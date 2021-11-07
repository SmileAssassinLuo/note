#### Stack

栈：后入先出的数据结构，JS的栈数据基于数组或者对象实现；

```javascript
 class Stack {
     constructor (){
         this._count = 0;
         this._items = {};
     }

     push( element ) {
         this._items[this._count] = element;
         this._count++;
     }
     size() {
         return this._count;
     }
     isEmpty(){
         return  this._count === 0;
     }
     pop(){
         if(isEmpty()) return undefined;

         this._count--;
         const deleteKey = this._items[this._count];
         delete this._items[this._count];
         return deleteKey;
     }
 }

const stack = new Stack();
stack.push(5);
stack.push(8);
console.log(stack);//{1}
console.log(Object.getOwnPropertyNames(stack)); //{2}
console.log(Object.keys(stack)); //{3}
console.log(stack._items); //{4}

//{2}输出结果是 ["count", "items"] 。这表示 count 和 items 属性是公开的，我们可以像行 {4} 那样直接访问它们。根据这种行为，我们可以对这两个属性赋新的值.比如 stack.count = 4;
```

在栈数据内部，保护内部的元素，只有我们暴露出的方法才能修改内部结构。对于 Stack 类来说，要确保元素只会被添加到栈顶，而不是栈底或其他任意位置（比如栈的中间）。

保护数据方式：

* 通过命名带下划线，只是约定俗成的方式为私有属性，并无法做到不可变更；

* 限定作用域 Symbol 实现类

  ```javascript
  const _items = Symbol('stackItems'); // {1}
  class Stack {
      constructor () {
      this[_items] = []; // {2}
  	}
  	// 栈的方法
  }
  const stack = new Stack();
  stack.push(5);
  stack.push(8);
  let objectSymbols = Object.getOwnPropertySymbols(stack);
  console.log(objectSymbols.length); //  输出 1
  console.log(objectSymbols); // [Symbol()]
  console.log(objectSymbols[0]); // Symbol()
  stack[objectSymbols[0]].push(1);
  stack.print(); //  输出 5, 8, 1
  
  //种方法创建了一个假的私有属性，因为 ES2015 新增的 Object.getOwnProperty-Symbols 方法能够取到类里面声明的所有 Symbols 属性.上述方式破坏了Stack类；
  //访问 stack[objectSymbols[0]] 是可以得到 _items 的。并且，_items 属性是一个数组，可以进行任意的数组操作，比如从中间删除或添加元素（使用对象进行存储也是一样的）。但我们操作的是栈，不应该出现这种行为.
  ```

  

* 通过WeakMap创建的栈数据，items 在 Stack 类里是真正的私有属性。采用这种方法，代码的可读性不强，而且在扩展该类时无法继承私有属性。鱼和熊掌不可兼得！

  ```javascript
  //使用场景：将十进制转换成二进制
  
  const items = new WeakMap(); 
  class Stack {
      constructor () {
     	 items.set(this, []); 
  	}
      push(element){
          const s = items.get(this); 
          s.push(element);
  	}
  	pop(){
          const s = items.get(this);
          const r = s.pop();
          return r;
  	}
  	isEmpty(){
          const siKey = items.get(this);
          return siKey.length === 0;
      	}
  	}
  
  function decimalToBinary(decNumber){
      const remStack = new Stack();
      let num = decNumber;
      let rem ;
      let binaryString = '';
  
      while(num > 0 ){
          rem = Math.floor(num % 2 );
          remStack.push(rem);
          num = Math.floor(num / 2 );
      }
      console.log(remStack);
      while(!remStack.isEmpty()){
      	//console.log(remStack.pop());
     	 	binaryString += remStack.pop().toString();
  	}
  
  	return binaryString;
  }
  console.log(decimalToBinary(233)); // 11101001
  // console.log(decimalToBinary(10)); // 1010
  // console.log(decimalToBinary(1000)); // 1111101000
  ```

  



