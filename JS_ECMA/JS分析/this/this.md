## this

	 this 是在运行时进行绑定的，并不是在编写时绑定，它的上下文取决于函数调用时的各种条件。


 		+ this 的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。
当一个函数被调用时，会创建一个活动记录（有时候也称为执行上下文）。这个记录会包
含函数在哪里被调用（调用栈）、函数的调用方法、传入的参数等信息。 this 就是记录的
其中一个属性，会在函数执行的过程中用到。

		+  this 实际上是在函数被调用时发生的绑定，它指向什么完全取决于函数在哪里被调用。

## 剖析this

#### 一.调用位置

​	调用位置就是函数在代码中被调用的位置（而不是声明的位置）；

​	寻找调用位置最重要的是分析调用栈（就是为了到达当前执行位置所调用的所有函数）；

```javascript
function baz() {
    // 当前调用栈是：baz
    // 因此，当前调用位置是全局作用域
    console.log( "baz" );
    bar(); // <-- bar 的调用位置
}
function bar() {
    // 当前调用栈是 baz -> bar
    // 因此，当前调用位置在 baz 中
    console.log( "bar" );
    foo(); // <-- foo 的调用位置
}
function foo() {
    // 当前调用栈是 baz -> bar -> foo
    // 因此，当前调用位置在 bar 中
    console.log( "foo" );
}
baz(); // <-- baz 的调用位置
```

​	调用栈想象成一个函数调用链，就像我们在前面代码段的注释中所写的一样。但是这种方法非常麻烦并且容易出错。另一个查看调用栈的方法是使用浏览器的调试工具。绝大多数现代桌面浏览器都内置了开发者工具，其中包含 JavaScript 调试器。就本例来说，你可以在工具中给 foo() 函数的第一行代码设置一个断点，或者直接在第一行代码之前插入一条 debugger;语句。运行代码时，调试器会在那个位置暂停，同时会展示当前位置的函数调用列表，这就是你的调用栈。因此，如果你想要分析 this 的绑定，使用开发者工具得到调用栈，然后找到栈中第二个元素，这就是真正的调用位置。

#### 二.绑定规则

+ 默认绑定： this 指向全局对象。如果使用严格模式（ strict mode ），那么全局对象将无法使用默认绑定，因此 this 会绑定
  到 undefined .

  如下foo() 是直接使用不带任何修饰的函数引用进行调用的，因此只能使用默认绑定，无法应用其他规则。

  ```javascript
  function foo() {
  	console.log( this.a );
  }
  var a = 2;
  foo(); // 2
  ```

+ 隐式绑定：需要考虑的规则是调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含;

  ```javascript
  function foo() {
  	console.log( this.a );
  }
  var obj = {
          a: 2,
          foo: foo
      };
  obj.foo(); // 2
  ```

  首先需要注意的是 foo() 的声明方式，及其之后是如何被当作引用属性添加到 obj 中的。但是无论是直接在 obj 中定义还是先定义再添加为引用属性，这个函数严格来说都不属于obj 对象。
  然而，调用位置会使用 obj 上下文来引用函数，因此你可以说函数被调用时 obj 对象“拥有”或者“包含”它。

  当函数引用有上下文对象时，隐式绑定规则会把函数调用中的 this 绑定到这个上下文对象.



​	`隐式丢失`

​	虽然 bar 是 obj.foo 的一个引用，但是实际上，它引用的是 foo 函数本身，因此此时的bar() 其实是一个不带任何修饰的函数调用，因此应用了默认绑定.

```javascript
function foo() {
	console.log( this.a );
}
var obj = {
		a: 2,
		foo: foo
	};
var bar = obj.foo; // 函数别名！
var a = "oops, global"; // a 是全局对象的属性
bar(); // "oops, global"

```

​	一种更微妙、更常见并且更出乎意料的情况发生在传入回调函数时:

```javascript
function foo() {
	console.log( this.a );
}
function doFoo(fn) {
    // fn 其实引用的是 foo
    fn(); // <-- 调用位置！
}
var obj = {
    a: 2,
    foo: foo
    };
var a = "oops, global"; // a 是全局对象的属性
doFoo( obj.foo ); // "oops, global"
```

+ 显式绑定：使用函数的 `call(..)` 和`apply(..)`方法。

  ```javascript
  function foo() {
  	console.log( this.a );
  }
  var obj = {
      a:2
  };
  foo.call( obj ); // 2
  ```

  ​	显式绑定仍然无法解决我们之前提出的丢失绑定问题.

  ​	解决丢失绑定问题：

     **一.硬绑定**

  ```javascript
  function foo() {
  	console.log( this.a );
  }
  var obj = {
      	a:2
      };
  var bar = function() {
  	foo.call( obj );
  };
  bar(); // 2
  setTimeout( bar, 100 ); // 2
  // 硬绑定的 bar 不可能再修改它的 this
  bar.call( window ); // 2
  ```

  创建了函数 `bar()` ，并在它的内部手动调用 `foo.call(obj) `，因此强制把 `foo `的 `this` 绑定到了 `obj` 。无论之后如何调用函数 `bar` ，它总会手动在 `obj` 上调用 `foo` 。这种绑定是一种显式的强制绑定，因此我们称之为硬绑定.
  
```javascript
  //硬绑定的典型应用场景就是创建一个包裹函数，传入所有的参数并返回接收到的所有值：
  function foo(something) {
  	console.log( this.a, something );
  	return this.a + something;
  }
  var obj = {
  		a:2
  	};
  var bar = function() {
  		return foo.apply( obj, arguments );
  };
  var b = bar( 3 ); // 2 3
  console.log( b ); // 5
  //另一种使用方法是创建一个 i 可以重复使用的辅助函数：
  function foo(something) {
  	console.log( this.a, something );
  	return this.a + something;
  }
  // 简单的辅助绑定函数
  function bind(fn, obj) {
  	return function() {
  		return fn.apply( obj, arguments );
  	};
  }
  var obj = {
  		a:2
  	};
  var bar = bind( foo, obj );
  var b = bar( 3 ); // 2 3
  console.log( b ); // 5
  
  //由于硬绑定是一种非常常用的模式，所以在 ES5 中提供了内置的方法 Function.prototype.bind ，它的用法如下：
  function foo(something) {
  	console.log( this.a, something );
  	return this.a + something;
  }
  var obj = {
      	a:2
      };
  var bar = foo.bind( obj );
  var b = bar( 3 ); // 2 3
  console.log( b ); // 5
  //bind(..) 会返回一个硬编码的新函数，它会把参数设置为 this 的上下文并调用原始函数
  ```
  
**二. API调用的“上下文”**
  
第三方库的许多函数，以及` JavaScript` 语言和宿主环境中许多新的内置函数，都提供了一个可选的参数，通常被称为“上下文”（`context`），其作用和 `bind(..)` 一样，确保你的回调函数使用指定的` this` 。
  
```javascript
  function foo(el) {
  	console.log( el, this.id );
  }
  var obj = {
  	id: "awesome"
  };
  // 调用 foo(..) 时把 this 绑定到 obj
  [1, 2, 3].forEach( foo, obj );
  // 1 awesome 2 awesome 3 awesome
  ```
  
+ `new`绑定：使用 `new` 来调用 `foo(..)`时，会构造一个新对象并把它绑定到 `foo(..)` 调用中的` this`上。 `new `是最后一种可以影响函数调用时 `this` 绑定行为的方法,称之为 new 绑定.

  ```javascript
  function foo(a) {
  	this.a = a;
  }
  var bar = new foo(2);
  console.log( bar.a ); // 2
  ```

  

#### **三.优先级**

函数调用中 `this` 绑定的四条规则，当调用位置可以应用多条规则时，必须给这些规则设定优先级。

1. 函数是否在 `new` 中调用（ `new `绑定）？如果是的话` this` 绑定的是新创建的对象。
`var bar = new foo()`
2. 函数是否通过 `call` 、 `apply` （显式绑定）或者硬绑定调用？如果是的话， `this` 绑定的是指定的对象。
`var bar = foo.call(obj2)`
3. 函数是否在某个上下文对象中调用（隐式绑定）？如果是的话， `this` 绑定的是那个上下文对象。
`var bar = obj1.foo()`
4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到` undefined` ，否则绑定到全局对象。
`var bar = foo()`



#### **四.绑定例外**

**1.被忽略的`this`**

如果将`null` 或者 `undefined` 作为 `this` 的绑定对象传入 `call `、 `apply `或者 `bind` ，这些值在调用时会被忽略，实际应用的是默认绑定规则：

```javascript
function foo(){
	console.log(this.a)
}
var a = 2;
foo.call( null ); // 2
```

那么什么情况下你会传入 null 呢？
一种非常常见的做法是使用 `apply(..) `来“展开”一个数组，并当作参数传入一个函数。
类似地， `bind(..) `可以对参数进行柯里化（预先设置一些参数），这种方法有时非常有用：

**2.更安全的 this**

一种“更安全”的做法是传入一个特殊的对象，把 this 绑定到这个对象不会对你的程序产生任何副作用.

 Object.create(null) 和 {} 很像，但是并不会创建 Object.prototype 这个委托，所以它比 {} “更空”.

```javascript
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}
// 我们的 DMZ 空对象
var ø = Object.create( null );
// 把数组展开成参数
foo.apply( ø, [2, 3] ); // a:2, b:3
// 使用 bind(..) 进行柯里化
var bar = foo.bind( ø, 2 );
bar(3); // a:2, b:3
```



**3.间接引用**

```javascript
//间接引用最容易在赋值时发生：
function foo() {
	console.log( this.a );
}

var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };
o.foo(); // 3
(p.foo = o.foo)(); // 2
```

赋值表达式 `p.foo = o.foo` 的返回值是目标函数的引用，因此调用位置是 `foo() `而不是`p.foo()` 或者 `o.foo()` 。根据我们之前说过的，这里会应用默认绑定。
**注意：**对于默认绑定来说，决定 this 绑定对象的并不是调用位置是否处于严格模式，而是函数体是否处于严格模式。如果函数体处于严格模式， this 会被绑定到 undefined ，否则this 会被绑定到全局对象。



**4.软绑定**









**小结**

如果要判断一个运行中函数的 this 绑定，就需要找到这个函数的直接调用位置。找到之后
就可以顺序应用下面这四条规则来判断 this 的绑定对象。
1. 由 new 调用？绑定到新创建的对象。

2. 由 call 或者 apply （或者 bind ）调用？绑定到指定的对象。

3. 由上下文对象调用？绑定到那个上下文对象。

4. 默认：在严格模式下绑定到 undefined ，否则绑定到全局对象。

一定要注意，有些调用可能在无意中使用默认绑定规则。如果想“更安全”地忽略 this 绑定，你可以使用一个 DMZ 对象，比如 ø = Object.create(null) ，以保护全局对象。ES6 中的箭头函数并不会使用四条标准的绑定规则，而是根据当前的词法作用域来决定this ，具体来说，箭头函数会继承外层函数调用的 this 绑定（无论 this 绑定到什么）。这其实和 ES6 之前代码中的 self = this 机制一样。