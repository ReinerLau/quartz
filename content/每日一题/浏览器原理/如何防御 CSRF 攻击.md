---
noteId: 1727841698803
---


# 如何防御 CSRF 攻击
---
- token：用户登录后，服务器生成一个随机 token 字符串，之后在每个请求都中添加 token 头信息，服务器验证每个请求的 token，避免只通过链接就能发起请求的问题，也避免请求自动携带 cookie 的单一验证问题
- SameSite 属性：设置 cookie 的 SameSite 属性，限制跨站请求发送 cookie
- Referer 头：验证请求是从同一域名发起的
