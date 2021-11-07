##### setTimeout and setInterval

`setTimeout`: 等待一段时间间隔后，再执行代码，只执行一次。

`setInterval`:根据执行毫秒数不断间隔执行代码。



问题：setInterval的参数函数执行时间过长，会导致时间间隔不准确。

用`setTimeout`替代`setInterval`

```javascript
  function newSetInterval(func,middleTime){
       function inSide(){
           func();
           setTimeout(inSide,middleTime)
       }
       setTimeout(inSide,middleTime);
  }
  function like(){
   console.log("hello");
  }
  newSetInterval(like,1000);
```



