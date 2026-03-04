---
layout: mypost
title: Mac 和 Linux 的 Homebrew安装与卸载（国内加速安装
categories: [linux]
---

# Mac 和 Linux 的 Homebrew安装与卸载（国内加速安装）

[Homebrew官网](https://link.zhihu.com/?target=https%3A//brew.sh/)

## macOS

### 常规安装（推荐）

```bash
$ /bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

### 极速安装（精简版）

```bash
$ /bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)" speed
```

### 卸载

```bash
$ /bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/HomebrewUninstall.sh)"
```

- [常见错误解决方案](https://link.zhihu.com/?target=https%3A//gitee.com/cunkai/HomebrewCN/blob/master/error.md)

## Linux

### 安装

```bash
$ rm Homebrew.sh ; wget https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh ; bash Homebrew.sh
```

### 卸载

```sh
$ rm HomebrewUninstall.sh ; wget https://gitee.com/cunkai/HomebrewCN/raw/master/HomebrewUninstall.sh ; bash HomebrewUninstall.sh
```
