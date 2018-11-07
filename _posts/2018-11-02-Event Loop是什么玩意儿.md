---
layout: post
title: Event Loop是什么玩意儿
date: 2018-11-02
categories: javascript
tags: javascript
---

## 相关概念

- Call Stack：执行栈，是同步任务执行环境
- Task Queue：任务队列，异步任务运行结果的等待环境
- Web APIs：浏览器中包含异步进程的服务
- Macrotask：可理解为执行栈中的一个任务（包括从任务队列进入到执行栈中的任务）
- Microtask：可理解为执行栈中任务完成后立即执行的任务
- Event Loop：主线程从Task Queue中读取事件（循环）的过程

**Macrotask范畴：**

- 同步任务代码块
- setTimeout、setInterval、setImmediate
- I/O
- UI渲染

**Microtask范畴：**

- Promise（在大部分浏览器环境中）
- process.nextTick
- MutationObserver
- Object.observe

## Event Loop事件循环过程

1. Call Stack中的同步任务被顺序执行。
2. Call Stack中的异步任务（宏任务）会被交给Web APIs去维护，执行结果被放到Task Queue中等待。
3. Call Stack中的异步任务（微任务）会被放到微任务队列中等待执行。
4. 当Call Stack中任务执行完毕后，会先执行完微任务队列的所有程序，再从Task Queue中读取第一个任务结果到Call Stack中运行（此时为同步任务）。
5. 重复1步骤，并按此顺讯依次执行。

![](https://img-blog.csdnimg.cn/20181107110749956.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xlcnJ5dGVuZw==,size_16,color_FFFFFF,t_70)

## 实例说明

**任务代码：**

<pre><code class="language-JavaScript">
console.log('start')

const interval = setInterval(() => {
  console.log('setInterval')
}, 0)

setTimeout(() => {
  console.log('setTimeout 1')
  Promise.resolve()
      .then(() => {
        console.log('promise 3')
      })
      .then(() => {
        console.log('promise 4')
      })
      .then(() => {
        setTimeout(() => {
          console.log('setTimeout 2')
          Promise.resolve()
              .then(() => {
                console.log('promise 5')
              })
              .then(() => {
                console.log('promise 6')
              })
              .then(() => {
                clearInterval(interval)
              })
        }, 0)
      })
}, 0)

Promise.resolve()
    .then(() => {
        console.log('promise 1')
    })
    .then(() => {
        console.log('promise 2')
    })
</code></pre>

**执行结果：**

<pre><code class="language-JavaScript">
start
promise 1
promise 2
setInterval
setTimeout 1
promise 3
promise 4
setInterval
setTimeout 2
promise 5
promise 6
</code></pre>

#### 参考：

[https://juejin.im/post/5a6309f76fb9a01cab2858b1]()
[https://juejin.im/post/5a6547d0f265da3e283a1df7]()
[http://www.ruanyifeng.com/blog/2014/10/event-loop.html]()

