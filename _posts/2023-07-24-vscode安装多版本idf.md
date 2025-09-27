---
layout: mypost
title: vscode安装多版本idf
categories: [esp32]
---

> ESP-IDF 是乐鑫官方的esp32开发框架，下面介绍vscode安装多个版本的idf

### 一、安装python环境
```bash
安装方法省略，建议安装最新的python3.11
```

### 二、vscode安装插件
```bash
搜索并安装插件 ==> Espressif IDF
```

创建 `idf` 目录如 `E:\esp-idf`，并创建多个版本的目录

![01](01.png)

并在每个版本的目录下创建 `tools` 目录

![02](02.png)

在vscode，输入快捷键 `Ctrl + Shift + p`，在上方的搜索栏输入 `ESP-IDF:Configure ESP-IDF extension`

![03](03.png)

点击进入配置 `ESP-IDF` 插件的页面，选择 `EXPRESS`

![04](04.png)

修改下载服务器和路径，点击安装

![05](05.png)

安装到python_env那一步如果卡住，需要开代理

![06](06.png)

安装成功如下

![07](07.png)

可重复上面步骤安装多个版本

### 三、切换版本
在vscode，输入快捷键 `Ctrl + Shift + p`，在上方的搜索栏输入 `ESP-IDF:Configure Paths`

![08](08.png)

点击配置路径，通过修改不同版本的路径，来切换版本

![09](09.png)

