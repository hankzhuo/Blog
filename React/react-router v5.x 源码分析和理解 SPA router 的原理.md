这篇文章主要讲的是分析 react-router 源码，版本是 **v5.x**，以及 SPA 路由实现的原理。

单页面应用都用到了路由 router，目前来看实现路由有两种方法 hash 路由和 H5 History API 实现。

而 react-router 路由，则是用到了 history 库，该库其实是对 hash 路由、history 路由、memory 路由（客户端）进行了封装。

下面先看看 hash 和 history 是怎样实现路由的。

# hash router

hash 是 location 的属性，在 URL 中是 #后面的部分。如果没有#，则返回空字符串。

hash 路由主要实现原理是：hash 改变时，页面不发生跳转，即 window 对象可以对 hash 改变进行监听（hashchange 事件），只要 url 中 hash 发生改变，就会触发回调。

利用这个特性，下面模拟实现一个 hash 路由。

实现步骤：
- 初始化一个类
- 记录路由、history 历史值
- 添加 hashchange 事件，hash 发生改变时，触发回调
- 初始添加所有路由
- 模拟前进和回退功能

代码如下：

**`hash.html`**
```html
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>hash router</title>
</head>
<body>
  <ul>
    <li><a href="#/">turn yellow</a></li>
    <li><a href="#/blue">turn blue</a></li>
    <li><a href="#/green">turn green</a></li>
  </ul>
  <button id='back'>back</button>
  <button id='forward'>forward</button>
  <script src="./hash.js"></script>
</body>
</html>
```
**`hash.js`**
```js
class Routers {
  constructor() {
    this.routes = {}
    this.currentUrl = ''
    this.history = []; // 记录 hash 历史值
    this.currentIndex = this.history.length - 1;  // 默认指向 history 中最后一个
    // 默认前进、后退
    this.isBack = false;
    this.isForward = false;

    this.onHashChange = this.onHashChange.bind(this)
    this.backOff = this.backOff.bind(this)
    this.forward = this.forward.bind(this)

    window.addEventListener('load', this.onHashChange, false);
    window.addEventListener('hashchange', this.onHashChange, false);  // hash 变化监听事件，support >= IE8
  }

  route(path, callback) {
    this.routes[path] = callback || function () { }
  }

  onHashChange() {
    // 既不是前进和后退，点击 a 标签时触发
    if (!this.isBack && !this.isForward) {
      this.currentUrl = location.hash.slice(1) || '/'
      this.history.push(this.currentUrl)
      this.currentIndex++
    }

    this.routes[this.currentUrl]()

    this.isBack = false
    this.isForward = false
  }
  // 后退功能
  backOff() {
    this.isBack = true
    this.currentIndex = this.currentIndex <= 0 ? 0 : this.currentIndex - 1

    this.currentUrl = this.history[this.currentIndex]
    location.hash = `#${this.currentUrl}`
  }
  // 前进功能
  forward() {
    this.isForward = true
    this.currentIndex = this.currentIndex >= this.history.length - 1 ? this.history.length - 1 : this.currentIndex + 1

    this.currentUrl = this.history[this.currentIndex]
    location.hash = `#${this.currentUrl}`
  }
}
// 初始添加所有路由
window.Router = new Routers();
Router.route('/', function () {
  changeBgColor('yellow');
});
Router.route('/blue', function () {
  changeBgColor('blue');
});
Router.route('/green', function () {
  changeBgColor('green');
});

const content = document.querySelector('body');
const buttonBack = document.querySelector('#back');
const buttonForward = document.querySelector('#forward')

function changeBgColor(color) {
  content.style.backgroundColor = color;
}
// 模拟前进和回退
buttonBack.addEventListener('click', Router.backOff, false)
buttonForward.addEventListener('click', Router.forward, false)
```

hash 路由兼容性好，缺点就是实现一套路由比较复杂些。

[在线预览](https://codesandbox.io/s/static-wfx60)

# history router

history 是 HTML5 新增的 API，允许操作浏览器的曾经在标签页或者框架里访问的会话历史记录。

history 包含的属性和方法：

- `history.state` 读取历史堆栈中最上面的值
- `history.length` 浏览历史中元素的数量
- `window.history.replaceState(obj, title, url)` 取代当前浏览历史中当前记录
- `window.history.pushState(obj, title, url)` 往浏览历史中添加历史记录，不跳转页面
- `window.history.popstate(callback)` 对历史记录发生变化进行监听，state 发生改变时，触发回调事件
- `window.history.back()` 后退，等价于浏览器返回按钮
- `window.history.forward()` 前进，等价于浏览器前进按钮
- `window.history.go(num)` 前进或后退几个页面，`num` 为负数时，后退几个页面

实现步骤：
- 初始化一个类
- 记录路由
- 添加 `popstate` 事件，state 发生改变时，触发回调
- 初始添加所有路由

代码如下：

**`history.html`**
```html
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>h5 router</title>
</head>
<body>
  <ul>
      <li><a href="/">turn yellow</a></li>
      <li><a href="/blue">turn blue</a></li>
      <li><a href="/green">turn green</a></li>
  </ul>
  <script src='./history.js'></script>
</body>
</html>
```

**`history.js`**

```js
class Routers {
  constructor() {
    this.routes = {};
    this._bindPopState();
  }

  route(path, callback) {
    this.routes[path] = callback || function () { };
  }

  go(path) {
    history.pushState({ path: path }, null, path);
    this.routes[path] && this.routes[path]();
  }

  _bindPopState() {
    window.addEventListener('popstate', e => {  // 监听 history.state 改变
      const path = e.state && e.state.path;
      this.routes[path] && this.routes[path]();
    });
  }
}

// 初始时添加所有路由
window.Router = new Routers();
Router.route('/', function () {
  changeBgColor('yellow');
});
Router.route('/blue', function () {
  changeBgColor('blue');
});
Router.route('/green', function () {
  changeBgColor('green');
});

const content = document.querySelector('body');
const ul = document.querySelector('ul');

function changeBgColor(color) {
  content.style.backgroundColor = color;
}

ul.addEventListener('click', e => {
  if (e.target.tagName === 'A') {
    debugger
    e.preventDefault();
    Router.go(e.target.getAttribute('href'));
  }
});


```

兼容性：`support >= IE10`

[在线预览](https://codesandbox.io/s/static-1tg7i)

# react-router

react-router 分为四个包，分别为 `react-router`、`react-router-dom`、`react-router-config`、`react-router-native`，其中 `react-router-dom` 是浏览器相关 API，`react-router-native` 是 React-Native 相关 API，`react-router` 是核心也是共同部分 API，`react-router-config` 是一些配置相关。

react-router 是 React指定路由，内部 API 的实现也是继承 React 一些属性和方法，所以 react-router 内 API 也是 React 组件。

react-router 还用到了 `history` 库，这个库主要是对 hash 路由、history 路由、memory 路由的封装。

下面我们讲到的是 react-router **v5.x** 版本，用到了 context，context 减少了组件之间显性传递 props，具体用法可以看看官方文档。

创建 context: 

```jsx
var createNamedContext = function createNamedContext(name) {
    var context = createContext();
    context.displayName = name;
    return context;
};

var context = createNamedContext("Router");
```

## Router

定义一个类 Router，它继承 React.Component 属性和方法，所以 Router 也是 React 组件。

```js
_inheritsLoose(Router, _React$Component);
// 等价于 
Router.prototype = Object.create(React.Component)  
```

在 Router 原型对象上添加 React 生命周期 `componentDidMount`、`componentWillUnmount`、`render` 方法。

一般情况下，Router 都是作为 Route 等其他子路由的上层路由，使用了 `context.Provider`，接收一个 value 属性，传递 value 给消费子组件。

```jsx
var _proto = Router.prototype; 
_proto.componentDidMount = function componentDidMount() {
    // todo
};

_proto.componentWillUnmount = function componentWillUnmount(){
    // todo
};

_proto.render = function render() {
    return React.createElement(context.Provider, props);
};

```

history 库中有个方法 `history.listen(callback(location))`  对 location 进行监听，只要 location 发生变化了，就会 setState 更新 location，消费的子组件也可以拿到更新后的 location，从而渲染相应的组件。

核心源码如下：

```jsx
var Router =
    function (_React$Component) {
        // Router 从 React.Component 原型上的继承属性和方法
        _inheritsLoose(Router, _React$Component);  

        Router.computeRootMatch = function computeRootMatch(pathname) {
            return {
                path: "/",
                url: "/",
                params: {},
                isExact: pathname === "/"
            };
        };

        function Router(props) { // 首先定义一个类 Router，也是 React 组件
            var _this;

            _this = _React$Component.call(this, props) || this; // 继承自身属性和方法
            _this.state = {
                location: props.history.location
            };

            _this._isMounted = false;
            _this._pendingLocation = null;

            if (!props.staticContext) {  // 如果不是 staticRouter，则对 location 进行监听
                _this.unlisten = props.history.listen((location) => { // 监听 history.location 变化，如果有变化，则更新 locaiton
                    if (_this._isMounted) {
                        _this.setState({
                            location: location
                        });
                    } else {
                        _this._pendingLocation = location;
                    }
                });
            }

            return _this;
        }

        var _proto = Router.prototype;  // 组件需要有生命周期，在原型对象上添加 componentDidMount、componentWillUnmount、render 方法

        _proto.componentDidMount = function componentDidMount() {
            this._isMounted = true;

            if (this._pendingLocation) {
                this.setState({
                    location: this._pendingLocation
                });
            }
        };

        _proto.componentWillUnmount = function componentWillUnmount() {
            if (this.unlisten) this.unlisten();  // 停止监听 location
        };

        _proto.render = function render() {
            // 使用了 React Context 传递 history、location、match、staticContext，使得所有子组件都可以获取这些属性和方法
            // const value = {
            //   history: this.props.history,
            //   location: this.state.location,
            //   match: Router.computeRootMatch(this.state.location.pathname),
            //   staticContext: this.props.staticContext
            // }

            // return (
            //   <context.Provider value={value}>
            //     {this.props.children}
            //   </context.Provider>
            // )
            return React.createElement(context.Provider, {
                children: this.props.children || null,
                value: {
                    history: this.props.history,
                    location: this.state.location,
                    match: Router.computeRootMatch(this.state.location.pathname),
                    staticContext: this.props.staticContext // staticContext 是 staticRouter 中的 API，不是公用 API
                }
            });
        };

        return Router;
    }(React.Component);
```

## Route

Route 一般作为 Router 的子组件，主要是匹配 path 和渲染相应的组件。

Route 使用了 `context.Consumer`（消费组件），订阅了 Router 提供的 context，一旦 location 发生改变，context 也会改变，判断当前 `location.pathname` 是否与子组件的 path 是否匹配，如果匹配，则渲染对应组件，否则就不渲染。

Route 因为有些库传递组件方式不同，所以有多种渲染，部分代码如下：

```jsx
const {children, render, component} = this.props
let renderEle = null;

// 如果有 children，则渲染 children
if (children &&  !isEmptyChildren(children)) renderEle = children

// 如果组件传递的是 render 及 match 是匹配的，则渲染 render
if (render && match) renderEle = render(props)

// 如果组件传递 component 及 match 是匹配的，则渲染 component 
if (component && match) renderEle = React.createElement(component, props)

return (
    <context.Provider value={obj}>
        {renderEle}
    </context.Provider>
)
```

核心源码如下：

```jsx
var Route =
    function (_React$Component) {
        _inheritsLoose(Route, _React$Component);

        function Route() {
            return _React$Component.apply(this, arguments) || this;
        }

        var _proto = Route.prototype;

        _proto.render = function render() {
            var _this = this;
            // context.Consumer 每个 Route 组件都可以消费 Router 中 Provider 提供的 context
            return React.createElement(context.Consumer, null, function (context$$1) {
                var location = _this.props.location || context$$1.location;
                var match = _this.props.computedMatch ? _this.props.computedMatch
                    : _this.props.path ? matchPath(location.pathname, _this.props) : context$$1.match;  // 是否匹配当前路径

                var props = _extends({}, context$$1, { // 处理用 context 传递的还是自己传递的 location 和 match
                    location: location,
                    match: match
                });

                var _this$props = _this.props,
                    children = _this$props.children,
                    component = _this$props.component,  
                    render = _this$props.render;

                if (Array.isArray(children) && children.length === 0) {
                    children = null;
                }

                // let renderEle = null

                // 如果有 children，则渲染 children
                // if (children &&  !isEmptyChildren(children)) renderEle = children

                // 如果组件传递的是 render 及 match 是匹配的，则渲染 render
                // if (render && props.match) renderEle = render(props)

                //  如果组件传递 component 及 match 是匹配的，则渲染 component 
                // if (component && props.match) renderEle = React.createElement(component, props)

                // return (
                //     <context.Provider value={props}>
                //        {renderEle}
                //     </context.Provider>
                // )

                return React.createElement(context.Provider, { // Route 内定义一个 Provider，给 children 传递 props
                    value: props
                }, children && !isEmptyChildren(children) ? children : props.match ? component ? React.createElement(component, props) : render ? render(props) : null : null);
            });
        };

        return Route;
    }(React.Component);
```
## Redirect

重定向路由：from 是从哪个组件来，to 表示要定向到哪里。

```jsx
 <Redirect from="home" to="dashboard" />
```

根据有没传属性 `push`，有传则是往 state 堆栈中新增（`history.push`），否则就是替代（`history.replace`）当前 state。

Redirect 使用了 `context.Consumer`（消费组件），订阅了 Router 提供的 context，一旦 location 发生改变，context 也会改变，则也会触发重定向。

源码如下：

```jsx
function Redirect(_ref) {
    var computedMatch = _ref.computedMatch,
        to = _ref.to,
        _ref$push = _ref.push,
        push = _ref$push === void 0 ? false : _ref$push;
    return React.createElement(context.Consumer, null, (context$$1) => { // context.Consumer 第三个参数是一个函数
        var history = context$$1.history,
            staticContext = context$$1.staticContext;

        // method 方法：判断是否是替换（replace）当前的 state，还是往 state 堆栈中添加（push)一个新的 state
        var method = push ? history.push : history.replace;
        // 生成新的 location
        var location = createLocation(computedMatch ? typeof to === "string" ? generatePath(to, computedMatch.params) : _extends({}, to, {
            pathname: generatePath(to.pathname, computedMatch.params)
        }) : to); 

        if (staticContext) { // 当渲染一个静态的 context 时（staticRouter)，立即设置新 location
            method(location);
            return null;
        }
        // Lifecycle 是一个 return null 的空组件，但定义了 componentDidMount、componentDidUpdate、componentWillUnmount 生命周期
        return React.createElement(Lifecycle, {
            onMount: function onMount() {
                method(location);
            },
            onUpdate: function onUpdate(self, prevProps) {
                var prevLocation = createLocation(prevProps.to);
                // 触发更新时，对比前后 location 是否相等，不相等，则更新 location
                if (!locationsAreEqual(prevLocation, _extends({}, location, {
                    key: prevLocation.key
                }))) {
                    method(location);
                }
            },
            to: to
        });
    });
}
```

## Switch

Switch 组件的子组件必须是 Route 或 Redirect 组件。

Switch 使用了 `context.Consumer`（消费组件），订阅了 Router 提供的 context，一旦 location 发生改变，context 也会改变，判断当前 `location.pathname` 是否与子组件的 path 是否匹配，如果匹配，则渲染对应的子组件，其他都不渲染。

源码如下：

```jsx
var Switch =
    function (_React$Component) {
        _inheritsLoose(Switch, _React$Component);

        function Switch() {
            return _React$Component.apply(this, arguments) || this;
        }

        var _proto = Switch.prototype;

        _proto.render = function render() {
            var _this = this;

            return React.createElement(context.Consumer, null, function (context$$1) {
                var location = _this.props.location || context$$1.location;
                var element, match; 
               
                React.Children.forEach(_this.props.children, function (child) {
                    if (match == null && React.isValidElement(child)) {
                        element = child;
                        var path = child.props.path || child.props.from;
                        match = path ? matchPath(location.pathname, _extends({}, child.props, {
                            path: path
                        })) : context$$1.match;
                    }
                });
                return match ? React.cloneElement(element, {
                    location: location,
                    computedMatch: match  // 加强版的 match
                }) : null;
            });
        };

        return Switch;
    }(React.Component);
```

## Link

Link 组件作用是跳转到指定某个路由。Link 实际是对 `<a>` 标签进行了封装。

点击时会触发以下：

- 改变 url，但使用了 `e.preventDefault()`，所以页面没有发生跳转。
- 根据是否传递属性 replace，有传就是替代当前 state（`history.replace`），否则是往 state 堆栈中新增（`history.push`），从而路由发生了改变。
- 路由发生了改变，由于 Router 中有对 location 进行监听，从而通过 context 传递给消费子组件，匹配 path 是否相同，渲染相应的组件。

核心源码如下：

```jsx
function LinkAnchor(_ref) {
    var innerRef = _ref.innerRef,
        navigate = _ref.navigate,
        _onClick = _ref.onClick,
        rest = _objectWithoutPropertiesLoose(_ref, ["innerRef", "navigate", "onClick"]); // 剩余属性

    var target = rest.target;
    return React.createElement("a", _extends({}, rest, {  // a 标签
        ref: innerRef ,
        onClick: function onClick(event) {
            try {
                if (_onClick) _onClick(event);
            } catch (ex) {
                event.preventDefault(); // 使用 e.preventDefault() 防止跳转
                throw ex;
            }

            if (!event.defaultPrevented && // onClick prevented default
                event.button === 0 && ( // ignore everything but left clicks
                    !target || target === "_self") && // let browser handle "target=_blank" etc.
                !isModifiedEvent(event) // ignore clicks with modifier keys
            ) {
                event.preventDefault();
                navigate(); // 改变 location
            }
        }
    }));
}

function Link(_ref2) {
    var _ref2$component = _ref2.component,
        component = _ref2$component === void 0 ? LinkAnchor : _ref2$component,
        replace = _ref2.replace,
        to = _ref2.to, // to 跳转链接的路径
        rest = _objectWithoutPropertiesLoose(_ref2, ["component", "replace", "to"]);

    return React.createElement(__RouterContext.Consumer, null, function (context) {
        var history = context.history;
        // 根据 to 生成新的 location
        var location = normalizeToLocation(resolveToLocation(to, context.location), context.location);
        var href = location ? history.createHref(location) : "";

        return React.createElement(component, _extends({}, rest, {
            href: href,
            navigate: function navigate() {
                var location = resolveToLocation(to, context.location);
                var method = replace ? history.replace : history.push;
                method(location); // 如果有传 replace，则是替换掉当前 location，否则往 history 堆栈中添加一个 location
            }
        }));
    });
}
```

## BrowserRouter

BrowserRouter 用到了 H5 history API，所以可以使用 pushState、replaceState 等方法。

源码中主要是用到了 history 库 createBrowserHistory 方法创建了封装过 history 对象。

把封装过的 history 对象传递给 Router 的 props。

```jsx
var BrowserRouter =
    function (_React$Component) {
        _inheritsLoose(BrowserRouter, _React$Component); // BrowserRouter 继承 React.Component 属性和方法

        function BrowserRouter() {
            var _this;

            for (var _len = arguments.length, args = new Array(_len), _key = 0; _key < _len; _key++) {
                args[_key] = arguments[_key];
            }

            _this = _React$Component.call.apply(_React$Component, [this].concat(args)) || this; // 继承自身属性和方法
            _this.history = createBrowserHistory(_this.props); // 创建 browser history 对象，是支持 HTML 5 的 history API
            return _this;
        }

        var _proto = BrowserRouter.prototype;

        _proto.render = function render() {
            return React.createElement(Router, { // 以 Router 为 element，history 和 children 作为 Router 的 props
                history: this.history,
                children: this.props.children
            });
        };

        return BrowserRouter;
    }(React.Component);
```

## HashRouter

HashRouter 与 BrowserRouter 的区别，主要是创建是以 window.location.hash 为对象，返回一个 history，主要是考虑到兼容性问题。

```jsx
var HashRouter =
    function (_React$Component) {
        _inheritsLoose(HashRouter, _React$Component);

        function HashRouter() {
            var _this;

            for (var _len = arguments.length, args = new Array(_len), _key = 0; _key < _len; _key++) {
                args[_key] = arguments[_key];
            }

            _this = _React$Component.call.apply(_React$Component, [this].concat(args)) || this;
            _this.history = createHashHistory(_this.props); // 创建 hash history 对象，兼容老浏览器 hash，其他和 BrowserRouter 没有大区别
            return _this;
        }

        var _proto = HashRouter.prototype;

        _proto.render = function render() {
            return React.createElement(Router, {
                history: this.history,
                children: this.props.children
            });
        };

        return HashRouter;
    }(React.Component);

```

# 总结

- react-router 分为四个包，分别为 `react-router`、`react-router-dom`、`react-router-config`、`react-router-native`，其中 `react-router-dom` 是浏览器相关 API，`react-router-native` 是 React-Native 相关 API，`react-router` 是核心也是共同部分 API，`react-router-config` 是一些配置相关。

- react-router 是 React指定路由，内部 API 的实现也是继承 React 一些属性和方法，所以说 react-router 内 API 也是 React 组件。

- react-router 还用到了 history 库，这个库主要是对 hash 路由、history 路由、memory 路由的封装。

- Router 都是作为 Route 等其他子路由的上层路由，使用了 `context.Provider`，接收一个 value 属性，传递 value 给消费子组件。

- history 库中有个方法 `history.listen(callback(location))`  对 location 进行监听，点击某个 Link 组件，改变了 location，只要 location 发生变化了，通过 context 传递改变后的 location，消费的子组件拿到更新后的 location，从而渲染相应的组件。


# 参考
- [react-router github](https://github.com/ReactTraining/react-router)
- [history github](https://github.com/ReactTraining/history)
- [寻找海蓝96【你了解前端路由吗？】](https://juejin.im/post/5ac61da66fb9a028c71eae1b)
- [History MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/History)