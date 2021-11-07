### 脚本 or 模块

现在 script 标签可代表两种类型，如果不加 type = 'module' ,则会被默认为脚本，反之则为模块；



脚本可以包含语句;

模块怎包含三种内容：import 声明,export 声明和语句，普通语句； 



#### import 声明

两种方式通过 import ：

```javascript
import "mod";

import mod from 'mod'
```

* 直接import 只是保证模块执行，无法或者其他信息；
* 带 from 的import 引入模块的部分信息，可以将它们变成本地变量



#### 带 from 的import

用法：

* import	  x 	from	"a.js"  引入模块中导出的默认值；
* import {a as x ,modify}  from "a.js"  引入模块中的变量；
* import    *    as   x   from  "a.js"   把模块中所有的变量以类似对象属性的方式引入； 

第一种还可以与后两种合并使用：**语法要求不带as的默认值永远在最前**

* import d , {a as x ,modify}  from "a.js"
* import d,  *    as   x   from  "a.js"



模块a：

```javascript
export  var  a = 1;

export function modify(){
    a = 2;
}
```

模块b:

```javascript
import { a, modify } from './a.js';

console.log(a);

modify();

console.log(a);
```

调用代表函数后，b模块的变量也跟着发生了变化，说明导入跟一般的赋值不一样，导入后的变量只是改变了名字，它仍然是与原来的变量是同一个；



#### export 声明

与import 相对，export 声明是导出；

导出变量有两种方式：独立使用export 声明，或者 直接在声明语句前添加 export 关键字；

```javascript
export {a,b,c};，

```

可以直接在export 后接 var const function 等；

特殊用法：

export default  表示导出一个默认变量值，可以用于function 和 class ，导出无变量名，可通过 import  x from 'a.js' 的方式引入；

还支持后面跟一个表达式

var  a = {};

export default a ;

**但是这里的行为跟导出变量是不一致的，这里导出的是值，导出是普通变量a的值，以后a的变化与导出的值无关，修改变量a，不会是的其他模块中引入的default值发生改变。**



