

#### Redux 状态容器和数据管理

##### Redux 三大原则

* 单一数据源.
* 状态不可变:修改数据前后数据源非同一个.
* 纯函数修改状态.



##### 不使用Redux

以 **TodoList** 为例，在不适用redux时，如何编写增删等方法。

共4个组件，父子组件关系如下 :

```
TodoList ---> Control

		 ---> Todos ---> TodoItem
```

编码：

```javascript
import React ,{useCallback, useState,useRef, useEffect} from 'react';
import './App.css';

//todolist 选项输入组件
let idSeq = Date.now();
function Control(props){
  const {addTodo} = props;
  const inputRef = useRef();
  const onSubmit = (e) => {
      e.preventDefault();
      console.log(document.forms);
      const newText = inputRef.current.value.trim();
      if(newText.length === 0) {
        return ;
      }
      addTodo({
        id:++idSeq,
        text:newText,
        compolete:false
      })
      inputRef.current.value = '';
  }
  return (
    <div className = 'control'> 
        <h3>todos</h3>
        <form onSubmit = {onSubmit} name = 'formTest'> 
            <input 
                type = 'text' 
                placeholder = 'what need to done?' 
                className = 'new-todo'    
                ref = {inputRef}
            />
            <button type ='reset'>reset</button>
        </form>
    </div>
  )
}
//裂表渲染组件
function Todos(props){
  const {todos,toggleTodo,removeTodo} = props;
  
  return (
      <ul>
        {
          todos.map(todoItem => {
            return <TodoItem
                      key = {todoItem.id}
                      todoItem = {todoItem}
                      toggleTodo = {toggleTodo}
                      removeTodo = {removeTodo}
                    />
          })
        }
      </ul>
  )
}
//列表渲染子组件
function TodoItem (props){
  const {
      todoItem:{
          id,
          text,
          compolete
        }, 
        toggleTodo,
        removeTodo
  } = props ;
  const onChange = () => {
    toggleTodo(id)
  }
  const onRemove = () => {
      removeTodo(id)
  }
  return (
      <li className = 'todo-item'>
          <input 
              type = 'checkbox'
              onChange = {onChange}
              checked = {compolete}
          />
          <label className = {compolete?'compolete':''}>
              {text}
          </label>
          <button onClick = {onRemove}>&#xd7; </button>
      </li>
  )
}

//TodoList 根组件
let _LS_KEY_TODOS_ = '$_LS_KEY_TODOS';
function TodoList() {
    const [todos,setTodos] = useState([]);
	//添加
    const addTodo = useCallback((todo) => {
        setTodos(todos => [...todos,todo] ); 
    },[])
    //删除
    const removeTodo = useCallback(( id ) => {
        setTodos(todos => todos.filter(todoItem =>{
          return todoItem.id !== id
        })
      )
    },[])
    //切换已完成方法
    const toggleTodo = useCallback((id) =>{
        setTodos(todos => todos.map(todoItem => {
          return todoItem.id === id?{
                ...todoItem,
                compolete:!todoItem.compolete
          }:todoItem
      }))
    },[])

    //进入组件获取localStorage下存储的数据;
    useEffect(() => {
        const localTodos = JSON.parse(localStorage.getItem(_LS_KEY_TODOS_) || "[]");
        setTodos(localTodos);
    },[])
    //todos改变时,修改localStorage下存储的数据;
    useEffect(() => {
      localStorage.setItem(_LS_KEY_TODOS_,JSON.stringify(todos))
      
    }, [todos])
  return (
    <div className="todo-list">
        <Control addTodo = {addTodo}/>
        <Todos removeTodo = {removeTodo} toggleTodo = {toggleTodo} todos = {todos}/>
    </div>
  );
}

export default TodoList;

```

```css
.todo-list {
  width: 550px;
  margin:130px auto;
  background: #fff;
  box-shadow: 0 2px 4px 0 rgba(0,0,0,0.2),0 25px 50px 0 rgba(0,0,0,0.1);
}
.control{
    width: 100%;
    font-size: 100px;
    text-align: center;
    margin:0;
    color:rgba(175.47.47.0.15);
}
.control form {
  display: flex;
}
.control .new-todo {
  padding: 16px 16px 16px 60px;
  border:0;
  outline:none;
  font-size: 24px;
  box-sizing: border-box;
  width: 100%;
  line-height: 1.4rm;
  box-shadow: inset 0-2px 1px rgba(0,0,0,0.3);
}
.todos {
  margin:0;
  padding: 0;
  list-style:  none;
}
.todo-item {
  margin:0;
  padding: 0;
  list-style: none;
  display: flex;
  align-items: center;
}
.todo-item input {
  display: block;
  width: 20px;
  height: 20px;
  margin:0 20px;
}
.todo-item label {
  flex:1;
  padding:15px,15px,15px,0;
  line-height: 1.2;
  display: block;
}
.todo-item label.compolete {
  text-decoration: line-through;
  color:red;
}
.todo-item button {
  border:0;
  outline: 0;
  display: block;
  width: 40px;
  text-align: center;
  font-size:30px;
  color:#cc9a9a;
}
```



##### 改造版本

对于多个数据，新建多个reducers ，每个数据对应自己的方法，由于dispatch派发新的数据更新通过唯一reducer方法，故多个reducers方法需整合为一个。

* redux是的诞生是为了给 React 应用提供「可预测化的状态管理」机制。

* Redux会将整个应用状态(其实也就是数据)存储到到一个地方，称为store

* 这个store里面保存一棵状态树(state tree)

* 组件改变state的唯一方法是通过调用store的dispatch方法，触发一个action，这个action被对应的reducer处理，于是state完成更新

* 组件可以派发(dispatch)行为(action)给store,而不是直接通知其它组件

* 其它组件可以通过订阅store中的状态(state)来刷新自己的视图

```javascript
import React ,{ useState,useRef, useEffect} from 'react';
import { createSet , createAdd , createRemove , createToggle} from './actionCreater';
import './App.css';

import reducer from './reducers';


/*
*   整合dispatch 和 create 函数
*/
function bindActionCreaters(actionCreater,dispatch){
  let ret = {} ;
  for(let key in actionCreater) {
      ret[key] = function (...args) {
        const actionCreate = actionCreater[key];
        const action = actionCreate(...args);
        dispatch(action);
      }
  }
  return ret;
}

function Control(props){
  const {addTodo} = props;
  const inputRef = useRef();
  const onSubmit = (e) => {
      e.preventDefault();
      const newText = inputRef.current.value.trim();
      if(newText.length === 0) {
        return ;
      }
    
      addTodo(newText);
      inputRef.current.value = '';
  }
  return (
    <div className = 'control'> 
        <h3>todos</h3>
        <form onSubmit = {onSubmit} name = 'formTest'> 
            <input 
                type = 'text' 
                placeholder = 'what need to done?' 
                className = 'new-todo'    
                ref = {inputRef}
            />
            <button type ='reset'>reset</button>
        </form>
    </div>
  )
}
function Todos(props){
  const {todos,removeTodo,toggleTodo} = props;
  
  return (
      <ul>
        {
          todos.map(todoItem => {
            return (<TodoItem
                      key = {todoItem.id}
                      todoItem = {todoItem}
                      removeTodo = {removeTodo}
                      toggleTodo = {toggleTodo}
                    />)
          })
        }
      </ul>
  )
}
function TodoItem (props){
  const {
      todoItem:{
          id,
          text,
          compolete
        }, 
        toggleTodo,
        removeTodo
  } = props ;
  const onChange = () => {
    toggleTodo(id)
  }
  const onRemove = () => {
    removeTodo(id)
  
  }
  return (
      <li className = 'todo-item'>
          <input 
              type = 'checkbox'
              onChange = {onChange}
              checked = {compolete}
          />
          <label className = {compolete?'compolete':''}>
              {text}
          </label>
          <button onClick = {onRemove}>&#xd7; </button>
      </li>
  )
}

let _LS_KEY_TODOS_ = '$_LS_KEY_TODOS';

//新建唯一数据源
let store = {
  todos:[],
  incrementCount:0
}

function TodoList() {
    const [todos,setTodos] = useState([]);
    const [incrementCount,setIncrementCount] = useState(0);

    //每次变化同步数据
    useEffect(() => {
      Object.assign(store,{
        todos,
        incrementCount
      })
     
    }, [todos,incrementCount])

    //派发新的数据更新
    const dispatch = (action) => {
          const setters = {
            todos : setTodos,
            incrementCount : setIncrementCount 
          }
          if('function' === typeof action){
            action(dispatch,() => store );
            return ;
          }
          //通过reducer 函数，返回更新后的state
          const newState = reducer(store,action);
          //遍历新的state，并将值通过 useState 更新
          for (let key in newState) {
            setters[key](newState[key]);
          }
    }

    //进入组件获取localStorage下存储的数据;
    useEffect(() => {
        const localTodos = JSON.parse(localStorage.getItem(_LS_KEY_TODOS_) || "[]");
        // const { setTodo } = bindActionCreaters({setTodo:createSet},dispatch);
        // setTodo(localTodos);
        dispatch(createSet(localTodos));
    },[])
    //todos改变时,修改localStorage下存储的数据;
    useEffect(() => {
      localStorage.setItem(_LS_KEY_TODOS_,JSON.stringify(todos))
      
    }, [todos])
  return (
    <div className="todo-list">
        <Control 
          {
            ...bindActionCreaters({
              addTodo:createAdd
            },dispatch)
          }
        />
        <Todos 
          {
            ...bindActionCreaters({
              removeTodo:createRemove,
              toggleTodo:createToggle,
            },dispatch)
          }
        todos = {todos}/>
    </div>
  );
}
export default TodoList;

```



actionCreater.js

```javascript
export function createSet(payload){
    return {
        type:"set",
        payload,
    }
}
let idSeq = Date.now();
export function createAdd(text){
    return (dispatch , getState) => {
        setTimeout(() => {
            const { todos } = getState() ;
            if(!todos.find(todoItem => todoItem.text === text)){
                dispatch({
                    type:"add",
                    payload:{
                        id:++idSeq,
                        text,
                        compolete:false
                    },
                })
            }
        }, 3000);
    }
}
export function createRemove(payload){
    return {
        type:"remove",
        payload,
    }
}
export function createToggle(payload){
    return {
        type:"toggle",
        payload,
    }
}
```

reducers.js

```javascript
const reducers = {
    todos(state,action){
      const {type,payload} = action;
     switch(type) {
       case 'set':
         return payload;
       case 'add':
         return [...state,payload];
       case 'remove':
         return state.filter(todoItem =>{
           return todoItem.id !== payload
         })
       case 'toggle':
         return state.map(todoItem => {
           return todoItem.id === payload?{
                 ...todoItem,
                 compolete:!todoItem.compolete
           }:todoItem
       })
       default:
        break;
     }

     return state;
    },
    incrementCount(state,action){
       const { type } = action ;
       switch(type){
          case 'set':
          case "add":
                return state  + 1 ;
          default:
                break; 
       }
       return state;
    }  
 }

 function combineReducers(reducers){
  return function reducer(state , action){
      const change = {};
      for( let key in reducers){
         change[key] = reducers[key](state[key],action);
      }
      return {
        ...state,
        ...change,
      }
  }
}

export default combineReducers(reducers);
```



