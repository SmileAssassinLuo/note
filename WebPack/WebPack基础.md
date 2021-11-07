##### WebPack

##### 一，模块化

以往页面通过引入`JS`文件的方式，来加载脚本，会有命名冲突，重复，和任意修改的问题。有命名空间，函数作用域控制等方式来解决命名冲突，变量唯一等。

```javascript
//利用作用域和闭包，实现一个基础的模块化;
//往顶层作用局window暴露的只有tell方法，name和gender是局部变量，不会被外界修改。
(function(window){
    var name = "SunSan";
    var gender = "F";
    function tell(){
        console.log(`名字叫${name},性别是：${gender}。`)
    }
    window.SunSanModule = {tell};
})(window)
```

模块化方案简单总结：

- CommonJS规范主要用于服务端编程，加载模块是同步的，这并不适合在浏览器环境，因为同步意味着阻塞加载，浏览器资源是异步加载的，因此有了AMD CMD解决方案。
- AMD规范在浏览器环境中异步加载模块，而且可以并行加载多个模块。不过，AMD规范开发成本高，代码的阅读和书写比较困难，模块定义方式的语义不顺畅。
- CMD规范与AMD规范很相似，都用于浏览器编程，依赖就近，延迟执行，可以很容易在Node.js中运行。不过，依赖SPM 打包，模块的加载逻辑偏重。
- **ES6 在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案**。**ES6 模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块**。

##### 二，WebPack

**webpack** 是一个用于现代 JavaScript 应用程序的 *静态模块打包工具*。

 webpack 中的几个术语：

- **module**：指在模块化编程中我们把应用程序分割成的独立功能的代码模块。
- **chunk**：指模块间按照引用关系组合成的代码块，一个 `chunk` 中可以包含多个` module`。
- **chunk group**：指通过配置入口点（`entry point`）区分的块组，一个`chunk group `中可包含一到多个 `chunk`。
- **bundling**：`webpack` 打包的过程。
- **asset/bundle**：打包产物。

###### (1)打包机制

* `webpack`与立即执行函数的关系。

* `webpack`打包的核心逻辑。

  * 大致结构：

    ```javascript
    (function(modules){
    	var installedModules ={};
    	function __webpack_require_(moduleId){
    		/*code */
    		//...
    	}
    	return _webpack_require_(0); // ([entry frle]) 
    	
     }([ /* modules array */ ]);
    ```

  * 核心逻辑

    ```javascript
     function _webpack_require (moduleId){
       // Check if module is in cache
       if(installedModules[moduleId]){
           return installedModules[moduleId].exports;
       }
       // Create a new module (and put it into the cache) 
       var module = installedModules[moduleId]={
           i: moduleId, 
           l:false, 
           exports:0
       };
       // Execute the module function 
       modules[moduleId].call(
            module.exports,
            module,module.exports ,
            _webpack_require_
           );
       // Flag the module as loaded 
       module.l =true;
       // Return the exports of the module 
       return module.exports;
     }
    ```

* Webpack的打包过程

  ![](D:\work\note\WebPack\img\webpack构建流程.png)

  

  这个过程核心完成了 **「内容转换 + 资源合并」** 两种功能，实现上包含三个阶段：

  1. 初始化阶段：**「初始化参数」**：从配置文件、 配置对象、Shell 参数中读取，与默认配置结合得出最终的参数**「创建编译器对象」**：用上一步得到的参数创建 `Compiler` 对象**「初始化编译环境」**：包括注入内置插件、注册各种模块工厂、初始化 RuleSet 集合、加载配置的插件等**「开始编译」**：执行 `compiler` 对象的 `run` 方法**「确定入口」**：根据配置中的 `entry` 找出所有的入口文件，调用 `compilition.addEntry` 将入口文件转换为 `dependence` 对象。

  2. 构建阶段：**「编译模块(make)」**：根据 `entry` 对应的 `dependence` 创建 `module` 对象，调用 `loader` 将模块转译为标准 JS 内容，调用 JS 解释器将内容转换为 AST 对象，从中找出该模块依赖的模块，再 递归 本步骤直到所有入口依赖的文件都经过了本步骤的处理**「完成模块编译」**：上一步递归处理所有能触达到的模块后，得到了每个模块被翻译后的内容以及它们之间的 **「依赖关系图」**。

     构建阶段流程：

     * 调用 `handleModuleCreate` ，根据文件类型构建 `module` 子类；
     * 调用 [loader-runner](https://www.npmjs.com/package/loader-runner) 仓库的 `runLoaders` 转译 `module` 内容，通常是从各类资源类型转译为 JavaScript 文本；
     * 调用 [acorn](https://www.npmjs.com/package/acorn) 将 JS 文本解析为 AST；
     * 遍历 AST，触发各种钩子在 `HarmonyExportDependencyParserPlugin` 插件监听 `exportImportSpecifier` 钩子，解读 JS 文本对应的资源依赖调用 `module` 对象的 `addDependency` 将依赖对象加入到 `module` 依赖列表中；
     * AST 遍历完毕后，调用 `module.handleParseResult` 处理模块依赖；
     * 对于 `module` 新增的依赖，调用 `handleModuleCreate` ，控制流回到第一步；
     * 所有依赖都解析完毕后，构建阶段结束；

     **这个过程中数据流 `module => ast => dependences => module` ，先转 AST 再从 AST 找依赖。这就要求 `loaders` 处理完的最后结果必须是可以被 acorn 处理的标准 JavaScript 语法，比如说对于图片，需要从图像二进制转换成类似于 `export default "data:image/png;base64,xxx"` 这类 base64 格式或者 `export default "http://xxx"` 这类 url 格式。**

  3. 生成阶段：**「输出资源(seal)」**：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 `Chunk`，再把每个 `Chunk` 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会**「写入文件系统(emitAssets)」**：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。

单次构建过程自上而下按顺序执行，下面是上述提及的各类技术名词释义：

- `Entry`：编译入口，webpack 编译的起点。
- `Compiler`：编译管理器，webpack 启动后会创建 `compiler` 对象，该对象一直存活知道结束退出。
- `Compilation`：单次编辑过程的管理器，比如 `watch = true` 时，运行过程中只有一个 `compiler` 但每次文件变更触发重新编译时，都会创建一个新的 `compilation` 对象。
- `Dependence`：依赖对象，webpack 基于该类型记录模块间依赖关系。
- `Module`：webpack 内部所有资源都。会以“module”对象形式存在，所有关于资源的操作、转译、合并都是以 “module” 为基本单位进行的。
- `Chunk`：编译完成准备输出时，webpack 会将 `module` 按特定的规则组织成一个一个的 `chunk`，这些 `chunk` 某种程度上跟最终输出一一对应。
- `Loader`：资源内容转换器，其实就是实现从内容 A 转换 B 的转换器。
- `Plugin`：webpack 构建过程中，会在特定的时机广播对应的事件，插件监听这些事件，在特定时间点介入编译过程。



​	

###### (2)打包配置文件

* `mkdir webpack-demo` 新建项目。
* `cd webpack-demo 并且 npm init -y ` 初始化。
* `npm install webpack webpack-cli --save` 安装最新版webpack及其脚手架，可加@指定版本，在`node_modules\.bin`目录下通过`webpack -v` 查看版本号。
* 执行 `npx webpack`，会将我们的脚本作为[入口起点](https://www.webpackjs.com/concepts/entry-points)，然后 输出 为 `main.js`。

第一种：通过使用`webpack-cli`,执行`npx webpack`,会在dist目录下打包成`main.js`。

第二种：在项目根目录新建`webpack.config.js`文件：可通过执行配置文件的方式进行打包工作，会在`dist`目录下输出`bundle.js`.

命令：`npx webpack --config webpack.config.js`

```javascript
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

##### 三，WebPack构建工程

通过`WebPack`构建`react`项目,初始化项目`npm init -y`;

1. 依次安装如下依赖，`yarn add --dev xxx`可以将依赖加载至`devDependencies`

   ```json
   {
     "name": "webpack-react-demo",
     "version": "1.0.0",
     "description": "",
     "main": "index.js",
     "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1",
       "server":"webpack server"
     },
     "keywords": [],
     "author": "",
     "license": "ISC",
     "dependencies": {
       "@babel/cli": "^7.14.8",
       "@babel/core": "^7.15.0",
       "@babel/preset-env": "^7.15.0",
       "@babel/preset-react": "^7.14.5",
       "babel-loader": "^8.2.2",
       "react": "^17.0.2",
       "react-dom": "^17.0.2",
       "webpack": "^5.51.1",
       "webpack-cli": "^4.8.0"
     },
     "devDependencies": {
       "html-webpack-plugin": "^5.3.2",
       "webpack-dev-server": "^4.0.0"
     }
   }
   ```

2. 编写`webpack.config.js`文件

   ```js
   const HtmlWebpackPlugin = require('html-webpack-plugin');
   const path = require('path');
   const webpack = require('webpack');
   module.exports = {
       entry:path.resolve(__dirname,'src/index.jsx'),
       resolve:{
           extensions:['.js','.jsx','.json']  // 忽略后缀
       },
       module:{
           rules:[
               {
                   test:/\.jsx?/,
                   exclude:/node_modules/,
                   use:{
                       loader:'babel-loader',
                       options:{
                           babelrc:false,//不要再去查找babelrc文件
                           presets:[
                               require.resolve("@babel/preset-react"),
                               [require.resolve("@babel/preset-env",{modules:false})]//设置为false，不会将JSX文件中的import export等语句转译
                           ]
                       }
                   }
               }
           ]
       },
       plugins:[
            //自动生成一个html，并加载打包后的js；
           new HtmlWebpackPlugin({
               template:path.resolve(__dirname,'src/index.html'),
               filename:'index.html'
           }),
           //增加代码热更新
           new webpack.HotModuleReplacementPlugin()
       ],
       devServer:{
           hot:true
       },
       //打包环境设置
       mode:"development"
   }
   
   ```

   

3. 执行`npx webpack `,会在`dist`目录下生成`index.html和main.js`。

4. 项目基础构建完成，执行`npx webpack server` (通过cli方式)，或者添加 `npm`脚本的方式执行`npm run server`。

##### 四，优化

###### (1)打包结果优化

###### (2)构建过程优化

###### (3)Tree-Shaking优化

​	**Tree-Shaking**：用于描述移除 JavaScript 上下文中的未引用代码(dead-code)。





##### loader

###### (1)loader的执行顺序,自定义loader

 	新建loader目录，并新建3个loader文件，命名`loader1`，`loader2`, `loader3`;

```js
//loader1:
module.exports = function(content,map,meta){
    console.log("111");
    return content;
}
module.exports.pitch = function(){
    console.log("pitch 111");
```

`webpack.config.js`：

```json
module.exports = {
    module:{
        rules:[
            {
                test:/\.js$/,
                //loader:'loader1'  //单个loader
                //多个loader,执行顺序从下往上，一次打印 333,222,111
                use:[
                    'loader1',
                    'loader2',
                    'loader3'
                ]
            }
        ]
    },
    //配置loader解析规则
    resolveLoader:{
        modules:[
            "node_modules",
            path.resolve(__dirname,'loader')
        ]
    }
}
```



执行`$ npx webpack --mode development`

```
log输出：
pitch 111
pitch 222
pitch 333
333
222
111
```

`loader`的`pitch`方法按顺序先执行，`loader`基本方法是按照倒叙执行。

###### (2)同步&异步loader

```js
//同步：以`loader1.js`为例：
module.exports = function(content,map,meta){
    console.log("111");
    //同步,调用内部callback方法
    this.callback(null,content);
   // return content;
}
module.exports.pitch = function(){
    console.log("pitch 111");
}

//异步:以`loader2.js`为例：
//会在222 打印之后会停顿。
module.exports = function(content,map,meta){
    console.log("222");
    const callback = this.async();
    setTimeout(() => {
        callback(null,content)
    }, 1000);
}
module.exports.pitch = function(){
    console.log("pitch 222");
}
```

###### (3)获取options的值

使用多个`loader`时`webpack`一般会配置成对象，例如下例`loader3`,

```js

const path = require('path');
module.exports = {
    module:{
        rules:[
            {
                test:/\.js$/,
                //loader:'loader1'  //单个loader
                //多个loader,执行顺序从下往上，一次打印 333,222,111
                use:[
                    'loader1',
                    'loader2',
                    {
                        loader:'loader3',
                        options:{
                            name : 'Isaiah',
                            age:18
                        }
                    }
                ]
            }

        ]
    },
    //配置loader解析规则
    resolveLoader:{
        modules:[
            "node_modules",
            path.resolve(__dirname,'loader')
        ]
    }
}
```

在`loader3.js`中获取`options`的值,并对值做校验

```js
//需要安装 loader-utils 和 schema-utils，
const { getOptions }  = require('loader-utils'); // 获取options内容
const { validate } = require('schema-utils');
const schema = require('../schema/schema');// 验证规则

module.exports = function(content,map,meta){
    const options = getOptions(this);
    console.log("333",options)
    //指定验证规则，options内容，以及提示，验证通过继续往下走，失败会在控制台打印错误
    validate(schema, options,{
        name:"loader3"
    }) 
    return content;
}
module.exports.pitch = function(){
    console.log("pitch 333");
}


```

`schema.json`验证规则,`additionalProperties`表示是否可追加属性，`true`代表追加时，验证不会报错。

```json
{
    "type":"object",
    "properties":{
        "name" : {
            "type" : "string",
            "descriptions":"名称~"
        }
    },
    "additionalProperties":true 
}
```

###### (4)babel-loader

自定义`babel-loader`

`webpack.config.js`

```js
const path = require('path');
module.exports = {
    module:{
        rules:[
            {
                test:/\.js$/,
                loader:"babelLoader",
                options:{
                    presets:["@babel/preset-env"]
                }
            }

        ]
    },
    //配置loader解析规则
    resolveLoader:{
        modules:[
            "node_modules",
            path.resolve(__dirname,'loader')
        ]
    }
}
```

`babelLoader.js`

```js
const { getOptions }  = require("loader-utils");
const { validate } = require("schema-utils");
const babelCustomize = require("@babel/core");//编译的核心库；

const babelSchema = require('../schema/babelSchema.json')

//babel.transformAsync:用来编译js的代码,并返回一个promise的异步方法
module.exports = function(content,map,meta){
    const option  = getOptions(this);
    validate(babelSchema,option,{
        name:"Babel Loader"
    });
    //创建异步
    const callback  = this.async();
    //所用 transformAsync 编译代码，返回promise对象，正常返回值result传给callback
    babelCustomize.transformAsync(content,option)
        .then((result) => {
            callback(null , result.code , result.map , meta)
        })
        .catch(e => callback(e))
}
```

`babelSchema.json`:

```json
{
    "type":"object",
    "properties":{
        "presets":{
            "type":"array"
        }
    },
    "additionalProperties":true
}
```



##### plugin

从形态上看，插件通常是一个带有 `apply` 函数的类。

`apply` 函数运行时会得到参数 `compiler` ，以此为起点可以调用 `hook` 对象注册各种钩子回调，例如： `compiler.hooks.make.tapAsync` ，这里面 `make` 是钩子名称，`tapAsync` 定义了钩子的调用方式，webpack 的插件架构基于这种模式构建而成，插件开发者可以使用这种模式在钩子回调中，插入特定代码。webpack 各种内置对象都带有 `hooks` 属性，比如 `compilation` 对象：

```javascript
class SomePlugin {
    apply(compiler) {
        compiler.hooks.thisCompilation.tap('SomePlugin', (compilation) => {
            compilation.hooks.optimizeChunkAssets.tapAsync('SomePlugin', ()=>{});
        })
    }
}
```

钩子的核心逻辑定义在 [Tapable](https://github.com/webpack/tapable) 仓库，内部定义了如下类型的钩子：

```javascript
const {        
    SyncHook,        
    SyncBailHook,        
    SyncWaterfallHook,        
    SyncLoopHook,        
    AsyncParallelHook,        
    AsyncParallelBailHook,        
    AsyncSeriesHook,        
    AsyncSeriesBailHook,        
    AsyncSeriesWaterfallHook 
	} = require("tapable");
```

不同类型的钩子根据其并行度、熔断方式、同步异步，调用方式会略有不同，插件开发者需要根据这些的特性，编写不同的交互逻辑.



执行流程：

`CopyWebpckPlugin：Compiler Hook environment, Time: 123ms
CopyWebpckPlugin_2：Compiler Hook environment, Time: 125ms
CopyWebpckPlugin：Compiler Hook entryOption, Time: 365ms
CopyWebpckPlugin_2：Compiler Hook entryOption, Time: 365ms
CopyWebpckPlugin：Compiler Hook afterPlugins, Time: 688ms
CopyWebpckPlugin_2：Compiler Hook afterPlugins, Time: 688ms
CopyWebpckPlugin：Compiler Hook compile, Time: 693ms
CopyWebpckPlugin_2：Compiler Hook compile, Time: 693ms`



未完，待补充。。。