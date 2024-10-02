---
noteId: 1727841699031
---


# CORS
---
正常情况下, 因为同源策略是浏览器的机制, 需要让浏览器判断资源是同源的才会去访问, 但通过 CORS 可以绕过这个机制

CORS 就是在发起跨域请求时让服务器返回特定的响应头给浏览器, 让浏览器判断是否可以放心地获取来自不同源的资源, 这些响应头包括:

- Access-Control-Allow-Origin: 指定允许访问资源的域
- Access-Control-Allow-Methods: 指定允许的[[每日一题之计算机网络#常见的 HTTP 请求方法|请求方法]]
- Access-Control-Allow-Headers: 指定允许的自定义请求头
- Access-Control-Allow-Credentials: 指定是否允许发送凭证, 比如 Cookie

跨域请求存在两种请求类型, 分别是**简单请求**和**预检请求**

简单请求是指浏览器只检查 `Accless-Control-Allow-Origin` 响应头来决定是否允许跨域的请求, 需要符合以下条件: 

- 请求方法: 只包括 `GET`、`POST`、`HEAD`
- 请求头: 只包括 `Accept`、`Content-type` 等基本请求头, 其中 `Content-Type` 只能是 `text/plain`、`application/x-www-form-urlencoded`、`multipart/form-data`

预检请求是指简单请求以外的请求, 会先发起 `OPTIONS` 请求询问服务器是否允许跨域请求, 浏览器通过判断服务器返回的头来判断是否继续发起实际请求, 比如 `Access-Control-Allow-Headers` 、`Access-Control-Allow-Methods` 等