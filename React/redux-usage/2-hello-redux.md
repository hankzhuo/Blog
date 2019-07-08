# 第二篇 Hello Redux

## 安装环境
理论与代码结合，实践动手，才能更好理解 Redux。

老样子，学一门技术，都是从简单的应用开始的。

首先，为了节省时间和以学习 Redux 的目的，使用 React 脚手架 [create-react-app](https://github.com/facebook/create-react-app) 安装 React 环境。

## React 内部 state

安装好环境后，按照提示把项目跑起来，修改 ```src/App.js``` 文件，

**src/App.js**

```
import React, { Component } from "react";
import HelloWorld from "./HelloWorld";

class App extends Component {
    constructor(props) {
      super(props)
      this.state = {
        tech: 'react'
      }
  }
  render() {
    return <HelloWorld tech={this.state.tech}/>
  }
}

export default App;
```

在 `src` 目录下创建一个 React 无状态组件 `<HelloWorld/>`:

```
import React from 'react'

const HelloWorld = ({tech}) => {
  return <div>
    Hello World <span>{tech}!</span>
  </div>
}

export default HelloWorld
```

可以看到，默认情况下是， React 内部通过 ``` this.state.tech```，传递给子组件。

## Redux 传递 state

接下来，把内部传递 `state` 修改成 Redux 传递的 `state`。

先 ```npm install redux --save``` 安装 Redux 库，安装成功后，在 ```src/App.js``` 中添加 ```redux``` 进来，如下：

```
import { createStore } from 'redux'
const store = createStore() 
```

通过 ```createStore()``` 是生成 `store`，`createStore` 函数接受一些参数，第一个参数是 `reducer`，如下：

```
const store = createStore(reducer) 
```

现在把 `reducer` 传进来，新建一个 `reducer`，在 `src/reducers` 目录下新建一个 `index.js` 文件。`reducer` 返回一个 `state`（这里先简单地返回一个 `state`），`index.js` 文件如下：

```
export default (state) => {
  return state
}
```

`createStore` 还可以接受第二个参数 `initialState`，`initialState` 是初始状态，修改后如下：

```
const initialState = { tech: "Redux"};
const store = createStore(reducer, initialState);
```

还记得上一篇文章，`state`（你的钱）是从 `store`（收营员） 返回的吗？使用 `store.getState().tech` 返回新的 `state`，`src/App.js` 完整代码如下：

```
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';
import HelloWorld from './HelloWorld'
import { createStore } from 'redux'
import reducer from './reducers'

const initialState = { tech: 'Redux'}
const store = createStore(reducer, initialState)

class App extends Component {
  render() {
    return <HelloWorld tech={store.getState().tech}/>
  }
}

export default App
```

现在为止，项目从组件内部 `state` 传递，现在从 `redux` 传递 `state` 了。
以上就是简单的 Hello Redux 入门程序。

## 源码

源码放在 `demo` 文件夹里：[源码链接](https://github.com/hankzhuo/redux-guide/tree/master/demo/demo1)

## 总结
这一篇文章主要讲了，从 React 中内部更新 state 到 Redux 获取 state。
下面总结下知识点：

1. Redux 是一个可预测的 JavaScript app 的状态管理
2. `createStore` 是一个工厂函数，用于生成 `store`
3. `createStore` 第一个参数 `reducer`，是必须传递的，第二个参数是初始 状态 `initialState`
4. `reducer` 是一个纯函数，接受两个参数，一个是初始化 `state`，另一个是 `action`
5. Reducer 总是返回一个新的 `state`
6. `store.getState()` 将会返回当前的 `state`
