#### Vue 初次渲染流程

模板编译：

<img src="D:\work\gitRespository\note\vue\img\模板渲染简易图.png" style="zoom:75%;" />

简易图示：细节待补充

<img src="D:\work\gitRespository\note\vue\img\vue初次渲染简易图.png" style="zoom:80%;" />



##### 一，initMixin 初始化构造函数 (初始化阶段)

配置Vue构造函数的_init方法

```JS
import { initMixin } from './utils/init.js'
function Vue(options){
    // 初始化传进来的options配置
    this._init(options)
}
initMixin(Vue);
```

```js
/* @flow */
//通过 initMixin 给构造函数添加原型方法
import config from '../config'
import { initProxy } from './proxy'
import { initState } from './state'
import { initRender } from './render'
import { initEvents } from './events'
import { mark, measure } from '../util/perf'
import { initLifecycle, callHook } from './lifecycle'
import { initProvide, initInjections } from './inject'
import { extend, mergeOptions, formatComponentName } from '../util/index'

let uid = 0

export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    // a uid
    vm._uid = uid++ ;// 唯一标识 ,每个组件都会存在的唯一ID，

    // a flag to avoid this being observed
    vm._isVue = true
    // merge options 合并配置项
    if (options && options._isComponent) {
      initInternalComponent(vm, options)
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
    /* 添加 _renderProxy 属性 */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self 暴露 _self 属性
    vm._self = vm
      
    //初始化生命周期函数，事件，render 函数，data,props
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props  在 初始化data,props 之前，初始化 injection
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')

    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
}
//以下是部分功能函数
export function initInternalComponent (vm: Component, options: InternalComponentOptions) {
  const opts = vm.$options = Object.create(vm.constructor.options)
  // doing this because it's faster than dynamic enumeration.
  const parentVnode = options._parentVnode
  opts.parent = options.parent
  opts._parentVnode = parentVnode

  const vnodeComponentOptions = parentVnode.componentOptions
  opts.propsData = vnodeComponentOptions.propsData
  opts._parentListeners = vnodeComponentOptions.listeners
  opts._renderChildren = vnodeComponentOptions.children
  opts._componentTag = vnodeComponentOptions.tag

  if (options.render) {
    opts.render = options.render
    opts.staticRenderFns = options.staticRenderFns
  }
}

export function resolveConstructorOptions (Ctor: Class<Component>) {
  let options = Ctor.options
  console.log('Ctor.options:',Ctor.options);
  if (Ctor.super) {
    const superOptions = resolveConstructorOptions(Ctor.super)
    const cachedSuperOptions = Ctor.superOptions
    if (superOptions !== cachedSuperOptions) {
      // super option changed,
      // need to resolve new options.
      Ctor.superOptions = superOptions
      // check if there are any late-modified/attached options (#4976)
      const modifiedOptions = resolveModifiedOptions(Ctor)
      // update base extend options
      if (modifiedOptions) {
        extend(Ctor.extendOptions, modifiedOptions)
      }
      options = Ctor.options = mergeOptions(superOptions, Ctor.extendOptions)
      if (options.name) {
        options.components[options.name] = Ctor
      }
    }
  }
  return options
}

function resolveModifiedOptions (Ctor: Class<Component>): ?Object {
  let modified
  const latest = Ctor.options
  const sealed = Ctor.sealedOptions
  for (const key in latest) {
    if (latest[key] !== sealed[key]) {
      if (!modified) modified = {}
      modified[key] = latest[key]
    }
  }
  return modified
}
```

###### 1，合并options配置

主线任务会合并`options`并在实例上挂载一个`$options`属性。合并什么东西了？这里是分两种情况的：

1. 初始化new Vue

   在执行`new Vue`构造函数时，参数就是一个对象，也就是用户的自定义配置；会将它和`vue`之前定义的原型方法，全局`API`属性；还有全局的`Vue.mixin`内的参数，将这些都合并成为一个新的`options`，最后赋值给一个的新的属性`$options`。

2. 子组件初始化

   如果是子组件初始化，除了合并以上那些外，还会将父组件的参数进行合并，如有父组件定义在子组件上的`event`、`props`等等。

   经过合并之后就可以通过`this.$options.data`访问到用户定义的`data`函数，`this.$options.name`访问到用户定义的组件名称

###### 2，初始化功能函数

* `initLifecycle `:确认组件的父子关系和初始化某些实例属性

  ```js
  export function initLifecycle (vm: Component) {
    const options = vm.$options
  
    // locate first non-abstract parent 找到第一个非抽象父组件  抽象组件 比如 transition
    let parent = options.parent
    //循环递归父组件，将父级赋值给实例属性vm.$parent，然后将当前实例push到找到的父级的$children实例属性内，从而建立组件的父子关系
    if (parent && !options.abstract) {
      while (parent.$options.abstract && parent.$parent) {
        parent = parent.$parent
      }
      parent.$children.push(vm)
    }
    //
    //找到父组件后赋值
    vm.$parent = parent
    vm.$root = parent ? parent.$root : vm // 让每一个子组件的$root属性都是根组件
  
    vm.$children = []
    vm.$refs = {}
  // _开头是私有实例属性是在这里定义的
    vm._watcher = null
    vm._inactive = null
    vm._directInactive = false
    vm._isMounted = false
    vm._isDestroyed = false
    vm._isBeingDestroyed = false
  }
  ```

  

* `initEvents`：主要作用是将父组件在使用`v-on`或`@`注册的自定义事件添加到子组件的事件中心中

  ```js
  export function initEvents (vm) {
    vm._events = Object.create(null)  // 事件中心
    ...
    const listeners = vm.$options._parentListeners  // 经过合并options得到的
    if (listeners) {
      updateComponentListeners(vm, listeners) 
    }
  }
  ```

  在`vue`中事件分为两种，他们的处理方式也各有不同:

  * 原生事件：编译阶段，根据事件绑定在html标签还是自定义标签判定。如果是`html`标签会在转为真实`dom`之后使用`addEventListener`注册浏览器原生事件。绑定事件是挂载`dom`的最后阶段，这里只是初始化阶段。

  * 自定义事件：在经历过合并`options`阶段后，子组件就可以从`vm.$options._parentListeners`读取到父组件传过来的自定义事件：

    ```js
    <child-components @select='handleSelect' />
    ```

    传过来的事件数据格式是`{select:function(){}}`这样的，在`initEvents`方法内定义`vm._events`用来存储传过来的事件集合。

    内部执行的方法`updateComponentListeners(vm, listeners)`主要是执行`updateListeners`方法。

    ```js
    // updateListeners
    export function updateListeners (
      on: Object,
      oldOn: Object,
      add: Function,
      remove: Function,
      createOnceHandler: Function,
      vm: Component
    ) {
      let name, def, cur, old, event
      for (name in on) {
        def = cur = on[name]
        old = oldOn[name]
        event = normalizeEvent(name)
        /* istanbul ignore if */
        if (__WEEX__ && isPlainObject(def)) {
          cur = def.handler
          event.params = def.params
        }
        if (isUndef(cur)) {
          process.env.NODE_ENV !== 'production' && warn(
            `Invalid handler for event "${event.name}": got ` + String(cur),
            vm
          )
        } else if (isUndef(old)) {
          if (isUndef(cur.fns)) {
            cur = on[name] = createFnInvoker(cur, vm)
          }
          if (isTrue(event.once)) {
            cur = on[name] = createOnceHandler(event.name, cur, event.capture)
          }
          add(event.name, cur, event.capture, event.passive, event.params)
        } else if (cur !== old) {
          old.fns = cur
          on[name] = old
        }
      }
      for (name in oldOn) {
        if (isUndef(on[name])) {
          event = normalizeEvent(name)
          remove(event.name, oldOn[name], event.capture)
        }
      }
    }
    ```

    这个方法有两个执行时机，首先是现在的**初始化阶段**，还一个就是**最后`patch`时的原生事件**也会用到。

    **它的作用是比较新旧事件的列表来确定事件的添加和移除以及事件修饰符的处理**。

    现在主要看自定义事件的添加，它的作用是借助之前定义的`$on`，`$emit`方法，完成父子组件事件的通信，首先使用`$on`往`vm.events`事件中心下创建一个自定义事件名的数组集合项，数组内的每一项都是对应事件名的回调函数，例如：

    ```js
vm._events.select = [function handleSelect(){}, ...]  // 可以有多个
    ```
    
    注册完成之后，使用`$emit`方法执行事件：

    ```js
this.$emit('select')
    ```
    
    首先会读取到事件中心内`$emit`方法第一个参数`select`的对象的数组集合，然后将数组内每个回调函数顺序执行一遍即完成了`$emit`做的事情。

    不知道大家有没有注意到`this.$emit`这个方法是在当前组件实例触发的，所以事件的原理可能跟大部分人理解的不一样，并不是父组件监听，子组件往父组件去派发事件。

    **而是子组件往自身的实例上派发事件，只是因为回调函数是在父组件的作用域下定义的，所以执行了父组件内定义的方法，就造成了父子之间事件通信的假象**。

    <img src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/3/16cf7abfde438ab4~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp" style="zoom:80%;" />

    

    知道这个原理特性后，我们可以做一些更`cool`的事情，例如：

    ```vue
<div>
      <parent-component>  // $on添加事件
        <child-component-1>
          <child-component-2>
            <child-component-3 />  // $emit触发事件
          </child-component-2>
        </child-components-1>
      </parent-component>
    </div>
    ```
    
    我们可不可以在`parent-component`内使用`$on`添加事件到当前实例的事件中心，而在`child-components-3`内找到`parent-component`的组件实例并在它的事件中心触发对应的事件实现跨组件通信了，答案是可以了！这一原理发现再开发组件库时会有一定帮助。

    **事件相关的处理方法**：

    ```js
/* @flow */
    
    import {
      tip,
      toArray,
      hyphenate,
      formatComponentName,
      invokeWithErrorHandling
    } from '../util/index'
    import { updateListeners } from '../vdom/helpers/index'
    
    export function initEvents (vm: Component) {
      //事件中心
      vm._events = Object.create(null)
      vm._hasHookEvent = false
      // init parent attached events 
      // 经过合并options得到的
      const listeners = vm.$options._parentListeners
      if (listeners) {
        updateComponentListeners(vm, listeners)
      }
    }
    
    let target: any
    
    function add (event, fn) {
      target.$on(event, fn)
    }
    
    function remove (event, fn) {
      target.$off(event, fn)
    }
    
    function createOnceHandler (event, fn) {
      const _target = target
      return function onceHandler () {
        const res = fn.apply(null, arguments)
        if (res !== null) {
          _target.$off(event, onceHandler)
        }
      }
    }
    
    export function updateComponentListeners (
      vm: Component,
      listeners: Object,
      oldListeners: ?Object
    ) {
      target = vm
      updateListeners(listeners, oldListeners || {}, add, remove, createOnceHandler, vm)
      target = undefined
    }
    
    export function eventsMixin (Vue: Class<Component>) {
      const hookRE = /^hook:/    //检测自定义事件名是否是hook:开头
      Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {
        const vm: Component = this
        if (Array.isArray(event)) {
          for (let i = 0, l = event.length; i < l; i++) {
            vm.$on(event[i], fn)
          }
        } else {
          //如果有对应事件名就push，没有创建为空数组然后push
          (vm._events[event] || (vm._events[event] = [])).push(fn)
          if (hookRE.test(event)) {
            //如果是hook:开头
          vm._hasHookEvent = true  // 标志位为true
          }
        }
        return vm
      }
    
      Vue.prototype.$once = function (event: string, fn: Function): Component {
        const vm: Component = this
        // this.$emit 执行的时候，会先从事件中心移除，然后利用 apply 绑定 this 到 vm 实例执行
        function on () {
          vm.$off(event, on)
          fn.apply(vm, arguments)
        }
        on.fn = fn
        vm.$on(event, on)
        return vm
      }
    
      Vue.prototype.$off = function (event?: string | Array<string>, fn?: Function): Component {
        const vm: Component = this
        // all 没有参数，清除事件中心所有事件
        if (!arguments.length) {
          vm._events = Object.create(null)
          return vm
        }
        // array of events 参数是数组，遍历执行回调，进行事件清除。
        if (Array.isArray(event)) {
          for (let i = 0, l = event.length; i < l; i++) {
            vm.$off(event[i], fn)
          }
          return vm
        }
        // specific event  找到指定的事件集合
        const cbs = vm._events[event]
        if (!cbs) {
          return vm
        }
        //回调函数不存在，直接将指定事件集合置为null
        if (!fn) {
          vm._events[event] = null
          return vm
        }
        // specific handler
        let cb
        let i = cbs.length
        while (i--) {
          cb = cbs[i]
          //从事件中心移除
          if (cb === fn || cb.fn === fn) {
            cbs.splice(i, 1)
            break
          }
        }
        return vm
      }
    
      Vue.prototype.$emit = function (event: string): Component {
        const vm: Component = this
    	...
        let cbs = vm._events[event]
        if (cbs) {
          cbs = cbs.length > 1 ? toArray(cbs) : cbs
          const args = toArray(arguments, 1)
          const info = `event handler for "${event}"`
          for (let i = 0, l = cbs.length; i < l; i++) {
            invokeWithErrorHandling(cbs[i], vm, args, vm, info)
          }
        }
        return vm
      }
    }
    ```
  
* `initRender`:挂载可以将`render`函数转为`vnode`的方法

  ```js
  export function initRender(vm) {
    vm._vnode = null
    ...
    vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)  //转化编译器的
    vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)  // 转化手写的
    ...
  }
  ```

  挂载`vm._c`和`vm.$createElement`两个方法，它们只是最后一个参数不同，这两个方法都可以将`render`函数转为`vnode`，从命名大家应该可以看出区别，`vm._c`转换的是通过编译器将`template`转换而来的`render`函数；而`vm.$createElement`转换的是用户自定义的`render`函数;

  ```js
  new Vue({
    data: {
      msg: 'hello Vue!'
    },
    render(h) { // 这里的 h 就是vm.$createElement
      return h('span', this.msg);  
    }
  }).$mount('#app');
  ```

  将内部定义的树形结构数据转为`Vnode`的实例;

*  `callHook(vm, 'beforeCreate')`:执行组件的`beforeCreate`钩子函数,会执行用户自定义的生命周期方法，如果有`mixin`混入的也一并执行;

  插件内部的`instanll`方法通过`Vue.use`方法安装时一般会选在`beforeCreate`这个钩子内执行，比如`vue-router`和`vuex`，此时可以访问Vue实例，做一些初始化的工作；

* `initInjections`:初始化`inject`，可以访问到对应的依赖

  * `provide`：提供一个对象或是返回一个对象的函数。
  * `inject`：是一个字符串数组或对象。

  ```js
   //1.vm.$options.inject为之前合并后得到的用户自定义的inject
   const result = resolveInject(vm.$options.inject, vm)
    if(result) { // 如果有结果
      toggleObserving(false)  // 刻意为之不被响应式 ，标识位，控制是否被响应式设置
      Object.keys(result).forEach(key => {
        ...
        //在vm下挂载key对应普通的值，这样就可以在当前实例使用this访问到inject内对应的依赖项
        defineReactive(vm, key, result[key])
      })
      toggleObserving(true)
    }
  ```

  ```js
  //2.双层循环遍历,找到父级的_provided 属性即退出循环
  export function resolveInject (inject, vm) {
    if (inject) {
      const result = Object.create(null)
      const keys = Object.keys(inject)  //省略Symbol情况，inject会在合并options时处理成对象
     /* 定义时：{ inject: ['app'] }
      格式化后： {  inject: {
                      app: {
                        from: 'app'
                      }
                    }
  				}
  */
      for (let i = 0; i < keys.length; i++) {
        const key = keys[i]
        const provideKey = inject[key].from
        let source = vm
        // 父组件在子组件之前初始化，所以 _provided 正常是存在的
        while (source) {
          if (source._provided && hasOwn(source._provided, provideKey)) { //hasOwn为是否有
            result[key] = source._provided[provideKey]
            break
          }
          source = source.$parent
        }
      ... //vue@2.5后新增设置inject默认参数相关逻辑
      }
      return result
    }
  }
  ```

* `initState`:初始化会被使用到的状态，状态包括`props`，`methods`，`data`，`computed`，`watch`五个选项。

  ```js
  export function initState (vm: Component) {
    vm._watchers = []
    const opts = vm.$options
    if (opts.props) initProps(vm, opts.props)
    if (opts.methods) initMethods(vm, opts.methods)
    if (opts.data) {
      initData(vm)
    } else {
      observe(vm._data = {}, true /* asRootData */)
    }
    if (opts.computed) initComputed(vm, opts.computed)
    if (opts.watch && opts.watch !== nativeWatch) {
      initWatch(vm, opts.watch)
    }
  }
  ```

  * `initProps`:检测子组件接受的值是否符合规则，以及通过`Object.defineProperty()`让对应的值可以用`this`直接访问,

  * `initMethods`: 将`methods`内的方法挂载到`this`下,验证一下方法是否符合规范，是否命名冲突等。

  * `initData`:初始化`data`，挂载到`this`下。**有个重要的点，之所以`data`内的数据是响应式的，是在这里初始化的，绑定细节另外说明**

    ```js
    function initData (vm: Component) {
      let data = vm.$options.data
      // 通过data.call(vm, vm)得到返回的对象,获取 data 对象
      data = vm._data = typeof data === 'function'
        ? getData(data, vm)  
        : data || {}
      if (!isPlainObject(data)) {
        data = {}
        process.env.NODE_ENV !== 'production' && warn(
          'data functions should return an object:\n' +
          'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
          vm
        )
      } 
      // proxy data on instance  验证data 的数据不要和 props ，methods，重名
      const keys = Object.keys(data)
      const props = vm.$options.props
      const methods = vm.$options.methods
      let i = keys.length
      while (i--) {
        const key = keys[i]
        if (process.env.NODE_ENV !== 'production') {
          if (methods && hasOwn(methods, key)) {
            warn(
              `Method "${key}" has already been defined as a data property.`,
              vm
            )
          }
        }
        if (props && hasOwn(props, key)) {
          process.env.NODE_ENV !== 'production' && warn(
            `The data property "${key}" is already declared as a prop. ` +
            `Use prop default value instead.`,
            vm
          )
          // 检查 key 是否以 $ or _ 开头 , 转换可以通过 this 直接访问
        } else if (!isReserved(key)) {
          proxy(vm, `_data`, key)
        }
      }
      // observe data 实现 data 响应式绑定
      observe(data, true /* asRootData */)
    }
    ```

  * `initComputed`: 见computed 和 watch 相关

  * `initWatch`: 见computed 和 watch 相关

* `initProvide`: 初始化`provide`为子组件提供依赖。`provide`选项应该是一个对象或是函数

  ```js
  export function initProvide (vm: Component) {
    const provide = vm.$options.provide
    if (provide) {
      vm._provided = typeof provide === 'function'
        ? provide.call(vm)
        : provide
    }
  ```

* `callHook(vm, 'created')`:执行用户定义的`created`钩子函数，有`mixin`混入的也一并执行。

`methods`定义不可以用箭头函数，原因是在初始化 `methods`时，会通过`methods[key].bind(vm)`将方法绑定在vue实例上，执行的时候是`this.xxx`,箭头函数的this是在父级上下文，在vue会导致undefined。



##### 二，模板编译(编译挂载阶段)

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
>   - 无template：拿`$mount()`根节点的outerHTML去解析成render函数的所需的格式，并使用调用render函数渲染
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

```js
new Vue ==> vm._init() ==> vm.$mount(el) ==> vm._render()  ==> vm.update(vnode) 
```

**具体解析细节待补充**

###### 1，vue的V-dom是如何处理的？



###### 2，updata首次的流程？





