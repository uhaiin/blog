---
layout: mypost
title: 兼容python和anaconda
categories: [python]
---

> 由于安装idf的时候使用anaconda的python环境出现问题，可能需要原装的python，下面是我现在安装python的最新解决方案。

> 使用官网python作为系统默认python，屏蔽anaconda的默认base环境，创建项目的时候再使用anaconda创建的虚拟环境。

### 1、屏蔽anaconda的python环境

```bash
修改anaconda安装路径下的 `python.exe` 为 `python-a.exe`
```
![10](10.png)

### 2、安装python

```bash
python官网地址 `https://www.python.org/downloads/windows/`
```
勾选添加 `python.exe` 到环境，直接安装

![11](11.png)

### 3、换源

清华源pypi文档路径 `https://mirrors.tuna.tsinghua.edu.cn/help/pypi/`
```bash
# 更新pip
python -m pip install --upgrade pip
# 设置镜像源地址
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```