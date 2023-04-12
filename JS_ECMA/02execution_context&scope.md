#### 一，JavaScript深入之执行上下文栈
 JS 引擎会在执行代码前做准备工作，准备工作是指执行到可执行代码时，会做的一些事情，叫执行上下文 execution context stack
 可执行代码:局代码、函数代码、eval代码；

 ##### 执行上下文栈
JavaScript 引擎创建了执行上下文栈（Execution context stack，ECS）来管理执行上下文
为了模拟执行上下文栈的行为，让我们定义执行上下文栈是一个数组：
    ECStack = []; 
   *  当 JS 解释器执行代码时，最开始是全局代码，初始化即向执行上下文栈压入全局执行上下文（globalContext），程序结束时被清空；
        ECStack = [ globalContext ]
   *  当执行一个函数时，就会创建一个执行上下文，并压入执行上下文栈，执行完毕，从栈中弹出；
      ```
          function fun3() {
              console.log('fun3')
          }
          function fun2() {
              fun3();
          }
          function fun1() {
              fun2();
          }
          fun1();
      ```
   
      // 伪代码 分析过程
      ```
        // fun1()
        ECStack.push(<fun1> functionContext);

        // fun1中竟然调用了fun2，还要创建fun2的执行上下文
        ECStack.push(<fun2> functionContext);

        // 擦，fun2还调用了fun3！
        ECStack.push(<fun3> functionContext);

        // fun3执行完毕
        ECStack.pop();

        // fun2执行完毕
        ECStack.pop();

        // fun1执行完毕
        ECStack.pop();

        // javascript接着执行下面的代码，但是ECStack底层永远有个globalContext
        ```
  * 每个执行上下文，都有三个重要属性：
          1.变量对象(Variable object，VO)
          2.作用域链(Scope chain)
          3.this
          
#### 二，JavaScript深入之变量对象
变量对象：是与执行上下文相关的数据作用域，存储了在上下文中定义的变量和函数声明。
        1. 全局上下文的变量对象，就是全局对象
        2. 函数上下文：用活动对象(activation object, AO)来表示变量对象。
执行过程：
        执行上下文的代码会分成两个阶段进行处理：分析和执行
        1. 分析：进入执行上下文
        进入执行上下文，代码还没执行，但是会分析，定义变量，函数声明等，此时的变量对象（即活动对象）包括
            1. 函数的所有形参 (如果是函数上下文)
            2. 函数声明
            3. 变量声明
          
            function foo(a) {
                var b = 2;
                function c() {}
                var d = function() {};
                b = 3;
                e = 3;
            }
            foo(1);
          
       
        AO = {
            arguments:{0:1,length:1},
            a:1,
            b:undefined,
            c:reference to function c(){},
            d:undefined
         }
          
注意点：
   若函数中的 "e" 没有通过 var 关键字声明，不会被存放在 AO 中。寻找时会去全局找。
   在进入执行上下文时，首先会处理函数声明，其次会处理变量声明，如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性
        
  2.代码执行:在代码执行阶段，会顺序执行代码，根据代码，修改变量对象的值.
    此时的 
              AO = {
                    arguments: {
                        0: 1,
                        length: 1
                    },
                    a: 1,
                    b: 3,
                    c: reference to function c(){},
                    d: reference to FunctionExpression "d"
                }
    
#### 三，JavaScript深入之作用域链 
 当查找变量的时候，会先从当前上下文的变量对象中查找，如果没有找到，就会从父级(词法层面上的父级)执行上下文的变量对象中查找，一直找到全局上下文的变量对象，也就是全局对象。
       这样由多个执行上下文的变量对象构成的链表就叫做作用域链。
        
 以一个函数的创建和激活两个时期来讲解作用域链是如何创建和变化的：
       1.函数创建
           首先函数的作用域在函数定义的时候就决定了，
           函数有一个内部属性 [[scope]]，当函数创建的时候，就会保存所有父变量对象到其中，
           可以理解 [[scope]] 就是所有父变量对象的层级链，但是注意：[[scope]] 并不代表完整的作用域链！
           eg:
            ```    
            function foo() {
                function bar() {
                    ...
                }
            }
            ```
            函数创建时，各自的[[ scope ]]为：
            foo.[[scope]] = [ globalContext.VO];
            bar.[[scope]] = [
                fooContext.AO,
                globalContext.VO
            ]
         
        2.函数激活
        函数激活，进入函数上下文，创建 VO/AO 后，就会将活动对象添加到作用域链的前端
        Scope = [AO].contat([[Scope]])
        作用域链创建完毕

   
  根据变量对象和执行上下文栈，捋一捋函数执行上下文中作用域链和变量对象的创建过程
  ```
    var scope = 'global scope';
    function checkscope(){
        var scope2 = 'local scope';
        return scope2
    }
    checkscope()
  ```
   执行过程：
       1. checkscope 函数被创建，保存作用域链到内部属性 [[scope]]
           ```
                checkscope.[[scope]] = [globalContext.VO]
           ```
   
       2. 执行 checkscope 函数，创建 checkscope 函数执行上下文，checkscope 函数执行上下文被压入执行上下文栈；
          ECStack = [ checkscopeContext,  globalContext ]
   
       3. checkscope 函数并不立刻执行，开始做准备工作，
          第一步：复制函数[[scope]]属性创建作用域链
            checkscopeContext = {
                Scope: checkscope.[[scope]],
             }
          第二步：用 arguments 创建活动对象，随后初始化活动对象，加入形参、函数声明、变量声明.
            ```
                checkscopeContext = {
                    AO: {
                        arguments: {
                            length: 0
                        },
                        scope2: undefined
                    }，
                    Scope: checkscope.[[scope]],
                }
            ```    
          第三步：将活动对象压入 checkscope 作用域链顶端
            ```
                checkscopeContext = {
                    AO: {
                        arguments: {
                            length: 0
                        },
                        scope2: undefined
                    },
                    Scope: [AO, [[Scope]]]
                }
            ```
   
       4.准备工作做完，开始执行函数，随着函数的执行，修改 AO 的属性值
        ```
            checkscopeContext = {
                AO: {
                    arguments: {
                        length: 0
                    },
                    scope2: 'local scope'
                },
                Scope: [AO, [[Scope]]]
            }
        ```
       5.查找到 scope2 的值，返回后函数执行完毕，函数上下文从执行上下文栈中弹出
        ```
            ECStack = [
                globalContext
            ];
        ```
      
 #### 四，JavaScript深入之从ECMAScript规范解读this
       ECMAScript 的类型分为语言类型和规范类型。
       ECMAScript 语言类型是开发者直接使用 ECMAScript 可以操作的。其实就是我们常说的Undefined, Null, Boolean, String, Number, 和 Object。
       规范类型相当于 meta-values，是用来用算法描述 ECMAScript 语言结构和 ECMAScript 语言类型的。规范类型包括：Reference, List, Completion, Property Descriptor, Property Identifier, Lexical Environment, 和 Environment Record。
       
       Reference 跟 this 的指向有密切关联
       Reference 类型就是用来解释诸如 delete、typeof 以及赋值等操作行为的。
       由三个组成部分，分别是：
              base value
              referenced name
              strict reference
        base value 就是属性所在的对象或者就是 EnvironmentRecord，它的值只可能是 undefined, an Object, a Boolean, a String, a Number, or an environment record 其中的一种。                   
        referenced name 就是属性的名称。
        举例说明：
            var str = 1;

            // 对应的Reference是：
            var strReference = {
                base: EnvironmentRecord,
                name: 'str',
                strict: false
            };

            var foo = {
                bar: function () {
                    return this;
                }
            }; 
            foo.bar(); // foo

            // bar对应的Reference是：
            var BarReference = {
                base: foo,
                propertyName: 'bar',
                strict: false
            };
        规范中还提供了获取 Reference 组成部分的方法，比如 GetBase 和 IsPropertyReference。
        1.GetBase
        GetBase(V). Returns the base value component of the reference V.
        返回 reference 的 base value。

        2.IsPropertyReference
        IsPropertyReference(V). Returns true if either the base value is an object or HasPrimitiveBase(V) is true; otherwise returns false.
        简单的理解：如果 base value 是一个对象，就返回true。

        GetValue:从 Reference 类型获取对应值的方法
        var foo = 1;
        var fooReference = {
            base: EnvironmentRecord,
            name: 'foo',
            strict: false
        };
            
        GetValue(fooReference) // 1;

        调用 GetValue，返回的将是具体的值，而不再是一个 Reference

        
        如何确定 this 值？
        1.计算 MemberExpression 的结果赋值给 ref

        2.判断 ref 是不是一个 Reference 类型

            2.1 如果 ref 是 Reference，并且 IsPropertyReference(ref) 是 true, 那么 this 的值为 GetBase(ref)

            2.2 如果 ref 是 Reference，并且 base value 值是 Environment Record, 那么this的值为 ImplicitThisValue(ref)

            2.3 如果 ref 不是 Reference，那么 this 的值为 undefined

        具体说明：  
        1.计算 MemberExpression 的结果赋值给 ref
            什么是 MemberExpression？看规范 11.2 Left-Hand-Side Expressions：
            MemberExpression :
                PrimaryExpression // 原始表达式 可以参见《JavaScript权威指南第四章》
                FunctionExpression // 函数定义表达式
                MemberExpression [ Expression ] // 属性访问表达式
                MemberExpression . IdentifierName // 属性访问表达式
                new MemberExpression Arguments // 对象创建表达式
            举例：
            ```
                function foo() {
                    console.log(this)
                }
                foo(); // MemberExpression 是 foo

                function foo() {
                    return function() {
                        console.log(this)
                    }
                }
                foo()(); // MemberExpression 是 foo()

                var foo = {
                    bar: function () {
                        return this;
                    }
                }
                foo.bar(); // MemberExpression 是 foo.bar

                //所以简单理解 MemberExpression 其实就是()左边的部分。

               
            ```
        2.判断 ref 是不是一个 Reference 类型。
            关键就在于看规范是如何处理各种 MemberExpression，返回的结果是不是一个Reference类型。
            
            eg:
            var value = 1;

            var foo = {
                value: 2,
                bar: function () {
                    return this.value;
                }
            }

            //示例1
            console.log(foo.bar());
            //示例2
            console.log((foo.bar)());
            //示例3
            console.log((foo.bar = foo.bar)());
            //示例4
            console.log((false || foo.bar)());
            //示例5
            console.log((foo.bar, foo.bar)());

            分析：
            1.foo.bar()
            var Reference = {
                    base: foo,
                    name: 'bar',
                    strict: false
            };
            该值是 Reference 类型，那么 IsPropertyReference(ref) 的结果是多少呢？
            前面我们已经铺垫了 IsPropertyReference 方法，如果 base value 是一个对象，结果返回 true。
            base value 为 foo，是一个对象，所以 IsPropertyReference(ref) 结果为 true。
            GetBase 也已经铺垫了，获得 base value 值，这个例子中就是foo，所以 this 的值就是 foo ，示例1的结果就是 2！

            2.(foo.bar)()
            实际上 () 并没有对 MemberExpression 进行计算，所以其实跟示例 1 的结果是一样的。

            3.(foo.bar = foo.bar)()
            因为使用了 GetValue，所以返回的值不是 Reference 类型，this 为 undefined，非严格模式下，this 的值为 undefined 的时候，其值会被隐式转换为全局对象。

            4.(false || foo.bar)()
            因为使用了 GetValue，所以返回的不是 Reference 类型，this 为 undefined

            5.(foo.bar, foo.bar)()
            因为使用了 GetValue，所以返回的不是 Reference 类型，this 为 undefined

            另外：
            function foo() {
                console.log(this)
            }

            foo(); 
            MemberExpression 是 foo，解析标识符，查看规范 10.3.1 Identifier Resolution，会返回一个 Reference 类型的值：

            var fooReference = {
                base: EnvironmentRecord,
                name: 'foo',
                strict: false
            };
            接下来进行判断：

            2.1 如果 ref 是 Reference，并且 IsPropertyReference(ref) 是 true, 那么 this 的值为 GetBase(ref)

            因为 base value 是 EnvironmentRecord，并不是一个 Object 类型，还记得前面讲过的 base value 的取值可能吗？ 只可能是 undefined, an Object, a Boolean, a String, a Number, 和 an environment record 中的一种。

            IsPropertyReference(ref) 的结果为 false，进入下个判断：

            2.2 如果 ref 是 Reference，并且 base value 值是 Environment Record, 那么this的值为 ImplicitThisValue(ref)

            base value 正是 Environment Record，所以会调用 ImplicitThisValue(ref)

            查看规范 10.2.1.1.6，ImplicitThisValue 方法的介绍：该函数始终返回 undefined。

            所以最后 this 的值就是 undefined。


            从场景角度看
            this一般有几种调用场景
            var obj = {a: 1, b: function(){console.log(this);}}
            1、作为对象调用时，指向该对象 obj.b(); // 指向obj
            2、作为函数调用, var b = obj.b; b(); // 指向全局window
            3、作为构造函数调用 var b = new Fun(); // this指向当前实例对象
            4、作为call与apply调用 obj.b.apply(object, []); // this指向当前的object
  
  
   #### 五，JavaScript深入之执行上下文 
        执行上下文的具体处理过程：
        分析第一段代码：
            ```
                var scope = "global scope";
                function checkscope(){
                    var scope = "local scope";
                    function f(){
                        return scope;
                    }
                    return f();
                }
                checkscope();
            ```
            1.执行全局代码，创建全局上下文，全局上下文被压入执行上下文栈
              ``` 
                ECStack = [ globalContext ]
              ```
            2.全局上下文初始化，
              ```
                globalContext = {
                    VO: [global],
                    Scope: [globalContext.VO],
                    this: globalContext.VO
                }
              ```
            3.初始化的同时，checkscope 函数被创建，保存作用域链到函数的内部属性[[scope]].
            ```
                checkscope.[[scope]] = [
                    globalContext.VO
                ]; 
            ```
            4.执行 checkscope 函数，创建 checkscope 函数执行上下文，checkscope 函数执行上下文被压入执行上下文栈
             ```
                ECStack = [
                    checkscopeContext,
                    globalContext
                ];
             ```
            5.checkscope 函数执行上下文初始化：
                1,复制函数 [[scope]] 属性创建作用域链，
                2,用 arguments 创建活动对象，
                3,初始化活动对象，即加入形参、函数声明、变量声明，
                4,将活动对象压入 checkscope 作用域链顶端。
            同时 f 函数被创建，保存作用域链到 f 函数的内部属性[[scope]]
                checkscopeContext = {
                    AO: {
                        arguments: {
                            length: 0
                        },
                        scope: undefined,
                        f: reference to function f(){}
                    },
                    Scope: [AO, globalContext.VO],
                    this: undefined
                }
   
            6.执行 f 函数，创建 f 函数执行上下文，f 函数执行上下文被压入执行上下文栈
                ECStack = [
                    fContext,
                    checkscopeContext,
                    globalContext
                ];
            7.f 函数执行上下文初始化, 以下跟第 5 步相同：
                1,复制函数 [[scope]] 属性创建作用域链
                2,用 arguments 创建活动对象
                3,初始化活动对象，即加入形参、函数声明、变量声明
                4,将活动对象压入 f 作用域链顶端
            ```
                fContext = {
                    AO: {
                        arguments: {
                            length: 0
                        }
                    },
                    Scope: [AO, checkscopeContext.AO, globalContext.VO],
                    this: undefined
                }
            ```
            8.f 函数执行完毕，f 函数上下文从执行上下文栈中弹出
            
            9.checkscope 函数执行完毕，checkscope 执行上下文从执行上下文栈中弹出
              ECStack = [
                globalContext
              ];
   
   
#### 六，JavaScript深入之闭包
MDN 对闭包的定义为：
    闭包是指那些能够访问自由变量的函数。
    那什么是自由变量呢？
    自由变量是指在函数中使用的，但既不是函数参数也不是函数的局部变量的变量。
    由此，我们可以看出闭包共有两部分组成：
    闭包 = 函数 + 函数能够访问的自由变量
   
    ECMAScript中，闭包指的是：
        从理论角度：所有的函数。因为它们都在创建的时候就将上层上下文的数据保存起来了。哪怕是简单的全局变量也是如此，因为函数中访问全局变量就相当于是在访问自由变量，这个时候使用最外层的作用域。
        从实践角度：以下函数才算是闭包：
                1,即使创建它的上下文已经销毁，它仍然存在（比如，内部函数从父函数中返回）
                2,在代码中引用了自由变量
   
     看看实际的闭包：
       ``` 
        var scope = "global scope";
        function checkscope(){
            var scope = "local scope";
            function f(){
                return scope;
            }
            return f;
        }

        var foo = checkscope();
        foo();
       ```
        执行过程：
            1,进入全局代码，创建全局执行上下文，全局执行上下文压入执行上下文栈
            2,全局执行上下文初始化
            3,执行 checkscope 函数，创建 checkscope 函数执行上下文，checkscope 执行上下文被压入执行上下文栈
            4,checkscope 执行上下文初始化，创建变量对象、作用域链、this等
            5,checkscope 函数执行完毕，checkscope 执行上下文从执行上下文栈中弹出
            6,执行 f 函数，创建 f 函数执行上下文，f 执行上下文被压入执行上下文栈
            7,f 执行上下文初始化，创建变量对象、作用域链、this等
            8,f 函数执行完毕，f 函数上下文从执行上下文栈中弹出

        当 f 函数执行的时候，checkscope 函数上下文已经被销毁了啊(即从执行上下文栈中被弹出)，怎么还会读取到 checkscope 作用域下的 scope 值呢？
        通过具体的执行过程后，我们知道 f 执行上下文维护了一个作用域链：
        fContext = {
            Scope: [AO, checkscopeContext.AO, globalContext.VO],
        }
        
        因为这个作用域链，f 函数依然可以读取到 checkscopeContext.AO 的值，说明当 f 函数引用了 checkscopeContext.AO 中的值的时候，即使 checkscopeContext 被销毁了，但是 JavaScript 依然会让 checkscopeContext.AO 活在内存中，f 函数依然可以通过 f 函数的作用域链找到它，正是因为 JavaScript 做到了这一点，从而实现了闭包这个概念。
 
        eg:
        ```
            var data = [];
            for (var i = 0; i < 3; i++) {
                data[i] = function () {
                    console.log(i);
                };
            }
            data[0]();
            data[1]();
            data[2]();
        ```
        当执行 data[0] 函数的时候，data[0] 函数的作用域链为：
        data[0]Context = {
            Scope: [AO, globalContext.VO]
        }
        data[0]Context 的 AO 并没有 i 值，所以会从 globalContext.VO 中查找，i 为 3，所以打印的结果就是 3。

        改成闭包
        ```
        var data = [];

        for (var i = 0; i < 3; i++) {
            data[i] = (function (i) {
                    return function(){
                        console.log(i);
                    }
            })(i);
        }

        data[0]();
        data[1]();
        data[2]();
        ```
        当执行 data[0] 函数的时候，data[0] 函数的作用域链发生了改变：
        data[0]Context = {
            Scope: [AO, 匿名函数Context.AO globalContext.VO]
        }
        匿名函数执行上下文的AO为：
        匿名函数Context = {
            AO: {
                arguments: {
                    0: 0,
                    length: 1
                },
                i: 0
            }
        }
        data[0]Context 的 AO 并没有 i 值，所以会沿着作用域链从匿名函数 Context.AO 中查找，这时候就会找 i 为 0，找到了就不会往 globalContext.VO 中查找了，即使 globalContext.VO 也有 i 的值(值为3)，所以打印的结果就是0。
        





         /

            


