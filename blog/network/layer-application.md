## 应用层

📖《计算机网络——自顶向下》第二章读书笔记。

## 1 应用层协议原理

### 1.1 网络应用程序体系结构

- 客户-服务器体系结构 (Client-Server architecture) ：搜索引擎、社交网络软件等。
- P2P 架构：文件共享 BitTorrent、迅雷 P2P 加速下载。

---

### 1.2 进程通信

**1. 客户和服务器进程**

在一对进程之间的通信会话场景中，**发起通信** (即在该会话开始时发起与其他进程的联系)的进程被标识为**客户**，在会话开始时**等待联系**的进程是**服务器**。

**2. 进程与计算机网络之间的接口**

进程通过一个称为套接字 (socket) 的软件接口向网络发送报文和从网络接收报文。

**3. 进程寻址**

通过 IP 和端口 (Port)，IP 标识主机，端口标识进程。如 Web 服务器用端口号 80 来标识；邮件服务器进程 (使用SMTP 协议) 用端口号 25来标识。

----

### 1.3 可供应用程序使用的运输服务

一个运输层协议能够为调用它的应用程序提供什么样的服务呢? 我们大体能够从四个方面对应用程序服务要求进行分类: 可靠数据传输、吞吐量、定时和安全性。

**1. 可靠数据传输**

网络通过分组交换的方式实现通信，分组在计算机网络中可能丢失，数据丢失可能会造成灾难性的后果。

因此，为了支持这些应用，必须做 一些工作以确保由应用程序的一端发送的数据正确、完全地交付给该应用程序的另一端。如果一个协议提供了这样的确保数据交付服务，就认为提供了可靠数据传输 (Reliable Data Transfer) 。

最为常见的，TCP 是可靠的，UDP 是不可靠的。

**2. 吞吐量**

可用吞吐量就是发送进程能够向接收进程交付比特的速率，越大越好。

**3. 定时**

运输层协议也能提供定时保证。如同具有吞吐量保证那样，定时保证能够以多种形式实现。一个保证的例子如: 发送方注入进套接字中的每个比特到达接收方的套接字不迟于 100ms 。

**4. 安全性**

运输协议能够加密由发送进程传输的所有数据，在接收主机中，运输层协议能够在 将数据交付给接收进程之前解密这些数据。

---

### 1.4 因特网提供的运输服务

**1. TCP**

TCP服务模型包括**面向连接**服务和**可靠数据传输**服务。

- 面向连接：在应用层数据报文开始流动之前，TCP让客户和服务器互相交换运输层控制信息。三次握手，四次挥手。
- 可靠数据传输：通信进程能够依靠 TCP，**无差错、按适当顺序交付所有发送的数据**。当应用程序的一端将字节流传进套接字时，它能够依靠 TCP 将相同的字节流交付给接收方的套接字，而**没有字节的丢失和冗余**。

**2. UDP**

UDP 是无连接的，不可靠传输的服务。即：两个进程通信前没有握手过程，当进程将一个报文发送进 UDP 套接字时，UDP 协议并不保证该报文将到达接收进程。不仅如此，到达接收进程的报文也可能是乱序到达的。

---

TCP 在应用层可以很容易地用SSL来加强以提供安全服务。对于吞吐量或定时保证，这些服务目前的因特网运输协议并没有提供。

常见的网络应用在应用层和传输层的的协议如下图所示。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210221163933.png" style="width:80%;" />



### 1.5 应用层协议

应用层协议 (application-layer protocol) 定义了运行在不同端系统上的应用程序进程如何相互传递报文。特别地，应用层协议定义了 :

- 交换的报文类型，例如请求报文和响应报文。
- 各种报文类型的语法，如报文中的各个字段及这些字段是如何描述的。
- 字段的语义，即这些字段中的信息的含义。
- 确定一个进程何时以及如何发送报文，对报文进行响应的规则。

有些应用层协议是由 RFC 文档定义的，例如，Web的应用层协议 HTTP 。

## 2 Web 和 HTTP

### 2.1 概况

Web 的应用层协议是超文本传输协议 (HyperText Transfer Protocol, HTTP), 它是 Web 的核心。

HTTP 定义了 Web 客户端向 Web 服务器请求 Web 页面的方式，以及服务器向客户传送 Web 页面的方式，它**使用 TCP 作为传输协议**。HTTP 由两个程序实现: 一个客户程序和一个服务器程序。客户程序和服务器程序运行在不同的端系统中，通过交换 HTTP 报文进行会话。

此外，HTTP 是一个**无状态协议** (Stateless Protocol) 。服务器向客户发送被请求的文件，而不存储任何关于该客户的状态信息。假如某个特定的客户在短短的几秒内两次请求同一个对象，服务器并不会因为刚刚为该客户提供了该对象就不再做出反应，而是重新发送该对象，就像服务器已经完全忘记不久之前所做过的事一样。

Web 服务器 (Web server) 实现了 HTTP 的服务器端，它用于存储 Web 对象，每个对象由 URL寻址。流行的 Web 服务器有 Apache 和Microsoft Internet Information Server (微软互联网信息服务器)。

Web 浏览器 (Web browser)，(例如 Internet Explorer 和 Firefox) 实现了 HTTP 的客户端。

---

### 2.2 持续连接和非持续连接

每个请求/响应对是经一个单独的 TCP 连接发送，还是所有的请求及其响应经相同的TCP连接发送。若为前者，则是非持续连接 (non-persistent connection) ，若为后者，则为持续连接 (persistent connection) 。

----

### 2.3 HTTP 报文格式

HTTP 报文有两种: 请求报文和响应报文。

**1. 请求报文**

例子：

```
GET /somedir/page.html HTTP/1.1 
Host: www.someschool.edu
Connection: close 
User-agent: Mozilla/5.0 
Accept-language: fr
```

解释：

- HTTP 请求报文的**第一行叫作请求行** (request line), 其**后继的行叫作首部行** (header line)。
- 请求行有 3 个字段: 方法字段、URL 字段和 HTTP 版本字段。方法字段可以取几种不同的值，包括GET、POST、HEAD、PUT 和 DELETE。
- `Connection: close` 表示客户端与服务器之间**使用非持续连接**，要求服务器在发送完被请求的对象后就关闭这条连接。
- `User-agent: Mozilla/5.0 `  用来指明用户代理，即向服务器发送请求的浏览器的类型。根据这个信息，服务器可以有效地为不同类型的用户代理实际发送相同对象的不同版本。
- `Accept-language: fr` 表示用户想得到该对象的法语版本。



请求报文的通用格式如下图所示。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210221170750.png" style="width:50%;" />

`sp` 是空格 (Space)，`CRLF` 表示换行。实体体是一个「诡异」的翻译，实际上是 Entity Body。如果使用 POST 方法提交一个表单，那么表单的内容就放在 Entity Body 中，通常使用 JSON 格式。

**2. 响应报文**

例子：

```
HTTP/1.1 200 OK
Connection: close
Date: Tue, 18 Aug 2015 15:44:04 GMT
Server: Apache/2.2.3 (CentOS)
Last-Modified: Tuer 18 Aug 2015 15:11:03 GMT 
Content-Length: 6821
Content-Type: text/html
(data data data data data ...)
```

解释：

- 3 个部分：一个初始状态行 (status line) , 6 个首部行 (headerline) ，然后是实体体 (entity body)。
- 状态行：协议版本、状态码、状态信息
- 首部行
  - `Connection: close` 告诉客户端，发送完报文后将关闭该 TCP 连接。
  - `Date` 表示服务器产生并发送该响应报文的日期和时间。
  - `Server` 表示该报文由 Apache Web 服务器产生。
  - `Last-Modified` 表示发送对象的创建或者最后修改的日期和时间。
  - `Content-Length` 表示发送对象的字节数。
  - `Content-Type` 表示发送的对象是 HTML 文本。
- `data` 就是发送的对象。

常见的 HTTP 状态码：

- 200 0K: 请求成功，信息在返回的响应报文中。
- 301 Moved Permanently: 请求的对象已经被永久转移了，新的 URL 定义在响应报文的 `Location` 首部中。客户软件将自动获取新的URL。
- 400 Bad Request: 一个通用差错代码，指示该请求不能被服务器理解。
- 404 Not Found: 被请求的文档不在服务器上。
- 505 HTTP Version Not Supported: 服务器不支持请求报文使用的HTTP协议版本。

HTTP 响应报文的通用格式如下图所示。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210221171849.png" style="width:50%;" />

### 2.4 用户与服务器的交互: cookie

这部分看得我比较无语 😓，实际上应该结合 session 来阐述比较好。

HTTP 是无状态的，简化了服务器程序的设计，允许开发可以同时处理数以千计的 TCP 连接的高性能 Web 服务器。然而一个 Web 站点通常希望能够识别用户，因为它希望把内容与用户身份联系起来。因此，需要使用 cookie 。

cookie 技术有 4 个组件：

- 在 HTTP 响应报文中的一个 cookie 首部行；
- 在 HTTP 请求报文中的一个 cookie 首部行；
- 在用户端中保留有一个 cookie 文件，并由用户的浏览器进行管理（一般文件大小不超过 4KB）；
- 位于 Web 站点的一个后端数据库。

 cookie 的工作流程如下图所示。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210221173208.png" style="width:70%;" />

### 2.5 Web 缓存器

Web 缓存器 (Web Cache) 也叫代理服务器 (Proxy Server)，在一个局域网内添加一台服务器，缓存目标服务器的数据。局域网内的客户端向缓存服务器发起 HTTP 请求，如果请求对象不存在，那么缓存服务器再向目标服务器请求，收到请求对象后缓存在本机上，同时向客户端转发。

局域网内的网速通常要比直接向目标服务器请求要快得多。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210222193926.png" style="width:40%;" />

### 2.6 条件 GET 方法

条件 GET 方法是 HTTP 中的一种机制，允许缓存服务器验证它缓存的对象是否是最新的。工作过程：

- 缓存服务器在 HTTP 请求加入首部 `Last-modified-since` 
- 目标服务器根据 `Last-modified-since` 的日期判断，如果请求对象在该日期之后有更改，那么发送最新对象；否则发送一个不带有 Entity Body 的响应报文。

## 3 电子邮件服务

邮件服务使用 SMTP 协议。

其他的还有：POP3、IMAP。



## 4 DNS

域名系统 (Domain Name System) 的主要任务是：

