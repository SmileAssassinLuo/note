### 一，创建数组

* 构造函数

* 字面量

* ES6 新增Array.from()  Array.of()

  ```javascript
  Array.from();将类数组转为数组；
  console.log(Array.from("Matt")); // ["M", "a", "t", "t"]
  
  const m = new Map().set(1, 2)
  	m.set(3, 4);
  const s = new Set().add(1)
      s.add(2)
      s.add(3)
      s.add(4);
  console.log(Array.from(m)); // [[1, 2], [3, 4]]
  console.log(Array.from(s)); // [1, 2, 3, 4]
  
  
  Array.of() 可以把一组参数转换为数组。这个方法用于替代在ES6之前常用的 Array.prototype.
  slice.call(arguments) ，一种异常笨拙的将 arguments 对象转换为数组的写法：
  console.log(Array.of(1, 2, 3, 4)); // [1, 2, 3, 4]
  console.log(Array.of(undefined)); // [undefined]
  ```

### 二，数组空位

​		使用数组字面量初始化数组时，可以使用一串逗号来创建空位（hole）。ECMAScript 会将逗号之间相应索引位置的值当成空位，ES6 规范重新定义了该如何处理这些空位

