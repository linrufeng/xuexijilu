---
layout: koa
title: 学习分享 koa 
date: 2018-09-06 17:55:19
tags:
---

koa是由express原班人马打造的一个更小、更富有表现力、更健壮的web框架。Koa 通过 node.js 实现了一个十分具有表现力的 HTTP 中间件框架。

在我眼中，koa的确是比express轻量的多，koa给我的感觉更像是一个中间件框架，koa只是一个基础的架子，需要用到的相应的功能时，用相应的中间件来实现就好，诸如路由系统等。

它和express 最大的不同 在于  express是基于回调来处理 而它 是基于 node 对 async/await 的支持,来代替回调。

### 执行顺序：

Koa 的中间件之间按照编码顺序在栈内依次执行，允许您执行操作并向下传递请求（downstream），之后过滤并逆序返回响应（upstream）。

### 功能 ： 

* HTTP 服务器通用的方法 
* 全部被集成了进去

### 安装 

`
$ npm install koa 
`

### koa 使用

打印 我们的标志性语言 hello world

```
const Koa = require('koa');
const app = new Koa();
app.listen(3000);
```

然后执行

`
 node demo.js

`

这时候发页面提示是NOT FOUND，因为我们并没有告诉浏览器应该显示什么内容。



### koa 原理

 koa是一个 **中间件框架** 可以采用两种不同的方法来实现中间件： 

* async function
* common function

#### Async 异步方法

Async 是一个流程控制工具包，提供了直接而强大的异步功能。基于 **JavaScript** 为 Node.js 设计，同时也可以直接在浏览器中使用。

#### 为何要使用 Async 机制

Async 为了解决 node.js 回调函数的嵌套问题。

#### Async 功能

* 流程控制
* 集合处理

series：用于依次执行多个方法，一个方法执行完毕后才会进入下 一方法，方法之间没有数据传递
tasks 需要执行多个方法。
tasks 可以以数组形式传入，也可以以 object 对象形式传入。每个方法都要一个回调方法
callback (err, result) 
用于处理错误或进入下一方法。当发生错误时（即：err 参数存在时），其后的方法会跳过，错误被传入最终回调方法中
不出错时，tasks 中回调结果将被写入 results 参数中，以数据或对象形式提供

```
Async.series(tasks,callback)
```



parallel：多个任务并发执行
waterfall：多个任务依次执行，上一任务的输出可做为下一任务的输入参数




#### 中间件原理分析

```
app.use(async (ctx, next) => {
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  console.log(`${ctx.method} ${ctx.url} - ${ms}ms`);
});
//// 中间件通常带有两个参数 (ctx, next), ctx 是一个请求的上下文（context）, 中间件通常带有两个参数 (c 
// next 是调用执行下游中间件的函数. 在代码执行完成后通过 then 方法返回一个 Promise.
app.use((ctx, next) => {
  const start = Date.now();
  return next().then(() => {
    const ms = Date.now() - start;
    console.log(`${ctx.method} ${ctx.url} - ${ms}ms`);
  });
});
```

随着node对于async/await的支持，koa 的中间件的 原理 koa-compose：

```
function compose (middleware) {
  // 参数检验
  return function (context, next) {
    // last called middleware #
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      // 最后一个中间件的调用
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      // 用Promise包裹中间件，方便await调用
      try {
        return Promise.resolve(fn(context, function next () {
          return dispatch(i + 1)
        }))
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```





