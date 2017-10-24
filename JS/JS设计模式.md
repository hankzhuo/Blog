## 前言

  设计模式几十种，阅读了《JavaScript设计模式与开发实践》这本书后，个人感觉js就是围绕对象来设计的，发现日常写代码能用上的并不多，下面是常用的几种设计模式。

## 模式列表

  - 单例模式
  - 策略模式
  - 模板方法模式
  - 职责链模式
  - 发布订阅模式

## 设计模式
### 单例模式

单一模式的核心是确保只有一个实例，并提供全局访问，在 JS 开发中，经常把用一个对象包裹，这样减少了全局变量的污染，比如 var  a = {}。

普通写法:
``` javascript

  // 每次点击点击按钮，都会创建一个 div
  var createLayer1 = (function () {
    var div = document.createElement('div');
      div.innerHTML = '我是内容';
      div.style.display = 'none';
      document.body.appendChild(div);
      return div;
  })()

  document.getElementById('#btn').onclick = function () {
    var layer1 = createLayer1();
    layer1.style.display = 'block';
  }

```
单例模式：
``` javascript

  //实例对象总是在我们调用方法时才被创建，而不是在页面加载好的时候就创建。  
  // 这样就不会每次点击按钮，都会创建一个 div 了
  var createLayer2 = function () {
    var div;
    return function () {
      if (!div) {
        document.createElement('div');
        div.innerHTML = '我是内容';
        div.style.display = 'none';
        document.body.appendChild(div);
      }
      return div;
    }
  }

  document.getElementById('#btn').onclick = function () {
    var layer2 = createLayer2();
    layer2.style.display = 'block';
  }

```

### 策略模式
策略模式代码非常优雅，最喜欢模式之一，也很便于修改，请看代码。

普通模式：
``` javascript

  var awardS = function (salary) {
    return salary * 4
  };

  var awardA = function (salary) {
    return salary * 3
  };

  var awardB = function (salary) {
    return salary * 2
  };

  var calculateBonus = function (level, salary) {
    if (level === 'S') {
      return awardS(salary);
    }
    if (level === 'A') {
      return awardA(salary);
    }
    if (level === 'B') {
      return awardB(salary);
    }
  };

  calculateBonus('A', 10000);

```

策略模式：

``` javascript

  var strategies = {
    'S': function (salary) {
      return salary * 4;
    },
    'A': function (salary) {
      return salary * 3;
    },
    'B': function (salary) {
      return salary * 2;
    }
  }

  var calculateBonus = function (level, salary) {
    return strategies[level](salary);
  }

  calculateBonus('A', 10000);

```

### 模板方法模式

模板方法模式使用了原型链的方法，封装性好，复用性差。

``` javascript

  var Coffee = function () {

  };
  Coffee.prototype.boilWater = function () {
    // todo
    console.log('把水煮沸');
  };
  Coffee.prototype.brewCoffee = function () {
    // todo
    console.log('冲咖啡');
  };
  Coffee.prototype.pourInCup = function () {
    // todo
    console.log('把咖啡倒进杯子');
  };
  Coffee.prototype.addSugarAndMilk = function () {
    // todo
    console.log('加糖和牛奶');
  };
  Coffee.prototype.init = function () {
    this.boilWater();
    this.brewCoffee();
    this.pourInCup();
    this.addSugarAndMilk();
  }

  var coffee = new Coffee();
  coffee.init();

```

### 职责链模式

没错，我刚开始写第一个项目时候就这么嵌套的，重复代码太多，逻辑太乱，维护下太差。

``` javascript

  var order = function (orderType, pay, stock) {
    // 500 元定金模式
    if (orderType === 1) {
      if (pay === true) {
        console.log('500元定金预购，得到100元优惠券');
      } else {
        if (stock > 0) {
          console.log('普通购买，无优惠券');
        } else {
          console.log('手机库存不足');
        }
      }
    // 200 元定金模式
    } else if (orderType === 2) {
      if (pay === true) {
        console.log('200元定金预购，得到50元优惠券');
      } else {
        if (stock > 0) {
          console.log('普通购买，无优惠券');
        } else {
          console.log('手机库存不足');
        }
      }
    // 没有定金模式  
    } else if (orderType === 3) {
      if (stock > 0) {
        console.log('普通购买，无优惠券');
      } else {
        console.log('手机库存不足');
      }
    } 
  }

  order(1, true, 500);

```

职责链，一系列可能处理请求的对象被连接成一条链，请求在这些对象之间依次传递，直到遇到一个可以处理它的对象，减少了很多重复代码。

``` javascript

  var order500 = function (orderType, pay, stock) {
    if (orderType === 1 && pay === true) {
      console.log('500元定金预购，得到100元优惠券');
    } else {
      order200(orderType, pay, stock);
    }
  }

  var order200 = function (orderType, pay, stock) {
    if (orderType === 2 && pay === true) {
      console.log('200元定金预购，得到50元优惠券');
    } else {
      orderNormal(orderType, pay, stock);
    }
  }

  var orderNormal = function (orderType, pay, stock) {
    if (stock > 0) {
      console.log('普通购买，无优惠券');
    } else {
      console.log('手机库存不足');
    }
  }

  order500(1, true, 500);
  order500(1, false, 500);
  order500(2, true, 500);

```  

### 发布-订阅模式

发布订阅模式，顾名思义，就是一个发布消息，一个监听消息，当有消息接收时处理消息。

``` javascript
// js前端
  window.onload = function () {
    var socket = io.connect('http://localhost:20122?token=abc');
    socket.on('connect', function() {
      socket.emit('message', ':chat socket')
    });
    socket.on('message', function(data) {
      alert(data)
    })
  };

  // 服务端
  io.on('connection', function (socket) {
    socket.on('chat message', function (msg) {
      console.log('receive a message: ' + msg)
      io.emit('chat message', msg);
    });
  })

``` 

可以参考我的另一个仓库地址，一个简单的实时聊天 [demo](https://github.com/hankzhuo/socket-example)

## 参考

 - 来自《JavaScript设计模式与开发实践》读书笔记
 - https://github.com/hankzhuo/socket-example