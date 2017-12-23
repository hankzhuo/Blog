
## 1，同步和异步区别？

  相信大家在工作过程中，肯定有或多或少听说过同步和异步。那么同步和异步到底是什么呢？它们之间有什么区别？举个栗子，煮开水，同步就是把水放上去烧，得一直等水开，中途不能做其他事情。而异步，则是把水放上去烧，让水在烧，你可以玩手机看电视，等水开了把火关掉。同样的，代码中也是一样，同步是现在发生的，异步是未来某个时刻发生的。
  
  同步代码：
  
    // 烧开水
    function boilWater() {
        var water;
        while(water_is_boiled) {
        // search
        }
        return water;
    }
    
    var boiledWater = boilWater();
    
    // 做其他事情
    doSomethingElse();
  
## 2，JS 运行机制

在介绍异步编程前，先介绍下JavaScript运行机制，因为JS 是单线程运行的，所以这意味着两段代码不能同时运行，而是必须一个接一个地运行，所以，在同步代码执行过程中，异步代码是不执行的。只有等同步代码执行结束后，异步代码才会被添加到事件队列中。
> (详细可以参考 [阮一峰-Event-Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html))。

## 3，JS 中异步有几种？

JS 中异步操作还挺多的，常见的分以下几种：
    
-   setTimeout (setInterval)
-   AJAX
-   Promise
-   async/await

### 3.1，setTimeout

    setTimeout(
      function() { 
        console.log("Hello!");
    }, 1000);

setTimout（setInterval）并不是立即就执行的，这段代码意思是，等 1s后，把这个 function 加入任务队列中，如果任务队列中没有其他任务了，就执行输出 'Hello'。

    var outerScopeVar; 
    helloCatAsync(); 
    alert(outerScopeVar);
    
    function helloCatAsync() {     
        setTimeout(function() {         
            outerScopeVar = 'hello';     
        }, 2000); 
    }
    
执行上面代码，发现 outerScopeVar 输出是 undefined，而不是 hello。之所以这样是因为在异步代码中返回的一个值是不可能给同步流程中使用的，因为 console.log(outerScopeVar) 是同步代码，执行完后才会执行 setTimout。

    helloCatAsync(function(result) {
    console.log(result);
    });
    
    function helloCatAsync(callback) {
        setTimeout(
            function() {
                callback('hello')
            }
        , 1000)
    }

把上面代码改成，传递一个callback，console 输出就会是 hello。

### 3.2， AJAX

    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function() {
        if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304 ) {
            console.log(xhr.responseText);
        } else {
            console.log( xhr.status);
        }
    }
    xhr.open('GET', 'url', false);
    xhr.send();
   

上面这段代码，xhr.open 中第三个参数默认为 false 异步执行，改为 true 时为同步执行。

### 3.3，Promise

语法：
> new Promise( function(resolve, reject) {...});

Promise 对象是由关键字 new 及其构造函数来创建的。这个“处理器函数”接受两个函数 resolve 和 reject 作为其参数。当异步任务顺利完成且返回结果值时，会调用 resolve 函数；而当异步任务失败且返回失败原因（通常是一个错误对象）时，会调用reject 函数。

new Promise 返回一个 promise 对象，在遇到 resolve 或 reject之前，状态一直是pending，如果调用 resolve 方法，状态变为 fulfilled，如果调用了 reject 方法，状态变为 rejected。


    function delay() {
        return new Promise(function(resolve, reject) {
            setTimeout(function() {
                resolve(666)
            }, 2000)
        })
    }
    
    delay()
        .then(function(value){
            console.log('resolve...',value);
        })
        .catch(function(err){
            cosole.log(err)
        })
    
上面代码中，2s 后调用 resolve 方法，然后调用 then 方法，没有调用 cacth方法。

**注意：then() 和 catch() 都是异步操作。**

下面 promise 和 ajax 结合例子：

    function ajax(url) {
        return new Promise(function(resolve, reject) {
            var xhr = new XMLHttpRequest();
            xhr.onreadystatechange = function() {
                if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304 ) {
                    resovle(xhr.responseText);
                } else {
                    reject( xhr.status);
                }
            }
            xhr.open('GET', url, false);
            xhr.send();
        });
    }
    
    ajax('/test.json')
        .then(function(data){
            console.log(data);
        })
        .cacth(function(err){
            console.log(err);
        });
    



> 学习1：[promise-mdn](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)

> 学习2：[promise-google](https://developers.google.com/web/fundamentals/primers/promises)

### 3.4，async/await

语法:

> async function name([param[, param[, ... param]]]) { statements }

调用 async 函数时会返回一个 promise 对象。当这个 async 函数返回一个值时，Promise 的 resolve 方法将会处理这个值；当 async 函数抛出异常时，Promise 的 reject 方法将处理这个异常值。

async 函数中可能会有**await 表达式，这将会使 async 函数暂停执行，等待 Promise 正常解决后继续执行 async 函数并返回解决结果(异步)**。

    function resolveAfter2Seconds(x) {
      return new Promise(resolve => {
        setTimeout(() => {
          resolve(x);
        }, 2000);
      });
    }
    
    async function add1(x) {
      var a = resolveAfter2Seconds(20);
      var b = resolveAfter2Seconds(30);
      return x + await a + await b;
    }
    
    add1(10).then(v => {
      console.log(v);  // 2s 后打印 60, 两个 await 是同时发生的，也就是并行的。
    });
    
    async function add2(x) {
      var a = await resolveAfter2Seconds(20);
      var b = await resolveAfter2Seconds(30);
      return x + a + b;
    }
    
    add2(10).then(v => {
      console.log(v);  // 4s 后打印 60，按顺序完成的。
    });

上面代码是 mdn 的一个例子。
1. 说明了 async 返回的是一个 promise对象；
2. await 后面跟的表达式是一个 promise，执行到 await时，函数暂停执行，直到该 promise 返回结果，并且暂停但不会阻塞主线程。
3. await 的任何内容都通过 Promise.resolve() 传递。
4. await 可以是并行（同时发生）和按顺序执行的。

看下面两段代码：

    async function series() {
        await wait(500);
        await wait(500);
        return "done!";
    }

    async function parallel() {
      const wait1 = wait(500);
      const wait2 = wait(500);
      await wait1;
      await wait2;
      return "done!";
    }

第一段代码执行完毕需要 1000毫秒，这段 await 代码是按顺序执行的；第二段代码执行完毕只需要 500 毫秒，这段 await 代码是并行的。


> 学习1：[async-mdn](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function)

> 学习2：[async-google](https://developers.google.com/web/fundamentals/primers/async-functions)
