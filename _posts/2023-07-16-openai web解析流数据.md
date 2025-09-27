---
layout: mypost
title: openai 网页版解析流数据EventStream
categories: [openai]
---

> openai 网页版使用event-stream来传递信息,可以使用 SSEClient 库来解析数据流

### 一、发起sse请求
```python
def request_sse(url, headers, data):
    """ 用于发起SSE请求 """
    # 创建一个队列 queue 和一个事件对象 e，用于在请求过程中传递数据和控制线程。
    queue, e = block_queue.Queue(), threading.Event()
    # 创建一个线程 t，并将异步方法 self._do_request_sse() 作为参数传递给 asyncio.run() 函数。
    t = threading.Thread(target=asyncio.run, args=(do_request_sse(url, headers, data, queue, e),))
    # 启动线程 t，开始执行异步的 SSE 请求。
    t.start()
    # 使用 queue.get() 两次分别获取队列中的两个数据项。
    # 第一个数据项表示 SSE 请求状态，第二个为数据流结束标志。
    # 调用 self.__generate_wrap() 方法，生成一个封装后的生成器对象，用于迭代 SSE 数据流。
    return queue.get(), queue.get(), generate_wrap(queue, t, e)
```

### 二、执行sse请求
```python
async def do_request_sse(self, url, headers, data, queue, event):
    """ 用于执行SSE请求 """

    # 使用 httpx.AsyncClient 创建一个异步的 HTTP 客户端，用于发送 SSE 请求。
    async with httpx.AsyncClient() as client:
        # 发起一个异步的 POST 请求，通过 client.stream() 方法发送请求并获取响应的数据流。
        async with client.stream('POST', url, json=data, headers=headers, timeout=600) as resp:
            # 在响应的数据流中，使用 async for 循环逐行读取数据。
            async for line in process_sse(resp):
                queue.put(line)
                # 中断请求
                if event.is_set():
                    await client.aclose()
                    break
            # 将 None 放入队列 queue，表示数据流结束
            queue.put(None)
```
### 三、处理see响应
```python
async def process_sse(resp):
    """ 用于处理SSE响应 """

    # 首先使用 yield 关键字返回响应的状态码（resp.status_code）和响应头部（resp.headers）。
    yield resp.status_code
    yield resp.headers
    # 检查响应的状态码是否为 200。如果不是 200，说明 SSE 请求出现错误。
    if resp.status_code != 200:
        yield await process_sse_except(resp)
        return
    # 在循环中，使用 resp.aiter_lines() 方法逐行读取响应的内容。
    async for utf8_line in resp.aiter_lines():
        # 检查每行内容是否以 'data: [DONE]' 开头，如果是，则表示数据流结束，跳出循环。
        if 'data: [DONE]' == utf8_line[0:12]:
            break
        # 检查每行内容是否以 'data: {' 开头，如果是，则表示该行是 JSON 格式的数据。
        if 'data: {' == utf8_line[0:7]:
            yield json.loads(utf8_line[6:])
async def process_sse_except(resp):
    """ 用于处理SSE响应的异常情况 """
    result = b''
    async for line in resp.aiter_bytes():
        result += line

    return json.loads(result.decode('utf-8'))
```
### 四、创建封装后的生成器
```python
def generate_wrap(queue, thread, event):
    """ 用于生成封装后的生成器 """
    while True:
        try:
            item = queue.get()
            # 如果获取到的数据为 None 则结束
            if item is None:
                break
            # 使用 yield 关键字将数据项逐个返回
            yield item
        except BaseException as e:
            event.set()
            thread.join()
            if isinstance(e, GeneratorExit):
                raise e
```