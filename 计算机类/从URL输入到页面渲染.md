# 从URL输入到页面渲染
**网络请求**  
**url 解析**  
浏览器判断是关键字还是 url,url 补全 关键字编码搜索  
完整的 url 应该包括 协议+主机+端口号+路径（参数+锚点）  
浏览器对我们的输入会进行转义，对不安全字符(安全字符指英文，数字，特殊字符) 进行百分号编码，基于 utf-8，一个中文等于 3 个字节，每个字节用百分号 16 进制表示  
encodeUrl 通常用于编码整个 url  
encodeUrlCompent 更加严格，会对？ % & 进行转义，用与处理链接后的参数  
**检查资源缓存**  
走缓存 请求头 size 会变成 memory cahe disk cache  
一般刷新页面取内存，重新打开标签取硬盘  
强制缓存 协商缓存  
**DNS 解析**  
浏览器 DNS 缓存 操作系统 DNS 缓存 路由器 DNS 缓存 服务器 DNS 查询 根域名服务器查询  
为节省时机可以在 html 头部做 DNS 的预解析  
**建立 TCP 三次连接**  
**TLS 协商密钥**  
非对称加密方式协商出一个密钥  
1.客户端发送一个随机值，以及需要的协议和加密方式  
2.服务端接受到客服端的随机值，发送自己的证书，并附加一个随机值，发回协议和加密方式  
3.客户发验证证书有效，生成一个随机值，通过证书中的共钥加密这个随机值，给服务端  
4.服务端用自己的私钥解密随机值，此时两端各有 3 个随机值。用这个 3 个随机值按照之前的约定的加密方式生成密钥，  
之后通信通过这个密钥进件对称加密解密  
**发送请求&接受请求**  
请求行 请求头 请求体  
**关闭 TCP 连接**  
客户端请求释放  
服务端收到 继续处理  
服务端请求释放  
客户单确认释放  
2MSL 之后正式关闭  
**浏览器渲染**  
构建 dom 树  
样式计算  
布局定位  
图层分层  
图层绘制  
显示

<br>
  
> 语雀地址 https://www.yuque.com/yuqueyonghuyv23kd/zxkn1f/kv6qgu