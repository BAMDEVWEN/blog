---
title: 'npm下载'
description: 'npm 的基本使用'
pubDate: 'Nov 18 2024'
tags: ['npm']
---

# npm拉取依赖

## 缓存机制 
- 优先使用cache
- ![下载流程](/download.png)

## npm的下载方式

```shell

sudo npm install -global [package name]
npm install git://github.com/package/path.git
npm install git://github.com/package/path.git#0.1.0
npm install path/to/somedir  //本地路径 #可以下载包，从本地导入


```

## node-gyp的使用

- windows-build-tools 会直接提供c++运行环境，不需要下载vs2017

- 或者下载vs2017

## node-gyp build 产生的问题

### 拉取不到node.js 的头文件依赖（npm 拉取）

### 问题分析

配置了镜像源，node.js的头文件都是到node.js的官网拉取，网络连接超时

### 解决方式

#### 配置翻墙（cmd就算开启了全局代理，默认不会走飞机场的端口，需要配置）

1. cmd 配置代理
   - set http_proxy=http://127.0.0.1:1189` 和 `set https_proxy=http://127.0.0.1:1189（端口为clash配置的转发端口）
   - 下载 electron 下载缓慢
   - 配置npm 国内镜像源
   - 可以使用netsh 永久配置（尝试配置生效，下载失败）

### 配置国内环境

1. npm config set registry https://registry.npmmirror.com 配置依赖包源
2.  set NODE_GYP_DISTURL=https://mirrors.ustc.edu.cn/node/ 配置node.js的镜像（清华）
3. npm install
