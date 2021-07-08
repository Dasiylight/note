# Redux

## 1. What is Redux

（1）Web 应用是一个状态机，视图与状态是一一对应的。
（2）所有的状态，保存在一个对象里面。

也就是说，当需要共享某个组件的状态或是某个状态，则需要Redux

## 2. 基本概念

### 2.1 Store

用于保存数据的地方（数据的容器），（这些数据是全局的，组件都可以获取），**整个应用只能有一个Store**  
createStore可以用于创建Store

```javascript
import { createStore } from 'redux';
const store = createStore(fn);
```

createStore接收fn(干啥用的？)作为参数，返回Store对象

### 2.2 State

Store中存储了所有数据，若想获取到某个数据，则需要用到State

```javascript
import { createStore } from 'redux';
const store = createStore(fn);

const state = store.getState();
```

Redux 规定， 一个 State 对应一个 View(啥是view？可能是视图？)。只要 State 相同，View 就相同。你知道 State，就知道 View 是什么样，反之亦然。

### 2.3 Action

Action用于改变State，是View对State发出的通知，它描述了当前发生的事情。

```javascript
const action = {
  type: 'ADD_TODO',
  payload: 'Learn Redux'
};
```

上面代码中，Action 的名称是ADD_TODO，它携带的信息是字符串Learn Redux。

### 2.4 Action Creator

定义一个函数，专门用于生成Action  

```javascript
const ADD_TODO = '添加 TODO';

function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}

const action = addTodo('Learn Redux');
```

### 2.5 store.dispatch()

store.dispatch()是 View 发出 Action 的唯一方法。

```javascript
import { createStore } from 'redux';
const store = createStore(fn);

store.dispatch({
  type: 'ADD_TODO',
  payload: 'Learn Redux'
});
```

接收一个Action对象，然后将他发送出去，可以结合Action Creator使用

```store.dispatch(addTodo('Learn Redux'));```

### 2.6 Reducer

Store收到Action ==> 给出新的State ==> View变化
Reducer：函数形式，接收Action和当前State，返回新的State。

Reducer无需手动调用，store.dispatch方法会触发 Reducer 的自动执行。为此，Store 需要知道 Reducer 函数，做法就是在生成 Store 的时候，将 Reducer 传入createStore方法。

```javascript
import { createStore } from 'redux';
const store = createStore(reducer);
```

上面代码中，createStore接受 Reducer 作为参数，生成一个新的 Store。以后每当store.dispatch发送过来一个新的 Action，就会自动调用 Reducer，得到新的 State。

Reducer是一个**纯函数**，只要是同样的输入，必定得到同样的输出，这是为了保证同样的State，必定得到同样的 View。因此，Reducer不是改变State，而是**创建新的State**.

### 2.7 store.subscribe()

这是一个监听函数，State变化时就执行函数

```javascript
import { createStore } from 'redux';
const store = createStore(reducer);

store.subscribe(listener);  
```

显然，只要把 View 的更新函数（对于 React 项目，就是组件的render方法或setState方法）放入listener，就会实现 View 的自动渲染。

如何解除监听:调用subscribe返回的函数

## 3. 如何实现一个Store？

## 4. 拆分Reducer

## 5. 工作流程

用户发出Action ==> Store自动调用Reducer，传入当前State和收到的Action，返回新的State ==> State变化，Store调用监听函数subscribe()，可以通过getState来得到当前的状态

怎样实现一个基础的redux？store + action + reducer

## 6. 中间件

可对store.dispatch（即发送action的步骤）进行修改重定义，来加入中间件。

## 7. react-redux


# redux-saga

## What is redux-saga

redux-saga用于管理应用程序的副作用（包括异步、访问缓存等），采用了ES6的Generator属性来处理异步

### Generator属性

```javascript
function* gen(){
    yield {
        // ...
    }
    yield{
        //..
    }
}


```