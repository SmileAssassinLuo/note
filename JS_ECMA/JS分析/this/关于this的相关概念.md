### this

**this:函数调用时的引用,决定了函数执行时刻的this值**

* 函数的this值由"调用它所使用的引用"决定,奥秘在于:我们获取函数的表达式,它实际返回的并非函数本身,而是一个Reference型,Reference类型由两部分组成,一个对象和一个属性值,不难理解o.showThis 产生的Reference 类型,即由对象 o 和属性 showThis 组成;

* 当做一些算数运算或者其他运算时,Reference 类型会被解引用,即获取真正的值(被引用的内容),来参与运算,而类似函数调用,delete等操作,都需要用到Reference 类型中的对象; 

```javascript
function showThis (){
	console.log(this);
}

var o = {
	showThis:showThis
};

showThis();//Window
o.showThis();//function showThis
```

修改一下案例,改为箭头函数后,不论用什么引用来调用它,都不影响 this 的值;

```javascript
const  showThis =  () => {
	console.log(this);
}

var o = {
	showThis:showThis
};

showThis();//Window
o.showThis();//Window
```

再修改一下，又会有不同的行为；

```javascript
class C {
    showThis(){
    	console.log(this);
	}
}


var o = new C();
var showThis = o.showThis;

showThis();  //undefined
o.showThis(); //c
```



### this 关键字的机制

在js标准中，为函数规定了保存定义时上下文的私有属性[[Environment]];

函数执行时，会创建一条当前上下文执行环境记录，记录的外层词法环境（`outer lexical environment`）会被设置成函数的`[[Environment]]`;

此动作即为切换上下文；如下代码

```javascript
var b = 2;
function foo() {
    console.log(b);
    console.log(a);
}

(() => {
    var a = 1;
    foo();
})();

//2
//ReferenceError: a is not defined
```



js通过一个栈来管理执行上下文，栈的每一项都包含一个链表，当函数调用时，会入栈一个新的执行上下文，函数调用结束，执行上下文被出栈。

JS标准定义了[[thisMode]]私有属性，有三个取值：

* lexical：表示从上下文中找this，对应了箭头函数
* global：表示当this为undefined 时，取全局对象，对应了普通函数
* strict： 当严格模式时使用，this 严格按照调用时传入的值，可能为null或undefined；（class 类设计默认按strict 模式执行）

函数创建新的执行上下文中的词法环境记录时，会根据[[thisMode]]来标记新纪录的[[ThisBindingStatus]]私有属性。代码执行遇到 this 时，会逐层检查当前词法环境记录中的[[ThisBindingStatus]]，当找到有 this 的环境记录时获取 this 的值。