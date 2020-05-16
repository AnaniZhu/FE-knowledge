# Event Loop

### 浏览器

- `script` 执行
- `Promise.then` 的回调、`MutationObserver`

- `requestAnimationFrame `
- `layout`
-  `resizeObserver`
- `paint`
- `setTimeout` & `setInterval` & `IO`
- `requestIdleCallback` & `IntersectionObserver`



### Node

- `Timer` 阶段。执行 `setTimeout` `setInterval`
- `Pending callback` 阶段。延迟到下一个事件循环的 IO 回调。
- `idle` 和 `prepare` 阶段。系统内置使用。
- `poll` 轮询阶段。处理除 `timer` `Pending callback` 以及 `setImmediate` 之外的 IO 回调
- `check` 检测阶段。执行 `setImmediate`
- `close callback` 关闭回调阶段。执行 `xxx.on('close')` 的回调函数


### 参考链接
- [面试题：说说事件循环机制(满分答案来了)](https://juejin.im/post/5e5c7f6c518825491b11ce93)
- [Node.js 事件循环，定时器和 `process.nextTick()`](https://nodejs.org/zh-cn/docs/guides/event-loop-timers-and-nexttick/)





## 0.1 + 0.2 为什么不等于 0.3 ?

- [JavaScript 浮点数陷阱及解法 ](https://github.com/camsong/blog/issues/9)
- [0.1 + 0.2不等于0.3？为什么JavaScript有这种“骚”操作？](https://juejin.im/post/5b90e00e6fb9a05cf9080dff)



## Object.is 与 ===

`Object.is` 解决两个问题:

- `+0` & `-0`
- `NaN` 和 `NaN`



## 什么是闭包

> *Closure is when a function is able to remember and access its lexical scope even when that function is executing outside its lexical scope.*



### `Curry`

函数柯里化就是将一个接受多个参数的函数，转换为接受单个参数的函数，并且该函数返回一个接受剩余参数且返回结果的新函数。