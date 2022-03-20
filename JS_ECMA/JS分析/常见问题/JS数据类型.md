##### JS 数据类型

* 为什么有的编程规范要求用 void 0 代替 undefined？
* 字符串有最大长度吗？0.1 + 0.2 不是等于 0.3 么？为什么 JavaScript 里不是这样的？
* ES6 新加入的 Symbol 是个什么东西？
* 为什么给对象添加的方法能用在基本类型上？. 运算符提供了装箱操作，它会根据基础类型构造一个临时对象，使得我们能在基础类型上调用对应对象的方法

JavaScript 模块会从**运行时**、**文法**和**执行过程**三个角度去剖析 JS 的知识体系，从运行时的角度去看 JavaScript 的类型系统。

> 运行时类型是代码实际执行过程中我们用到的类型。所有的类型数据都会属于 7 个类型之一。从变量、参数、返回值到表达式中间结果，任何 JavaScript 代码运行过程中产生的数据，都具有运行时类型。

**JavaScript** 语言规定了 **7 **种语言类型。语言类型广泛用于**变量**、**函数参数**、**表达式**、**函数返回值**等场合。

1. **Undefined**；
2. **Null**；
3. **Boolean**；
4. **String**；
5. **Number**；
6. **Symbol**；
7. **Object**；

###### Undefined Null

任何变量在赋值前是 Undefined 类型、值为 undefined，一般我们可以用全局变量 undefined（就是名为 undefined 的这个变量）来表达这个值，或者 **void 运算**来把任意一个表达式变成 undefined 值。

null 就是有个值是 null。

```js
var und1 = undefined;
var und2;
var und3 = void 0;
console.log(und1,und2,und3);//undefined undefined undefined
var null1 = null;
console.log(null1);//null
```

###### Boolean

true 和 false.

###### String 

String 用于表示文本数据。String 有最大长度是` 2^53 - 1`,String是指字符串的 UTF16 编码，我们字符串的操作 `charAt`、`charCodeAt`、`length` 等方法针对的都是 UTF16 编码。所以，字符串的最大长度，实际上是受字符串的编码长度影响的。

* **length()**:length并不是直接返回直觉上字符串的长度，而是返回在Unicode编码状态下编码码点的长度。

* **charAt()**:返回给定索引位置的字符，由传给方法的整数参数指定。
* **charCodeAt()**:返回指定索引位置的码元值，索引以整数指定。
* **fromCharCode()**: 方法用于根据给定的 UTF-16 码元创建字符串中的字符。可以接受任意多个数值，并返回将所有数值对应的字符拼接起来的字符串：

为正确解析既包含单码元字符又包含代理对字符的字符串，可以使用` codePointAt()` 来代替`charCodeAt()` ;`fromCodePoint()` 来代替`fromCharCode()` ;

* **codePointAt()**：
* **fromCodePoint()**：

```js
 let str2 = 'abcde';
 console.log(str2.charAt(0));    //a
 console.log(str2.charCodeAt(0)); //97
 console.log(String.fromCharCode(0x61, 0x62, 0x63, 0x64, 0x65)); // "abcde"
 console.log(String.fromCodePoint(0x1F60A)); // 😊
 console.log(String.fromCharCode(97, 98, 55357, 56842, 100, 101));//ab😊de
 let str3 = "b😊d";
 console.log(str3.length); //4
 console.log(str3.codePointAt(0)); //98
 console.log(str3.codePointAt(1)); //128522
 console.log(str3.codePointAt(2)); //56842
 console.log(str3.codePointAt(3)); //100
 console.log(String.fromCodePoint(98, 128522,100)); //b😊d
```

> Note：现行的字符集国际标准，字符是以 Unicode 的方式表示的，每一个 Unicode 的码点表示一个字符，理论上，Unicode 的范围是无限的。UTF 是 Unicode 的编码方式，规定了码点在计算机中的表示方法，常见的有 UTF16 和 UTF8。 Unicode 的码点通常用 U+??? 来表示，其中 ??? 是十六进制的码点值。 0-65536（U+0000 - U+FFFF）的码点被称为基本字符区域（基本多语言平面）（BMP）。



`let string1 = new String("hello world");`

String 对象的方法可以在所有字符串原始值上调用。

3个继承的方法 `valueOf()` 、`toLocaleString()`和` toString() `都返回对象的原始字符串值。



###### Number

```js
//浮点数运算的精度问题导致等式左右的结果并不是严格相等，而是相差了个微小的值。
console.log( 0.1 + 0.2 == 0.3); //false

//使用 JavaScript 提供的最小精度值,检查等式左右两边差的绝对值是否小于最小精度，才是正确的比较浮点数的方法
console.log( Math.abs(0.1 + 0.2 - 0.3) <= Number.EPSILON); //true
```



###### Symbol

Symbol 可以具有字符串类型的描述，但是即使描述相同，Symbol 也不相等.

```js
var mySymbol = Symbol("my symbol");
console.log(mySymbol);	//Symbol(mySymbol)

//使用 Symbol.iterator 来自定义 for…of 在对象上的行为：
var o = Object.create(null);
 o[Symbol.iterator] = function(){
     var v = 0 ;
     return {
         next:function(){
             return {
                 value:v++,done:v > 10 
             }
         }
     }
 }
 for(var v of o) {
     console.log(v);
 }
```



###### Object

在 JavaScript 中，对象的定义是“属性的集合”。属性分为数据属性和访问器属性，二者都是 key-value 结构，key 可以是字符串或者 Symbol 类型。

代码可以把对象的方法在基本类型上使用：

`    console.log("abc".charAt(0)); //a`

甚至我们在原型上添加方法，都可以应用于基本类型，比如以下代码，在 Symbol 原型上添加了 hello 方法，在任何 Symbol 类型变量都可以调用:

```js
    Symbol.prototype.hello = () => console.log("hello");

    var a = Symbol("a");
    console.log(typeof a); //symbol，a并非对象
    a.hello(); //hello，有效
```

为什么给对象添加的方法能用在基本类型上？. 运算符提供了装箱操作，它会根据基础类型构造一个临时对象，使得我们能在基础类型上调用对应对象的方法.



##### 类型转换

![](D:\work\note\JS_ECMA\JS分析\img\类型转换.webp)



###### 装箱转换

每一种基本类型` Number、String、Boolean、Symbol` 在对象中都有对应的类，所谓装箱转换，正是把基本类型转换为对应的对象，它是类型转换中一种相当重要的种类。



全局的 `Symbol` 函数无法使用` new` 来调用，但我们仍可以利用装箱机制来得到一个 `Symbol` 对象，我们可以利用一个函数的 `cal`l 方法来强迫产生装箱。

```js
var symbolObject = (function(){ return this; }).call(Symbol("a"));

console.log(typeof symbolObject); //object
console.log(symbolObject instanceof Symbol); //true
console.log(symbolObject.constructor == Symbol); //true
```
使用内置的` Object` 函数，我们可以在 `JavaScript` 代码中显式调用装箱能力：

```js
var symbolObject = Object(Symbol("a"));

console.log(typeof symbolObject); //object
console.log(symbolObject instanceof Symbol); //true
console.log(symbolObject.constructor == Symbol); //true
```

每一类装箱对象皆有私有的 Class 属性，这些属性可以用 `Object.prototype.toString` 获取：

```js
var symbolObject = Object(Symbol("a")); 
console.log(Object.prototype.toString.call(symbolObject)); //[object Symbol]
```

在 JavaScript 中，没有任何方法可以更改私有的 Class 属性，因此 `Object.prototype.toString` 是可以准确识别对象对应的基本类型的方法，它比 `instanceof` 更加准确。但需要注意的是，`call` 本身会产生装箱操作，所以需要配合` typeof` 来区分基本类型还是对象类型。

###### 拆箱转换

在 JavaScript 标准中，规定了 ToPrimitive 函数，它是对象类型到基本类型的转换（即，拆箱转换）。

对象到 String 和 Number 的转换都遵循“先拆箱再转换”的规则。通过拆箱转换，把对象变成基本类型，再从基本类型转换为对应的 String 或者 Number.

拆箱转换会尝试调用 `valueOf` 和 `toString `来获得拆箱后的基本类型。如果 `valueOf `和 `toString` 都不存在，或者没有返回基本类型，则会产生类型错误 TypeError。

```JS

var o = {
	valueOf : () => {console.log("valueOf"); return {}},
	toString : () => {console.log("toString"); return {}}
}

o * 2
// valueOf
// toString
// TypeError
```

定义了一个对象 o，o 有 `valueOf` 和 `toString` 两个方法，这两个方法都返回一个对象，然后我们进行 `o*2` 这个运算的时候，你会看见先执行了` valueOf`，接下来是 `toString`，最后抛出了一个 `TypeError`，这就说明了这个拆箱转换失败了。

到 String 的拆箱转换会优先调用 `toString`。我们把刚才的运算从 `o*2` 换成 `String(o)`，那么你会看到调用顺序就变了。

```JS
var o = {
	valueOf : () => {console.log("valueOf"); return {}},
	toString : () => {console.log("toString"); return {}}
}

String(o)
// toString
// valueOf
// TypeError
```

在 ES6 之后，还允许对象通过显式指定 @@toPrimitive Symbol 来覆盖原有的行为。

```JS
var o = {
	valueOf : () => {console.log("valueOf"); return {}},
	toString : () => {console.log("toString"); return {}}
}

o[Symbol.toPrimitive] = () => {console.log("toPrimitive"); return "hello"}

console.log(o + "")
// toPrimitive
// hello
```



##### JavaScript 运行时的类型系统 总结

除了这七种语言类型，还有一些语言的实现者更关心的规范类型。

* List 和 Record： 用于描述函数传参过程。

* Set：主要用于解释字符集等。

* Completion Record：用于描述异常、跳出等语句执行过程。

* Reference：用于描述对象属性访问、delete 等。

* Property Descriptor：用于描述对象的属性。

* Lexical Environment 和 Environment Record：用于描述变量和作用域。

* Data Block：用于描述二进制数据。

  

  有一个说法是：程序 = 算法 + 数据结构，运行时类型包含了所有 JavaScript 执行时所需要的数据结构的定义。