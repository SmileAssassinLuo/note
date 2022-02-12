#### Axios

[Axios](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Faxios%2Faxios) 是一个基于 Promise 的 HTTP 客户端，同时支持浏览器和 Node.js 环境。它是一个优秀的 HTTP 客户端，被广泛地应用在大量的 Web 项目中。

##### 一，特性

- 支持 Promise API；
- 能够拦截请求和响应；
- 能够转换请求和响应数据；
- 客户端支持防御 [CSRF](https://link.juejin.cn/?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FCross-site_request_forgery) 攻击；
- 同时支持浏览器和 Node.js 环境；
- 能够取消请求及自动转换 JSON 数据。

##### 二，HTTP 拦截器的设计与实现

 Axios 提供了请求拦截器和响应拦截器来分别处理请求和响应，它们的作用如下：

- 请求拦截器：该类拦截器的作用是在请求发送前统一执行某些操作，比如在请求头中添加 token 字段。
- 响应拦截器：该类拦截器的作用是在接收到服务器响应后统一执行某些操作，比如发现响应状态码为 401 时，自动跳转到登录页




