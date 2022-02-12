#### VUE中的实例方法

##### 一，数据相关

与数据相关的实例方法有3个，分别是vm.$set`、`vm.$delete`和`vm.$watch，是在`stateMixin`函数中挂载到`Vue`原型上的；

```js
import {set,del} from '../observer/index'

export function stateMixin (Vue) {
    Vue.prototype.$set = set
    Vue.prototype.$delete = del
    Vue.prototype.$watch = function (expOrFn,cb,options) {}
}
```

###### (1),vm.$watch

* 用法:  `vm.$watch( expOrFn, callback, [options] )`

  观察 `Vue` 实例变化的一个表达式或计算属性函数。回调函数得到的参数为新值和旧值。表达式只接受监督的键路径。对于更复杂的表达式，用一个函数取代。

  **注意**：在变异 (不是替换) 对象或数组时，旧值将与新值相同，因为它们的引用指向同一个对象/数组。`Vue` 不会保留变异之前值的副本。

* 参数:

  * `{string | Function} expOrFn`
  * `{Function | Object} callback`
  * `{Object} [options]`
    * `{boolean} deep`
    * `{boolean} immediate`

* 返回值:`{Function} unwatch`

* 示例

  ```js
  // 键路径
  vm.$watch('a.b.c', function (newVal, oldVal) {
    // 做点什么
  })
  
  // 函数
  vm.$watch(
    function () {
      // 表达式 `this.a + this.b` 每次得出一个不同的结果时
      // 处理函数都会被调用。
      // 这就像监听一个未被定义的计算属性
      return this.a + this.b
    },
    function (newVal, oldVal) {
      // 做点什么
    }
  )
  ```

  `vm.$watch` 返回一个取消观察函数，用来停止触发回调：

  ```js
  var unwatch = vm.$watch('a', cb)
  // 之后取消观察
  unwatch()
  ```

  

* 原理



###### (2),vm.$set

* 用法 

  - 向响应式对象中添加一个属性，并确保这个新属性同样是响应式的，且触发视图更新。它必须用于向响应式对象上添加新属性，因为 `Vue` 无法探测普通的新增属性 (比如 `this.myObject.newProperty = 'hi'`)
  - **注意**：对象不能是 `Vue` 实例，或者 `Vue` 实例的根数据对象。

* 参数

  - `{Object | Array} target`
  - `{string | number} propertyName/index`
  - `{any} value`

* 返回值 :设置的值

* 示例

* 原理

  数据变化侦测的时，对于`object`型数据，当我们向`object`数据里添加一对新的`key/value`或删除一对已有的`key/value`时，`Vue`是无法观测到的；而对于`Array`型数据，当我们通过数组下标修改数组中的数据时，`Vue`也是是无法观测到的；

  正是因为存在这个问题，所以`Vue`设计了`set`和`delete`这两个方法来解决这一问题。

  

###### (3),vm.$delete

* 用法 
* 参数
* 返回值
* 示例
* 原理





