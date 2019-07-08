这篇文章主要讲的是分析 react-router 源码，版本是 **v5.x**，以及路由实现的原理。

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

# react-router-dom

# react-router

# 学习源码提交记录

- [学习源码提交记录](https://github.com/hankzhuo/learning-react-router)

# 参考
- [react-router github](https://github.com/ReactTraining/react-router)
- [history github](https://github.com/ReactTraining/history)
- [寻找海蓝96【你了解前端路由吗？】](https://juejin.im/post/5ac61da66fb9a028c71eae1b)
- [History MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/History)