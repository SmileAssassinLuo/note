#### Set

Set 无重复元素的列表，通常用于检测某一个值在集合中是否存在。

#### Map

 Map经常被用于缓存频繁取用的数据     



#### ES5 模拟Set，Map

es5中用对象属性来模拟set和 map，

```javascript
    var set = Object.create(null);
    set.foo = true;
    if(set.foo){
        console.log('hello. is exit')
    }

    var map = Object.create(null);
    map.foo = 'bar';
    var value = map.foo;
    console.log(value) //bar
```

弊端：如果需要对象的属性名都是字符串类型，且唯一；难以保证

```javascript
var  map = Object.create(null);
map[5] = foo;
console.log(map['5']); // foo

//map[5] 和 map['5'] 的值相同

var key1 = {},key2 = {};
map[key1] = 'bar';
console.log(map[key2]) ; // bar

//map[key1]和map[key2] 会被自动转换成 [Object Object];
```



#### Set 使用

```javascript
var set = new Set();
set.add(5);
set.add('5');
console.log(set.size); //2
set.add(0);
set.add(+0);
set.add(-0);
console.log(set.size); //3  0,+0,-0 被认为是相等的，只会被添加一次；
set.add("0");
set.add("-0");
console.log(set.size); //5
set.add({});
set.add({});
console.log(set.size); //7  空 {} 也是独立的

//亦可接受可迭代对象作为参数,数组，Set，Map等，重复值不会被添加;
var set2 = new Set([1,2,4,5,5,5,5]);
console.log(set2.size); //4

//其他api
console.log( set2.has(1)); // true
set2.delete(2); //删除
console.log(set2); //Set(3) {1, 4, 5}
set2.clear();   //清空
console.log(set2.size); //0

//forEach:回调函数的三个参数，值，键（索引），本遍历的本身
let set3 = new Set(['a','b']);
set3.forEach((value, key, ownerSet) =>{
    console.log(`${key}---${value}`); //a--a b--b
    console.log(ownerSet === set3) ;  //true true
});
//forEach第二个参数传入this，让其指定到processor对象，或者使用箭头函数，不用绑定this；
let set4 = new Set([1,2]);
let processor = {
    output(value){
        console.log(value);
    },
    process(dataSet){
        // dataSet.forEach(function(value){
        //     this.output(value);
        // },this)
        dataSet.forEach(value => this.output(value));
    }
}
processor.process(set4);


//Set集合更适合来跟踪多个值，而且又可以通过forEach()方法操作集合中的每一个值，不可通过索引直接访问集合元素，
//有需要，转成数组
let set5 = new Set(['1','2']);
let arr = [...set5];
console.log(arr[0]);

//已有基本类型元素数组，返回去重后的数组
let arr2 = [1,2,33,4,1,4,3,4,2,5];
function eliminateDuplicates(items){
    return [...new Set(items)]
}
console.log(eliminateDuplicates(arr2)); //(6) [1, 2, 33, 4, 3, 5]

//值得注意的地方
let set6 = new Set();
let key = {};
set6.add(key);
key = null ;
console.log(set6.size); //1
console.log([...set6][0]) // {}


// WeakMap
/*
   当 JS 代码在网页中运行，同时你想保持与 DOM 元素的联系，在该元素可能被其他脚本移除的情况下，
   你应当不希望自己的代码保留对该 DOM 元素的最后一个引用（这种情况被称为内存泄漏）.

    因此ES6的 WeakSet ，该类型只允许存储对象弱引用，而不能存储基本类型的值。对象的弱引用在它自己成为该对象的唯一引用时，
    不会阻止垃圾回收。
*/   
let weakSet1 = new WeakSet();
let key1 = {};
weakSet1.add(key1);
console.log( weakSet1.has(key1)); //true
key1 = null ;
console.log( weakSet1.has(key1)); //false
//当此代码被执行后， Weak Set 中的  key1  引用就不能再访问了

//无法证实，但 JS 引擎确实是这样做了。 
```

#### WeakSet

1. 对于  WeakSet  的实例，若调用  add()  方法时传入了非对象的参数，就会抛出错误（has()  或  delete()  则会在传入了非对象的参数时返回  false  ）；
2. Weak Set 不可迭代，因此不能被用在  for-of  循环中；
3. Weak Set 无法暴露出任何迭代器（例如  keys()  与  values()  方法），因此没有任何编程手段可用于判断 Weak Set 的内容；
4. Weak Set 没有  forEach()  方法；
5. Weak Set 没有  size  属性.



#### Map 使用

```javascript
let map = new Map();
map.set("name", "Nicholas");
map.set("age", 25);
console.log(map.size); // 2
console.log(map.has("name")); // true
console.log(map.get("name")); // "Nicholas"
console.log(map.has("age")); // true
console.log(map.get("age")); // 25
map.delete("name");
console.log(map.has("name")); // false
console.log(map.get("name")); // undefined
console.log(map.size); // 1
map.clear();
console.log(map.has("name")); // false
console.log(map.get("name")); // undefined
console.log(map.has("age")); // false
console.log(map.get("age")); // undefined
console.log(map.size); // 0


//Map的初始化,该数组中的每一项也必须是数组，内部数组的首个项会作为键，第二项则为对应值
let map2  = new Map ([['name','Isaiah'],['age',18]])
console.log(map2.get("name"));
console.log(map2.get("age"));

map2.forEach((value,key,owner) => {
    console.log(value,key);
    console.log(map2 === owner);
})


/*
      Weak Map 对 Map 而言，就像 Weak Set 对 Set 一样： Weak 版本都是存储对象弱引用的方式。在 Weak Map 中，所有的键都必须是对象
     （尝试使用非对象的键会抛出错误），而且这些对象都是弱引用，不会干扰垃圾回收.
      api:set(),has(),get(),delete()
    */
let weakMap1 = new WeakMap();
let mapKey = Object.create(null);
weakMap1.set(mapKey,'123');
console.log(weakMap1.has(mapKey));// true
console.log(weakMap1.get(mapKey));// 123

weakMap1.delete(mapKey);
console.log(weakMap1.has(mapKey));// false
console.log(weakMap1.get(mapKey)); //undefined

// 私有属性
/*
    * Weak Map 的主要用途是关联数据与 DOM 元素，但仍然还存在许多可能的用法;
    */
//Weak Map 的一个实际应用就是在对象实例中存储私有数据:
//在 ES5 中能够创建几乎真正私有的数据，只要在创建对象时使用类似下面的模式：
var Person = (function() {
    var privateData = {},
        privateId = 0;
    function Person(name) {
        Object.defineProperty(this, "_id", { value: privateId++ });
        privateData[this._id] = {
            name: name
        };
    }
    Person.prototype.getName = function() {
        return privateData[this._id].name;
    };
    return Person;
}());

//问题在于  privateData  中的数据永不会消失，因为在对象实例被销毁时没有任何方法可以获知该数据，  privateData  对象就将永远包含多余的数据。这个问题现在可以换用 Weak Map 来解决了，如下：
let Person2 = (function() {
    let privateData = new WeakMap();
    function Person2(name) {
        privateData.set(this, { name: name });
    }
    Person2.prototype.getName = function() {
        return privateData.get(this).name;
    };
    return Person2;
}());
```





