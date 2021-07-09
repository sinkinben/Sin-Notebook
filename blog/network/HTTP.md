## HTTP 协议

超文本传输协议 (Hypertext Transfer Protocol, HTTP) 是一个**应用层**协议，使用 TCP 进行可靠传输。

参考：https://github.com/CyC2018/cs-notes

（其实大部分都是 Copy and Paste 😅）

## 基础概念

基础概念分为 3 部分：请求报文、响应报文、URL。

2 中报文的通用格式如下图所示。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210224155657.png" style="width:70%;" />

### 请求报文

例子：

```http
GET http://www.example.com/ HTTP/1.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cache-Control: max-age=0
Host: www.example.com
If-Modified-Since: Thu, 17 Oct 2019 07:18:26 GMT
If-None-Match: "3147526947+gzip"
Proxy-Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 xxx

param1=1&param2=2
```

请求报文的结构：

- 第一行：请求方法、URL、协议版本
- 接下来的多行都是请求首部 Header，每个首部都有一个首部名称，以及对应的值。
- 一个空行用来分隔首部和内容主体 Entity Body
- 最后是请求的内容主体



### 响应报文

例子：

```http
HTTP/1.1 200 OK
Age: 529651
Cache-Control: max-age=604800
Connection: keep-alive
Content-Encoding: gzip
Content-Length: 648
Content-Type: text/html; charset=UTF-8
Date: Mon, 02 Nov 2020 17:53:39 GMT
Etag: "3147526947+ident+gzip"
Expires: Mon, 09 Nov 2020 17:53:39 GMT
Keep-Alive: timeout=4
Last-Modified: Thu, 17 Oct 2019 07:18:26 GMT
Proxy-Connection: keep-alive
Server: ECS (sjc/16DF)
Vary: Accept-Encoding
X-Cache: HIT

<!doctype html>
<html>
<head>
    <title>Example Domain</title>
	// 省略... 
</body>
</html>
```

响应报文结构：

- 第一行：协议版本、状态码以及状态描述（比如最常见的 200 OK 和 404 Not Found）
- 接下来多行也是首部内容
- 一个空行分隔首部和内容主体
- 最后是响应的内容主体



### URL

区分几个名词：

- URL (Uniform Resource Locator): 统一资源定位符
- URI (Uniform Resource Identifier): 统一资源标识符
- URN (Uniform Resource Name): 统一资源名称

URL 的一般形式：

```
<Protocol>://<host>:<port>/<path>
```

协议字段 `<Protocol>` 常见的有： `http, https, ftp` ；`<host>` 一般是主机的域名，也可以是指定的 IP 。URL 不区分大小写。

HTTP 的默认端口是 80 ，因此一般省略，所以最常见的 URL 形式就是 `http(s)://<host>/<path>`。

### 特点

- 无状态：服务器不会记住客户端请求过的资源，请求的次数等信息，这可以简化服务器端的代码设计。
- HTTP 本身的无连接的。虽然使用 TCP 作为传输协议，但通信双方在交换 HTTP 报文前不需要建立 HTTP 连接。

虽然 HTTP 是无状态的，但在实际工程当中，服务端通常需要知道请求的客户端是谁（比如注册登录功能），从而决定客户端访问资源的权限，这种需求一般通过 cookie 和 session 实现。



## HTTP 状态码

服务器返回的 **响应报文** 中第一行为状态行，包含了状态码以及原因短语，用来告知客户端请求的结果。

| 状态码 |               类别               |            含义            |
| :----: | :------------------------------: | :------------------------: |
|  1XX   |  Informational（信息性状态码）   |     接收的请求正在处理     |
|  2XX   |      Success（成功状态码）       |      请求正常处理完毕      |
|  3XX   |   Redirection（重定向状态码）    | 需要进行附加操作以完成请求 |
|  4XX   | Client Error（客户端错误状态码） |     服务器无法处理请求     |
|  5XX   | Server Error（服务器错误状态码） |     服务器处理请求出错     |

文档都写着啊：https://developer.mozilla.org/en-US/docs/Web/HTTP/Status

为什么面试要问这种阴间的八股文 😤 ，人的大脑当磁盘用，太浪费了。



**1XX：信息**

- **100 Continue** ：表明到目前为止都很正常，客户端可以继续发送请求或者忽略这个响应。



**2XX：成功**

- **200 OK**
- **204 No Content** ：请求已经成功处理，但是返回的**响应报文不包含实体的主体部分**。一般在只需要从客户端往服务器发送信息，而不需要返回数据时使用。
- **206 Partial Content** ：表示客户端进行了范围请求，响应报文包含由 Content-Range 指定范围的实体内容。



**3XX：重定向**

- **301 Moved Permanently** ：永久性重定向
- **302 Found** ：临时性重定向
- **303 See Other** ：和 302 有着相同的功能，但是 303 明确要求客户端应该采用 GET 方法获取资源。
- **304 Not Modified** ：如果请求报文首部包含一些条件，例如：If-Match，If-Modified-Since，If-None-Match，If-Range，If-Unmodified-Since，如果不满足条件，则服务器会返回 304 状态码。
- **307 Temporary Redirect** ：临时重定向，与 302 的含义类似，但是 307 要求浏览器不会把重定向请求的 POST 方法改成 GET 方法。

⚠️ 注意：虽然 HTTP 协议规定 301、302 状态下重定向时不允许把 POST 方法改成 GET 方法，但是大多数浏览器都会在 301、302 和 303 状态下的重定向把 POST 方法改成 GET 方法。



**4XX：客户端错误**

- **400 Bad Request** ：请求报文中存在语法错误。
- **401 Unauthorized** ：该状态码表示发送的请求需要有认证信息（BASIC 认证、DIGEST 认证）。如果之前已进行过一次请求，则表示用户认证失败。
- **403 Forbidden** ：请求被拒绝。
- **404 Not Found**



**5XX：服务器错误**

- **500 Internal Server Error** ：服务器正在执行请求时发生错误。
- **503 Service Unavailable** ：服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。



## HTTP 方法

|  方法   |               作用               |                             描述                             |
| :-----: | :------------------------------: | :----------------------------------------------------------: |
|   GET   |             获取资源             |         当前网络请求中，绝大部分使用的是 GET 方法。          |
|  POST   |             传输实体             | POST 主要用来传输数据（如提交表单），而 GET 主要用来获取资源。 |
|  HEAD   |           获取报文首部           | 和 GET 方法类似，但是不返回报文实体主体部分；<br />主要用于确认 URL 的有效性以及资源更新的日期时间等。 |
|   PUT   |             上传文件             | 由于自身不带验证机制，任何人都可以上传文件，因此存在安全性问题，一般不使用该方法。 |
|  PATCH  |        对资源进行部分修改        | PUT 也可以用于修改资源，但是只能完全替代原始资源，PATCH 允许部分修改。 |
| DELETE  |             删除文件             |           与 PUT 功能相反，并且同样不带验证机制。            |
| OPTIONS |          查询支持的方法          | 查询指定的 URL 能够支持的方法。<br />会返回 `Allow: GET, POST, HEAD, OPTIONS` 这样的内容。 |
| CONNECT | 要求在与代理服务器通信时建立隧道 | 使用 SSL（Secure Sockets Layer，安全套接层）和 TLS（Transport Layer Security，传输层安全）协议把通信内容加密后经网络隧道传输。 |
|  TRACE  |             追踪路径             | 服务器会将通信路径返回给客户端。发送请求时，在 Max-Forwards 首部字段中填入数值，每经过一个服务器就会减 1，当数值为 0 时就停止传输。 |





## HTTP 首部

有 4 种类型的首部字段：通用首部字段、请求首部字段、响应首部字段和实体首部字段。

[文档](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers#:~:text=An%20HTTP%20header%20consists%20of%20its%20case-insensitive%20name,its%20value.%20Whitespace%20before%20the%20value%20is%20ignored.)都写着啊，我真的无语了（为什么面试不能打开文档对着回答呢 🤒️ ）。

写在这当手册用一下吧。

### 通用首部字段

| 首部字段名 | 说明 |
| :--: | :--: |
| Cache-Control | 控制缓存的行为 |
| Connection | 控制不再转发给代理的首部字段、管理持久连接|
| Date | 创建报文的日期时间 |
| Pragma | 报文指令 |
| Trailer | 报文末端的首部一览 |
| Transfer-Encoding | 指定报文主体的传输编码方式 |
| Upgrade | 升级为其他协议 |
| Via | 代理服务器的相关信息 |
| Warning | 错误通知 |



### 请求首部字段

| 首部字段名 | 说明 |
| :--: | :--: |
| Accept | 用户代理可处理的媒体类型 |
| Accept-Charset | 优先的字符集 |
| Accept-Encoding | 优先的内容编码 |
| Accept-Language | 优先的语言（自然语言） |
| Authorization | Web 认证信息 |
| Expect | 期待服务器的特定行为 |
| From | 用户的电子邮箱地址 |
| Host | 请求资源所在服务器 |
| If-Match | 比较实体标记（ETag） |
| If-Modified-Since | 比较资源的更新时间 |
| If-None-Match | 比较实体标记（与 If-Match 相反） |
| If-Range | 资源未更新时发送实体 Byte 的范围请求 |
| If-Unmodified-Since | 比较资源的更新时间（与 If-Modified-Since 相反） |
| Max-Forwards | 最大传输逐跳数 |
| Proxy-Authorization | 代理服务器要求客户端的认证信息 |
| Range | 实体的字节范围请求 |
| Referer | 对请求中 URI 的原始获取方 |
| TE | 传输编码的优先级 |
| User-Agent | HTTP 客户端程序的信息 |



### 响应首部字段

| 首部字段名 | 说明 |
| :--: | :--: |
| Accept-Ranges | 是否接受字节范围请求 |
| Age | 推算资源创建经过时间 |
| ETag | 资源的匹配信息 |
| Location | 令客户端重定向至指定 URI |
| Proxy-Authenticate | 代理服务器对客户端的认证信息 |
| Retry-After | 对再次发起请求的时机要求 |
| Server | HTTP 服务器的安装信息 |
| Vary | 代理服务器缓存的管理信息 |
| WWW-Authenticate | 服务器对客户端的认证信息 |



### 实体首部字段


| 首部字段名 | 说明 |
| :--: | :--: |
| Allow | 资源可支持的 HTTP 方法 |
| Content-Encoding | 实体主体适用的编码方式 |
| Content-Language | 实体主体的自然语言 |
| Content-Length | 实体主体的大小 |
| Content-Location | 替代对应资源的 URI |
| Content-MD5 | 实体主体的报文摘要 |
| Content-Range | 实体主体的位置范围 |
| Content-Type | 实体主体的媒体类型 |
| Expires | 实体主体过期的日期时间 |
| Last-Modified | 资源的最后修改日期时间 |



## HTTPS

HTTPS，Hyper Text Transfer Protocol over SecureSocket Layer ，SecureSocket Layer 是我们平常所说的 SSL 加密协议。

HTTP 具有以下安全性问题：

- 使用明文进行通信，内容可能会被窃听；
- 不验证通信方的身份，通信方的身份有可能遭遇伪装；
- 无法证明报文的完整性，报文有可能遭篡改。

HTTPS 并不是新协议，而是让 HTTP 先和 SSL（Secure Sockets Layer）通信，再由 SSL 和 TCP 通信，也就是说 HTTPS 使用了隧道进行通信。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210224170323.jpg" style="width:67%;" />

### 加密

按照个人理解，加密实际上就是把一个 `key` 通过特定哈希算法，映射到一个值 `hash(key)` 。

**1. 对称密钥加密**

对称密钥加密（Symmetric-Key Encryption），加密和解密使用同一密钥。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210224170832.png" style="width:60%;" />

**2. 非对称密钥加密**

非对称密钥加密，又称公开密钥加密（Public-Key Encryption），加密和解密使用不同的密钥（公钥和私钥）。

公开密钥所有人都可以获得，通信发送方获得接收方的公开密钥之后，就可以使用公开密钥进行加密，接收方收到通信内容后使用私有密钥解密。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210224170935.png" style="width:60%;" />

**3. HTTPS 采用的加密方式**

上面提到对称密钥加密方式的传输效率更高，但是无法安全地将密钥 Secret Key 传输给通信方。而非对称密钥加密方式可以保证传输的安全性，因此我们可以利用非对称密钥加密方式将 Secret Key 传输给通信方。HTTPS 采用混合的加密机制，正是利用了上面提到的方案：

- 使用非对称密钥加密方式，传输对称密钥加密方式所需要的 Secret Key，从而保证安全性;
- 获取到 Secret Key 后，再使用对称密钥加密方式进行通信，从而保证效率。（下图中的 Session Key 就是 Secret Key）

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210225123944.png" style="width:80%;" />

### 认证

HTTPS 通过使用 **数字证书** 来对通信方进行认证。

数字证书认证机构（CA，Certificate Authority）是客户端与服务器双方都可信赖的第三方机构。

服务器的运营人员向 CA 提出公开密钥的申请，CA 在判明提出申请者的身份之后，会对已申请的公开密钥做数字签名，然后分配这个已签名的公开密钥，并将该公开密钥放入公开密钥证书后绑定在一起。

进行 HTTPS 通信时，服务器会把证书发送给客户端。客户端取得其中的公开密钥之后，先使用数字签名进行验证，如果验证通过，就可以开始通信了。

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210225124854.png" style="width:70%;" />

## GET 与 POST 比较

**作用**

GET 用于获取资源，而 POST 用于传输实体主体。



**参数**

GET 和 POST 的请求都能使用额外的参数，但是 GET 的参数是以查询字符串出现在 URL 中，而 POST 的参数存储在实体主体中（一般是 JSON 格式）。**不能因为 POST 参数存储在实体主体中就认为它的安全性更高，因为照样可以通过一些抓包工具（Fiddler）查看。**

因为 URL 只支持 ASCII 码，因此 GET 的参数中如果存在中文等字符就需要先进行编码。例如 `中文` 会转换为 `%E4%B8%AD%E6%96%87`，而空格会转换为 `%20`。POST 参数支持标准字符集。

GET 方法例子：

```
GET /test/demo_form.asp?name1=value1&name2=value2 HTTP/1.1
```

POST 方法例子：

```
POST /test/demo_form.asp HTTP/1.1
Host: w3schools.com
name1=value1&name2=value2
```



**安全**

安全的 HTTP 方法不会改变服务器状态，也就是说它只是可读的。

GET 方法是安全的，而 POST 却不是，因为 POST 的目的是传送实体主体内容，这个内容可能是用户上传的表单数据，上传成功之后，服务器可能把这个数据存储到数据库中，因此状态也就发生了改变。

安全的方法除了 GET 之外还有：HEAD、OPTIONS。

不安全的方法除了 POST 之外还有 PUT、DELETE。



**幂等性**

幂等的 HTTP 方法，同样的请求被执行一次与连续执行多次的效果是一样的，服务器的状态也是一样的。换句话说就是，幂等方法不应该具有副作用（统计用途除外）。**所有的安全方法也都是幂等的。**

在正确实现的条件下，GET，HEAD，PUT 和 DELETE 等方法都是幂等的，而 POST 方法不是。这一点也不难理解：GET 一般是请求资源，多次请求，所得到的结果总归是一样的；但 POST 方法常用于提交表单，服务器可能需要向数据库插入一条记录，如果多次 POST，有可能插入多条记录。



**可缓存**

如果要对响应进行缓存，需要满足以下条件：

- 请求报文的 HTTP 方法本身是可缓存的，包括 GET 和 HEAD，但是 PUT 和 DELETE 不可缓存，POST 在多数情况下不可缓存的。
- 响应报文的状态码是可缓存的，包括：200, 203, 204, 206, 300, 301, 404, 405, 410, 414, and 501。
- 响应报文的 Cache-Control 首部字段没有指定不进行缓存。



**XMLHttpRequest**

> XMLHttpRequest 是一个 API，它为客户端提供了在客户端和服务器之间传输数据的功能。它提供了一个通过 URL 来获取数据的简单方式，并且不会使整个页面刷新。这使得网页只更新一部分页面而不会打扰到用户。XMLHttpRequest 在 AJAX 中被大量使用。

- 在使用 XMLHttpRequest 的 POST 方法时，浏览器会先发送 Header 再发送 Data。但并不是所有浏览器会这么做，例如火狐就不会。
- 而 GET 方法 Header 和 Data 会一起发送。

