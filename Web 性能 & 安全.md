# Web 性能 & 安全

## 性能

### 优化

主要分为以下几大类

1. 减少 `http` 请求数量或缩短请求时间
    - 静态资源压缩（js, css, img）(缩短请求时间)
    - 静态资源 cdn (缩短请求时间，绕开单域名请求最大并发数)
    - 精灵图 （减少 `http` 请求），WebP 格式图片，图标 SVG
    - 合理采用内联图片，首屏外图片懒加载
    - 合理利用缓存 （减少 `http` 请求）
    - 升级至 `http 2.0`
    - 代码分割，降低首屏时间
2. 请求前置, 减少用户等待时间
    - `preload`
        - 预加载静态资源
    - `prefetch`
        - 浏览器在空闲时获取将来可能用到的资源并放入缓存
    - `prerender`
        - 后台预渲染整个页面
    - `dns-prefetch`
        - 提前 dns 解析
    - `preconnect`
        - `HTTP` 请求正式发送前预先执行的操作：`DNS`  解析 、`TCP` 握手、`TLS` 协商
    - `Script` async & defer
        - `async`: 异步加载 `script` ，加载完立即执行，先加载完先执行。
        - `defer`: 异步加载 `script`, 等到文档解析后， `DOMContentLoaded` 事件触发前执行，按照加载顺序执行(现代浏览器不一定遵循)。
3. 浏览器
    - `CSS` 放页面顶部，`Script` 放页面底部
    - `CSS` 选择器嵌套层级不要太深
    - 事件委托 （减少内存）
4. 减少回流与重绘
    - `DOM` 部分 API (`offsetXXX` `clientXXX` `scrollXXX` `getComputedStyle` `getBoundingClientRect` ...) 结果多用变量存储缓存，防止每次调用触发强制渲染
    - 大量样式修改尽量采用移除/添加类名的方法，避免 `style` 逐个修改
    - 大量 DOM 操作合理利用 `documentFragment`
    - 优先采用 CSS 动画，JS 动画采用 `requestAnimationFrame`, 复杂动画元素可先脱离文档流，动画结束回归文档流

### 回流 & 重绘

回流: 页面发生重新布局就触发了回流

重绘: 页面中元素样式的改变并不影响它在文档流中的位置时会发生重绘

浏览器会将回流和重绘放置到一个队列，当发生渲染前或任务数量&时间间隔达到了一定阈值，浏览器会清空队列。

当脚本中执行了某些 `API` 时，浏览器也会立即清除队列:

- `clientXXX`
- `offsetXXX`
- `scrollXXX`
- `style.width/height`
- `getComputedStyle`
- `getBoundingClientRect`

### 白屏时间 & 首屏时间

#### 白屏时间

概念：页面开始加载到页面开始显示内容所花费的时间

白屏时间统计方案:

- 一般认为解析完 Head 或者开始解析 body 就是白屏的结束时间，可在对应处放置 `script` 脚本计算时间

#### 首屏时间

概念：页面开始加载到首屏内容全部渲染完成所花费的时间

首屏时间统计方案:
- 因为浏览器解析 DOM 是顺序解析，对于内容固定且没有异步资源的首屏情况下，可在首屏末尾放置 `script` 标签计算首屏时间(`Date.now`)。
- 如果首屏有图片，一般图片都是最慢加载完成的，可以遍历首屏所有图片，监听 `load` 事件取最大值

### 渐进增强 & 优雅降级

#### 渐进增强

一开始只对低版本浏览器构建功能，保证基本功能，再根据高级浏览器针对交互和效果优化

#### 优雅降级

一开始构建完整的功能，再针对及版本浏览器做兼容

### 浏览器缓存

下列按照缓存优先级排序

- 强缓存
    - `Cache-Control: max-age=<seconds>`
    - `expired: <Date>`
      - 缺点：客户端可能与服务端时间不一致
    - `(Date - Last-modified) / 10`
- 协商缓存
    - `If-None-Match` & `ETag`
      - 缺点：需要根据文件内容计算出哈希值，针对大文件得用特殊算法（分片、......），比较占用 CPU 和耗时
    - `If-Modified-Since` & `Last-modified`
      - 缺点1：有时候文件的更新时间改了，但是文件内容并没有修改
      - 缺点2：最小单位是秒，同一秒可能存在多次修改

#### Cache-control
- `max-age=<seconds>`
    - 缓存的最大时间，单位秒
    - 协同 `must-revalidate` 一起使用，缓存过期前使用缓存，否则去服务器进行有效期校验
- `no-store`
    - 禁用缓存, 强制使用最新资源
- `no-cache`
    - 忽略强缓存，直接走协商缓存
- `public`
    - 可以被中间人缓存（代理、CDN）
- `private`
    - 中间人不能缓存此响应
- `s-maxage`
    - 针对代理服务器的缓存时间

#### `Pragma: no-cache`

同 `Cache-control: no-store`, 可兼容至 `http1.0`

#### 参考链接

- [理解浏览器缓存以及304状态码](https://juejin.im/post/5a142fab6fb9a044fb076322)

### CDN

[CDN是什么？使用CDN有什么优势？](https://www.zhihu.com/question/36514327/answer/193768864)

## 安全

### XSS

跨站脚本攻击。通过主域恶意脚本达到攻击的目的。

类别:
- 存储型
- 反射型
- DOM 型

防范:
- 服务端业务对输入字符进行长度限制和特殊转义
- `cookie` 设置为 `http-only` 防止被恶意获取
- 前端尽量避免 `innerHTML` & `outerHTML` & `document.write` 等属性方法，如必须使用需做好字符过滤和转义
- [CSP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP)

#### 参考链接
- [前端安全系列（一）：如何防止XSS攻击？](https://tech.meituan.com/2018/09/27/fe-security.html)

### CSRF

用户访问一个恶意网站，该恶意网站构造请求访问 A 网站，该请求会自动携带上同源 (`A`) 的 `cookie`，导致恰好用户前不久访问过 A 网站，其 `cookie` 中的 `sessionId` 还没过期，这个 `sessionId` 被请求携带从而通过服务端校验，从而请求被正确执行。


防护:
- 验证 `referer` 和 `Origin`
- 采用 `token` 由客户端主动发送，防止请求会携带同源 `cookie`的特性被利用。
    - 前提是不被 `XSS` 攻击，如果已经被 `XSS` 攻击，无论 `token` 存在哪，都会被攻击者拿到。
- 设置 `cookie` 的`sameSite`
- 对敏感接口增加验证码校验

#### 参考链接
- [前端安全系列之二：如何防止CSRF攻击？](https://juejin.im/post/5bc009996fb9a05d0a055192)

### 同源策略

同源策略是浏览器对网页安全性的一种防护措施。

同源必须满足 `同协议`、`同主机号`、`同端口` 三个条件，只要有一个不满足，就属于不同源，从而会产生跨域问题。

### 跨域

因浏览器同源策略的限制，产生了跨域问题。

广义上的跨域一般指的是不同域之间的接口(`ajax`、`fetch`)通信问题。

除此之外，浏览器中不同窗口 (`iframe`, `window.open`产生的页面)也存在跨域问题。

非同源下有以下限制：

- `ajax` / `fetch` 通信
- `cookie` / `localstorage` / `indexDB` 获取
- 获取 `DOM `

#### 跨域方案
#### `CORS`

`CORS` 是一种机制，通过额外的 `HTTP` 头部来告诉浏览器 web 应用允许访问不同源服务器上的资源。

`CORS` 算是目前最主流也相对来说最可靠的跨域方案，所的控制都在服务端完成。

服务端可通过添加响应头部来告知浏览器允许跨域的行为和能力:

- `Access-Control-Allow-Origin`。允许请求的 `origin`。可设置为通配符 `*` 表示信任所有请求。当设置为通配符时，`Acess-Control-Allow-Credentials` 无效。
- `Access-Control-Allow-Methods`。在对 `OPTIONS` 预检请求中明确了客户端访问资源允许的请求方法。
- `Access-Control-Allow-Credentials`。允许通信携带 `cookie`，要协同客户端 `xhr.withCredentials = true` 一起使用。只有两者都设置了，服务端 `Set-Cookie` 才有效。
- `Access-Control-Allow-Headers`。在对 `OPTIONS` 预检请求中明确了客户端访问资源允许的请求首部。默认允许以下简单头部:
    - `Accept`
    - `Accept-Language`
    - `Content-Language`
    - `Content-Type` 为 `application/x-www-form-urlencoded` 或 `text/plain` 或 `multipart/form-data`
    - 还有一些客户端头部...详见 [MDN 简单头部](https://developer.mozilla.org/zh-CN/docs/Glossary/%E7%AE%80%E5%8D%95%E5%A4%B4%E9%83%A8)
- `Access-Control-Expose-Headers`。允许`xhr.getResponseHeader()`访问到的响应首部。默认值如下:
    - `Cache-Control`
    - `Content-Language`
    - `Content-Type`
    - `Last-modified`
    - `Pragma`
- `Access-Control-Max-Age`。`OPTIONS` 预检请求的有效时间（s）。默认为 `0`， 复杂请求每次发送前都要发送一个预检请求。

`CORS`也需要设置对应的请求首部字段:
- `Origin`
- `Access-Control-Request-Method`
- `Access-Control-Request-Headers`

不过浏览器已经默认设置了这些首部字段，无需开发者手动设置。

跨域请求分为「简单请求」和「非简单请求」，当请求属于非简单请求时，浏览器会先发送一个预检请求(`OPTIONS`)，浏览器根据服务端返回的响应首部来决定此非简单请求是否可以发送。

「简单请求」的定义:

使用下列方法之一:
- `GET`
- `POST`
- `HEAD`

不得设置一下集合之外的请求首部字段:
- `Accept`
- `Accept-Language`
- `Content-Language`
- `Content-Type`
    - `application/x-www-form-urlencoded`
    - `application/plain`
    - `multipart/form-data`
- `DPR`
- `Downlink`
- `Save-Data`
- `Viewport-Width`
- `Width`

只要不满足上面任一条件，皆属于「非简单请求」

##### 限制
- 浏览器兼容问题（`IE10+`）

#### `JSONP`

##### 概念

`JSONP` 利用浏览器不对 `script` 标签做同源策略限制的特性而实现。

##### 原理

`JSONP` 原理就是创建一个 `script` 标签，`src` 属性设置为接口的地址并传入参数和回调函数名，同时在全局上定义这个回调函数。当请求返回会自动执行返回的内容 (`script`标签特性)，请求返回内容则是执行回调函数并传参(`callback({xxx})`)，从而达到返回数据的目的。

##### 限制

- 只支持 `GET`
- 存在安全问题
    - `JSONP 劫持` (`CSRF`)
        - 产生问题和防护同 [CSRF](#CSRF)
    - `XSS`
        - 因为 `callback` 一般都支持自定义，可以改成 `script` 脚本恶意攻击。
        - 防护: `Content-Type` 改成 `application/json`


#### `postMessage`
多窗口 (`window`) 之间的通信。调用 `postMessage` 方法和监听 `message` 事件达到通信的目的。

#### `iframe + hash`

通过增加一个和主域 `url` 一致的 `iframe`做中间代理来传递数据。

假设 A， B 两个不同域的页面，B 为 A 的 `iframe`，再增加一个 C 作为 B 的 `iframe`，且 C 和 A 属于同域。

A -> B 通信

A 向 B 传输数据只需要将数据放到 B `iframe.src` 的 `url hash` 上，B 监听 `hashchange` 拿到数据.

B -> A 通信

B 将要传递的数据放入到 C `iframe.src` 的 `url hash` 上，C 监听 `hashchange` 事件后执行 `window.parent.parent.xxxx = location.hash` 将数据传递至 A 上的某个变量上（或者执行回调 `window.parent.parent.callback(location.hash)`）

#### `document.domain`

将多个页面(`iframe`) 的 `document.domain` 设置为一个域下，就可以进行通信

#### `window.name`

利用了当 `iframe` `src` 属性改变后 `window.name` 依旧不变的特性进行数据传输

#### `Webscoket`

`websocket` 不受同源策略限制

#### 同源策略 & 跨域参考链接
- [前端跨域整理](https://juejin.im/post/5815f4abbf22ec006893b431)
- [同源策略 MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)



## 监控

- 错误监控。（统一上报）
  - `window.onerror`
  - 框架自带的 `error handler`
- 性能监控
  - `performance` API



> 数据上报和埋点可用 1 像素的 gif 图片

