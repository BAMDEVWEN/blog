---
title: 'FFI-NAPI'
description: 'ffi-napi调用外部函数的基本使用'
pubDate: 'Dec 30 2024'
tags: ['Node.js','FFI-NAPI']
---
# Node-ffi
## 介绍
### 为什么使用
可以调用C++、C生成的dll动态链接库，主要通过此库调用硬件函数读取数据或者输出数据
### 原理 
Node.js 与动态链接库操作同一块内存，通过Node.js Buffer（核心）进行操作
### 缺点
- 调用时，运行时处于黑盒环境，出错不能直接输出错误，需要自己翻阅资料解决，
- 需要c语言基础（AI帮你读也可以，能理解就ok），厂商提供的文档不详细（可能我自己看不懂），方便理解阅读文档，写出对应的调用函数
- 为什么要用Node.js 去调用，因为接收时就是Node.js项目，Node.js 语法较为简单，方便上手
### 基本函数的使用
#### ref.refType(type)
返回一个表示指向 type 类型的指针的类型
#### ref.alloc(type)
分配一块内存，这块内存的大小与 type 对应的大小相同，并且返回一个指向该内存区域的指针
### 使用的情况（复杂一点点的）
#### 获取句柄
```js
//当函数为此类型时      'XXXX': ['long', ['int', 'int','pointer']],句柄为入参,同时也为出餐，需要提供一个指向Buffer的缓冲区的指针
const READERHANDLE = ref.refType(ref.types.void);
const phReaderPointer = ref.alloc(READERHANDLE);
//调用函数入参
xx.XXX(1,1,phReaderPointer);
//取出句柄
const handle=phReaderPointer.deref();
```
#### 结构体
```js
const StructType = require('ref-struct-napi');
const refArray = require('ref-array-napi')
const StringArray = refArray(ref.types.char, 256);
const TrackInfo = StructType({
    Contents: refArray(StringArray, 3),  // 3 个字符串数组，每个最大
    Lengths: refArray(ref.types.int, 3),           // 3 个整数，表示
    Status: refArray(ref.types.byte, 3)           // 3 个字节，表示
});
//这里是否需要填充数据，取决于dll动态链接库里德处理逻辑，如果未填充，读取不到数据，请填充
const trackInfo = new TrackInfo({
            Contents: new Array(3).fill(new Array(256).fill('\0')),
            Lengths: [0, 0, 0],
            Status: [0, 0, 0],
        });
```
#### 其他（遇到再更新）
#### 提示
- 大部分需要传十六进制，可以传入十进制，值需要相等，因为底层都是二进制进行传递的（如果存在理解误差，可联系我纠正）
- 十六进制不区分大小写，默认会以小写展示

