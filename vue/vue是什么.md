#### vue是什么？

在引入`vue.js`完整版文件或者通过脚手架安装了`vue`运行时版本后，会初始化Vue构造函数的全局方法，全局组件等；

##### 一，Vue 构造函数初始化

```js
/* @flow */

import config from '../config'
import { initUse } from './use'
import { initMixin } from './mixin'
import { initExtend } from './extend'
import { initAssetRegisters } from './assets'
import { set, del } from '../observer/index'
import { ASSET_TYPES } from 'shared/constants'
import builtInComponents from '../components/index'
import { observe } from 'core/observer/index'

import {
  warn,
  extend,
  nextTick,
  mergeOptions,
  defineReactive
} from '../util/index'

export function initGlobalAPI (Vue: GlobalAPI) {
  // config
  const configDef = {}
  configDef.get = () => config
  if (process.env.NODE_ENV !== 'production') {
    configDef.set = () => {
      warn(
        //不要替换 Vue.config 对象，而是设置单个字段
        'Do not replace the Vue.config object, set individual fields instead.'
      )
    }
  }
  Object.defineProperty(Vue, 'config', configDef)

  // exposed util methods.
  // NOTE: these are not considered part of the public API - avoid relying on
  // them unless you are aware of the risk.
  Vue.util = {
    warn,
    extend,
    mergeOptions,
    defineReactive
  }

  Vue.set = set
  Vue.delete = del
  Vue.nextTick = nextTick

  // 2.6 explicit observable API
  Vue.observable = <T>(obj: T): T => {
    observe(obj)
    return obj
  }

  Vue.options = Object.create(null)
  ASSET_TYPES.forEach(type => {
    Vue.options[type + 's'] = Object.create(null)
  })

  // this is used to identify the "base" constructor to extend all plain-object
  // components with in Weex's multi-instance scenarios.
  Vue.options._base = Vue

  extend(Vue.options.components, builtInComponents)

  initUse(Vue)
  initMixin(Vue)
  initExtend(Vue)
  initAssetRegisters(Vue)
}

```

初始化Vue全局的方法，属性，配置项，内置组件，数据劫持的方法，工具函数等；

* `initGlobalAPI(Vue)`: 劫持`Vue.config`属性，只可get不能set；

* 初始化`Vue.set = set`, `Vue.delete = del`,`Vue.nextTick = nextTick`方法；

* 初始化，`Vue.observable`方法；

* `Vue.util`:初始化一些工具函数，比如 警告提示，合并对象，`nextTick`等；

* 初始化`Vue.options`对象,包括组件，过滤器，指令等

  ```js
  Vue.options = {
  	isVue:boolean,
  	_base:Vue,
      components:{
         keep-alive
         transition
         transition-group
      },
      directives:{},
      filters:{},
  }
  ```

* `initUse(Vue)`: 初始化`Vue.use`方法，通过此方法，插件安装在Vue对象上，比如`Element`, 并返回`Vue`对象，可供链式调用；

*  `initMixin(Vue)`：`Vue.mixin`方法  

* `initExtend(Vue)`：`Vue.extend`方法  

* `initAssetRegisters(Vue)`：`Vue.component`，`Vue.directive`，`Vue.filter`方法

##### 2,new Vue(options)

```js
function Vue(options) {
  ...
  this._init(options)
}
```

在`vue`的内部，`_`符号开头定义的变量是供内部私有使用的，而`$` 符号定义的变量是供用户使用的，而且用户自定义的变量不能以`_`或`$`开头，以防止内部冲突。

```js
// src/core/instance/index.js
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'

function Vue(options) {
  ...
  this._init(options)
}
65
initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)
```

不采用`ES6`的`class`来定义,可以把`vue`的功能拆分到不同的目录中去维护，将`vue`的构造函数传入到以下方法内：

- `initMixin(Vue)`：定义`_init`方法。
- `stateMixin(Vue)`：定义数据相关的方法`$set`,`$delete`,`$watch`方法。
- `eventsMixin(Vue)`：定义事件相关的方法`$on`，`$once`，`$off`，`$emit`。
- `lifecycleMixin(Vue)`：定义`_update`，及生命周期相关的`$forceUpdate`和`$destroy`。
- `renderMixin(Vue)`：定义`$nextTick`，`_render`将render函数转为`vnode`。

这里有部分`API`和`xxxMixin`定义的原型方法功能是类似或相同的，如`this.$set`和`Vue.set`他们都是使用`set`这样一个内部定义的方法。

这里需要提一下`vue`的架构设计，它的架构是分层式的。最底层是一个`ES5`的构造函数，再上层在原型上会定义一些`_init`、`$watch`、`_render`等这样的方法，再上层会在构造函数自身定义全局的一些`API`，如`set`、`nextTick`、`use`等(以上这些是不区分平台的核心代码)，接着是跨平台和服务端渲染(这些暂时不在讨论范围)及编译器。将这些属性方法都定义好了之后，最后会导出一个完整的构造函数给到用户使用，而`new Vue`就是启动的钥匙。



##### 3,源码目录

代码结构是如何组建起来的?

```
|-- dist  打包后的vue版本
|-- flow  类型检测，3.0换了typeScript
|-- script  构建不同版本vue的相关配置
|-- src  源码
    |-- compiler  编译器
    |-- core  不区分平台的核心代码
        |-- components  通用的抽象组件
        |-- global-api  全局API
        |-- instance  实例的构造函数和原型方法
        |-- observer  数据响应式
        |-- util  常用的工具方法
        |-- vdom  虚拟dom相关
    |-- platforms  不同平台不同实现
    |-- server  服务端渲染
    |-- sfc  .vue单文件组件解析
    |-- shared  全局通用工具方法
|-- test 测试

```

* flow：`javaScript`是弱类型语言，使用`flow`以定义类型和检测类型，增加代码的健壮性。

* src/compiler：将`template`模板编译为`render`函数。
* src/core：与平台无关通用的逻辑，可以运行在任何`javaScript`环境下，如`web`、`Node.js`、`weex`嵌入原生应用中。
* src/platforms：针对`web`平台和`weex`平台分别的实现，并提供统一的`API`供调用。
* src/observer：`vue`检测数据数据变化改变视图的代码实现。
* src/vdom：将`render`函数转为`vnode`从而`patch`为真实`dom`以及`diff`算法的代码实现。
* dist：存放着针对不同使用方式的不同的`vue`版本。

##### 5,vue 版本

`vue`使用的是`rollup`构建的，具体怎么构建的不重要，总之会构建出很多不同版本的`vue`。按照使用方式的不同，可以分为以下三类：

- UMD：通过`<script>`标签直接在浏览器中使用。
- CommonJS：使用比较旧的打包工具使用，如`webpack1`。
- ES Module：配合现代打包工具使用，如`webpack2`及以上。

而每个使用方式内又分为了完整版和运行时版本，这里主要以`ES Module`为例，由于在`vue`的内部是只认`render`函数的，我们来自己定义一个`render`函数，也就是这么个东西:

```js
new Vue({
  data: {
    msg: 'hello Vue!'
  },
  render(h) {
    return h('span', this.msg);
  }
}).$mount('#app');
```

但是因为有`vue-loader`，它会将我们在`template`内定义的内容编译为`render`函数，而这个编译就是区分完整版和运行时版本的关键所在，完整版就自带这个编译器，而运行时版本就没有，如下面这段代码如果是在运行时版本环境下就会报错了：

```js
new Vue({
  data: {
    msg: 'hello Vue!'  
  },
  template: `<div>{{msg}}</div>`
})
```

`vue-cli`默认是使用运行时版本的，更改或覆盖脚手架内的默认配置，将其更改为完整版即可通过编译：`'vue$': 'vue/dist/vue.esm.js'`，推荐还是使用运行时版本。

- 请问`runtime`和`runtime-only`这两个版本的区别？

- 主要是两点不同：

1. 最明显的就是大小的区别，带编译器会比不带的版本大`6kb`。
2. 编译的时机不同，编译器是运行时编译，性能会有一定的损耗；运行时版本是借助`loader`做的离线编译，运行性能更高。