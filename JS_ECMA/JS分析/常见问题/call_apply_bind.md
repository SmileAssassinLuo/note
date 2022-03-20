#### call 和apply 

`call()` 和 `apply()` 都是函数原型上的用来改变this指向的方法，区别就在于传参的方式不一样；类似功能的还有`bind()`。

在 参数数量超过3个时，`call()` 的性能稍微优于 `apply()` .

手动实现：原理思路如下

1. **将函数设为对象的属性**
2. **执行该函数**
3. **删除该函数**

`call()实现：`

```javascript
       //想要bar 执行的时候，将this指向egg，以取得egg的name属性；
       var egg  = {
           name :"name_egg "
       }
       function bar(a,b,c,d){
           return {
               name:this.name,
               a:a,b:b,c:c,d:d
           }
       }
       //call 挂载在 Function.prototype 对象上的一个函数方法，
       /* 将要执行的bar函数置为egg对象的一个属性,
          就可以通过egg.bar() 获取到egg的name属性.
       */
       Function.prototype.newCall = function(ctx){
           var ctx = ctx || window; // 指向null 时设为window
           //console.log(this);//this是 由于bar函数调用，指向bar函数。隐式绑定
           //此时就可以将this，定义为形参的方法
           var f = Symbol('f');//唯一
           ctx.f = this;
           var args = [];
           //将arguments的元素改为动态的"argument[i]"存入数组，
           //这样args的元素就是["arguments[1]" , "arguments[2]", "arguments[3]" ,"arguments[4]"]
           //eval()可以将字符串的js执行，将 'ctx.f(' + args +')' 拼接成字符串，数组拼接字符串会调用toString方法。
           // 'arguments[1],arguments[2],arguments[3],arguments[4]'
           //eval()的参数即为 'ctx.f(arguments[1],arguments[2],arguments[3],arguments[4])';
           for(var i = 1; i < arguments.length ;i++){
               args.push("arguments["+i+"]");
           }
           //执行函数
          var result =  eval('ctx.f(' + args +')');
           //删除函数，恢复为初始定义的对象，不可改写对象。
           delete ctx.f;
           return result;
           
       }
       var res = bar.newCall(egg,"param1","param2","param3","param4");
```

`apply()实现：`

```javascript
    //想要bar 执行的时候，将this指向egg，以取得egg的name属性；
    var egg  = {
        name :"name_egg "
    }
    function bar(a,b,c,d){
        return {
            name:this.name,
            a:a,b:b,c:c,d:d
        }
    }
    Function.prototype.newApply = function(ctx,arr){
        var ctx = ctx || window;
        var result;
        //console.log(this);//this是 由于bar函数调用，指向bar函数。隐式绑定
        //此时就可以将this，定义为形参的方法
        var f = Symbol('f');
        ctx.f = this;
         //判断第二个参数是否存在。
        if(!arr){
             result = ctx.f();
        }else{
         //将数组元素通过 "arr[i]",存入新数组，并进行字符串拼接；
         var args = [];
         for(var i = 0; i < arr.length ;i++){
             args.push("arr["+i+"]");
         }
         //执行函数
          result =  eval('ctx.f(' + args +')');
        }
        //删除函数，恢复为初始定义的对象，不可改写对象。
        delete ctx.f;
        return result;
        
    }
    var res = bar.newApply(egg,["param1","param2","param3","param4"]);
```

##### bind

`bind()` 方法创建一个新的函数，在 `bind()` 被调用时，这个新函数的 `this` 被指定为 `bind()` 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。

```javascript
var egg = {
    name:"name_egg"
}
var name = 'window_name';
var bar = function(a,b,c){
    console.log(this.name);
    console.log(a,b,c);
}
bar.prototype.colotion = "hello";
Function.prototype.newBind = function(obj){
    if(typeof this !== 'function' ){
        throw new TypeError ("非函数调用者");
    }
    var that = this;
    var arr1 = Array.prototype.slice.call(arguments,1);
    var o = function(){};
    //通过返回具名函数才能进行原型绑定；
    var newFun = function(){
        //因为使用了new 构造函数，就会发生this绑定，此时的this 指向了bindBar函数的实例，而不是bar函数
        var arr2 = Array.prototype.slice.call(arguments);
        var arrSum = arr1.concat(arr2);
        //如果发生了new ，bar就通过apply改为指向this;
        //没有的话，就正常绑定obj
        if(this instanceof o){
            that.apply(this,arrSum);
        }else{
            that.apply(obj,arrSum);
        }

    }
    //newFun.prototype = that.prototype; //一般不建议直接通过原型连接
    //通过一个空函数的进行桥接原型。
    o.prototype = that.prototype;
    newFun.prototype = new o();
    return newFun;
}

//var bindBar2 =  bar.newBind(egg,"param1","param2")("param3");
var bindBar =  bar.newBind(egg,"param1","param2")
var bindBarSon = new bindBar("param3");
console.log(bindBarSon);
//    var barBind2 = bar.bind(egg,"1","2");
//    var barBind2Son = new barBind2("3");
//    console.log(barBind2Son);
```





#### 防抖 debounce 和节流 throttle 

**防抖**：原理就是，你尽管触发事件，但是我一定在事件触发 n 秒后才执行，如果你在一个事件触发的 n 秒内又触发了这个事件，那我就以新的事件的时间为准，n 秒后才执行，总之，就是要等你触发完事件 n 秒内不再触发事件，我才执行。

常见的使用场景：

- 搜索框搜索输入。只需要用户最后一次输入完再发送请求
- 手机号、邮箱格式的输入验证检测
- 窗口大小的 resize 。只需窗口调整完成后，计算窗口的大小，防止重复渲染。

```javascript
//html :<input id = 'btn' type = "button" value = "剁手按钮" />

//防抖思路
 //1.监听执行函数
 //2.清除延时器
 //3.开启延时器
 var btn = document.querySelector("#btn");
 function payMoney(e,...arg){
     console.log("已剁");
     console.log(e)
     console.log(arg); //(3) ['canshu1', 'canshu2', PointerEvent]
 }
 //func(); 直接执行函数，会导致监听动作未点击时，函数就会执行一次；
 //返回一个函数，将执行函数包裹起来；只有在点击时，才执行；
 //防抖，就设置延时时间，增加settimeout,和延时时间参数
 // 清除延时，使用clearTimeout,
 //由于setTimeout 导致 this 指向发生了变化，所以在回调函数执行前，应该保存 this , 并通过 call 等 绑定 this 指向
 // 参数，传入的参数是给 payMoney 使用,如 payMoney 函数 需要传参，可通过 bind
 function debounce(func,delay){
     let timer = null ;
     return function(){
        // JavaScript 在事件处理函数中会提供事件对象 event
         let args = Array.from(arguments); 
         let context = this ;
         clearTimeout(timer);
         timer =  setTimeout(function(){
             func.apply(context,args);
             console.log(context);                       //<input id="btn" type="button" value="剁手按钮">
         },delay)
     }
 }
btn.addEventListener("click",debounce(payMoney , 1000));
//btn.addEventListener("click",debounce(payMoney , 1000).bind(btn,"canshu1"));  //this 指向 window,需修正 ;
// btn.addEventListener("click",payMoney); //this 指向 input
```

##### 函数节流（throttle）的应用场景

在一定的间隔时间内执行一次，场景有：

- 滚动加载，加载更多或滚动到底部监听
- 谷歌搜索框，搜索联想功能
- 高频点击提交，表单重复提交
- 省市信息对应字母快速选择

```javascript
function coloring(){
    let r = Math.floor(Math.random()*255);
    let g = Math.floor(Math.random()*255);
    let b = Math.floor(Math.random()*255);
    document.body.style.background = `rgb(${r},${g},${b})`;
}
function throttle1(func ,delay){
	let timer;
    return function () {
        if(timer){
             return;
        }
        let context = this;
        let args = arguments;
        timer = setTimeout(function(){
            func.apply(context,args);
            timer = null ;
        },delay)
	}
}
function throttle(func ,delay){
    let pre = 0;
    return function () {
        let context = this;
        let args = arguments;
        let now = new Date();
        if(now - pre > delay){
            func.apply(context,args);
            pre = now ;
        }
    }
}

window.addEventListener("resize",throttle(coloring,1000));
```



