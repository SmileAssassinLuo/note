

#### 一，模块化

早期**JavaScript** 开发很容易存在**全局污染**和**依赖管理**混乱问题。

#### 二，Commonjs

`Commonjs` 的提出，弥补 Javascript 对于模块化，没有统一标准的缺陷。nodejs 借鉴了 `Commonjs` 的 Module ，实现了良好的模块化管理。

目前 `commonjs` 广泛应用于以下几个场景：

- `Node` 是 CommonJS 在服务器端一个具有代表性的实现；
- `Browserify` 是 CommonJS 在浏览器中的一种实现；
- `webpack` 打包工具对 CommonJS 的支持和转换；也就是前端应用也可以在编译之前，尽情使用 CommonJS 进行开发。

##### 1.commonjs 使用与原理

在使用  规范下，有几个显著的特点。

- 在 `commonjs` 中每一个 `js` 文件都是一个单独的模块，我们可以称之为 `module`；
- 该模块中，包含 `CommonJS` 规范的核心变量: `exports`、`module.exports`、`require`；
- `exports` 和 `module.exports` 可以负责对模块中的内容进行导出；
- `require` 函数可以帮助我们导入其他模块（自定义模块、系统模块、第三方库模块）中的内容；

###### commonjs 实现原理

每个模块文件上存在 `module`，`exports`，`require`三个变量，然而这三个变量是没有被定义的，但是我们可以在 Commonjs 规范下每一个` js` 模块上直接使用它们。在 `nodejs` 中还存在 `__filename` 和 `__dirname` 变量.

* `module`:记录当前模块的信息。
* `require`:引入模块的方法。
* `exports`: 当前模块导出的属性。

在编译时，`Commonjs`对`js`代码块进行了首尾包装，

```javascript
(function(exports,require,module,__filename,__dirname){
    const sayName = require('./hello.js');
    module.exports  = function say(){
        return {
            name:sayName(),
            author:"isaiah"
        }
    }
})
```

- 在 Commonjs 规范下模块中，会形成一个包装函数，我们编写的代码将作为包装函数的执行上下文，使用的 `require` ，`exports` ，`module` 本质上是通过形参的方式传递到包装函数中的。

  包装函数：

  ```javascript
  function wrapper (script) {
      return '(function (exports, require, module, __filename, __dirname) {' + 
          script +
       '\n})'
  }
  ```

- 包装函数执行：

  ```javascript
  const modulefunction = wrapper(`
      const sayName = require('./hello.js');
      module.exports  = function say(){
          return {
              name:sayName(),
              author:"isaiah"
          }
      }
  `)
  ```

- 上述模拟一个包装函数功能，返回的包装函数为一个字符串。

  ```javascript
  runInThisContext(modulefunction)((module.exports, require, module, __filename, __dirname)
  ```

- 在模块加载的时候，会通过 **runInThisContext** (可以理解成 **eval** ) 执行 `modulefunction` ，传入`require` ，`exports` ，`module` 等参数。最终我们写的 nodejs 文件就这么执行了。

 ##### 2.require 文件加载流程

以`nodejs`中文件加载为例：

```javascript
const fs = require('fs');  // 1.核心模块
const sayName = require('./hello.js');// 2.文件模块 
const crypto = require('crypto-js'); // 3.第三方自定义模块
```

- ① 为 nodejs 底层的核心模块。
- ② 为我们编写的文件模块，比如上述 `sayName`
- ③ 为我们通过 npm 下载的第三方自定义模块，比如 `crypto-js`。

根据`require`的参数，分为核心模块，文件模块，自定义模块；

* 核心模块：在`NODE	`源码中被编译成二进制代码，加载速度最快；
* 文件模块：以路径为标识，require会根据路径加载并缓存起来。
* 自定义模块：查找规则如下
  - 在当前目录下的 `node_modules` 目录查找。
  - 如果没有，在父级目录的 `node_modules` 查找，如果没有在父级目录的父级目录的 `node_modules` 中查找。
  - 沿着路径向上递归，直到根目录下的 `node_modules` 目录。
  - 在查找过程中，会找 `package.json` 下 main 属性指向的文件，如果没有  `package.json` ，在 node 环境下会以此查找 `index.js` ，`index.json` ，`index.node`。

##### 3.require 模块引入与处理

`CommonJS` 模块**同步加载**并**执行**模块文件，`CommonJS` 模块在执行阶段分析模块依赖，采用**深度优先遍历**（depth-first traversal），执行顺序是父 -> 子 -> 父；

举例说明：

`a.js:`

```javascript
const getMes = require('./b')
console.log('我是 a 文件')
exports.say =  function(){
    const message = getMes()
    console.log(message)
}
console.log(exports === module.exports);
```

`b.js:`

```js
const say = require('./a')
const  object = {
   name:'《React进阶实践指南》',
   author:'我不是外星人'
}

console.log('我是 b 文件')
//exports.name = "jajj";
module.exports = function(){
    return object
}
console.log(module.exports);
console.log(exports === module.exports);
```

`main.js:`

```js
const a = require('./a')
const b = require('./b')

console.log('node 入口文件')
```

`node ./main.js`:

输出：

<!-- 	我是 b 文件
		[Function]
		false
		我是 a 文件
		true
		node 入口文件	-->

**分析：**

- `main.js` 和 `a.js` 模块都引用了 `b.js` 模块，但是 `b.js` 模块只执行了一次。
- `a.js` 模块 和 `b.js` 模块互相引用，但是没有造成循环引用的情况。
- 执行顺序是父 -> 子 -> 父；



##### 4.require 加载原理

先了解下`module` 和 `Module`。

**`module`** ：在 Node 中每一个 js 文件都是一个 module ，module 上保存了 exports 等信息之外，还有一个 **`loaded`** 表示该模块是否被加载。

- 为 `false` 表示还没有加载；
- 为 `true` 表示已经加载

**`Module`** ：以 nodejs 为例，整个系统运行之后，会用 `Module` 缓存每一个模块加载的信息

部分`require`源码：

```js
 // id 为路径标识符
function require(id) {
   /* 查找  Module 上有没有已经加载的 js  对象*/
   const  cachedModule = Module._cache[id]
   
   /* 如果已经加载了那么直接取走缓存的 exports 对象  */
  if(cachedModule){
    return cachedModule.exports
  }
  /* 创建当前模块的 module  */
  const module = { exports: {} ,loaded: false , ...}

  /* 将 module 缓存到  Module 的缓存属性中，路径标识符作为 id */  
  Module._cache[id] = module
  /* 加载文件 */
  runInThisContext(wrapper('module.exports = "123"'))(module.exports, require, module, __filename, __dirname)
  /* 加载完成 *//
  module.loaded = true 
  /* 返回值 */
  return module.exports
}
```

- require 会接收一个参数——文件标识符，然后分析定位文件，分析过程我们上述已经讲到了，加下来会从 Module 上查找有没有缓存，如果有缓存，那么直接返回缓存的内容。
- 如果没有缓存，会创建一个 module 对象，缓存到 Module 上，然后执行文件，加载完文件，将 loaded 属性设置为 true ，然后返回 module.exports 对象。借此完成模块加载流程。
- 模块导出就是 return 这个变量的其实跟 a = b 赋值一样， 基本类型导出的是值， 引用类型导出的是引用地址。
- exports 和 module.exports 持有相同引用，因为最后导出的是 module.exports， 所以对 exports 进行赋值会导致 exports 操作的不再是 module.exports 的引用。

##### 5.require 避免重复加载，避免循环引用

require 如何避免重复加载的，首先加载之后的文件的 `module` 会被缓存到 `Module` 上，比如一个模块已经 require 引入了 a 模块，如果另外一个模块再次引用 a ，那么会直接读取缓存值 module ，所以无需再次执行模块。

**分析整个流程：**

- ① 首先执行 `node main.js` ，那么开始执行第一行 `require(a.js)`；
- ② 那么首先判断 `a.js` 有没有缓存，因为没有缓存，先加入缓存，然后执行文件 a.js （**需要注意 是先加入缓存， 后执行模块内容**）;
- ③ a.js 中执行第一行，引用 b.js。
- ④ 那么判断 `b.js` 有没有缓存，因为没有缓存，所以加入缓存，然后执行 b.js 文件。
- ⑤ b.js 执行第一行，再一次循环引用 `require(a.js)` 此时的 a.js 已经加入缓存，直接读取值。接下来打印 `console.log('我是 b 文件')`，导出方法。
- ⑥ b.js 执行完毕，回到 a.js 文件，打印 `console.log('我是 a 文件')`，导出方法。
- ⑦ 最后回到 `main.js`，打印 `console.log('node 入口文件')` 完成这个流程。

**不过这里我们要注意问题：**

- 如上第 ⑤ 的时候，当执行 b.js 模块的时候，因为 a.js 还没有导出 `say` 方法，所以 b.js 同步上下文中，获取不到 say。



修改`b.js:`

```javascript
const say = require('./a')
const  object = {
   name:'《React进阶实践指南》',
   author:'我不是外星人'
}
console.log('我是 b 文件')
console.log('打印 a 模块' , say)

setTimeout(()=>{
    console.log('异步打印 a 模块' , say)
},0)

module.exports = function(){
    return object
}
```

`a.js`:

```js
const getMes = require('./b')
console.log('我是 a 文件')
exports.say =  function(){
    const message = getMes()
    console.log(message)
}
console.log(exports === module.exports);
```

`mian.js:`

```js
const a = require('./a')
const b = require('./b')
console.log('node 入口文件')
```

`node ./main.js`:

输出：

<!-- 	我是 b 文件
		打印a模块 {}
		false
		我是 a 文件
		true
		node 入口文件
		异步打印a模块 { say: [Function] }	-->

`打印a模块 {}`是一个空对象，那么如何获取到 say 呢，有两种办法：

- 一是用动态加载 a.js 的方法，require动态加载。
- 二个就是如上放在异步中加载。

我们注意到 a.js 是用 `exports.say` 方式导出的，再次修改`a.js:`改为 `module.exports`导出：

```js
const getMes = require('./b')
console.log('我是 a 文件')
module.exports =  function(){
    const message = getMes()
    console.log(message)
}
console.log(exports === module.exports);
```

`node ./main.js`:

输出：

<!-- 	我是 b 文件
		打印a模块 {} 
		false        
		我是 a 文件  
		false        
		node 入口文件
		异步打印a模块 {}	-->

异步输出也是一个空对象。**WHY?**,`exports`和`module.exports`的区别在哪？



##### 6.exports 和 module.exports

举例说明

`test.js`

```js
exports.name = `<<ddd>>`;
exports.author = "ddd";
exports.say = function(){
    console.log(222);
}
console.log(module.exports);
```

`main.js:`

```js
const a = require('./a')
console.log(a)
```

输出：

<!-- 	{ name: '<<ddd>>', author: 'ddd', say: [Function] } 
		{ name: '<<ddd>>', author: 'ddd', say: [Function] } 	-->

`exports` 就是传入到当前模块内的一个对象，本质上就是 `module.exports`。两者指向同一个引用。



**问题：为什么 exports={} 直接赋值一个对象就不可以呢？** 比如我们将如上 `a.js` 修改一下：

```javascript
exports={
    name:'《React进阶实践指南》',
    author:'我不是外星人',
    say(){
        console.log(666)
    }
}
```

输出：

<!-- {}	-->



###### module.exports 使用

从上述 `require` 原理实现中，我们知道了 exports 和 module.exports 持有相同引用，因为最后导出的是 module.exports 。那么这就说明在一个文件中，我们最好选择 `exports` 和 `module.exports` 两者之一，如果两者同时存在，很可能会造成覆盖的情况发生。比如如下情况：

```js
exports.name = 'alien' // 此时 exports.name 是无效的
module.exports ={
    name:'《React进阶实践指南》',
    author:'我不是外星人',
    say(){
        console.log(666)
    }
}
```

输出：

<!-- { name: '《React进阶实践指南》', author: '我不是外星人', say: [Function: say] }	--> 

#### Q & A

1 那么问题来了？**既然有了 `exports`，为何又出了 `module.exports`?**

答：如果我们不想在 commonjs 中导出对象，而是只导出一个**类或者一个函数**再或者其他属性的情况，那么 `module.exports` 就更方便了，如上我们知道 `exports` 会被初始化成一个对象，也就是我们只能在对象上绑定属性，但是我们可以通过 `module.exports` 自定义导出出对象外的其他类型元素。

```js
let a = 1
module.exports = a // 导出函数

module.exports = [1,2,3] // 导出数组

module.exports = function(){} //导出方法
```

2 与 `exports` 相比，`module.exports` 有什么缺陷 ？

答：`module.exports` 当导出一些函数等非对象属性的时候，也有一些风险，就比如循环引用的情况下。对象会保留相同的内存地址，就算一些属性是后绑定的，也能间接通过异步形式访问到。但是如果 module.exports 为一个非对象其他属性类型，在循环引用的时候，就容易造成属性丢失的情况发生了。



#### 三，ES Module

`Nodejs` 借鉴了 `Commonjs` 实现了模块化 ，从 `ES6` 开始， `JavaScript` 才真正意义上有自己的模块化规范，Es Module 的产生有很多优势，比如:

- 借助 `Es Module` 的静态导入导出的优势，实现了 `tree shaking`。
- `Es Module` 还可以 `import()` 懒加载方式实现代码分割。

在 `Es Module` 中用 `export` 用来导出模块，`import` 用来导入模块。但是 `export` 配合 `import` 会有很多种组合情况。

##### (1).导入导出

###### 默认导出 export default

导出`a.js`

```js
const name = '《React进阶实践指南》'
const author = '我不是外星人'
const say = function (){
    console.log('hello , world')
}
export default {
    name,
    author,
    say
} 
```

导入`a.js`

```js
import mes from './a.js'
console.log(mes) //{ name: '《React进阶实践指南》',author:'我不是外星人', say:Function }
```

###### 混合导入｜导出

导出模块：`a.js`

```js
export const name = '《React进阶实践指南》'
export const author = '我不是外星人'

export default  function say (){
    console.log('hello , world')
}
```

导入模块：`main.js` 中有几种导入方式：

第一种：

```js
import theSay , { name, author as  bookAuthor } from './a.js'
console.log(
    theSay,     // ƒ say() {console.log('hello , world') }
    name,       // "《React进阶实践指南》"
    bookAuthor  // "我不是外星人"
)
```

第二种：

```js
import theSay, * as mes from './a'
console.log(
    theSay, // ƒ say() { console.log('hello , world') }
    mes // { name:'《React进阶实践指南》' , author: "我不是外星人" ，default:  ƒ say() { console.log('hello , world') } }
)
```

- 导出的属性被合并到 `mes` 属性上， `export` 被导入到对应的属性上，`export default` 导出内容被绑定到 `default` 属性上。`theSay` 也可以作为被 `export default` 导出属性。

###### 无需导入模块，只运行模块

```js
import 'module' 
```

- 执行 module 不导出值  多次调用 `module` 只运行一次。

###### 动态导入

```js
const promise = import('module')
```

- `import('module')`，动态导入返回一个 `Promise`。为了支持这种方式，需要在 webpack 中做相应的配置处理。

###### 执行特性

ES6 module 和 Common.js 一样，对于相同的 js 文件，会保存静态属性。但是与 Common.js 不同的是 ，`CommonJS` 模块同步加载并执行模块文件，**ES6 模块提前加载并执行模块文件，ES6 模块在预处理阶段分析模块依赖，在执行阶段执行模块**，两个阶段都采用深度优先遍历，执行顺序是子 -> 父。

**`main.js`**

```js
console.log('main.js开始执行')
import say from './a'
import say1 from './b'
console.log('main.js执行完毕')
```

**`a.js`**

```js
import b from './b'
console.log('a模块加载')
export default  function say (){
    console.log('hello , world')
}
```

**`b.js`**

```js
console.log('b模块加载')
export default function sayhello(){
    console.log('hello,world')
}
```

- `main.js` 和 `a.js` 都引用了 `b.js` 模块，但是 b 模块也只加载了一次。
- 执行顺序是子 -> 父

效果如下：

<!-- 	b模块加载
		a模块加载
		main.js开始执行       
		main.js执行完毕	-->



###### 不能修改import导入的属性

```
a.js
export let num = 1
export const addNumber = ()=>{
    num++
}
```

`main.js`中

```
import {  num , addNumber } from './a'
num = 2
```

如果直接修改，那么会报错。如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/2KticQlBJtdyuU4RV8phMsekvbWsWVw2DS1N86xCXE8ewucAE6AWOHoGO4T8JBmfQ3PzzZphknpowamP4pmk5Kw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**属性绑定**

所以可以在 `main.js` 中这么修改。

```
import {  num , addNumber } from './a'

console.log(num) // num = 1
addNumber()
console.log(num) // num = 2
```

- 如上属性 num 的导入是绑定的。

接下来对 import 属性作出总结：

- 使用 import 被导入的模块运行在严格模式下。
- 使用 import 被导入的变量是只读的，可以理解默认为 const 装饰，无法被赋值
- 使用 import 被导入的变量是与原变量绑定/引用的，可以理解为 import 导入的变量无论是否为基本类型都是引用传递。

##### (2).import() 动态引入

`import()` 返回一个 `Promise` 对象， 返回的 `Promise` 的 then 成功回调中，可以获取模块的加载成功信息。

```js
//main.js
setTimeout(() => {
    const result  = import('./b')
    result.then(res=>{
        console.log(res)
    })
}, 0);
//b.js
export const name ='alien'
export default function sayhello(){
    console.log('hello,world')
}
```

打印如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/2KticQlBJtdyuU4RV8phMsekvbWsWVw2DL4PmgLzdoTEnu7bc1yxtgxIV938sVJBw9ecMe30g4yc3TzMk3hjk0Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)13.jpg

从打印结果可以看出 `import()`的基本特性。

- `import()` 可以动态使用，加载模块。
- `import()` 返回一个 `Promise` ，成功回调 then 中可以获取模块对应的信息。`name` 对应 name 属性， `default` 代表 `export default` 。`__esModule` 为 es module 的标识。

##### (3).import() 可以做一些什么

###### 动态加载

- 首先 `import()` 动态加载一些内容，可以放在条件语句或者函数执行上下文中。

```js
if(isRequire){
    const result  = import('./b')
}
```

###### 懒加载

- `import()` 可以实现懒加载，举个例子 vue 中的路由懒加载；

```js
[
   {
        path: 'home',
        name: '首页',
        component: ()=> import('./home') ,
   },
]
```

###### React中动态加载

```react
const LazyComponent =  React.lazy(()=>import('./text'))
class index extends React.Component{   
    render(){
        return <React.Suspense fallback={ <div className="icon"><SyncOutlinespin/></div> } >
               <LazyComponent />
           </React.Suspense>
    }
```

`React.lazy` 和 `Suspense` 配合一起用能够有动态加载组件的效果。`React.lazy` 接受一个函数，这个函数需要动态调用 `import()` 。

`import()` 这种加载效果，可以很轻松的实现**代码分割**。避免一次性加载大量 js 文件，造成首次加载白屏时间过长的情况。

##### (4).tree shaking 实现

Tree Shaking 在 Webpack 中的实现，是用来尽可能的删除没有被使用过的代码，一些被 import 了但其实没有被使用的代码。比如以下场景：

`a.js`：

```js
export let num = 1
export const addNumber = ()=>{
    num++
}
export const delNumber = ()=>{
    num--
}
```

`main.js`：

```js
import {  addNumber } from './a'
addNumber()
```

- 如上 `a.js` 中暴露两个方法，`addNumber`和 `delNumber`，但是整个应用中，只用到了 `addNumber`，那么构建打包的时候，`delNumber`将作为没有引用的方法，不被打包进来。

#### 四，Commonjs 和 Es Module 总结

##### (1).Commonjs 总结

`Commonjs` 的特性如下：

- CommonJS 模块由 JS 运行时实现。
- CommonJs 是单个值导出，本质上导出的就是 exports 属性。
- CommonJS 是可以动态加载的，对每一个加载都存在缓存，可以有效的解决循环引用问题。
- CommonJS 模块同步加载并执行模块文件。

##### (2).es module 总结

`Es module` 的特性如下：

- ES6 Module 静态的，不能放在块级作用域内，代码发生在编译时。
- ES6 Module 的值是动态绑定的，可以通过导出方法修改，可以直接访问修改结果。
- ES6 Module 可以导出多个属性和方法，可以单个导入导出，混合导入导出。
- ES6 模块提前加载并执行模块文件，
- ES6 Module 导入模块在严格模式下。
- ES6 Module 的特性可以很容易实现 `Tree Shaking` 和 `Code Splitting`。

