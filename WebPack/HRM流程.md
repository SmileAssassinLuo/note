#### 浏览器热更新

浏览器的热更新，指的是我们在本地开发的同时打开浏览器进行预览，当代码文件发生变化时，浏览器自动更新页面内容的技术。这里的自动更新，表现上又分为自动刷新整个页面，以及页面整体无刷新而只更新页面的部分内容。

#### webpack 中的热更新原理

 `webpackDevServer` 中 `HMR` 的基本流程，完整的 HMR 功能主要包含了三方面的技术：

1. `watch` 示例中体现的，对本地源代码文件内容变更的监控。
2. `instant reload` 示例中体现的，浏览器网页端与本地服务器端的 `Websocket` 通信。
3. `hmr` 示例中体现的，也即是最核心的，模块解析与替换功能。

<img src="https://s0.lgstatic.com/i/image/M00/41/C7/Ciqc1F82OZmAFYuKAAC7WNDPQB4766.png" style="zoom:80%;" />

举例说明: 变更`css`时,使用 `style-loader`将`css`注入`style` 标签

控制台中看到的 `style-loader` 处理后的模块代码，只看其热替换相关的部分:

```js
//为了清晰期间，我们将模块名称注释以及与热更新无关的逻辑省略，并将 css 内容模块路径赋值为变量 cssContentPath 以便多处引用，实际代码可从示例运行时中查看 
var cssContentPath = "./node_modules/css-loader/dist/cjs.js!./src/style.css" 
var api = __webpack_require__("./node_modules/style-loader/dist/runtime/injectStylesIntoStyleTag.js"); 
            var content = __webpack_require__(cssContentPath); 
... 
var update = api(content, options); 
... 
module.hot.accept( 
  cssContentPath, 
  function(){ 
    content = __webpack_require__(cssContentPath); 
    ... 
    update(content); 
  } 
) 
module.hot.dispose(function() { 
  update(); 
});
```

**在运行时调用 API 实现将样式注入新生成的 style 标签，并将返回函数传递给 update 变量。然后，在 module.hot.accept 方法的回调函数中执行 update(content)，在 module.hot.dispose 中执行 update()。通过查看上述 API 的代码，可以发现 update(content) 是将新的样式内容更新到原 style 标签中，而 update() 则是移除注入的 style 标签.**



`module.hot`是什么呢?

module.hot 实际上是一个来自 webpack 的基础插件 ***HotModuleReplacementPlugin***，该插件作为热替换功能的基础插件，其 API 方法导出到了 module.hot 的属性中.

**hot.accept** 方法传入依赖模块名称和回调方法，当依赖模块发生更新时，其回调方法就会被执行，而开发者就可以在回调中实现对应的替换逻辑，即上面的用更新的样式替换原标签中的样式。另一个 **hot.dispose** 方法则是传入一个回调，当代码上下文的模块被移除时，其回调方法就会被执行。例如当我们在源代码中移除导入的 CSS 模块时，运行时原有的模块中的 update() 就会被执行，从而在页面移除对应的 style 标签。

**假如代码中并未包含对热替换插件 API 的调用，代码的解析也没有配置额外能对特定代码调用热替换 API 的 Loader,则不会观察到热更新，而是看到了整个页面的刷新.**

总结:热替换的实现，既依赖 webpack 核心代码中 HotModuleReplacementPlugin 所提供的相关 API，也依赖在具体模块的加载器中实现相应 API 的更新替换逻辑。因此，在配置中开启 hot:true 并不意味着任何代码的变更都能实现热替换，除了示例中演示的 style-loader 外， vue-loader、 react-hot-loader 等加载器也都实现了该功能。当开发时遇到 hmr 不生效的情况时，可以优先确认对应加载器是否支持该功能，以及是否使用了正确的配置。





