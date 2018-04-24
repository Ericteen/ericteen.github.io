---
layout:     post
title:      "Event Loop 在浏览器和 NodeJS 中的差异"
subtitle:   "event loop"
date:       2018-03-10 12:00:00
author:     "Ericteen"
header-img: "img/post-bg-03.jpg"
---
# Event Loop 在浏览器和 NodeJS 中的差异

## Prerequisite

JS 是单线程的，Event Loop 机制来实现异步

> 事件循环发生的前提: 所有代码都在主线程调用栈完成执行。
>
> 事件循环发生的时机； 当主线程中的任务清空之后，事件循环任务队列中的任务。

讨论的主题是，Event Loop 机制在浏览器和 NodeJS 中的区别。

## Browsing Context

Event Loop 在 HTML 规范中的定义

> 为了协调事件、用户交互、脚本、UI渲染、网络请求等行为，用户引擎必须使用Event Loop。EL包含两类：基于browsing contexts，基于worker。二者独立。

### 图解 Event Loop

![Event Loop](/img/event-loop/event-loop.jpg)

- JavaScript 的特点就是单线程，而这个线程中拥有唯一的一个事件循环
- JavaScript 代码在执行过程中，除了依靠函数调用栈确定函数执行顺序外，还依靠任务队列来确定另一些代码的执行
- 一个线程中，事件循环是唯一的，而任务队列却可以有多个
- 任务队列又分为macro-task（宏任务）与micro-task（微任务），在最新标准中，它们被分别称为task与jobs。
- macro-task大概包括：script(整体代码), setTimeout, setInterval, setImmediate, I/O, UI rendering。
- micro-task大概包括: process.nextTick, Promise, Object.observe(已废弃), MutationObserver(html5新特性)
- setTimeout/Promise等我们称之为任务源。而进入任务队列的是他们指定的具体执行任务。
- 同步任务直接进入调用栈中执行
- 等待调用栈中任务执行完毕，由 Event Loop 将异步任务推入主执行栈中执行
- 事件循环的顺序，决定了JavaScript代码的执行顺序。它从script(整体代码)开始第一次循环。之后全局上下文进入函数调用栈。直到调用栈清空(只剩全局)，然后执行所有的micro-task。当所有可执行的micro-task执行完毕之后。循环再次从macro-task开始，找到其中一个任务队列执行完毕，然后再执行所有的micro-task，这样一直循环下去。

### task

一个 Event Loop 中有一个或多个task队列，来自不同任务源的 task 会放入不同的 task 队列中，比如，用户代理会为鼠标键盘事件分配一个 task 队列，为其他的事件分配另外的队列。

典型的任务源有如下几个([generic task sources](https://www.w3.org/TR/html5/webappapis.html#generic-task-sources)):

- DOM 操作任务源：响应 DOM 操作
- 用户交互任务源：鼠标事件
- 网络任务源：响应网络活动
- history traversal任务源：当调用history.back()等类似的api时，将任务插进task队列

task在网上也被成为macrotask 可能是为了和 microtask 做对照。但是规范中并不是这么描述任务的。

除了上述task来源，常见的来源还有 数据库操作、setTimeout/setInterval等，可以概括为以下几种：

- script 代码
- setTimeout/setInterval
- I/O
- UI 交互
- setImmediate(nodejs 环境中)

### Microtask

一个EL中只有一个microtask队列，通常下面几种任务被认为是microtask：

- promise（promise的then和catch才是microtask，本身其内部的代码并不是）
- MutationObserver
- process.nextTick(nodejs 环境中)

### Event Loop 循环过程

1. 在所有task队列中选择一个最早进队列的task，用户代理可以选择任何task队列，如果没有可选的任务，则跳到Microtasks步骤
2. 将上边选择的task设置为正在运行的task
3. Run: 运行被选择的task
4. 将event loop的 currently running task 置为 null
5. 从task队列里移除前边Run里运行的task
6. Microtasks: 执行microtasks任务检查点。（也就是执行microtasks队列里的任务）
7. 更新渲染
8. 如果这是一个worker event loop，但是task队列中没有任务，并且WorkerGlobalScope对象的closing标识为true，则销毁EL，中止这些步骤，然后 run a worker
9. 返回到第1步

简化以上步骤：

一个宏任务，所有微任务（，更新渲染），一个宏任务，所有微任务（，更新渲染）......

执行完microtask队列里的任务，有可能会渲染更新。在一帧以内的多次dom变动浏览器不会立即响应，而是会积攒变动以最高60HZ的频率更新视图。

![图示](/img/event-loop/browser-el.jpg)

例子：

```javascript
setTimeout(() => console.log('setTimeout1'), 0);    // 1#
setTimeout(() => {                  // 2#
    console.log('setTimeout2');     // 2-1#
    Promise.resolve().then(() => {  // 2-2#
        console.log('promise2');        // 2-2-1#
        Promise.resolve().then(() => {  // 2-2-2#
            console.log('promise3');
        })
        console.log(5)                  // 2-2-3#
    })
    setTimeout(() => console.log('setTimeout4'), 0);    // 2-3#
}, 0);
setTimeout(() => console.log('setTimeout3'), 0);    // 3#
Promise.resolve().then(() => {  // 4#
    console.log('promise1');
})
```

所以，运行结果为：

```javascript
promise1
setTimeout1
setTimeout2
promise2
5
promise3
setTimeout3
setTimeout4
```
