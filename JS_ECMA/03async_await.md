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

一个提交表单的页面，里面有姓名、地址等巴拉巴拉的信息，其中有一项是手机验证码，我们不得不等待手机验证码接口通过，才能发出后续的请求操作，这时候接口之间就存在了彼此依赖的关系，`async`跟`await`就有了用武之地，让异步请求之间可以按顺序执行.

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



另外一篇文章说明:

#### Promise



#### async await

##### 一，async

字面理解，`async` 是“异步”的简写，而 `await` 可以认为是 `async wai`t 的简写。所以应该很好理解 `async` 用于申明一个 `function` 是异步的，而 `await` 用于等待一个异步方法执行完成。

`async` 函数返回的是一个 `Promise` 对象。`async` 函数（包含函数语句、函数表达式、`Lambda`表达式）会返回一个 `Promise` 对象。如果在函数中 `return` 一个直接量，`async` 会把这个直接量通过 `Promise.resolve()` 封装成 `Promise` 对象。

##### 二，await

 `async` 函数返回一个 `Promise` 对象，所以 `await` 可以用于等待一个 `async` 函数的返回值——这也可以说是 `await` 在等 `async` 函数，但要清楚，它等的实际是一个返回值。注意到 await 不仅仅用于等 `Promise` 对象，它可以等任意表达式的结果，所以，`await` 后面实际是可以接普通函数调用或者直接量的.

```javascript
function getSomething() {
    return "something";
}

async function testAsync() {
    return Promise.resolve("hello async");
}

async function test() {
    const v1 = await getSomething();
    const v2 = await testAsync();
    console.log(v1, v2);
}

test();
```

如果它等到的不是一个 `Promise` 对象，那 `await` 表达式的运算结果就是它等到的东西。

如果它等到的是一个 `Promise` 对象，`await` 就忙起来了，它会阻塞后面的代码，等着 `Promise` 对象 `resolve`，然后得到 `resolve` 的值，作为 `await` 表达式的运算结果。

> 看到上面的阻塞一词，心慌了吧……放心，这就是 await 必须用在 async 函数中的原因。async 函数调用不会造成阻塞，它内部所有的阻塞都被封装在一个 Promise 对象中异步执行。





如果一个函数本身就返回 Promise 对象，加 `async` 和不加 `async` 还是有一点点区别：加了 `async` 之后外面得到 Promise 对象并不是 `return` 的那一个，参阅代码：

```javascript
(() => {
    let promise;
    async function test() {
        promise = new Promise(resolve => resolve(0));promise.then(res => console.log(res))
        promise.mark = "hello";
        return promise;
    }

    const gotPromise = test();
    console.log(`is same object?: ${promise === gotPromise}`);  // false
    console.log(`promise.mark: ${promise.mark}`);               // hello
    console.log(`gotPromise.mark: ${gotPromise.mark}`);         // undefined
})();
//:10 is same object?: false
//:11 promise.mark: hello
//:12 gotPromise.mark: undefined
//:4 0
```

##### 三，async/await 的优势在于处理 then 链

单一的 `Promise` 链并不能发现 `async/await` 的优势，但是，如果需要处理由多个 `Promise` 组成的 `then` 链的时候，优势就能体现出来了（很有意思，`Promise` 通过 `then` 链来解决多层回调的问题，现在又用 `async/await` 来进一步优化它）。

假设一个业务，分多个步骤完成，每个步骤都是异步的，而且依赖于上一个步骤的结果。我们仍然用 `setTimeout` 来模拟异步操作：

```js
function getTaskTime(n){
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve(n+200)
        },n)
    })
}

function step1(x){
    console.log(`step1 with ${x}`);
    return getTaskTime(x);
}

function step2(y){
    console.log(`step2 with ${y}`)
    return getTaskTime(y)
}
function step3(z){
    console.log(`step3 with ${z}`)
    return getTaskTime(z)
}

// function doIt(){
//     console.time('doIt');
//     const time1 = 300;
//     step1(time1)
//         .then(time2 => step2(time2))
//         .then(time3 => step3(time3))
//         .then(time => {
//             console.log(time);
//             console.timeEnd('doIt')
//         })
// }
// doIt();

async function doIt() {
    console.time("doIt");
    const time1 = 300;
    const time2 = await step1(time1);
    const time3 = await step2(time2);
    const result = await step3(time3);
    console.log(`result is ${result}`);
    console.timeEnd("doIt");
}

doIt();
```

输出结果 `result` 是 `step3()` 的参数 `700 + 200` = `900`。`doIt()` 顺序执行了三个步骤，一共用了 `300 + 500 + 700 = 1500` 毫秒，和 `console.time()/console.timeEnd()` 计算的结果一致。



**现在把业务要求改一下，仍然是三个步骤，但每一个步骤都需要之前每个步骤的结果。**

```javascript
function takeLongTime(n){
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve(n+200)
        },n)
    })
}
function step1(n) {
    console.log(`step1 with ${n}`);
    return takeLongTime(n);
}

function step2(m, n) {
    console.log(`step2 with ${m} and ${n}`);
    return takeLongTime(m , n);
}

function step3(k, m, n) {
    console.log(`step3 with ${k}, ${m} and ${n}`);
    return takeLongTime(k , m , n);
}
async function doIt() {
    console.time("doIt");
    const time1 = 300;
    const time2 = await step1(time1);
    const time3 = await step2(time1, time2);
    const result = await step3(time1, time2, time3);
    console.log(`result is ${result}`);
    console.timeEnd("doIt");
}

doIt();

// step1 with 300
// step2 with 300 and 500
// step3 with 300, 500 and 1000
// result is 2000
// doIt: 2.935s

//promise 那一堆参数处理，就是 Promise 方案的死穴—— 参数传递太麻烦
function doIt2() {
    console.time("doIt");
    const time1 = 300;
    step1(time1)
        .then(time2 => {
            return step2(time1, time2)
                .then(time3 => [time1, time2, time3]);
        })
        .then(times => {
            const [time1, time2, time3] = times;
            return step3(time1, time2, time3);
        })
        .then(result => {
            console.log(`result is ${result}`);
            console.timeEnd("doIt");
        });
}

doIt2();
```

