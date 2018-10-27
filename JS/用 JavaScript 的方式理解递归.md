## 1. 递归是啥?

递归概念很简单，“自己调用自己”（下面以函数为例）。

在分析递归之前，需要了解下 JavaScript 中“压栈”（`call stack`） 概念。

## 2. 压栈与出栈

**栈**是什么？可以理解是在内存中某一块区域，这个区域比喻成一个箱子，你往箱子里放些东西，这动作就是**压栈**。所以最先放下去的东西在箱子最底下，最后放下去的在箱子最上面。把东西从箱子中拿出来可以理解为**出栈**。


所以得出结论，我们有个习惯，拿东西是从上面往下拿，最先放下去的东西在箱子的最底下，最后才能拿到。


在 JavaScript 中，调用函数时，都会发生压栈行为，遇到含 `return` 关键字的句子或执行结束后，才会发生出栈（pop）。


来看个例子，这个段代码执行顺序是怎样的？


    function fn1() {
        return 'this is fn1'
    }
    
    function fn2() {
        fn3()
        return 'this is fn2'
    }
    
    function fn3() {
        let arr = ['apple', 'banana', 'orange']
        return arr.length
    }
    
    function fn() {
        fn1()
        fn2()
        console.log('All fn are done')
    }
    
    fn()


上面发生了一系列压栈出栈的行为：

 1.  首先，一开始栈（箱子）是空的
 2.  函数 `fn` 执行，`fn` 先压栈，放在栈（箱子）最底下
 2.  在函数 fn 内部，先执行函数 `fn1`，`fn1` 压栈在 `fn` 上面
 3.  函数 `fn1` 执行，遇到 `return` 关键字，返回 `this is fn1`，`fn1` 出栈，箱子里现在只剩下 `fn` 
 4.  回到 `fn` 内部，`fn1` 执行完后，函数 `fn2` 开始执行，`fn2` 压栈，但是在 `fn2` 内部，遇到 `fn3`，开始执行 `fn3`，所以 `fn3` 压栈
 5.  此时栈中从下往上顺序为：`fn3` <= `fn2` <= `fn`，`fn` 在最底下
 6.  以此类推，函数 `fn3` 内部遇到 `return` 关键字的句子，`fn3` 执行完结束后出栈，回到函数 `fn2` 中 ，`fn2` 也是遇到 `return` 关键字，继续出栈。
 7.  现在栈中只有 `fn` 了，回到函数 `fn` 中，执行 `console.log('All fn are done'`) 语句后，`fn` 出栈。
 8.  现在栈中又为空了。


上面步骤容易把人绕晕，下面是流程图：

![压栈出栈流程图](https://user-gold-cdn.xitu.io/2018/10/25/166abe8b1fcb295e?w=899&h=574&f=png&s=12039)

再看下在 chrome 浏览器环境下执行：
 
 ![chrome 执行顺序](https://user-gold-cdn.xitu.io/2018/10/25/166abe8b2012ddb5?w=2256&h=1828&f=gif&s=2288256)


## 3. 递归

先看下简单的 JavaScript 递归

    function sumRange(num) {
      if (num === 1) return 1;
      return num + sumRange(num - 1)
    }
    
    sumRange(3) // 6
    
上面代码执行顺序：
1. 函数 `sumRange(3)` 执行，`sumRange(3)` 压栈，遇到 `return` 关键字，但这里还马上不能出栈，因为调用了 `sumRange(num - 1)` 即 `sumRange(2)`
2. 所以，`sumRange(2)` 压栈,以此类推，`sumRange(1)` 压栈，
3. 最后，`sumRange(1)` 出栈返回 1，`sumRange(2)` 出栈，返回 2，`sumRange(3)` 出栈
4. 所以 `3 + 2 + 1` 结果为 6

看流程图

 ![递归执行顺序](https://user-gold-cdn.xitu.io/2018/10/25/166abe8b202bfeca?w=882&h=947&f=png&s=31428)

**所以，递归也是个压栈出栈的过程。**

递归可以用非递归表示，下面是上面递归例子等价执行
    
    // for 循环
    function multiple(num) {
        let total = 1;
        for (let i = num; i > 1; i--) {
            total *= i
        }
        return total
    }
    multiple(3)

## 4. 递归注意点

如果上面例子修改一下，如下：

    function multiple(num) {
        if (num === 1) console.log(1)
        return num * multiple(num - 1)
    }
    
    multiple(3) // Error: Maximum call stack size exceeded

上面代码第一行没有 `return` 关键字句子，因为递归没有终止条件，所以会一直压栈，造成内存泄漏。

**递归容易出错点**
1. 没有设置终止点
2. 除了递归，其他部分的代码
3. 什么时候递归合适

 
好了，以上就是 JavaScript 方式递归的概念。

下面是练习题。
 
## 6. 练习题目

 1. 写一个函数，接受一串字符串，返回一个字符串，这个字符串是将原来字符串倒过来。
 2. 写一个函数，接受一串字符串，同时从前后开始拿一位字符对比，如果两个相等，返回 `true`，如果不等，返回 `false`。
 3. 编写一个函数，接受一个数组，返回扁平化新数组。
 4. 编写一个函数，接受一个对象，这个对象值是偶数，则让它们相加，返回这个总值。
 5. 编写一个函数，接受一个对象，返回一个数组，这个数组包括对象里所有的值是字符串

## 7. 参考答案

### 参考1

    function reverse(str) {
       if(str.length <= 1) return str; 
       return reverse(str.slice(1)) + str[0];
    }
    
    reverse('abc')
    
### 参考2

    function isPalindrome(str){ 
        if(str.length === 1) return true;
        if(str.length === 2) return str[0] === str[1]; 
        if(str[0] === str.slice(-1)) return isPalindrome(str.slice(1,-1)) 
        return false; 
    }
    
    var str = 'abba'
    isPalindrome(str)
    
### 参考3

    function flatten (oldArr) {
       var newArr = [] 
       for(var i = 0; i < oldArr.length; i++){ 
        if(Array.isArray(oldArr[i])){ 
            newArr = newArr.concat(flatten(oldArr[i])) 
        } else { 
            newArr.push(oldArr[i])
         }
       }
       return newArr;
    }
    
    flatten([1,[2,[3,4]],5])

### 参考4

    function nestedEvenSum(obj, sum=0) {
        for (var key in obj) { 
            if (typeof obj[key] === 'object'){ 
                sum += nestedEvenSum(obj[key]); 
            } else if (typeof obj[key] === 'number' && obj[key] % 2 === 0){ 
                sum += obj[key]; 
            }
         } 
        
         return sum;
    }
    
    nestedEvenSum({c: 4,d: {a: 2, b:3}})

### 参考5
    function collectStrings(obj) {
        let newArr = []
        for (let key in obj) {
            if (typeof obj[key] === 'string') {
                newArr.push(obj[key])
            } else if(typeof obj[key] === 'object' && !Array.isArray(obj[key])) {
               newArr = newArr.concat(collectStrings(obj[key]))
            }
        }
        return newArr
    }
    
    var obj = {
            a: '1',
            b: {
                c: 2,
                d: 'dd'
            }
        }
    
    collectStrings(obj)

## 5. 总结
递归精髓是，往往要先想好常规部分是怎样的，在考虑递归部分，下面是 JavaScript 递归常用到的方法
    
 - 数组：`slice`, `concat`
 - 字符串: `slice`, `substr`, `substring`
 - 对象：`Object.assign`
