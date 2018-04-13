## 关于Http
> * TCP的三次握手建立连接
>   * 第一次：建立连接时，客户端发送SYN包(syn=j)到服务器，并进入SYN_SEND状态，等待服务器确认；
>   * 第二次：服务器收到SYN包，向客户端返回ACK（ack=j+1），同时自己也发送一个SYN包（syn=k），即SYN+ACK包，此时服务器进入SYN_RCVD状态；
>   * 第三次：客户端收到服务器的SYN＋ACK包，向服务器发送确认包ACK(ack=k+1)，此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手。
> * 四次挥手断开连接
>   * 第一次： TCP客户端发送一个FIN，用来关闭客户到服务器的数据传送。
>   * 第二次：服务器收到这个FIN，它发回一个ACK，确认序号为收到的序号加1。和SYN一样，一个FIN将占用一个序号。
>   * 第三次：服务器关闭客户端的连接，发送一个FIN给客户端。
>   * 第四次：客户端发回ACK报文确认，并将确认序号设置为收到序号加1。

## 关于Https建立连接过程
https不是一种新协议，而是http+SSL/TLS的结合体。它解决了http被监听、被篡改、被伪装的问题。
> * 共享密钥加密：共享密钥的加密密钥和解密密钥是相同的，所以又称为**对称密钥**
> * 公开密钥加密：加密算法是公开的，密钥是保密的。公开密钥分为私有密钥和公有密钥，公有密钥是公开的，任何人(客户端)都可以获取，客户端使用公有密钥加密数据，服务端用私有密钥解密数据。

* 客户端发送请求到服务端
* 服务端返回证书和公开密钥（证书由权威第三方机构颁发，并对公开密钥做了签名）
* 客户端验证证书和公开密钥的有效性，如果有效，则随机生成共享密钥并用公开密钥加密发送给服务端
* 服务端先使用私有密钥解密，再用解开的共享密钥对数据加密发送给客户端
* 客户端使用共享密钥解密数据
* SSL加密建立

## 关于部分Http状态码解释
* 2开头：请求成功
  * 200：服务器已成功处理了请求
  * 204：服务器成功处理了请求，但没有返回任何内容
* 3开头：请求被重定向
  * 301：**永久**重定向。请求的资源分配了新url，以后均使用新url
  * 302：**临时性**重定向。请求的资源分配了新url，本次请求临时使用新url
  * 303：请求的资源路径发生改变，在302的基础上，明确使用GET方法请求新url
  * 304：Not Modified. 服务器端资源未改变，可直接使用之前未过期的缓存，不返回内容
* 4开头：请求错误
  * 400：Bad Request. 请求体内存在语法错误，如url含有非法字符，或是json格式不正确
  * 401：Unauthorized. 需要重新进行身份验证
  * 403：Forbidden. 服务器拒绝请求
  * 404：Not Found. 服务器找不到给定资源
  * 405：Method Not Allowed. 请求方式（GET、POST、DELETE）与后台规定不一致
  * 408：Request Timeout. 请求超时
  * 415：Unsupported Media Type. content-type后台不支持
* 5开头：服务器错误
  * 500：Internal Server Error. 
  * 501：Not Implemented.
  * 502：Bad Gateway. 
  * 503：Service Unavailable. 目前无法使用，如超载或停机维护
  * 504：Gateway Timeout.
  * 505：TTP Version Not Unsupported.