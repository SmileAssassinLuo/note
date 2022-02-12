##### ECMAScript 中函数的参数就是局部变量

* 参数为值类型

* 参数为引用类型

  ```javascript
  /**
   *  ECMAScript 中函数的参数就是局部变量：
   * 
   * obj 指向的对象保存在全局作用域的堆内存上。很多开发者错误地认为，
   * 当在局部作用域中修改对象而变化反映到全局时，就意味着参数是按引用传递的。
   * 但是：setName2函数中重写obj时，obj 只是本地的一个引用，在函数执行完毕就销毁；
   */
  function setName(obj) {
      obj.name = 'Isaiah'
  }
  let person = new Object();
  setName(person);
  console.log(person.name); //Isaiah
  
  /**
   * 如果 person 是按引用传递的，那么 person 应该自动将指针改为指向 name 为 "Lucy" 的对象
   */
  function setName2(obj){
      obj.name = 'Isaiah';
      obj = new Object();
      obj.name = 'Lucy';
  }
  
  let person2 = new Object();
  setName2(person2);
  console.log(person2.name); //Isaiah
  ```

  

