这是分析 redux 源码系列第一篇。

下面是 Redux v4.x API 源码的解读。

## `createStore`

> createStore(reducer, [preloadedState], enhancer)

`createStore` 方法接受三个参数，第一个参数为 `reducer`，第二个参数是 `preloadedState` （可选），第三个是参数是 `enhancer`。返回一个 对象 `store`。`store` 中包含方法 `dispatch`、`getState`、`subscribe`、`replaceReducer`、`[$$observable]`。

参数 `reducer` 有可能是一个 `reducer`，如果有个多个 `reducer`，则通过  `combineReducer` 函数执行后返回函数作为 `reducer`。

参数 `preloadedState` 代表初始状态，很多时候都不传，不传该参数时候且 `enhancer` 是函数情况，`createStore` 函数内部会把 `enhancer` ，作为第二个参数使用，源码如下：

```javascript
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }
```

参数 `enhancer` 是 `store` 增强器，是一个函数， `createStore` 作为 `enhancer` 函数的参数，这里用到了函数式编程，返回函数又传递 `reducer` 和 `preloadState` 参数，执行最终返回一个增强的 `store`。`enhancer` 常有的是 `applyMiddleware()` 返回值，`react-devrools()` 等工具，源码如下：

```javascript
  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }

    return enhancer(createStore)(reducer, preloadedState)
  }

```

介绍方法之前，下面是一些变量：

```javascript
  let currentReducer = reducer   // 当前的 reducer
  let currentState = preloadedState    // 当前的 state
  let currentListeners = []   // 当前的监听者
  let nextListeners = currentListeners  //  监听者的备份
  let isDispatching = false  // 是否正在执行 dispatch 动作
```

### `dispatch `

> dispatch(action)

`dispatch` 方法，接受一个参数`action`，`action` 是一个对象，该对象必须包括 `type` 属性，否则会报错。`dispatch` 函数内部，主要是执行了 `reducer() ` 函数，得到最新 `state`，赋值给 `currentState`，执行订阅者，`dispatch` 函数返回一个 `action` 对象。源码如下：

```javascript
  function dispatch(action) {
    if (typeof action.type === 'undefined') {
      throw new Error(
        'Actions may not have an undefined "type" property. ' +
          'Have you misspelled a constant?'
      )
    }

    if (isDispatching) {  // 如果正在 dispatch，再次触发 dispatch，则报错
      throw new Error('Reducers may not dispatch actions.')
    }

    try {
      isDispatching = true  // 表示正在执行 `dispatch`
      currentState = currentReducer(currentState, action)    // 执行最新的 Reducer，返回最新的 state
    } finally {
      isDispatching = false
    }

    const listeners = (currentListeners = nextListeners)  // 每次 dispatch 执行，订阅者都会执行
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    return action
  }

```

### `getState`

> getState()

获取最新的 state，就是返回 currentState。源码如下：

```javascript
  function getState() {
    return currentState
  }
```

### `subscribe`

添加事件订阅者，每次触发 `dispatch(action)`
时候，订阅者都会执行。返回取消订阅方法 `unSubscribe`。这里采用了观察者模式/发布-订阅者模式。源码如下：

注意：订阅者必须是函数，否则报错。

```javascript
function subscribe(listener) {
    if (typeof listener !== 'function') {  // 订阅者必须是函数
      throw new Error('Expected the listener to be a function.')
    }

    if (isDispatching) {
      throw new Error(
        'You may not call store.subscribe() while the reducer is executing. ' +
          'If you would like to be notified after the store has been updated, subscribe from a ' +
          'component and invoke store.getState() in the callback to access the latest state. ' +
          'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.'
      )
    }

    let isSubscribed = true

    nextListeners.push(listener)     // 订阅消息，每次 dispatch(action) 时候，订阅者都会接受到消息

    return function unsubscribe() {  // 返回取消订阅方法
      if (!isSubscribed) {
        return
      }

      if (isDispatching) {
        throw new Error(
          'You may not unsubscribe from a store listener while the reducer is executing. ' +
            'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.'
        )
      }

      isSubscribed = false

      const index = nextListeners.indexOf(listener) 
      nextListeners.splice(index, 1)  // 取消订阅者
    }
  }
```


### `observable`

让一个对象可被观察。被观察对象必须是对象，否则报错。这里作者用到了第三方库 `symbol-observable`，对 `currentState` 进行观察。源码如下：

`ECMAScript  Observables` 目前只是草案，还没正式使用。

```javascript
function observable() {
    const outerSubscribe = subscribe
    return {
      subscribe(observer) {
        if (typeof observer !== 'object' || observer === null) {
          throw new TypeError('Expected the observer to be an object.')
        }

        function observeState() {
          if (observer.next) {
            observer.next(getState())   // 对 currentState 进行监听
          }
        }

        observeState()
        const unsubscribe = outerSubscribe(observeState)
        return { unsubscribe }
      },

      [$$observable]() {
        return this
      }
    }
  }
  ```

### `replaceReducer`

替换` reducer`，一般用不到。源码也简单，把 `nextReducer` 赋值给 `currentReducer` 。源码如下：

```javascript
  function replaceReducer(nextReducer) {
    if (typeof nextReducer !== 'function') {
      throw new Error('Expected the nextReducer to be a function.')
    }

    currentReducer = nextReducer
  }
```

上面 `dispatch`、`getState`、`subscribe`、`replaceReducer`、`[$$observable]` 这些方法使用了闭包，一直保持对 `currentState`、`currentReducer`、`currentListeners`、`nextListeners`、`isDispatching` 等变量的引用。比如，`dispatch `改变 `currentState`，相应的其他方法中，`currentState` 也会跟着变，所以这些方法之间是存在联系的。


## `applyMiddleware`

执行中间件的方法，中间件可以有很多，有自定义的如：打印日志，也有比较知名的 如：`redux-chunk`、`redux-promise`、`redux-saga` 等，后续会分析。

`applyMiddleware()` 通常是上面提到的 `createStore` 方法的第三个参数 `enhancer`，源码如下：

 ```javascript
enhancer(createStore)(reducer, preloadedState)
 ```

结合上面代码可以等价于

 ```javascript
applyMiddleware(...middlewares)(createStore)(reducer, preloadedState)
 ```
`applyMiddleware` 使用了函数式编程，接收第一个参数元素为 `middlewares `的数组，返回值接收`createStore` 为参数的函数，返回值接收 `dispatch` 函数，接收一个 `action` 如 `(reducer, state)`，最终返回增强版的 `store`。

源码如下：

```javascript
function applyMiddleware(...middlewares) {
  return createStore => (...args) => {  
    const store = createStore(...args)  //  args 为 (reducer, initialState)
    let dispatch = () => {
      throw new Error(
        'Dispatching while constructing your middleware is not allowed. ' +
          'Other middleware would not be applied to this dispatch.'
      )
    }
    
    const middlewareAPI = {   // 给 middleware 的 store
      getState: store.getState,
      dispatch: (...args) => dispatch(...args) 
    }
    const chain = middlewares.map(middleware => middleware(middlewareAPI))  // chain 是一个参数为 next 的匿名函数的数组（中间件执行返回的函数）
    dispatch = compose(...chain)(store.dispatch)  // 经过 compose 函数后返回的结果，是经过 middlewares 包装后的 dispatch 

    return {   // 返回 dispatch 增强后的 store
      ...store,
      dispatch
    }
  }
}

```

## `compose`

源码如下：

```javascript
 function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args))) 
}
```

虽然源代码没几行，作者实现方式太厉害了，如果理解起来有点费劲，可以看看下面例子分析：

```javascript
var arr = [
  function fn1(next){
    return action => {
      // next 为 fn2 return 函数（dispatch 函数）
      const returnValue = next(action)
      return returnValue
    }
  },
  function fn2(next) {
    return action => {
      // next 为 fn3 return 函数（dispatch 函数）
      const returnValue = next(action)
      return returnValue
    }
  },
  function fn3(next3) {
    return action => {
      // next 为 dispatch。
      const returnValue = next3(action)
      return returnValue
    }
  },
]

var fn = arr.reduce((a, b) => (args) => a(b(args)))
fn // (args) => a(b(args)) 等价于 (dispatch) => fn1(fn2(fn3(dispatch)))

// reduce函数 执行顺序：
// 第一次：
// 变量 a: ƒ fn1(args)
// 变量 b: ƒ fn2(args)
// 返回值: (args) => fn1(fn2(args))

// 第一次返回值赋值给 a
// 变量 a:  (args) => fn1(fn2(args))
// 变量 b: ƒ fn3(args)

// 第二次：
// 变量 a :  (args) => fn1(fn2(args))
// 变量 b: ƒ fn3(args)
// 返回值: (args) => fn1(fn2(fn3(args)))
```

所以作者的用意是 `compose(f, g, h)`  返回 `(...args) => f(g(h(...args))) `，其中 `f、g、h` 是一个个中间件 `middleware`。

结合上面 `applyMiddleware` 源码有一段 `dispatch = compose(...chain)(store.dispatch)` ，所以 `args` 为 `store.dispatch`，`(store.dispatch) => f(g(h(store.dispatch)))`，执行第一个中间件`h(store.dispatch)`，返回 `dispatch` 函数（参数为 action 的函数）作为下一个中间件的参数，通过一层层的传递，最终返回一个经过封装的  `dispatch`  函数。

下面是打印日志中间件例子：

```javascript
const logger = store => next => action => {
  console.log('dispatch', action)
  next(action)
  console.log('finish', action)
}

const logger2 = store => next2 => action2 => {
  console.log('dispatch2', action2)
  next2(action2)
  console.log('finish2', action2)
}
...
const store = createStore(rootReducer, applyMiddleware(logger, logger2))
```

注意：但如果在某个中间件中使用了 `store.dipsatch()`，而不是 `next()`，那么就会回到起始位置，会造成无限循环了。

每次触发一个 dispatch，中间件都会执行，打印的顺序为  `dispatch distapch2 finish2 finish`。


## `combineReducer`

这个 API 理解为：如果有多个 reducer，那么需要把它们合成一个 reducer，在传给函数 `createStore(reducer)`。

核心源码如下：

```javascript
function combineReducers(reducers) {
  const reducerKeys = Object.keys(reducers)
  const finalReducers = {}
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i]

    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]  // reducer 一定是函数
    }
  }
  const finalReducerKeys = Object.keys(finalReducers)

  return function combination(state = {}, action) {  // 返回一个新的 reducer 函数
    let hasChanged = false
    const nextState = {}
    for (let i = 0; i < finalReducerKeys.length; i++) { // 每次执行一次 dipatch(action)，所有的 reducer 都会执行
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      const previousStateForKey = state[key]
      const nextStateForKey = reducer(previousStateForKey, action)
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    return hasChanged ? nextState : state  // 判断 state 是否有变化，如果有则返回新的 state
  }
}

```

## `bindActionCreators`

这个 API 可以理解为：生成 action 的方法。主要是把 dispatch 也封装进 `bindActionCreator(actionCreator, dispatch)` 方法里面去，所以调用时候可以直接触发 `dispatch(action)`，不需要在手动调用 dispatch，比如 `dispatch(fetchPeople({type: TYPE, text: 'fetch people'}))`，使用这个 API 后，则直接 `fetchPeople({type: TYPE, text: 'fetch people'})`

源码如下：

```javascript
function bindActionCreator(actionCreator, dispatch) {
  return function() {
    return dispatch(actionCreator.apply(this, arguments))
  }
}

function bindActionCreators(actionCreators, dispatch) {
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }

  const boundActionCreators = {}
  for (const key in actionCreators) {
    const actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  return boundActionCreators
}

```