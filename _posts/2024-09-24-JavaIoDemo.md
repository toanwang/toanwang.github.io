---
layout:     post
title:      Java Io Demo
subtitle:   JavaBase
date:       2024-09-19
author:     toan
catalog:	true
tags:
    - IO
    - NIO
---

## IO模型

### BIO

* 阻塞`IO`，每一个线程只能处理一个链接
  * 服务端： `ServerSocket(PORT)`
    * `serverSocket.accept()`：阻塞，等待链接，获取`Socket`对象
  * 客户端：`Socket(HOST, PORT)`
* 面向流：使用`OutputStream`和`InputStream`

### NIO

面向缓冲区(块)

<img src="https://cdn.nlark.com/yuque/0/2024/png/1102741/1727234806260-da28324d-a742-4ed8-af00-f81bfac34835.png?x-oss-process=image%2Fformat%2Cwebp" alt="image.png" style="zoom:50%;" referrerpolicy="no-referrer"/>

#### 三个核心模块

* `Buffer`：双向的，可以从文件、网络等读取数据
  * `allocate()`：分配大小
  * 主要的三个参数
    * `position()`：下一次读取位置
    * `limit()`：缓冲区终点，不能读取超过限制的位置
    * `capacity()`：容量，不可变
  * `clear()`：写模式，清空缓冲
  * `compact()`：写模式，不清空缓冲
  * `flip()`：读模式
* `Channel`：`channel`必须通过`buffer`进行读写操作，绑定`IP+PORT`
  * `FileChannel`：文件读写
  * `DatagramChannel`：`UDP`数据读写
  * `ServerSocketChannel`：服务端
    * 使用`bind(new InetSocketAddress(PORT))`绑定端口号
    * 使用`ServerSocketChannel.accept()`获取链接的`SocketChannel`
    * 使用`SelectionKey`获取有数据的`SocketChannel`
    * 使用`SocketChannel.read(buffer)`，将数据读到`buffer`中
    * 使用`sChannel.register(selector, SelectionKey.OP_READ);`，将`socketChannel`注册到`selector`
  * `SocketChannel`：客户端
    * `SocketChannel.open(new InetSocketAddress(IP, PORT))` 通道绑定服务器
    * 使用`sChannel.register(selector, SelectionKey.OP_READ);`，将`socketChannel`注册到`selector`
* `Selector`：绑定`channel`
  * `Selector.open()`：获取监视器对象
  * `Selector.select()`：阻塞，获取所有通道中事件
    * 通过下面的事件通知机制，阻塞获取时，保证肯定有数据可以被获取到，避免长时间无效阻塞
  * `Selector.selectedKeys()`：获取所有的`selectedKey`
    * 每个`selectdKey`有一个`SelectableChannel`和`Selector`
    * 事件分为四种
      * `OP_READ`：`1<<0`，读操作
      * `OP_WRITE`：`1<<2`，写操作
      * `OP_CONNECT`：`1<<3`，连接已经建立
      * `OP_ACCEPT`：`1<<4`，有新网络可以连接

#### 想到的问题

服务端`selector`上注册的`channel`数量，一般为`count(client) + 1`

* `serverSocketChannel`一般以`OP_ACCEPT`事件注册到`selector`上，监听新建立的连接，有连接建立时，获取`socketChannel`
* `socketChannel`一般以`OP_READ OP_WRITE`注册到`selector`上，监听读写事件

`serverSocketChannel.accept()` 和 `socketChannel.open()` 得到的两个`channel`有什么区别？

* `serverSocketChnnel`常用操作：`open() bind() accept()`
* `socketChannel`常用操作：`open() connect()`
* `serverSocketChannel.accept()`得到的`channel` 和 `socketChannel`可以理解为一个网络连接的两个端点，通过操作两个`sockerChannel`实现网络通信

### 零拷贝

* `mmap`：适合小数据读写
  * 通过内存映射，将文件映射到内核缓冲区，减少用户空间到内核空间的拷贝
* `sendfile`：适合大文件传输
  * 不经过用户态，直接在内核空间拷贝，减少一次上下文切换

### 同步异步

两者描述的维度是不一样的，可以互相组合

* `NIO`和`BIO`是`IO`模型，读取数据时，是否阻塞内核线程【操作系统层面】
* 同步异步是描述`api`语义【接口层面】

|      | NIO                | BIO                               |
| ---- | ------------------ | --------------------------------- |
| 同步 | `Tomcat websocket` | 正常读写`socket`【早期 `Tomcat`】 |
| 异步 | `Netty`            | 几乎不存在                        |

## Netty

https://dongzl.github.io/netty-handbook/#/_content/chapter05?id=_58-netty-%e6%a8%a1%e5%9e%8b

介绍的很详细

## 相关链接

[JavaIO-demo 代码示例](https://github.com/toanwang/SpringComponent/tree/master/IO-demo/src/main/java/org/io/base/NIO)

[Netty-demo](https://github.com/toanwang/SpringComponent/tree/master/Netty-demo)

参考连接

* [知乎-epollo](https://zhuanlan.zhihu.com/p/63179839)
* [Netty学习手册](https://dongzl.github.io/netty-handbook/#/_content/chapter02)