#### Vue 初次渲染流程

模板编译：

<img src="D:\work\gitRespository\note\vue\img\模板渲染简易图.png" style="zoom:75%;" />

简易图示：细节待补充

<img src="D:\work\gitRespository\note\vue\img\vue初次渲染简易图.png" style="zoom:80%;" />

##### 一，initMixin 初始化构造函数

配置Vue构造函数的_init方法

```JS
import { initMixin } from './utils/init.js'
function Vue(options){
    // 初始化传进来的options配置
    this._init(options)
}
initMixin(Vue);

//通过 initMixin 给构造函数添加原型方法
import { initState } from './initState'
export const initMixin = (Vue) => {
    Vue.prototype._init = (options) =>{
        const vm = this;
        vm.$options = options;
        initState(vm)
    }
 
}
```

##### 二，new Vue 实例

实例化Vue时，即会调用 `_init`方法，对参数 options 进行初始化；

```js
const vm = new Vue({
	data(){return {}},
	computed:{},
	created(){},
	render:() => {}
})
```

将配置参数**options **初始化：

* 1.给 vm 实例，添加 `$options`属性，值便为 参数 **options**；
* 2.对**$options**参数的data取值，通过`Object.defineProperty()`对数据进行劫持，实现对数据源的监控；分别为对象和数组提供不同的监控方式，详见 [vue数据响应原理](D:\IsaiahLuo\note\vue\02vue数据响应原理.md)；（重点）
* 3.其他属性(props,computed,watch等)，均通过不同的方法，对其进行解析；

```js
function initState(vm) {
    // 获取vm上的$options对象，也就是options配置对象
    const opts = vm.$options
    if (opts.props) {
        initProps(vm)
    }
    if (opts.methods) {
        initMethods(vm)
    }
    if (opts.data) {
        // 如有有options里有data，则初始化data
        initData(vm)
    }
    if (opts.computed) {
        initComputed(vm)
    }
    if (opts.watch) {
        initWatch(vm)
    }
}
module.exports = { initState: initState }
```

##### 三，模板编译

当我们新建一个 vue 实例,是如何渲染到页面的呢？

```js
let vue = new Vue({
  render: h => h(App)
}).$mount('#app')
console.log(vue)
```

根据官网的图示分析：vue的生命周期函数

<img src="https://cn.vuejs.org/images/lifecycle.png" alt="vue生命周期" style="zoom: 50%;" />

> 1. 渲染到哪个根节点上：判断有无el属性，有的话直接获取el根节点，没有的话调用$mount去获取根节点
> 2. 渲染哪个模板：
>
> - 有render：这时候优先执行render函数，render优先级 > template
> - 无render：
>   - 有template：拿template去解析成render函数的所需的格式，并使用调用render函数渲染
>   - 无template：拿el根节点的outerHTML去解析成render函数的所需的格式，并使用调用render函数渲染
>
> 3.渲染的方式：无论什么情况，最后都统一是要使用render函数渲染
>
> 作者：Sunshine_Lin
> 链接：https://juejin.cn/post/69695636404164362

###### (1),$mount 函数的实现

```js
// init.js
const { initState } = require('./state')
const { compileToFunctions } = require('./compiler/index.js')

function initMixin(Vue) {
    Vue.prototype._init = function (options) {
        const vm = this
        vm.$options = options
        initState(vm)
        if(vm.$options.el) {
            vm.$mount(vm.$options.el)
        }
    }
    // 把$mount函数挂在Vue的原型上
    Vue.prototype.$mount = function(el) {
        // 使用vm变量获取Vue实例（也就是this）
        const vm = this
        // 获取vm上的$options
        // $options是在_init的时候就绑再vm上了
        const options = vm.$options
        // 获取传进来的dom
        el = document.querySelector(el)
        // el = {}
        // 如果options里没有render函数属性
        if (!options.render) {
            // 获取options里的template属性
            let template = options.template
            // 如果template属性也没有，但是dom获取到了
            if (!template && el) {
                // 那就把dom的outerHTML赋值给template属性
                template = el.outerHTML
            }
            // 如果有template属性有的话
            if (template) {
                // 那就把template传入compileToFunctions函数，生成一个render函数
                const render = compileToFunctions(template)
                // 把生成的render函数赋值到options的render属性上
                options.render = render
            }
        }
        // 记得return出Vue实例（也就是this）
        // 为了let vue = new Vue().$mount('#a')之后，能通过vue变量去访问这个Vue实例
        return this
    }
}
module.exports = {
    initMixin: initMixin
}
```

###### (2),解析template成`抽象语法树（AST）`，将`AST`转换成渲染函数

将一堆字符串模板解析成抽象语法树`AST`后，就可以对其进行各种操作处理了，处理完后用处理后的`AST`来生成`render`函数。

其具体流程可大致分为三个阶段：

<img src="D:\work\gitRespository\note\vue\img\模板编译.png" style="zoom:75%;" />



1. 模板解析阶段：将一堆模板字符串用正则等方式解析成抽象语法树`AST`；

   HTML解析器是主线，先用HTML解析器进行解析整个模板，在解析过程中如果碰到文本内容，那就调用文本解析器来解析文本，如果碰到文本中包含过滤器那就调用过滤器解析器来解析；

   **具体解析细节待补充**

2. 优化阶段：遍历`AST`，找出其中的静态节点，并打上标记；

   * 在`AST`中找出所有静态节点并打上标记；

   * 在`AST`中找出所有静态根节点并打上标记；

   **具体解析细节待补充**

3. 代码生成阶段：将`AST`转换成渲染函数；

   在此阶段要生成`render`函数字符串，根据已有的这个`AST`来生成对应的`render`函数。生成`render`函数的过程其实就是一个递归的过程，从顶向下依次递归`AST`中的每一个节点，根据不同的`AST`节点类型创建不同的`VNode`类型。

   **具体解析细节待补充**

   

***模板编译整理流程***:

![](D:\work\gitRespository\note\vue\img\模板编译整体流程.jpg)



##### 四，执行 _render  获得V-dom，执行 _updata 转为真实 dom 并渲染

**具体解析细节待补充**

