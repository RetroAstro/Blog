> ### JavaScript 异步编程

### 前言

自己着手准备写这篇文章的初衷是觉得如果想要更深入的理解 JS，异步编程则是必须要跨过的一道坎。由于这里面涉及到的东西很多也很广，在初学 JS 的时候可能无法完整的理解这一概念，即使在现在来看还是有很多自己没有接触和理解到的知识点，但是为了跨过这道坎，我仍然愿意鼓起勇气用我已经掌握的部分知识尽全力讲述一下 JS 中的异步编程。如果我所讲的一些概念或术语有错误，请读者向我指出问题所在，我会立即纠正更改。

### 同步与异步

我们知道无论是在浏览器端还是在服务器 ( Node ) 端，JS 的执行都是在单线程下进行的。我们以浏览器中的 JS 执行线程为例，在这个线程中 JS 引擎会创建执行上下文栈 ( 全局、函数、eval )，这个时候我们的代码则会像一系列的任务一样在这些执行栈中按照后进先出 ( LIFO ) 的方式依次执行。而同步最大的特性就是会阻塞后面任务的执行，比如此时 JS 正在执行大量的计算，这个时候就会使线程阻塞从而导致页面渲染加载不连贯 ( 在浏览器端的 Event Loop 中每次执行栈中的任务执行完毕后都会去检查并执行事件队列里面的任务直到队列中的任务为空，而事件队列中的任务又分为微队列与宏队列，当微队列中的任务执行完后才会去执行宏队列中的任务，而在微队列任务执行完到宏队列任务开始之前浏览器的 GUI 线程会执行一次页面渲染 ( UI rendering )，这也就解释了为什么在执行栈中进行大量的计算时会阻塞页面的渲染 ) 。

与同步相对的异步则可以理解为在异步操作完成后所要做的任务，它们通常以回调函数或者 Promise 的形式被放入事件队列，再由事件循环 ( Event Loop ) 机制在每次轮询时检查异步操作是否完成，若完成则按事件队列里面的执行规则来依次执行相应的任务。也正是得益于事件循环机制的存在，才使得异步任务不会像同步任务那样完全阻塞 JS 执行线程。

异步操作一般包括  **网络请求** 、**文件读取** 、**数据库处理**

异步任务一般包括  **setTimout / setInterval** 、**Promise** 、**requestAnimationFrame ( 浏览器独有 )** 、**setImmediate ( Node 独有 )** 、**process.nextTick ( Node 独有 )** 、**etc ...**

> **注意：** 在浏览器端与在 Node 端的 Event Loop 机制是有所不同的，下面给出的两张图简要阐述了在不同环境下事件循环的运行机制，由于 Event Loop 不是本文内容的重点，但是 JS 异步编程又是建立在它的基础之上的，故在下面给出相应的阅读链接，希望能够帮助到有需要的读者。

**浏览器端**

![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee30db2997?imageslim)

**Node 端**

![](https://user-gold-cdn.xitu.io/2018/5/21/163817de4a1ca52c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**阅读链接**

* **[深入理解 JS 事件循环机制 ( 浏览器篇 )](http://lynnelv.github.io/js-event-loop-browser)**
* **[深入理解 JS 事件循环机制 ( Node.js 篇 )](http://lynnelv.github.io/js-event-loop-nodejs)**

### 为异步而生的 JS 语法

回望历史，在最近几年里 ECMAScript 标准几乎每年都有版本的更新，也正是因为有像 ES6 这种在语言特性上大版本的更新，到了现今的 8102 年， JS 中的异步编程相对于那个只有回调函数的远古时代有了很大的进步。下面我将介绍 callback 、Promise 、generator 、async / await 的基本用法，为后文几种异步模式不同版本的逻辑代码实现打下基础。

#### callback

回调函数并不算是 JS 中的语法但它却是解决异步编程问题中最常用的一种方法，所以在这里有必要提出来，下面举一个例子，大家看一眼就懂。

```js
const foo = function (x, y, cb) {
    setTimeout(() => {
        cb(x + y)
    }, 2000)
}

// 使用 thunk 函数，有点函数柯里化的味道，在最后处理 callback。
const thunkify = function (fn) {
    return function () {
        let args = Array.from(arguments)
        return function (cb) {
            fn.apply(null, [...args, cb])
        }
    }
}

let fooThunkory = thunkify(foo)

let fooThunk1 = fooThunkory(2, 8)
let fooThunk2 = fooThunkory(4, 16)

fooThunk1((sum) => {
    console.log(sum) // 10
})

fooThunk2((sum) => {
    console.log(sum) // 20
})
```

#### Promise

在 ES6 没有发布之前，作为异步编程主力军的回调函数一直被人诟病，其原因有太多比如回调地狱、代码执行顺序难以追踪、后期因代码变得十分复杂导致无法维护和更新等，而 Promise 的出现在很大程度上改变了之前的窘境。话不多说先直接上代码看看它的用法，然后再总结下我自己认为在 Promise 中很重要的几个点。

```js
const foo = function () {
    let args = [...arguments]
    let cb = args.pop()
    setTimeout(() => {
        cb(...args)
    }, 2000)
}

const promisify = function (fn) {
    return function () {
        let args = [...arguments]
        return function (cb) {
            return new Promise((resolve, reject) => {
                fn.apply(null, [...args, resolve, reject, cb])
            })
        }
    }
}

const callback = function (x, y, isAdd, resolve, reject) {
    if (isAdd) {
        setTimeout(() => {
            resolve(x + y)
        }, 2000)
    } else {
        reject('Add is not allowed.')
    }
}

let promisory = promisify(foo)

let p1 = promisory(4, 16, false)
let p2 = promisory(2, 8, true)

p1(callback)
.then((sum) => {
    console.log(sum)
}, (err) => {
    console.error(err) // Add is not allowed.
})

p2(callback)
.then((sum) => {
    console.log(sum) // 10
    return 'evil 😡'
})
.then((unknown) => {
    throw new Error(unknown)
})
.catch((err) => {
    console.error(err) // Error: evil 😡
})
```

### 常见异步模式

在 JS 异步编程的世界里，很多时候我们会遇到因为是异步操作而出现的特定问题，而针对这些问题所提出的解决方案 ( 逻辑代码 ) 就是异步编程的核心，在这里我更愿意把一个问题和对应的解决方案统称为模式，在一种模式中问题总是一样的，但是解决方案却因为 JS 强大的语言特性而变得多样化。下面我将介绍常见的几种异步模式，并通过上文所讲的几种 JS 语法来实现每一种模式下四个版本的逻辑代码。

**并发交互模式**

当我们在同时执行多个异步任务时，这些任务的返回结果到达的时间往往是不确定的，因而会产生以下两种常见的场景：

1. 多个异步任务同时执行，等待所有任务都返回结果后才开始进行下一步的操作。
2. 多个异步任务同时执行，只返回最先完成异步操作的那个任务的结果然后再进行下一步的操作。

**并发协作模式**

有时候我们会遇到在异步任务中处理大量数据的情景，由于会阻塞线程故我们可以将其分割成多个步骤或多批任务，使得其他并发进程 ( 如渲染进程 ) 有机会将自己的运算插入到事件循环队列中交替进行。

**事件监听模式**

采用事件驱动的形式，任务的执行不取决于代码的顺序，而取决于某个事件是否发生，若事件发生则该事件指定的回调函数就会被执行。

**发布 / 订阅模式**

我们假定，存在一个"信号中心"，某个任务执行完成，就向信号中心"发布" ( publish ) 一个信号，其他任务可以向信号中心"订阅" ( subscribe ) 这个信号，从而知道什么时候自己可以开始执行。这种方法与"事件监听"类似，但是明显优于后者，因为我们可以通过查看"消息中心"，了解存在多少信号、每个信号有多少订阅者，从而监控程序的运行。