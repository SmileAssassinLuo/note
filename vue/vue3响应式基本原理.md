  ------------------------------------------------------------------------------------------
#### 响应式基础
#####    1.1,响应式的基本实现
    创建了⼀个⽤于存储副作⽤函数的桶 bucket，它是 Set 类型。接着定义原始数据targetData , proxyObj是原始数据的代理对象，我们分别设置了 get 和 set 拦截函数，
    ⽤于拦截读取和设置操作当读取属性时将副作⽤函数 effect 添加到桶⾥，即 bucket.add(effect)，然后返回属性值；
    当设置属性值时先更新原始数据，再将副作⽤函数从桶⾥取出并重新执⾏，这样我们就实现了响应式数据.
    
const bucket1 = new Set();// 存储副作⽤函数的桶

const targetData1 = {text:"hello"};

const proxyObj1 = new Proxy(targetData1,{
    get(target,key){
        // 将副作⽤函数 effect 添加到存储副作⽤函数的桶中
        bucket1.add(effect1);
        return target[key]
    },
    set(target, key ,newVal){
        target[key] = newVal;
        // 把副作⽤函数从桶⾥取出并执⾏
        bucket1.forEach(fn => fn());
        // 把副作⽤函数从桶⾥取出并执⾏
        return true
    }
})

function effect1(){
    document.getElementById("root").innerText = proxyObj1.text;
}
effect1();
setTimeout(() => {
    proxyObj1.text = "hello vue3"
}, 1000);
  
   ------------------------------------------------------------------------------------------
#####    1.2,完善响应式
          副作用函数名字可以动态自定义，且可以使用匿名函数
  
  
//用一个全局变量存储被注册的副作用函数
let activeEffect2
//effect 函数用于注册副作用函数
function effect2(fn){
    activeEffect2 = fn;//当调⽤ effect 注册副作⽤函数时，将副作⽤函数 fn 赋值给 activeEffect
    // 执行副作用
    fn()
}

const bucket2 = new Set();
const targetData2 = {text:"动态副作用函数名"};
const proxyObj2 = new Proxy(targetData2,{
    get(target,key){
        if(activeEffect2){
            bucket2.add(activeEffect2)
        }
        return target[key]
    },
    set(target,key,newVal){
        target[key] = newVal;
        bucket2.forEach(fn => fn());
        return true
    }
})
effect2(
    () => {
        console.log('effect run');
        document.getElementById("root").innerText = proxyObj2.text;
    }
)
setTimeout(() => {
    // proxyObj2.text = "修改后的动态副作用函数名"
    proxyObj2.noExist = "不存在的属性";
},2000)
  

  -----------------------------------------------------------------------------------------------
####    1.3,存在的问题
    问题： 没有在副作⽤函数与被操作的⽬标字段之间建⽴明确的联系。
    说明：  导致给 proxyObj2 设置一个不存在的属性时也会执行作用函数，因为并没有读取proxyObj2.noExist的
           值，不应该执行副作用函数；
    解决方式：重新设计数据结构，不简单使用Set类型的数据作为“桶”了。
    解析：  代码中存在的角色:
               1，被操作(读取)的代理对象 proxyObj2；
               2，被操作(读取)的字段 text ；
               3，使⽤ effect2 函数注册的副作⽤函数 effectFn。
            他们之间是一种树型结构
                target
                    └── key
                         └── effectFn
                如果有两个副作⽤函数同时读取同⼀个对象的属性值：
                target
                    └── key
                         └── effectFn1  
                         └── effectFn2 
                如果⼀个副作⽤函数中读取了同⼀个对象的两个不同属性：
                target
                    └── key1
                         └── effectFn  
                    └── key2
                         └── effectFn  
                如果在不同的副作⽤函数中读取了两个不同对象的不同属性：
                 target
                    └── key1
                         └── effectFn1  
                    └── key2
                         └── effectFn2  
                总之就是字段被操作，只执行相关联的副作用，而其他副作用不应该执行
    
    新的数据结构:
            使用 WeakMap 代替 Set 作为桶的数据结构
            构建数据结构的⽅式，我们分别使⽤了 WeakMap、Map 和 Set：
                WeakMap 由 target --> Map 构成；
                Map 由 key --> Set 构成。
   
   
    const bucket3 = new WeakMap();
    const targetData3 = {text:"新的数据结构桶"};
    const proxyObj3 = new Proxy(targetData3,{
        get(target,key){
            track(target,key);
            return target[key]
        },
        set(target,key,newVal){
            target[key] = newVal;
            trigger(target,key);
        }
    })
    let activeEffect3 ;
    function effect3(fn){
        activeEffect3 = fn;
        fn()
    }
    effect3(
        () => {
            console.log('effect3 run');
            document.getElementById("root").innerText = proxyObj3.text;  
        }
    )
  
    -----------------------------------------------------------------------------------------------------------
 ####  1.4 ，优化
        在⽬前的实现中，当读取属性值时，我们直接在 get 拦截函数⾥编写把副作⽤函数收集到“桶”⾥的这部分逻辑，
        但更好的做法是将这部分逻辑单独封装到⼀个track 函数中，函数的名字叫 track 是为了表达追踪的含义。
        同样，我们也可以把触发副作⽤函数重新执⾏的逻辑封装到 trigger 函数中：
   
   ```
    // 在 get 拦截函数内调⽤ track 函数追踪变化
   
   function track(target,key){
        if(!activeEffect3) return target[key];
        // 根据 target 从“桶”中取得 depsMap，它也是⼀个 Map 类型：key --> effects
        let depsMap = bucket3.get(target);
        if(!depsMap){
            bucket3.set(target, (depsMap = new Map()))
        }
        let deps = depsMap.get(key);
        if(!deps){
            depsMap.set(key,(deps = new Set()));// 如果 deps 不存在，同样新建⼀个 Set 并与 key 关联
        }
        // 最后将当前激活的副作⽤函数添加到“桶”⾥
        deps.add(activeEffect3);
    }
     // 在 set 拦截函数内调⽤ trigger 函数触发变化
    function trigger(target,key){
        // 根据 target 从桶中取得 depsMap，它是 key --> effects
        const depsMap = bucket3.get(target);
        if(!depsMap) return;
        // 根据 key 取得所有副作⽤函数 effects
        const effects = depsMap.get(key);
        effects && effects.forEach(fn => fn())
    }

    setTimeout(() => {
        // proxyObj2.text = "修改后的动态副作用函数名"
        proxyObj3.noExist = "不存在的属性aaaa";
    },2000)
  ```

  ------------------------------------------------------------------------------------
####   2.1 分⽀切换与 cleanup
       分⽀切换的定义:
       const data = {ok:true,text:"hello world"}     
       const obj = new Proxy(data, {}) 
       effect(function effectFn() {
            document.body.innerText = obj.ok ? obj.text : 'not'
       })
       在 effectFn 函数内部存在⼀个三元表达式，根据字段 obj.ok 值的不同会执⾏不同的代码分⽀
       分⽀切换可能会产⽣遗留的副作⽤函数
       data
         └── ok
              └── effectFn
         └── text 
              └── effectFn
        
    问题1，当 data.ok 修改为false，并触发副作⽤函数重新执⾏后，由于此时字段 obj.text 不会被读取，
          只会触发字段 obj.ok 的读取操作，所以理想情况下
          副作⽤函数 effectFn 不应该被字段obj.text 所对应的依赖集合收集；
   
   解决方式：当副作⽤函数执⾏完毕后，会重新建⽴联系，但在新的联系中不会包含遗留的副作⽤函数；
            如果我们能做到每次副作⽤函数执⾏前，将其从相关联的依赖集合中移除即可。
            要将⼀个副作⽤函数从所有与之关联的依赖集合中移除，就需要明确知道哪些依赖集合中包含它，因
此我们需要重新设计副作⽤函数。
   
     
//  在 effect 内部我们定义新的 effectFn 函数，并为其添加 effectFn.deps 属性，用来存储所有包含当前
//  副作用的的依赖集合
    const bucket4 = new WeakMap();
    const targetData4 = {ok:true,text:"分⽀切换和cleanup"};
    let activeEffect4 ;
    const proxyObj4 = new Proxy(targetData4,{
        get(target,key){
            track(target,key);
            return target[key]
        },
        set(target,key,newVal){
            target[key] = newVal;
            trigger(target,key);
        }
    })
    function effect4(fn){
        const effectFn = () => {
            // 调⽤ cleanup 函数完成清除⼯作
            cleanup(effectFn) // 新增

           // 当 effectFn 执⾏时，将其设置为当前激活的副作⽤函数
           activeEffect4 = effectFn;
           fn()
        }
        // activeEffect.deps ⽤来存储所有与该副作⽤函数相关联的依赖集合
        effectFn.deps = [];
        //执行副作用
        effectFn();
    }
    //那么 effectFn.deps 数组中的依赖集合是如何收集的呢？其实是在 track 函数中：
    function track(target,key){
        if(!activeEffect4) return target[key];
        // 根据 target 从“桶”中取得 depsMap，它也是⼀个 Map 类型：key --> effects
        let depsMap = bucket4.get(target);
        if(!depsMap){
            bucket4.set(target, (depsMap = new Map()))
        }
        let deps = depsMap.get(key);
        if(!deps){
            depsMap.set(key,(deps = new Set()));// 如果 deps 不存在，同样新建⼀个 Set 并与 key 关联
        }
        // 最后将当前激活的副作⽤函数添加到“桶”⾥
        deps.add(activeEffect4);
        // deps 是一个与当前副作用函数存在联系的依赖集合
        // 将其添加到 activeEffect.deps 数组中
        activeEffect4.deps.push(deps)
    }
    function trigger(target,key){
        // 根据 target 从桶中取得 depsMap，它是 key --> effects
        const depsMap = bucket4.get(target);
        if(!depsMap) return;
        // 根据 key 取得所有副作⽤函数 effects
        const effects = depsMap.get(key);

        const effectToRun = new Set(effects);        // 新增
        effectToRun.forEach(effectFn=> effectFn())   // 新增
        //effects && effects.forEach(fn => fn())    // 删除
    }
//cleanup 函数接收副作⽤函数作为参数，遍历副作⽤函数的 effectFn.deps 数组，
//          该数组的每⼀项都是⼀个依赖集合，然后将该副作⽤函数从依赖集合中移除，
//          最后重置 effectFn.deps 数组。
    function cleanup(effectFn){
        for(let i=0;i < effectFn.deps.length;i++){
            // deps 是依赖集合
            const deps = effectFn.deps[i];
            // 将 effectFn 从依赖集合set中移除
            deps.delete(effectFn)
        }
        // 最后需要重置 effectFn.deps 数组
        effectFn.deps.length = 0
    }
    effect4(
        () => {
            console.log('effect4 run');
            document.getElementById("root").innerText = proxyObj4.ok ? proxyObj4.text:"not" 
        }
    )
    setTimeout(() => {
        proxyObj4.ok = false
    },2000)
  
  *------------------------------------------------------------------------------------
 ####  3.1,嵌套的 effect 与 effect 栈
       什么时候会发生嵌套？当组件发生嵌套时，便发生了 effect 嵌套；
       在⼀个 effect 中执⾏ Foo 组件的渲染函数：
            effect(() => {
                    Foo.render() 
            })
        当组件发⽣嵌套时，例如 Foo 组件渲染了 Bar 组件：发⽣了 effect 嵌套，它相当于：
         effect(() => {
            Foo.render()
            // 嵌套
            effect(() => {
                Bar.render()
            })
         })

   
  *
   ⽤全局变量 activeEffect 来存储通过 effect 函数注册的副作⽤函数，
   这意味着同⼀时刻activeEffect 所存储的副作⽤函数只能有⼀个。
   当副作⽤函数发⽣嵌套时，内层副作⽤函数的执⾏会覆盖 activeEffect 的值，并且永远不会恢复到原来的值。
   这时如果再有响应式数据进⾏依赖收集，即使这个响应式数据是在外层副作⽤函数中读取的，
   它们收集到的副作⽤函数也都会是内层副作⽤函数，这就是问题所在。
   为了解决这个问题，我们需要⼀个副作⽤函数栈 effectStack，在副作⽤函数执⾏时，
   将当前副作⽤函数压⼊栈中，待副作⽤函数执⾏完毕后将其从栈中弹出，
   并始终让 activeEffect 指向栈顶的副作⽤函数
   
   
  
//用全局变量存储当前激活的 effect 函数
let activeEffect5;
// effect 栈
const effectStack = [];
function effect5(fn){
    const effectFn = () => {
        cleanup(effectFn);
        activeEffect5 = effectFn;
        effectStack.push(effectFn);
        fn();
        // 在当前副作⽤函数执⾏完毕后，将当前副作⽤函数弹出栈，并把 activeEffect 还原为之前的值
        effectStack.pop();
        activeEffect5 = effectStack[effectStack.length-1];
    }
    effectFn.deps = [];
    effectFn();
}
  
  *------------------------------------------------------------------------------------
 ####  4.1 避免⽆限递归循环
        问题：在effect 注册的副作⽤函数内有⼀个⾃增操作 obj.foo++，该操作会引起栈溢出
                const data = { foo: 1 }
                const obj = new Proxy(data, {})
                effect(() => obj.foo++);

                // obj.foo++ -----> obj.foo = obj.foo + 1
                既会读取 obj.foo 的值，⼜会设置 obj.foo 的值，⽽这就是导致问题的根本原因；

        解决方式：
            读取和设置操作是在同⼀个副作⽤函数内进⾏的。此时⽆论是 track 时收集的副作⽤函数，
            还是 trigger 时要触发执⾏的副作⽤函数，都是activeEffect。基于此，
            我们可以在trigger 动作发⽣时增加守卫条件：如果 trigger 触发执⾏的副作⽤函数
            与当前正在执⾏的副作⽤函数相同，则不触发执⾏。
   
  
    function trigger(target,key){
        // 根据 target 从桶中取得 depsMap，它是 key --> effects
        const depsMap = bucket4.get(target);
        if(!depsMap) return;
        // 根据 key 取得所有副作⽤函数 effects
        const effects = depsMap.get(key);
        const effectToRun = new Set(effects);  
        // 如果 trigger 触发执⾏的副作⽤函数与当前正在执⾏的副作⽤函数相同，则不触发执⾏
        effects && effects.forEach(effectFn => {
            if(effectFn !== activeEffect5){  // 新增
                effectToRun.add(effectFn)
            }
        })  
        effectToRun.forEach(effectFn=> effectFn())   
        //effects && effects.forEach(fn => fn())    
    }
  
  *------------------------------------------------------------------------------------
####   5.1 调度执行
        可调度性是响应系统⾮常重要的特性。⾸先我们需要明确什么是可调度性。
        所谓可调度，指的是当trigger 动作触发副作⽤函数重新执⾏时，有能⼒决定副作⽤函数执⾏的时机、次数以及⽅式。
   
  *
const bucket6 = new WeakMap();
const targetData6 = {ok:true,text:"调度执行",foo:0};
let activeEffect6 ;
const effectStack6 = [];
const proxyObj6 = new Proxy(targetData6,{
    get(target,key){
        track(target,key);
        return target[key]
    },
    set(target,key,newVal){
        target[key] = newVal;
        trigger(target,key);
    }
})
function effect6(fn,options={}){
    const effectFn = () => {
        cleanup6(effectFn);
        activeEffect6 = effectFn;
        effectStack6.push(effectFn);
        fn();
        // 在当前副作⽤函数执⾏完毕后，将当前副作⽤函数弹出栈，并把 activeEffect 还原为之前的值
        effectStack6.pop();
        activeEffect6 = effectStack6[effectStack6.length-1];
    }
    // 将 options 挂载到 effectFn 上 add
    effectFn.options = options;

     // activeEffect.deps ⽤来存储所有与该副作⽤函数相关联的依赖集合
     effectFn.deps = [];
     //执行副作用
     effectFn();
}
//那么 effectFn.deps 数组中的依赖集合是如何收集的呢？其实是在 track 函数中：
function track(target,key){
    if(!activeEffect6) return target[key];
    // 根据 target 从“桶”中取得 depsMap，它也是⼀个 Map 类型：key --> effects
    let depsMap = bucket6.get(target);
    if(!depsMap){
        bucket6.set(target, (depsMap = new Map()))
    }
    let deps = depsMap.get(key);
    if(!deps){
        depsMap.set(key,(deps = new Set()));// 如果 deps 不存在，同样新建⼀个 Set 并与 key 关联
    }
    // 最后将当前激活的副作⽤函数添加到“桶”⾥
    deps.add(activeEffect6);
    // deps 是一个与当前副作用函数存在联系的依赖集合
    // 将其添加到 activeEffect.deps 数组中
    activeEffect6.deps.push(deps)
}
function trigger(target,key){
    // 根据 target 从桶中取得 depsMap，它是 key --> effects
    const depsMap = bucket6.get(target);
    if(!depsMap) return;
    // 根据 key 取得所有副作⽤函数 effects
    const effects = depsMap.get(key);
    const effectToRun = new Set(effects);  
    // 如果 trigger 触发执⾏的副作⽤函数与当前正在执⾏的副作⽤函数相同，则不触发执⾏
    effects && effects.forEach(effectFn => {
        if(effectFn !== activeEffect6){  // 新增
            effectToRun.add(effectFn)
        }
    })  
    effectToRun.forEach(effectFn=> {
        // 如果⼀个副作⽤函数存在调度器，则调⽤该调度器，并将副作⽤函数作为参数传递
        if(effectFn.options.scheduler){
            effectFn.options.scheduler(effectFn); //add 
        }else{
            effectFn()
        }
    })   
    //effects && effects.forEach(fn => fn())    
}
//cleanup 函数接收副作⽤函数作为参数，遍历副作⽤函数的 effectFn.deps 数组，
//          该数组的每⼀项都是⼀个依赖集合，然后将该副作⽤函数从依赖集合中移除，
//          最后重置 effectFn.deps 数组。
function cleanup6(effectFn){
    for(let i=0;i < effectFn.deps.length;i++){
        // deps 是依赖集合
        const deps = effectFn.deps[i];
        // 将 effectFn 从依赖集合set中移除
        deps.delete(effectFn)
    }
    // 最后需要重置 effectFn.deps 数组
    effectFn.deps.length = 0
}
const jobQueue = new Set();
//使用 Promise.resolve() 创建一个promise 实例，用它将一个任务添加到微任务队列
const p = Promise.resolve();
// 一个标志代表是否正在刷新队列
let isFlushing = false;
function flushJob(){
    // 如果队列正在刷新，则什么都不做
    if(isFlushing) return
    // 设置为 true，代表正在刷新
    isFlushing =true;
    // 在微任务队列中刷新 jobQueue 队列
    p.then(() => {
        jobQueue.forEach(job => job())
    }).finally(() => {
        isFlushing =false
    })
}
effect6(
    () => {
        console.log('proxyObj6.foo:',proxyObj6.foo);
    },
    // Holding scheduling  add ⽀持调度 options
    {
        // 调度器 scheduler 是⼀个函数,利用任务队列即可实现控制执行顺序
        scheduler (fn){
            //setTimeout(fn); //控制执行顺序

            // 控制执行次数
            // 每次调度时，将副作⽤函数添加到 jobQueue 队列中
            jobQueue.add(fn);
            // 调⽤ flushJob 刷新队列
            flushJob();
        },
    }
)
   
//按照顺序打印 0，1 ，end;
//若需要调整顺序为0，end，1 ；如何处理？就需要响应系统⽀持调度。
  *
   响应系统⽀持调度:
        1，控制副作用执行顺序
        2，控制执行次数，连续自增应只执行一次；中不包含过渡状态，基于调度器我们可以很容易地实现此功能
   
  *
proxyObj6.foo++;
proxyObj6.foo++;  //如何控制执行次数，连续自增应只执行一次
console.log('end');
  
  *
   整段代码的效果是，连续对 obj.foo 执⾏两次⾃增操作，会同步且连续地执⾏两次 scheduler 调度函数，
   这意味着同⼀个副作⽤函数会被 jobQueue.add(fn) 语句添加两次，但由于 Set 数据结构的去重能⼒，
   最终 jobQueue 中只会有⼀项，即当前副作⽤函数.
   类似地，flushJob 也会同步且连续地执⾏两次，但由于 isFlushing 标志的存在，
   实际上 flushJob 函数在⼀个事件循环内只会执⾏⼀次，即在微任务队列内执⾏⼀次。
   当微任务队列开始执⾏时，就会遍历 jobQueue 并执⾏⾥⾯存储的副作⽤函数。
   由于此时 jobQueue 队列内只有⼀个副作⽤函数，所以只会执⾏⼀次，并且当它执⾏时，字段obj.foo 的值已经是 2 了。
   

  *
 ####  6.1 计算属性 computed 与 lazy    
   
  *
   
const bucket6 = new WeakMap();
const targetData6 = {text:"computed",foo:0,bar:1};
let activeEffect6 ;
const effectStack6 = [];
const proxyObj6 = new Proxy(targetData6,{
    get(target,key){
        track(target,key);
        return target[key]
    },
    set(target,key,newVal){
        target[key] = newVal;
        trigger(target,key);
    }
})
function effect6(fn,options={}){
    const effectFn = () => {
        cleanup6(effectFn);
        activeEffect6 = effectFn;
        effectStack6.push(effectFn);
        const res = fn();
        // 在当前副作⽤函数执⾏完毕后，将当前副作⽤函数弹出栈，并把 activeEffect 还原为之前的值
        effectStack6.pop();
        activeEffect6 = effectStack6[effectStack6.length-1];
        return res
    }
    // 将 options 挂载到 effectFn 上 add
    effectFn.options = options;
     // activeEffect.deps ⽤来存储所有与该副作⽤函数相关联的依赖集合
     effectFn.deps = [];
    if(!options.lazy){
        effectFn();
    }
    return effectFn
}
//那么 effectFn.deps 数组中的依赖集合是如何收集的呢？其实是在 track 函数中：
function track(target,key){
    if(!activeEffect6) return target[key];
    // 根据 target 从“桶”中取得 depsMap，它也是⼀个 Map 类型：key --> effects
    let depsMap = bucket6.get(target);
    if(!depsMap){
        bucket6.set(target, (depsMap = new Map()))
    }
    let deps = depsMap.get(key);
    if(!deps){
        depsMap.set(key,(deps = new Set()));// 如果 deps 不存在，同样新建⼀个 Set 并与 key 关联
    }
    // 最后将当前激活的副作⽤函数添加到“桶”⾥
    deps.add(activeEffect6);
    // deps 是一个与当前副作用函数存在联系的依赖集合
    // 将其添加到 activeEffect.deps 数组中
    activeEffect6.deps.push(deps)
}
function trigger(target,key){
    // 根据 target 从桶中取得 depsMap，它是 key --> effects
    const depsMap = bucket6.get(target);
    if(!depsMap) return;
    // 根据 key 取得所有副作⽤函数 effects
    const effects = depsMap.get(key);
    const effectToRun = new Set(effects);  
    // 如果 trigger 触发执⾏的副作⽤函数与当前正在执⾏的副作⽤函数相同，则不触发执⾏
    effects && effects.forEach(effectFn => {
        if(effectFn !== activeEffect6){  // 新增
            effectToRun.add(effectFn)
        }
    })  
    effectToRun.forEach(effectFn=> {
        // 如果⼀个副作⽤函数存在调度器，则调⽤该调度器，并将副作⽤函数作为参数传递
        if(effectFn.options.scheduler){
            effectFn.options.scheduler(effectFn); //add 
        }else{
            effectFn()
        }
    })   
    
}
function cleanup6(effectFn){
    for(let i=0;i < effectFn.deps.length;i++){
        // deps 是依赖集合
        const deps = effectFn.deps[i];
        // 将 effectFn 从依赖集合set中移除
        deps.delete(effectFn)
    }
    // 最后需要重置 effectFn.deps 数组
    effectFn.deps.length = 0
}
function computed(getter){
    // value ⽤来缓存上⼀次计算的值
    let value;
     // dirty 标志，⽤来标识是否需要重新计算值，为 true 则意味着“脏”，需要计算
    let dirty = true;
    const effectFn = effect6(getter,{
        lazy:true,
        // 添加调度器，在调度器中将 dirty 重置为 true
        scheduler(){
           if(!dirty){
             dirty = true;
             // 当计算属性依赖的响应式数据变化时，⼿动调⽤ trigger 函数触发响应
             trigger(obj,"value")
           }
        }
    })
    const obj = {
        get value(){
            if(dirty){
                value = effectFn();
                dirty = false
            }
            // 当读取 value 时，⼿动调⽤ track 函数进⾏追踪
            //track(obj, 'value')
            return value
        }
    }
    return obj
}

const sumRes = computed(() => proxyObj6.foo + proxyObj6.bar)
console.log(sumRes.value);
proxyObj6.foo++;
console.log(sumRes.value);
  
  *
   现在我们实现的计算属性只做到了懒计算，也就是说，只有当你真正读取 sumRes.value 的值时，
   它才会进⾏计算并得到值。但是还做不到对值进⾏缓存，即假如我们多次访问 sumRes.value 的值，
   会导致 effectFn 进⾏多次计算，即使 obj.foo 和 obj.bar 的值本⾝并没有变化：
  

    ------------------------------------------------------------------------------------
####   7.1 监听器 watch   
   
const bucket6 = new WeakMap();
const targetData6 = {text:"computed",foo:0,bar:1};
let activeEffect6 ;
const effectStack6 = [];
const proxyObj6 = new Proxy(targetData6,{
    get(target,key){
        track(target,key);
        return target[key]
    },
    set(target,key,newVal){
        target[key] = newVal;
        trigger(target,key);
    }
})
function effect6(fn,options={}){
    const effectFn = () => {
        cleanup6(effectFn);
        activeEffect6 = effectFn;
        effectStack6.push(effectFn);
        const res = fn();
        // 在当前副作⽤函数执⾏完毕后，将当前副作⽤函数弹出栈，并把 activeEffect 还原为之前的值
        effectStack6.pop();
        activeEffect6 = effectStack6[effectStack6.length-1];
        return res
    }
    // 将 options 挂载到 effectFn 上 add
    effectFn.options = options;
     // activeEffect.deps ⽤来存储所有与该副作⽤函数相关联的依赖集合
     effectFn.deps = [];
    if(!options.lazy){
        effectFn();
    }
    return effectFn
}
//那么 effectFn.deps 数组中的依赖集合是如何收集的呢？其实是在 track 函数中：
function track(target,key){
    if(!activeEffect6) return target[key];
    // 根据 target 从“桶”中取得 depsMap，它也是⼀个 Map 类型：key --> effects
    let depsMap = bucket6.get(target);
    if(!depsMap){
        bucket6.set(target, (depsMap = new Map()))
    }
    let deps = depsMap.get(key);
    if(!deps){
        depsMap.set(key,(deps = new Set()));// 如果 deps 不存在，同样新建⼀个 Set 并与 key 关联
    }
    // 最后将当前激活的副作⽤函数添加到“桶”⾥
    deps.add(activeEffect6);
    // deps 是一个与当前副作用函数存在联系的依赖集合
    // 将其添加到 activeEffect.deps 数组中
    activeEffect6.deps.push(deps)
}
function trigger(target,key){
    // 根据 target 从桶中取得 depsMap，它是 key --> effects
    const depsMap = bucket6.get(target);
    if(!depsMap) return;
    // 根据 key 取得所有副作⽤函数 effects
    const effects = depsMap.get(key);
    const effectToRun = new Set(effects);  
    // 如果 trigger 触发执⾏的副作⽤函数与当前正在执⾏的副作⽤函数相同，则不触发执⾏
    effects && effects.forEach(effectFn => {
        if(effectFn !== activeEffect6){  // 新增
            effectToRun.add(effectFn)
        }
    })  
    effectToRun.forEach(effectFn=> {
        // 如果⼀个副作⽤函数存在调度器，则调⽤该调度器，并将副作⽤函数作为参数传递
        if(effectFn.options.scheduler){
            effectFn.options.scheduler(effectFn); //add 
        }else{
            effectFn()
        }
    })   
    
}
function cleanup6(effectFn){
    for(let i=0;i < effectFn.deps.length;i++){
        // deps 是依赖集合
        const deps = effectFn.deps[i];
        // 将 effectFn 从依赖集合set中移除
        deps.delete(effectFn)
    }
    // 最后需要重置 effectFn.deps 数组
    effectFn.deps.length = 0
}
function computed(getter){
    // value ⽤来缓存上⼀次计算的值
    let value;
     // dirty 标志，⽤来标识是否需要重新计算值，为 true 则意味着“脏”，需要计算
    let dirty = true;
    const effectFn = effect6(getter,{
        lazy:true,
        // 添加调度器，在调度器中将 dirty 重置为 true
        scheduler(){
           if(!dirty){
             dirty = true;
             // 当计算属性依赖的响应式数据变化时，⼿动调⽤ trigger 函数触发响应
             trigger(obj,"value")
           }
        }
    })
    const obj = {
        get value(){
            if(dirty){
                value = effectFn();
                dirty = false
            }
            // 当读取 value 时，⼿动调⽤ track 函数进⾏追踪
            //track(obj, 'value')
            return value
        }
    }
    return obj
}
//watch 函数除了可以观测响应式数据，还可以接收⼀个 getter 函数：
// 如何获取新值和旧值
function watch(source,cb, options = {}){
    let getter 
    // 如果 source 是函数，说明⽤户传递的是 getter，所以直接把 source 赋值给 getter
//     ⾸先判断 source 的类型，如果是函数类型，说明⽤户直接传递了 getter 函数，这时直接使⽤⽤
// 户的 getter 函数；如果不是函数类型，那么保留之前的做法，即调⽤ traverse 函数递归地读取。这
// 样就实现了⾃定义 getter 的功能，同时使得 watch 函数更加强⼤.
    if(typeof source === "function"){
        getter = source
    }else{
        getter = () => traverse(source)
    }
    // 定义新值和旧值
    let oldValue,newValue

    // 提取 scheduler 调度函数为一个独立的 job 函数
    const job = () => {
        newValue = effectFn();
        cb(newValue,oldValue);
        oldValue = newValue;
    }

    const effectFn =  effect6(
        () => getter(),
        {
            lazy:true,
            scheduler:() => {
                if(options.flush === "post"){
                    const p = Promise.resolve();
                    p.then(job);
                }else{
                    job()
                }
            },
        }
    )
    //除了指定回调函数为⽴即执⾏之外，还可以通过其他选项参数来指定回调函数的执⾏时机
    if(options.immediate){
        // 当 immediate 为 true 时⽴即执⾏ job，从⽽触发回调执⾏
        job()
    }else{
        // ⼿动调⽤副作⽤函数，拿到的值就是旧值
        oldValue = effectFn();
    }

}
function traverse(value,seen = new Set()){
    // 如果要读取的数据是原始值，或者已经被读取过了，那么什么都不做
    if(typeof value !== 'object' || value === null || seen.has(value)) return 
    //将数据添加到 seen 中，代表遍历地读取过了，避免循环引起的死循环
    seen.add(value);
    for(const k in value){
        traverse(value[k],seen)
    }
    return value
}
watch(proxyObj6,() => {
    console.log('数据变化了');
})
proxyObj6.foo++;
  








