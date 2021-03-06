## 计算机网络面试题目

面试题目笔记。

## 打卡题目

一些基础概念。

### 五层模型

- 物理层
- 链路层
- 网络层
- 运输层
- 应用层



## HTTP 



### HTTP 中 GET 和 POST 有什么区别

- 作用：一般情况下，GET 用于获取资源（其实也可以修改），而 POST 用于传输实体主体。
- 参数：GET 直接写在 URL 当中，POST 写在 HTTP 请求报文的实体 (Entity Body) 部分
- 幂等性：GET 一般是请求资源，多次请求，所得到的结果总归是一样的；但 POST 方法常用于提交表单，服务器可能需要向数据库插入一条记录，如果多次 POST，有可能插入多条记录。
- 可缓存：GET 可缓存，POST 不可缓存



## TCP

### 4 种拥塞控制算法

- 4 种算法：慢开始、拥塞避免、快重传、快恢复。
- 2 个变量：`threshold` 表示拥塞的阈值； `cwnd`  表示拥塞窗口的大小。
- 2 个阶段：指数增长和线性增长，分别为慢开始和拥塞避免  2 个阶段。
- 2 种处理方法
  - 如果出现超时，表明网络出现拥塞，那么 `ssthresh = cwnd / 2, cwnd = 1` .
  - 如果出现 3-ACK，表明出现丢包，那么 `ssthresh = cwnd / 2, cwnd = ssthresh` .



### 服务器存在大量 TIME-WAIT 连接

TIMEWAIT 是主动关闭的一方才有的，我们设计的服务器是 **主动关闭型** 的，才需要考虑这个问题。

TIMEWAIT 过多，引起的主要问题是端口占用过多。

解决方案：

- 在应用层面：尽量避免频繁关闭连接，如业务优化，或者使用长连接等；或者改为客户端主动关闭。
- 在系统层面：
  - 缩短 MSL 的时间
  - 增加进程的端口数量



## 其他

### 输入一个 URL 后发生的事情

- 如果本地上已经有这个 URL 指向的 HTML 缓存，那么直接从本地读取。如果没有，那么执行后面的步骤。
- DNS 域名解析（按下列优先级顺序查找），找到 IP 地址
  - 浏览器 DNS 混存
  - 本地 DNS 缓存（假设已经更新了 host 文件指定的内容）
  - 向本地配置的首选DNS服务器发起域名解析请求（包括迭代查询和递归查询 2 种方式）
    - 次序为：本地域名服务器，根域名服务器，顶级域名服务器，权威（权限）域名服务器

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210310153013.png" style="width:60%;" />

- 建立 TCP 连接（三次握手 🤝 ）
  - 可以手动通过 `telnet` 命令建立 TCP 连接
- 客户端发起 HTTP 请求
- 服务端返回 HTTP 响应（在实体部分带有 HTML ）
- 浏览器解析 HTML



**与之相关问题**

- 如何绕过 DNS 服务器？
  - 方法一：URL 直接输入 IP 地址（不输入域名）
  - 方法二：在本地 host 文件写入 `url->ip` 的映射对
- 什么是 DNS 劫持？
  - DNS 服务器一方返回一个假的 IP 地址。



### 什么是跨域？

跨域，指的是浏览器不能执行其他网站的脚本。比如：

- 非跨域请求：`http://www.123.com/index.html` 调用 `http://www.123.com/server.php` 
- 跨域请求：`http://www.123.com/index.html` 调用 `http://www.456.com/server.php`

Vue 中通过**代理**的方式解决跨域问题（在 `vue.config.js` 中配置）。将域名发送给本地的服务器（启动 vue 项目的服务，loclahost:8080），再由本地的服务器去请求真正的服务器。





### cookie 和 session

为什么需要 cookie 和 session 呢？因为 HTTP 协议是无状态的，这就需要额外的机制来实现保存客户端与服务端交互状态的功能（比如记住登陆状态）。

主要区别如下：

- cookie 数据保存在客户端，session 数据保存在服务端。但是，`session-id` 需要保留在客户端，`cookie-id` 需要保存在服务端，这很好理解，因为要互相识别身份。
- 安全性：cookie 不是很安全，别人可以分析存放在本地的 cookie 并进行 cookie 欺骗，如果需要安全性应当使用 session。
- 重要信息应放在 session，其他对安全要求不高的信息可保存在 cookie 。
- 如果 session 过多，服务器性能会降低，需要提高性能的时候，可考虑使用 cookie 。





