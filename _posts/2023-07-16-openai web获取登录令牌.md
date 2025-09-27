---
layout: mypost
title: openai 网页版获取登录令牌
categories: [openai]
---

> openai 网页版获取登录令牌access_token

### 开始、定义变量
```python
email = '登录的邮箱'
password = '密码'
mfa = None # 账号是否开启多因素认证
user_agent = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) ' \
                    'Chrome/109.0.0.0 Safari/537.36'
session = requests.Session()
```
### 第一步、获取预身份验证cookie
```python
def part_one() -> str:
    default_api_prefix = 'https://ai-{}.fakeopen.com'.format((datetime.now() - timedelta(days=1)).strftime('%Y%m%d'))
    url = '{}/auth/preauth'.format(default_api_prefix)
    resp = session.get(url, allow_redirects=False, timeout=100)

    if resp.status_code == 200:
        json = resp.json()
        return part_two(json['preauth_cookie'])
```
### 第二步、构建身份验证 URL
```python
def part_two(preauth: str) -> str:
    code_challenge = 'w6n3Ix420Xhhu-Q5-mOOEyuPZmAsJHUbBpO8Ub7xBCY'
    code_verifier = 'yGrXROHx_VazA0uovsxKfE263LMFcrSrdm4SlC-rob8'

    url = 'https://auth0.openai.com/authorize?client_id=pdlLIX2Y72MIl2rhLhTE9VV9bN905kBh&audience=https%3A%2F' \
            '%2Fapi.openai.com%2Fv1&redirect_uri=com.openai.chat%3A%2F%2Fauth0.openai.com%2Fios%2Fcom.openai.chat' \
            '%2Fcallback&scope=openid%20email%20profile%20offline_access%20model.request%20model.read' \
            '%20organization.read%20offline&response_type=code&code_challenge={}' \
            '&code_challenge_method=S256&prompt=login&preauth_cookie={}'.format(code_challenge, preauth)
    return part_three(code_verifier, url)
```
### 第三步、构建验证请求并发送 GET 请求，获取授权码
```python
def part_three(code_verifier, url: str) -> str:
    headers = {
        'User-Agent': user_agent,
        'Referer': 'https://ios.chat.openai.com/',
    }
    resp = session.get(url, headers=headers, allow_redirects=True, timeout=100)

    if resp.status_code == 200:
        try:
            url_params = parse_qs(urlparse(resp.url).query)
            state = url_params['state'][0]
            return part_four(code_verifier, state)
        except IndexError as exc:
            raise Exception('Rate limit hit.') from exc
```
### 第四步、输入邮箱账号，构建验证请求并发送 POST 请求
```python
def part_four(code_verifier: str, state: str) -> str:
    email = '登录的邮箱账号'
    url = 'https://auth0.openai.com/u/login/identifier?state=' + state
    headers = {
        'User-Agent': user_agent,
        'Referer': url,
        'Origin': 'https://auth0.openai.com',
    }
    data = {
        'state': state,
        'username': email,
        'js-available': 'true',
        'webauthn-available': 'true',
        'is-brave': 'false',
        'webauthn-platform-available': 'false',
        'action': 'default',
    }
    resp = session.post(url, headers=headers, data=data, allow_redirects=False, timeout=100)

    if resp.status_code == 302:
        return part_five(code_verifier, state)
```
### 第五步、获取重定向的URL
```python
def part_five(code_verifier: str, state: str) -> str:
    url = 'https://auth0.openai.com/u/login/password?state=' + state
    headers = {
        'User-Agent': user_agent,
        'Referer': url,
        'Origin': 'https://auth0.openai.com',
    }
    data = {
        'state': state,
        'username': email,
        'password': password,
        'action': 'default',
    }

    resp = session.post(url, headers=headers, data=data, allow_redirects=False, timeout=100)
    if resp.status_code == 302:
        location = resp.headers['Location']
        if not location.startswith('/authorize/resume?'):
            raise Exception('Login failed.')
        return part_six(code_verifier, location, url)
```
### 第六步、解析验证结果并执行进一步的操作
```python
def __part_six(code_verifier: str, location: str, ref: str) -> str:
    url = 'https://auth0.openai.com' + location
    headers = {
        'User-Agent': user_agent,
        'Referer': ref,
    }

    resp = session.get(url, headers=headers, allow_redirects=False, timeout=100)
    if resp.status_code == 302:
        location = resp.headers['Location']
        if location.startswith('/u/mfa-otp-challenge?'):
            return part_seven(code_verifier, location)

        if not location.startswith('com.openai.chat://auth0.openai.com/ios/com.openai.chat/callback?'):
            raise Exception('Login callback failed.')
        return get_access_token(code_verifier, resp.headers['Location'])
```
### 第七步、（可选）模拟用户输入MFA多因素认证代码进行验证
```python
def part_seven(code_verifier: str, location: str) -> str:
    url = 'https://auth0.openai.com' + location
    data = {
        'state': parse_qs(urlparse(url).query)['state'][0],
        'code': mfa,
        'action': 'default',
    }
    headers = {
        'User-Agent': user_agent,
        'Referer': url,
        'Origin': 'https://auth0.openai.com',
    }

    resp = session.post(url, headers=headers, data=data, allow_redirects=False, timeout=100)
    if resp.status_code == 302:
        location = resp.headers['Location']
        if not location.startswith('/authorize/resume?'):
            raise Exception('MFA failed.')
        return part_six(code_verifier, location, url)
```
### 最后、获取访问令牌
```python
def get_access_token(code_verifier: str, callback_url: str) -> str:
    """ 获取访问令牌，根据回调 URL 中的授权码请求访问令牌。 """
    url_params = parse_qs(urlparse(callback_url).query)

    url = 'https://auth0.openai.com/oauth/token'
    headers = {
        'User-Agent': user_agent,
    }
    data = {
        'redirect_uri': 'com.openai.chat://auth0.openai.com/ios/com.openai.chat/callback',
        'grant_type': 'authorization_code',
        'client_id': 'pdlLIX2Y72MIl2rhLhTE9VV9bN905kBh',
        'code': url_params['code'][0],
        'code_verifier': code_verifier,
    }
    resp = session.post(url, headers=headers, json=data, allow_redirects=False, timeout=100)

    if resp.status_code == 200:
        json = resp.json()
        access_token = json['access_token']
        return access_token

```