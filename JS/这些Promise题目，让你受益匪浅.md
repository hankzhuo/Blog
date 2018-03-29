## 引言

promise我看了好机会，但做题时候都会出错，后来我知道学一个知识点，光看不动手敲代码是不够的。下面几道题做完，真的受益匪浅，题目大部分来自网上，文章后面会提到。

首先，promise 基础知识可以看 mdn。

>  [promise-mdn](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)

## 题目

### 第一题

    const request = (url) => {
      return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        xhr.open('GET', url);
        xhr.onload = () => resolve(xhr.responseText);
        xhr.onerror = () => reject(xhr.statusText);
        xhr.send();
      })
    }
    
    request(url1)
      .then(data1 => request(data1.url))
      .then(data2 => request(data2.url))
      .catch(err => { 
         throw new Error(err)
      })
        
```request``` 是个```promise```对象，有三个状态 ```pedding、fulfilled、rejected```。初始状态为 ```pendding```，调用 ```resolve```时，状态变为 ```fulfilled```，调用```rejected```时，状态变为 ```rejected```。


### 第二题

    const promise  = new Promise((resolve, reject) => {
      // resolve('fulfilled');
      reject('111');
    })
    
    promise.then(result => {
      console.log('success...', result)
    }, err => {
      console.log("err...", err);
    })

```promise.then(onFulfilled, onRejected) ```，```then```接受两个参数，```onFulfilled``` 成功时调用，```onRejected``` 失败时候调用。 一次只会调用其中一个函数。

### 第三题

    const p1 = new Promise((resolve, reject) => {
        resolve(1);
    });
    
    const p2 = new Promise((resolve, reject) => {
        resolve(2);
    });
    
    const p3 = new Promise((resolve, reject) => {
        resolve(3);
    });
    
    Promise.all([p1, p2, p3])
      .then(data => console.log(data))  // [1, 2, 3]
      .catch(err => console.log('err...',err))
    
    Promise.race([p1, p2, p3])
      .then(data => console.log(data)) // 1
      .catch(err => console.log("err...", err));

- ```Promise.all()```等待所有完成后才调用 ```then```
- ```Promise.race()``` 谁先处理完就直接调用后面函数
- ```Promise.resolve()``` 返回一个 ```fulfilled``` 状态的```promise```对象
- ```Promise.reject()``` 返回一个 ```rejected``` 状态的promise对象    


### 第四题

    const promise = new Promise((resolve, reject) => {
      console.log(1);
      resolve('resolve');
      console.log(2);
    });
    
    console.log(promise);
    
    promise.then(() => {
      console.log(3)
    });
    
    console.log(4); 
    
    // 1 2 Promise { 'resolve' } 4 3

- Promise 构造函数是同步执行的，promise.then 中的函数是异步执行的
- 注意：上面 promise 内，resolve() 是同步代码，所以立刻被执行了

### 第五题

    const promise1 = new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve('success')
      }, 1000)
    })
    const promise2 = promise1.then(() => {
      throw new Error('error!!!')
    })
    
    console.log('promise1', promise1)
    console.log('promise2', promise2)
    
    setTimeout(() => {
      console.log('promise1', promise1)
      console.log('promise2', promise2)
    }, 2000)
    
    // promise1 Promise { <pending> }
    // promise2 Promise { <pending> }
    // promise1 Promise { 'success' }
    // promise2 Promise {
    //  <rejected> Error: error!!!

```promise``` 有 3 种状态：```pending、fulfilled``` 或 ```rejected```。状态改变只能是 ```pending->fulfilled``` 或者 ```pending->rejected```，状态一旦改变则不能再变。上面 ```promise2``` 并不是 ```promise1```，而是返回的一个新的 ```Promise``` 实例

### 第六题
    
    Promise.resolve()
      .then(() => {
        return new Error("error!!!");
      })
      .then(res => {
        console.log("then: ", res);
      })
      .catch(err => {
        console.log("catch: ", err);
      });
      
    //输出 then: Error: error!!!

- ```.then``` 或者 ```.catch``` 中 ```return``` 一个 ```error``` 对象并不会抛出错误，所以不会被后续的 .catch 捕获
- 因为返回任意一个非 ```promise``` 的值都会被包裹成 ```promise ```对象，即 ```return new Error('error!!!') ```等价于 ```return Promise.resovle(new Error('error!!!'))```

#### 6.1

    Promise.resolve()
      .then(() => {
        throw new Error("error!!!");
        // return Promise.reject(new Error("error!!!"));
      }, err => {
        console.log('then1: ', err)
      })
      .then(res => {
        console.log("then2: ", res);
      })
      .catch(err => {
        console.log("catch: ", err);
      });
      
    //输出 catch:  Error: error!!!

第一个 ```then``` 抛出错误，需要后面的捕获错误。

#### 6.2

    Promise.reject(1)
      .then(() => {
        return new Error('error!!!')
      })
      .then((res) => {
        console.log('then: ', res)
      })
      .catch((err) => {
        console.log('catch: ', err)
      });
      
      // 输出 catch 1  
     
前面两个``` then ```并没有捕捉错误，所以错误抛到了最后面的```catch```，中间 ```then``` 并不执行了。


#### 6.3

    Promise.reject(1)
      .then(() => {
        return new Error("error!!!");
      })
      .then(res => {
        console.log("then: ", res);
      }, err => {
        console.log("haha", err);
      })
      .catch(err => {
        console.log("catch: ", err);
      });
      
    //输出 haha 1
  
 错误被第二个 ```then ```捕捉了，最后的 ```catch``` 也就不执行了。

### 第七题

    var p = new Promise(function(resolve, reject){
      resolve(1);
    });
    p.then(function(value){               //第一个then
      console.log(value);
      return value*2;
    }).then(function(value){              //第二个then
      console.log(value);
    }).then(function(value){              //第三个then
      console.log(value);
      return Promise.resolve('resolve'); 
    }).then(function(value){              //第四个then
      console.log(value);
      return Promise.reject('reject');
    }).then(function(value){              //第五个then
      console.log('resolve: '+ value);
    }, function(err){
      console.log('reject: ' + err);
    })
    
    // 1 
    // 2
    // undefined ，因为上一个 then() 没有返回值
    // resolve 
    // reject: reject

### 第八题

    Promise.resolve(1)
      .then(2)
      .then(Promise.resolve(3))
      .then(console.log);
      
    // 1
    
```.then ```或者 ```.catch``` 的参数期望是函数，传入非函数则会发生值穿透。

### 第九题

    const promise = Promise.resolve().then(() => {
      return promise;
    });
    promise.catch(console.error);
    
    // TypeError: Chaining cycle detected for promise #<Promise>

```.then ```或 ```.catch`` 返回的值不能是``` promise``` 本身，否则会造成死循环。


## 参考
- [promise-mdn](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [Promise 必知必会（十道题）
](https://juejin.im/post/5a04066351882517c416715d)
- [八段代码彻底掌握 Promise
](https://juejin.im/post/597724c26fb9a06bb75260e8)