#### Async_Await

##### 一，基本概念

###### 1、Async—声明一个异步函数(async function someName(){...})

- 自动将常规函数转换成Promise，返回值也是一个Promise对象
- 只有async函数内部的异步操作执行完，才会执行then方法指定的回调函数
- 异步函数内部可以使用await

###### 2、Await—暂停异步的功能执行(var result = await someAsyncCall();)

- 放置在Promise调用之前，await强制其他代码等待，直到Promise完成并返回结果
- 只能与Promise一起使用，不适用与回调
- 只能在async函数内部使用

###### 3、使用小贴士：async函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而await命令就是内部then命令的语法糖。

##### 二，使用方式

###### 1、使用 async

```javascript
async function foo(){}
const foo = async function(){}
let obj = {async foo(){}}
obj.foo().then(() => console.log('hello  async'))
```

###### 2、使用await

await的用法则相对简单了许多，他后面需要是一个Promise对象，如果不是则会被转成Promise对象。只要其中一个如果Promise对象变为reject状态，那么整个async函数都会中断操作。

如果状态是resolve，则在函数条用后`.then(res)` 即可取到resolve的状态。

如果状态是reject,需要用`try{}catch(err){}` 捕获。

```javascript
async function foo(){
	let awaitResult = await fetch('/xxx');
    return awaitResult;
}
foo().then(res => console.log(res))
```

##### 三，使用场景

###### 1、多个不想关的异步操作

需要执行多个异步操作时，

```javascript
async function foo(){
    //let a = await fetch('/aaa');
    //let b = await fetch('/bbb');
    let results = await Promise.all([fetch('/aaa'),fetch('/bbb')])
    return results;
}
foo().then(results => console.log(results))
```

###### 2、当一个异步操作依赖前一个异步操作的返回值时

一个提交表单的页面，里面有姓名、地址等巴拉巴拉的信息，其中有一项是手机验证码，我们不得不等待手机验证码接口通过，才能发出后续的请求操作，这时候接口之间就存在了彼此依赖的关系，Async跟Await就有了用武之地，让异步请求之间可以按顺序执行.

`.json()`:

返回一个被解析为[`JSON`](https://developer.mozilla.org/zh-CN/docs/Web/API/JSON)格式的promise对象.

```javascript
//使用promise
const aPromise = fetch('/aaa')
aPromise
.then(res => response.json())
.then(json => {
    const bPromise = fetch('/bbb')
    return bPromise
})
.then(res => res.json())
.then(json =>{
    console.log(json.respCode)
})
.catch(err => {
    console.log(err)
})
```

```javascript
//async await
async function postForm(){
    try{
        const res1 = await fetch('/aaa');
        const json1 = res1.json()
        let json2;
        if(json1.respCode === 200){
            const res2 = await fetch('/bbb');
            json2 = res2.json()
        }
        return json2
    }catch(err => console.log(err))
}
postForm().then(res => console.log(res))
```



