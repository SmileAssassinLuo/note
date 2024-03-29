### 域名的解析



就像 IP 地址必须转换成 MAC 地址才能访问主机一样，域名也必须要转换成 IP 地址，这个过程就是“域名解析”。目前全世界有几亿个站点，有几十亿网民，而每天网络上发生的 HTTP 流量更是天文数字。这些请求绝大多数都是基于域名来访问网站的，所以 DNS 就成了互联网的重要基础设施，必须要保证域名解析稳定可靠、快速高效。DNS 的核心系统是一个三层的树状、分布式服务，基本对应域名的结构：

+ 根域名服务器（Root DNS Server）：管理顶级域名服务器，返回“com”“net”“cn”等顶级域名服务器的 IP 地址；
+ 顶级域名服务器（Top-level DNS Server）：管理各自域名下的权威域名服务器，比如 com 顶级域名服务器可以返回 apple.com 域名服务器的 IP 地址；
+ 权威域名服务器（Authoritative DNS Server）：管理自己域名下主机的 IP 地址，比如 apple.com 权威域名服务器可以返回 www.apple.com 的 IP 地址

例如，你要访问“www.apple.com”，就要进行下面的三次查询：

+ 访问根域名服务器，它会告诉你“com”顶级域名服务器的地址；
+ 访问“com”顶级域名服务器，它再告诉你“apple.com”域名服务器的地址；
+ 最后访问“apple.com”域名服务器，就得到了“www.apple.com”的地址。



在核心 DNS 系统之外，还有两种手段用来减轻域名解析的压力，并且能够更快地获取结果，基本思路就是**“缓存”**。

顺序：浏览器缓存->操作系统缓存->hosts->dns 