---
title: CS144 Winter2025 Check0
description: 关于CS144第零个Check的个人解法
date: 2025-05-15
categories:
    - CS144
    - 解题思路
tags:
    - C++
    - 学习笔记
comment: true
---

## 写在前面

本文取自 Stanford CS144 Winter2025 minnow Check0，笔者为计算机网络初学者，如有错误表述或不恰当的地方，请指正！

欢迎与我[联系](mailto:yutaki23@163.com)

## 1 Set up GNU/Linux on your computer

题目给了我们五个选项来作为本实验的环境，笔者在这里选择了第 3 条，使用虚拟机下载 Ubuntu 24.04 进行学习。

值得注意的是，对于 Winter 2025 这个版本，似乎必须要是有 Ubuntu 24.04，但截至目前（2025.5.14）依然还有很多人在使用 Ubuntu 22.04 进行开发，在此，笔者只是提个醒，使用低版本的操作系统无法下载这些包，无法进行开发。

## 2 Networking by hand

本题需要我们完成两项任务，检索网页和发送电子邮件（没有斯坦福账号，不做解释），这两项任务引出了一个概念

**可靠双向字节流**：在自己的终端上输入一串字符，这串字符最终会以相同的顺序传递给服务器。

### 2.1 Fetch a Web page

跟着题目要求一步一步在终端上输入即可，唯一值得注意的是，输入的速度要快一点，慢了会失败。

## 3 Writing a network program using an OS stream Socket

接下来需要编写一些程序来实现**可靠双向字节流**这一操作。这种功能被称为**流式套接字**，就像是一个 IO 输入输出流一样，当两个流式套接字连接时，*写入其中一个套接字的字节最终会以相同顺序从另一台计算机的套接字中读出*。

本实验只需要实现 `webget`，创建 TCP 流套接字，连接到 Web 服务器并获取页面，也就是我们在 2.1 节中所做的任务。

### 3.1 Let’s get started—setting up the repository on your VM and On GitHub

从 GitHub 上 clone repo 即可，按自己的方法来，不赘述。

### 3.2 Compiling the starter code

跟着教程一步步编译即可。遇到的问题是，笔者最开始使用 WSL2 的时候，编译无法通过，换成 VMware 才能通过。

### 3.3 Modern C++: mostly safe but still fast and low-level

在本项目中，向我们规定了哪些语法应该使用，哪些不该使用。

运用了许多现代 C++的语法。

### 3.4 Reading the Minnow support code

题目要求我们看一下一些提到的公共接口

`socket.hh`

-  `Socket` 继承于 `FileDescriptor`
	- `bind(address)` 将 Socket 绑定到指定的网络地址，常用于服务器监听
	- `bind_to_device(device_name)` 将 Socket 绑定到指定网络接口设备
	- `connect(address)` 建立指定对端地址的 TCP 连接
	- `shutdown(how)` 关闭
	- `local_address()` 获取 Socket 绑定的本地地址信息
	- `peer_address()` 获取已连接 Socket 的对端地址信息
	- `set_reuseaddr()` 允许地址重用
	- `throw_if_error()` 检查 Socket 错误状态，用于非阻塞 Socket 的场景，发现错误时抛出异常
- `DatagramSocket` 继承于 `Socket`
	- `recv(source_address, payload)` 接受数据报
	- `sendto(destination, payload)` 发送数据报到指定位置
	- `send(payload)` 发送到已连接的地址，需要先 connect
- `UDPSocket` 继承于 `DatagramSocket`
	- `UDPSocket()` 默认构造函数
- `TCPSocket` 继承于 `Socket`
	- `TCPSocket()` 默认构造函数
	- `listen(int backlog = 16)` 启动监听
	- `accept()` 接受新连接并返回新 TCPSocket
- `PacketSocket` 继承于 `DatagramSocket`
	- `PacketSocket(int type, int protocol)` 默认构造函数
	- `set_promiscuous()` 设置混杂模式
- `LocalStreamSocket` 继承于 `Socket`
	- `LocalStreamSocket(FileDescriptor&& fd)` 从 fd 构造函数
- `LocalDatagramSocket` 继承于 `DatagramSocket`
	- `LocalDatagramSocket()` 默认构造未绑定的构造函数

`file_descriptor.hh`

- `FDWrapper()` 构造函数
- `~FDWrapper()` 析构函数
- `read()` 从文件描述读取数据到缓冲区（单缓冲区（字符串） 多缓冲区（数组））
- `write()` 尝试写入数据并返回写入的字节数（字符串视图或数组）
- `close()` 关闭底层文件描述符
- `duplicate()` 显式复制文件描述符，增加引用计数
- `set_blocking()` 设置文件描述为阻塞或非阻塞模式
- `size_t()` 返回文件大小
- `fd_num()` 获取底层文件描述符编号
- `eof()` 检查是否到达文件末尾
- `closed()` 检查描述符是否已关闭
- `read_count() / write_count()` 获取读写操作次数统计

### 3.5 Writing webget

本题要求我们完成一个类似于上文 2.1 节提起的**从互联网中获取网页**的内容，我认为比较重要的点有以下几点

1. 这整个过程是一个**双向可靠**的，也就是说我主机端向服务端发送了什么内容，那么服务端就会以相同的顺序获取这些内容。
2. 题目告诉我们要使用之前 `Http` 的格式，一定要看清楚哪里有空格，哪里没有空格
明白这两点，那么大致就会有个思路，接下来就靠自己努力，

一些容易踩坑的点，题目中也给我们标明了，注意即可，

代码如下，

```cpp
void get_URL( const string& host, const string& path )
{
  Address address = Address(host, "http");
  TCPSocket socket;
  socket.connect(address);
  socket.write(
    "GET " + path + " HTTP/1.1\r\n" +
    "Host: " + host + "\r\n" +
    "Connection: close\r\n\r\n");

  while (!socket.eof()) {
    string output;
    socket.read(output);
    cout << output;
  }
}
```

## 4 An in-memory reliable byte stream

**注：本流程已笔者做题时的正常思路为主，可能在后面会有其他补充，请看到最后~**

本题需要完成一个**可靠的字节流**，其中我认为比较重要的点有以下

1. 字节首先被写入输入端，以**相同顺序**写出到输出端
2. 字节流是有限的

### 4.1  `push(string data)`

将外部数据写入字节流的缓冲区，但不可以超过容量。

首先来看看**容量**是什么，在 `ByteStream` 初始化时，会给定一个 `capacity`，这是这个字节流的容量，但容量只由这一个来决定吗？不是，我们推送数据的时候，会创建一个适当的数据结构来表示缓冲区，在最开始，什么都没有，缓冲区内什么也没有，当然可以以总容量的大小来推送，但下一次就不一定了，若当前缓冲区存在一些字节，那我们此时的容量就会变成 `总容量-缓冲区大小`。

其次，用户在传递数据的时候，如果比一开始设置的容量就要大、亦或者是比我们 `总容量-缓冲区大小` 的容量还要大，应该怎么办？此时应该选择这两者之间最小的一个，也就是 `min(数据大小,此时容量大小)`。

想到这，就会有一个大概的思路，首先要选择一个合适的数据结构，因为传递的是 `string`，我在这里也选择 `string` 作为缓冲区的数据结构。

1. 判断当前可以使用的容量
2. 判断数据大小和容量哪个更小
3. 添加到缓冲区
因此，我们可以写出代码
```cpp
//byte_stream.hh
protected:
  std::string buffer_;
```

```cpp
// byte_stream.cc
void Writer::push( string data )
{
  uint64_t available_bytes = capacity_ - buffer_.size();
  uint64_t write_len = std::min(available_bytes, data.size());
  buffer_.append(data.substr(0, write_len));
}
```

### 4.2 `close()` & `is_closed()`

标记流已经结束，不再写入。

要完成此函数，可以添加一个布尔标记，判断此时是否已经结束

```cpp
// byte_stream.hh
protected:
  bool is_closed_;
```

```cpp
// byte_stream.cc
void Writer::close()
{
  is_closed_ = true;
}
bool Writer::is_closed() const
{
  return is_closed_;
}
```

### 4.3 `available_capacity () `

当前可推送的字节数

很明显，这就是我们在 4.1 节中想过的当前可以使用的容量大小，

```cpp
uint64_t Writer::available_capacity() const
{
  return capacity_ - buffer_.size();
}
```

### 4.4 `bytes_pushed () `

返回目前累计推送到字节流的总字节数

我们可以维护一个字节数的变量，每推送了一定值的字节后，就往这个变量上加一定值。所以，我们需要在上文 4.1 节中的 `push` 函数加上一行，来表示我们已推送的总字节数。并且，还可以将 4.3 节抽象到此处。

```cpp
// byte_stream.hh
protected:
  uint64_t bytes_pushed_;
```

```cpp
// byte_stream.cc
void Writer::push( string data )
{
  uint64_t available_bytes = available_capacity();
  uint64_t write_len = std::min(available_bytes, data.size());
  buffer_.append(data.substr(0, write_len));
  bytes_pushed_ += write_len;
}
uint64_t Writer::bytes_pushed() const
{
  return bytes_pushed_;
}
```

### 4.5 `peek()`

本题需要查看缓冲区的可读取的数据，但不移除任何数据。

可读取的数据就是还在缓冲区内存放的数据，也就是 `size`
```cpp
string_view Reader::peek() const
{
  return {buffer_.data(), buffer_.size()};
}
```
### 4.6 `pop(uint64_t len)`

本题需要从缓冲区中移除指定长度的数据，这些数据表示为已经被读取过的。

我们要先判断一下要移除的这些数据，与我当前缓冲区内的数据大小哪个大，如果移除的数据比我本身的都大，那就有错误了。其次，我们要移除的是已经被读取过的，所以最开始的一定是在第一位，因为先读取的会一步步排到最前面。

```cpp
void Reader::pop( uint64_t len )
{
  len = std::min(len, buffer_.size());
  buffer_.erase(0, len);
}
```

### 4.7 `is_finished()`

判断此时数据流是否已经结束，结束的标志有

1. 已经关闭
2. 缓冲区内已无任何数据

```cpp
bool Reader::is_finished() const
{
  return is_closed_ && buffer_.empty();
}
```

### 4.8 `bytes_buffered()`

给出当前缓冲区内的数据大小

```cpp
uint64_t Reader::bytes_buffered() const
{
  return buffer_.size();
}
```

### 4.9 `bytes_popped()`

给出截至目前为止有多少数据被移除了。

我们依然可以维护一个变量，每移除一定量的数据后就往这个变量上添加。所以，要在上文 `pop()` 函数再添加一些内容。

```cpp
protected:
  uint64_t bytes_popped_;
```

```cpp
// byte_stream.cc
void Reader::pop( uint64_t len )
{
  len = std::min(len, buffer_.size());
  buffer_.erase(0, len);
  bytes_popped_ += len;
}
uint64_t Reader::bytes_popped() const
{
  return bytes_popped_;
}
```

### 4.10 构造函数

我们已经维护了如此多的变量，对于 C++来说，初始化一个对象也有很多讲究，我们在 `protected` 处给出的变量，因此回到最开始的内容，在构造函数时要**按顺序**来给出，否则会报错。

```cpp
protected:
  uint64_t capacity_;
  bool error_ {};
  std::string buffer_;
  bool is_closed_;
  uint64_t bytes_pushed_;
  uint64_t bytes_popped_;
};
```

```cpp
ByteStream::ByteStream( uint64_t capacity ) :
  capacity_( capacity ),
  buffer_(),
  is_closed_(false),
  bytes_pushed_(0),
  bytes_popped_(0) {}
```

## 写在后面

不愧是著名的 CS144，光是第一个 check 就让我如此难受，总耗费时长大概在 12 小时左右，对于网络编程的不了解吃了很大亏。