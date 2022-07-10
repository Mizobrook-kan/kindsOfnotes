**前言**

最近在了解HTTPS的建立过程，RFC 2818规定了HTTPS的总体流程是先初始化连接，再由TLS加密。这篇文章主要介绍TLS 1.3的连接建立过程。

**连接建立流程**

~~这里还是请出类似TCP的连接建立图，TLS连接建立过程在RFC 8446定义，~~

TLS连接的建立过程与TCP连接建立过程类似，RFC 8446的握手协议描述了连接的建立过程，再配合RFC 2818描述的总流程，画出来的过程图基本是图1。

TLS连接建立可以分为三个阶段。第一阶段是密钥交换阶段，想要发起TLS连接请求的客户端向服务器发送ClientHello，服务器响应ServerHello作为应答，双方在各自的消息里面会记录需要交换的密钥，客户端的消息体里需要提供选择的加密算法。这个阶段受密钥交换模式控制，1.3支持三种交换模式：ECDHE、仅PSK、带ECDHE的PSK，默认是ECDHE模式，也是双方首次通信时的模式。

![1647995352560.png](image/TLS 1.3 /1647995352560.png)

第二阶段基本可以看作是一个过渡阶段，第一阶段结束后立即向客户端发送TLS连接的建立信息，比如采取的签名算法、TLS版本。

**密钥交换阶段**

~~ClientHello和ServerHello两条消息决定客户端和服务器能够建立哪种~~

~~signature_algorithm字段表示客户端可以接受哪些签名算法，TLS 1.3从大方向来说支持三种签名算法，ECDSA/EdDSA/RSA。signature_algorithm_cert指定证书应该使用哪种签名算法，缺少这个字段也是可以的，此时signature_algorithm字段用作证书的生成算法。如果不是PSK模式的话，就会使用ECDHE+证书的认证方式。~~

~~如果客户端缺少一些关键配置，服务器会发过来HelloRetryRequest，这条消息里面会告诉客户端缺少哪些东西，比如服务器想要cookie。客户端需要修改自己的ClientHello消息再次向服务器发送。~~

**证书验证**

CertificateVerify消息包含两个字段，一个是使用的签名算法，一个是使用该算法生成的数字签名。CertificateVerify消息如果是服务器发送的，签名算法必须是客户端发送ClientHello时signature_algorithms字段的其中一个。CertificateVerify消息如果是客户端发送的，签名算法必须是CertificateRequest消息中supported_signature_algorithms字段的其中一个。同时，签名算法必须与密钥兼容，RSA签名就只能使用RSASSA-PSS算法。

收到CertificateVerify消息的一方开始验证签名是否有效，检查数字签名的内容有没有缺失、能不能在对方发来的Certificate消息中找到公钥。

**一些碎碎念**
