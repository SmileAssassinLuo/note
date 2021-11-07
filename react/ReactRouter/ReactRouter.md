#### ReactRouter 官方资源

react-router [https://github.com/ReactTraining/react-router](https://github.com/ReactTraining/react-router)

react-router doc [https://reactrouter.com/](https://reactrouter.com/)

react-router doc CN [https://react-guide.github.io/react-router-cn/](https://react-guide.github.io/react-router-cn/)



### packgage

React Router是React团队开发的基于React框架架构所实现的路由库，有多个版本。

* react-router
* react-router-dom
* react-router-native
* react-router-config

前端写web页面多数是使用**react-router-dom**这个库，那么react-router和react-router-dom有什么区别呢？

react-router-dom是基于react-router再封装的一个带有React DOM组件的库，其中包括了Link、HashRouter、BrowserRouter等组件提供给开发者通过使用标签的方式控制路由跳转。



#### React-Router-Dom

web开发常用的 **react-router-dom**有3个核心的组件（Primary Componentes）：

* routers：路由器，BrowserRouter 和 HashRouter.
* route matchers： 路由匹配，比如 Route ，Switch.
* navigation:导航链接，比如 Link ,NavLink,Redirect.

使用上述components ，需要 **import { xxx,xxx,xxx }  from 'react-router-dom'**.



#### routers

BrowserRouter ：这一种很自然，比如 `/` 对应 `Home页` ，`/about` 对应 `About 页`，但是`这样的设计需要服务器端渲染`，因为`用户可能直接访问任何一个 URL，服务器端必须能对 /的访问返回 HTML，也要对 /about的访问返回 HTML`。BrowserRouter支持这种URL。

HashRouter：这一种看起来不自然，但是实现更简单。`只有一个路径 /，通过 URL 后面的 # 部分来决定路由`，/#/ 对应 Home 页，/#/about 对应 About 页。因为URL中#之后的部分是不会发送给服务器的，所以，`无论哪个 URL，最后都是访问服务器的 / 路径，服务器也只需要返回同样一份 HTML`就可以，`然后由浏览器端解析 # 后的部分，完成浏览器端渲染`。HashRouter支持这种URL。



#### Switch 



#### 嵌套路由



#### 路由传参





#### react-router-dom 的hooks

* useParams
* useHistory
* useLocation
* useRouteMatch

