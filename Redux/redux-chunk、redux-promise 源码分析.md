上一篇文章里，action 都是同步的，也就是说 dispatch(action)，经过中间件，更新得到 state 是同步的。

下面介绍几种 action 是异步执行的。


## `redux-thunk`

redux-chunk 是 redux 的一个中间件 middleware，核心是创建一个异步的 action。

具体是怎么实现的呢？首先了解一个概念，什么是 thunk?

### `thunk`

thunk 是一个函数，函数内有一个表达式，目的是为了延时执行。

```js
let x = 1 + 2;
let foo = () => 1 + 2;
```

只有 `foo()` 执行时，才返回 3，那么 foo 就是 chunk 函数。

`redux-thunk` 源码很少，但又是经典，源码如下：

```js
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
export default thunk;

```

如源码所示，redux-thunk 是个中间件，和其他中间件区别就是，判断 action 类型是一个函数时候，执行这个 action。而同步 action 执行返回是一个对象。

例子：

```js
// dispatch(action) 时，经过中间件时，执行 next(action)，更新得到 state 是同步的。
const loading = text => ({
  type: 'LOADING',
  text
})

const cancelLoading = text => ({
  type: 'CANCEL_LOADING',
  text
})

// dispatch(action) 时，经过中间件时，执行 action(...args)，更新得到 state 是异步的。
const fetchPeople = (url, params) => {
  return ({dispatch, getState}) => {
    dispatch(loading('start loading'));
    fetch(url, params).then(delayPromise(3000)).then(data => {
      dispatch(cancelLoading('cancel loading'));
    });
  };
};
```

[在线例子](https://codesandbox.io/s/lpo94263q7)

## `redux-promise`

redux-promise 也是中间件 middleware，原理和 redux-thunk 差不多，主要区别是 redux-promise 用到了 promise 异步。

源码如下：

```js
import isPromise from 'is-promise';
import { isFSA } from 'flux-standard-action';

export default function promiseMiddleware({ dispatch }) {
  return next => action => {
    if (!isFSA(action)) {
      return isPromise(action) ? action.then(dispatch) : next(action);
    }

    return isPromise(action.payload)
      ? action.payload
          .then(result => dispatch({ ...action, payload: result }))
          .catch(error => {
            dispatch({ ...action, payload: error, error: true });
            return Promise.reject(error);
          })
      : next(action);
  };
}
```

redux-promise 兼容了 FSA 标准（结果信息放在 payload 里），实现过程就是判断 action 或 action.payload 是否为 promise 对象，是则执行 then().catch()，否则 next(action)。