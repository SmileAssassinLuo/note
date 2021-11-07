#### PWA

Progressive Web App  渐进式网络app，技术组成包括：

* Service Work 服务工作线程
  * 常驻内存运行
  * 代理网络请求
  * 依赖HTTPS
* Promise
* fetch
* cache API 
* Notification API  : 消息推送



##### Service Work 

```javascript
navigator.serviceWorker.register('./sw.js',{
    scope:'/'}).then(register => {
    	console.log(register)
    },error => {
    	console.log(error)
    })
    
//sw.js
self.addEventListener('install', event => {
    console.log('install',event);   
    event.waitUntil(self.skipWaiting())
    //
})
self.addEventListener('activate', event => {
    console.log('activate',event);
    event.waitUntil(self.clients.claim())
})
self.addEventListener('fetch', event => {
    console.log('fetch',event);

})
```



##### Promise



##### fetch

```javascript
// const xhr = new XMLHttpRequest();
// xhr.responseType = 'json';
// xhr.onreadystatechange  = () => {
//     if(xhr.readyState == 4){
//         if(xhr.status >= 200 && xhr.status < 300){
//             console.log(xhr.response)
//         }
//     }
// }
// xhr.open('GET','./userInfo.json',true);
// xhr.send(null)

fetch('./userInfo.json').then(response => response.json()).then(info => console.log(info));
```



##### cache API

```javascript
//service work 三个生命周期函数；
const CACEH_NAME = 'cache-v3';
self.addEventListener('install', event => {
    console.log('install',event);
    //install 的时候写入缓存   
    event.waitUntil(caches.open(CACEH_NAME).then(cache => {
        cache.addAll([
            '/',
            './sw.css'
        ])
    }))
    
})
self.addEventListener('activate', event => {
    console.log('activate',event);
    
    /*
    *清除原有缓存是在 activate 生命周期中处理，改变缓存名字v3，
    *在浏览器控制台 application 中，查看存储选项，缓存资源大小。
    *然后点击 Service Work 中 skipWaiting, 会发现存储选项中，缓存资源减小了。
    */
    event.waitUntil(caches.keys().then(cacheNames => {
        return Promise.all(cacheNames.map(cacheName =>{
            if(cacheName !== CACEH_NAME){
               return  caches.delete(cacheName)
            }
        }))
    }))
})
self.addEventListener('fetch', event => {
    console.log('fetch',event);
    //fetch 的时候，查看是否有更新;
    event.respondWith(caches.open(CACEH_NAME).then(cache => {
        return cache.match(event.request).then(response => {
            if(response) {
                return response
            }
            /*有更新，通过cache.put()传入请求，以及一份response的clone。将网站shuted down,
             *刷新浏览器发现资源文件来自Service Work。类似web app 可以缓存资源。
             *
            */
            return fetch(event.request).then(response => {
                cache.put(event.request,response.clone());
                return response;
            })
        })
    }))


})
```



##### Notification 

浏览器页面中：

```javascript
//在浏览器控制台中
Notification.permission //查看浏览器通知属性 三个值 default ,granted ,denied 

Notification.requestPermission().then(permision => console.log(permision)) //在default 下会弹出询问窗口

//chrome 浏览器 地址栏前的 感叹号点击可以修改非默认情况下的值；

//在允许通知(granted)的情况下，调用下面语句，会在浏览器侧边弹出通知；
new Notification('hello world ',{body:'this is a notification'})
```

service work中：

```javascript
//在浏览器控制台 Console选项下，清除console 选项的右边，页面上下文 TOP 点击切换到service work下
Notification.permission  // "denied" : 默认是不允许的。
otification.requestPermission() ; //报错不存在，

// 理解：service work 上下文中无页面，也不知在哪里可弹。

//如何在 service work 中弹出通知呢？
//第一步：在页面上下文，修改为允许通知
//第二步：切换到service work 上下文
//第三步：调用 self.registration.showNotification('hello');相当于在页面中注册的registration
```



##### 项目中使用PWA

workbox :

create react app 的webpack 中已经依赖了workbox-webpack-plugin 插件，通过 npm run build 后，打包后的文件中除了service-work之外还会有一个 **precache-manifest.786edeb5b2aa3a9f4920d9a4c4aa4ca7**  的文件，就是 声明周期 **install** 的缓存数组的文件。

