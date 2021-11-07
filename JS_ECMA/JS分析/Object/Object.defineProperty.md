#### 数据绑定和劫持

`Object.defineProperty()`,监控对象属性，变化时操作其他内容；

```js
let data = {
    name :"hellp",
    address:"罗湖海联大厦",
    options:[
        {name : "forms"},
        {name : "atasuo"}
    ]
}

function observal(obj){
    if(!obj || typeof obj !== 'object'){
        return 
    }
    Object.keys(obj).forEach( (item) => {
        defineReactive(obj,item,obj[item])
    });
}

function defineReactive(data,key,val){
    observal(val);
    Object.defineProperty(data,key,{
        enumerable : true,
        configurable : true,
        get(){
            return val
        },
        set(newVal){
            console.log(`${val} 的值发生了改变，变成了${newVal}`)
            val = newVal
            //值发生改变，更新界面
        }
    })
}
observal(data);
data.name = "hello world";
data.options[0].name = 'shenzhen forms';
```

在此基础上，将`data`对象的属性添加到`vm`对象上,实现 `data` 和 `vm` 对象的绑定，为每一个添加到`vm`上的属性，都指定一`getter/setter`，在`getter/setter`内部去操作（读/写）`data`中对应的属性。

```js
let data = {
    name :"hellp",
    address:"罗湖海联大厦",
    options:[
        {name : "forms"},
        {name : "atasuo"}
    ]
}
const vm = {};

function observal(targetVm , obj){
    if(!obj || typeof obj !== 'object'){
        return 
    }
    Object.keys(obj).forEach( (item) => {
        defineReactive(targetVm,obj,item,obj[item]);
        
    });
}

function defineReactive(vm,data,key,val){
    observal(val);
    Object.defineProperty(vm,key,{
        enumerable : true,
        configurable : true,
        get(){
            return data[key]
        },
        set(newVal){
            console.log(`${val} 的值发生了改变，变成了${newVal}`)
            data[key] = newVal;
            //值发生改变，更新界面
            //...
        }
    })
}
observal(vm,data);

// vm.address = "nanshan"
console.log(vm);
console.log(data);

// data.name = "hello world";
// console.log(vm);
// console.log(data);

```

直通通过对象`vm = data;vm._data = data;`形成代理；

```js
let data = {
    name :"hellp",
    address:"罗湖海联大厦",
    options:[
        {name : "forms"},
        {name : "atasuo"}
    ],
    hosdd:{
        dkjf:{ddd:"dfsdf"}
    }
}

let vm = {};

function observal(obj){
    if(!obj || typeof obj !== 'object'){
        return 
    }
    Object.keys(obj).forEach( (item) => {
        defineReactive(obj,item,obj[item]);
        
    });
}

function defineReactive(data,key,val){
    observal(val);
    Object.defineProperty(data,key,{
        enumerable : true,
        configurable : true,
        get(){
            return val
        },
        set(newVal){
            console.log(`${val} 的值发生了改变，变成了${newVal}`)
            val = newVal;
            //值发生改变，更新界面
        }
    })
}
observal(data);
vm = data;
vm._data = data;

console.log(vm);
console.log(data);

data.name = "hello world";
console.log(vm);
console.log(data);

```

