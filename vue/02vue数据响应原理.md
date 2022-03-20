#### vue 数据响应原理

##### 一，vue@2.x 

**简单来说就是使用`Object.defineProperty`这个`API`为数据设置`get`和`set`。当读取到某个属性时，触发`get`将读取它的组件对应的`render watcher`收集起来；当重置赋值时，触发`set`通知组件重新渲染页面。如果数据的类型是数组的话，还做了单独的处理，对可以改变数组自身的方法进行重写，因为这些方法不是通过重新赋值改变的数组，不会触发`set`，所以要单独处理。响应系统也有自身的不足，所以官方给出了`$set`和`$delete`来弥补。**



在 vue @2.x的版本中，通过 `Object.defineProperty()`对数据进行劫持，实现对数据源的监控。

从配置项 options 中取得 data 后，给实例对象添加`_data`属性，并简化访问data的方式。

```js
import  { observable } from './observer'
export const initData = (vm) =>{
    const data = vm.$options.data;
    data = vm._data = typeof data == 'function' ? data.call(vm):data||{};
    //通过代理 vm._data.a 可以通过 vm.a 访问 
    for(key in data ){
        proxyData(vm,_data,key)
    }
    observable(data)  
}
const proxyData = (obj,sourceData,key) =>{
    Object.defineProperty(obj,key,{
        get(){
            return obj[sourceData][key]
        },
        set(newVal){
            obj[sourceData][key] = newVal ;
        }
    })
}
```

重点是 **observable**函数的执行，会将<strong style='color:red'>`data`</strong>对象的成员，进行深层的响应式绑定；

vue 双向绑定的原理解析,**发布订阅模式（举例  商家 - 快递公司 - 消费者 ）**

* 1.使用 Object.defineProperty 劫持数据变化，**发布者**角色；
* 2.watcher 数据变化更新视图，**订阅者**角色；
* 3.Dep 一个依赖收集容器，也就是消息**订阅器Dep**，用来容纳所有的“订阅者”；

订阅器Dep主要负责收集订阅者，然后当数据变化的时候后执行对应订阅者的更新函数

observe.js

```js
import Dep from './dep'
import { arrayMethods } from './array'
class Observer{
    constructor(val){
        this.val = val;
        this.dep = new Dep();
        Object.defineProperty(val,'__ob__',this);
        if(Array.isArray(val)){
            //兼容，比如IE10下返回 false 
            if('__proto__' in {}){
                //重写 Array.prototype 上的七个方法
                protoAugment(val, arrayMethods);
            }
            this.observeArray(val)
        }else{
            this.walk(val)
        }
    }
    observeArray(items){
        for(let i=0;i<items.length;i++){
            observe(items[i])
        }
    }
    walk(obj){
        const keys = Object.keys(obj);
        for(let i = 0;i< keys.length;i++){
            defineReactive(obj,keys[i],obj[keys[i]])
        }
    }
}
const protoAugment = (target,src) =>{
    target.__proto__ = src;
}
const defineReactive = (observeObj,key,val) => {
    observe(val);
    const dep = new Dep()
    Object.defineProperty(observeObj,key,{
        get(){
              // 调用依赖收集器中的addSub，用于收集当前属性与Watcher中的依赖关系
            dep.depend()
            return val
        },
        set(newVal){
            if(newVal === val) retrun 
            val = newVal
             // 当值发生变更时，通知依赖收集器，更新每个需要更新的Watcher，
             // 这里每个需要更新通过什么断定？dep.subs
            dep.notify();
        }
    })
}
export function observe(value) {
    // 如果传进来的是对象或者数组，则进行响应式处理
    if (Object.prototype.toString.call(value) === '[object Object]' || Array.isArray(value)) {
        return new Observer(value)
    }
}
```

array.js：数组方法重写

```js
const arrayProto = Array.prototype;
const arrayMethods = Object.create(arrayProto);
const methodsToPatch = [
    'push',
    'pop',
    'shift',
    'unshift',
    'splice',
    'sort',
    'reverse'
  ]
methodsToPatch.forEach(function(method) {
    // cache original method
    const original = arrayProto[method]  // 方法名
    def(arrayMethods, method, function mutator (...args) {
      const result = original.apply(this, args) 
      const ob = this.__ob__
      let inserted
      switch (method) {
        case 'push':
        case 'unshift':
          inserted = args
          break
        case 'splice':
          inserted = args.slice(2)
          break
      }
      if (inserted) ob.observeArray(inserted)
      // notify change
      ob.dep.notify()
      return result
    })
})

export default  {
    arrayMethods
}
```

dep.js

添加订阅者的操作设计在 getter 里面，这是为了 watcher 初始化时触发，因此需要判断是否要添加订阅者。在setter函数里面，如果数据变化，就会去通知所有订阅者，订阅者们就会去执行对应的更新的函数。

```js
class Dep{
    constructor(){
        this.subs = [];
    }
    addSubs(sub){
        this.subs.push(sub)
    }
    depend(){
         //判断是否增加订阅者，
        if(Dep.target){
            console.log('添加了watcher');
            // 搜集依赖，最终会调用上面的 addSub 方法,将当前 watcher 添加到 subs
            this.addSubs(Dep.target)
        }
    }
    notify(){
        const watchSubs = this.subs.slice();
        console.log('通知 watcher 更新视图');
        watchSubs.forEach(sub => {
            sub.update();
        })
    }
}
export default Dep
```

watch.js

订阅者 watcher 

订阅者Watcher在初始化的时候需要将自己添加进订阅器Dep中，调用属性，在 get 时会添加。

细节：只有初始化时才需要添加订阅者，所以需要做一个判断操作，在订阅器上做一下手脚： 在Dep.target 上缓存下订阅者，添加成功后再将其去掉就可以了。

具体实现，过程分析：

  订阅者Watcher 是一个 类，在它的构造函数中，定义了一些属性：

​    vm:一个Vue的实例对象；

​    exp:是node节点的v-model或v-on：click等指令的属性值。如v-model="name"，exp就是name;

​    cb:是Watcher绑定的更新函数;

当我们去实例化一个渲染 watcher 的时候，首先进入 watcher 的构造函数逻辑，就会执行它的 this.get() 方法，进入 get 函数，首先会执行：`Dep.target = this` ; 

 实际上就是把 Dep.target 赋值为当前的渲染 watcher,执行this.addSub(Dep.target),即把当前的 watcher 订阅到这个数据持有的 dep 的 subs 中.

```js
import Dep from './dep'
class Watch{
    constructor(vm,exp,cb){
        this.vm = vm;
        this.exp = exp;
        this.cb = cb;
        this.value = this.get();
    }
    get(){
        Dep.target = this;
        let value = this.vm.data[this.exp];
        Dep.target = null;
        return value

    }
    update(){
        let value = this.vm.data[this.exp];
        let oldValue  = this.value;
        if(value !== oldValue){
            this.value = value;
            this.cb.call(this.vm,value,oldValue)
        } 
    }
}
export default Watch
```





##### 二，vue@3.x

###### 1,proxy 和 reflection

###### 2,vue 数据绑定原理

 





