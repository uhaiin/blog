---
layout: mypost
title: docker部署flask应用
categories: [docker,python]
---

> 使用docker和flask框架部署python项目为在线接口

### 一、创建新的conda虚拟环境
```bash
conda create -n openai python==3.10
```
创建项目如`flask_demo`并使用该环境。

先运行下面命令，如果生成的`requirements.txt`有内容，需要全部`uninstall`，以保证最开始是无第三方依赖的干净环境。项目打包时候再次运行
```bash
pip freeze > requirements.txt
```
### 二、创建flask应用
在当前项目下创建`app.py`，内容如下
```python
from flask import Flask, request
# 实例化Flask对象
app = Flask(__name__)

@app.route('/', methods=['GET', 'POST'])
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```
### 三、创建Dockerfile文件
在当前项目下创建`Dockerfile`，内容如下
```bash
FROM python:3.10
WORKDIR /flask_demo

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple/
COPY . /flask_demo

CMD ["python3","app.py"]
```
### 四、打包镜像
将项目文件夹如`flask_demo`上传到linux服务器。并进入`flask_demo`目录下，运行打包命令
```bash
docker build -f ./Dockerfile -t flask_demo:1.0 .
```
### 五、运行容器
- 运行容器
```bash
docker run -d --name flask_docker_web -p 5000:5000 flask_demo:1.0
```
- 挂载目录，运行容器
```bash
docker run -d --name flask_docker_web2 -v /root/flask_docker:/flask_demo -p 5000:5000 flask_demo:1.0
```

> 记得打开5000端口的防火墙，即可通过`ip:5000`访问到`Hello World！`