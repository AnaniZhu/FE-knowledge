# webpack 相关

## 优化

- 静态资源压缩
- 小图片内联Base64
- ES6 `tree-shaking`
- 多进程打包
- 利用 loader 缓存
- `dll`
- 代码分割懒加载



## Hash

- `hash`
  - 每次打包采用一个 `hash`
- `chunkHash`
  - 每一个 `chunk` 一个 hash
- `contentHash`
  - 根据文件内容生成的 `hash`。每个文件不一样。



## 运行流程
1. 初始化参数。用 `yargs` 将配置文件与命令行选项解析合并为最终的 `options`
2. `webpack` 初始化
   1. 根据配置项初始化 `compiler`
   2. 注册 `NodeEnvironmentPlugin` (设置 `Node_ENV` 环境变量插件)
   3. 注册配置项插件（用户定义的 plugins）
   4. 初始化基础操作(注册 webpack 内置基础插件)
3. 调用 `compiler.run` 开始编译, 创建 `compilation` 对象
4. 根据配置中的 `entry` 确定入口， 调用 `make` 分析入口文件，创建模块对象
   1. `make` 事件触发 `SingleEntryPlugin`  执行
   2. 根据入口文件通过对应的工厂方法创建模块，并保存到 `compilation` 上
   3. 对模块进行 `build`
      1. 调用 `loader` 对文件进行处理
      2. 使用 `acorn` 生成 `AST` 对其进行遍历
      3. 分析模块中是否有导入语句(`require`, `import`, `import()`)，将依赖模块加入 `dependency` 数组
      4. 当前模块处理完毕之后处理 `dependency` 依赖模块，异步对依赖模块进行 `build`, 如果模块还有依赖模块，进行递归处理
5. 调用 `compilation.seal` 进行封装， 根据入口和模块的依赖关系组装成一个个 `chunk` (`splitChunkPlugin`，异步 `import`)
6. 通过不同的 `chunk` 模板生成不同的文件，生成的资源保存在 `compilation.assets` 中
7. 最后根据 `output` 配置的路径和文件名，将文件内容输出到文件系统中

在上述过程中, `webpack` 会在特定时间点广播出特定的事件，插件在监听的事件触发后会执行一系列逻辑，被调用 `webpack` 提供的 `API` 改变 `webpack` 的运行结果

## loader

### loader 是什么?

`loader` 本质上为一个函数，接受源文件资源或上一个 `loader` 的返回值作为参数, 返回处理后的结果。

`loader` 传递返回值有两种方式:
- 直接 `return` 返回值。
- 调用 `this.callback(err, code, map, meta)` 传递返回值，参数分别为:
  -  `err` 错误对象
  -  `code` 处理过后的代码
  -  `map` 代码对应的 `sourcemap`
  -  `meta` 任意值，常用来传递代码对应的 `AST`，防止后续重复生成。`webpack` 发现 loader 传递了 `AST`, 则会采用传递过来的 `AST`, 否则会自己生成。

如果 `loader` 含有异步操作，可调用 `let callback = this.async()` 告知 `webpack` 进入等待状态, 该方法会返回一个 `callback` 函数, 该函数参数与上述的 `this.callback` 相同，调用则会继续执行下一个 `loader`

可通过设置 `loader.raw = true` 告知 `webpack` 此 `loader` 需要二进制资源的参数( `loader` 函数参数默认为 `UTF-8`)。

### loader 有哪几种类型?
- `loader` 分为 `4` 种类型, 按执行顺序排序则为:
  - 前置 `loader` (`pre loader`)
  - 普通 `loader` (`normal loader`)
  - 行内 `loader` (`inline loader`)
    - `inline loader` 可在头部加前缀标识
      - `!` 表示忽略所有普通 `loader`
      - `-!` 表示忽略所有前置、普通 `loader`
      - `!!` 表示忽略所有的前置、普通和后置的 `loader`
  - 后置 `loader` (`post loader`)

### loader 的执行顺序
- `loader` 执行顺序如下:
  - 先执行 `pitching` 阶段: 从左到右依次执行 `loader.pitch` 函数, 如果返回值不为 `undefined`，则跳过自身和剩下的 `loader`
  - 然后从右到左依次执行 `loader` 函数，如果返回值不为 `undefined`，则会作为下一个 `loader` 的参数

最左侧的 `loader` 必须返回一段 JS 代码，才能被 `webpack`识别，`webpack` 只识别 js 代码。

### pitch

可通过设置 `loader.pitch` 定义 `pitching` 阶段，其会影响 `loader` 执行。

可通过 `this.data` 在 `loader.pitch` 和 `loader` 之间共享数据。

`pitch` 函数接受三个参数
- `remainingRequest` 剩下的 loaders 字符串
- `precedingRequest` 之前的 loaders 字符串
- `data` 共享数据对象

#### example
```js
// 假设注册了 3 个 loader: ['style-loader', 'css-loader', 'less-loader']
// 现在对 test.css 文件进行处理

// 假设这里是 css-loader 的实现
function CSSLoader (content) {
  // this.data.xxx
}

CSSLoader.pitch = function (remainingRequest, precedingRequest, data) {
  // 下面 this.request, remainingRequest 和 precedingRequest 中的路径都是绝对路径，此处简写
  // 可以通过 loaderUtils.stringRequest(this, request) 转成相对路径

  // this.request: 'style-loader!css-loader!less-loader!test.css'
  // remainingRequest: 'less-loader!test.css'
  // precedingRequest: 'style-loader!test.css'
  // data: {}
}

module.exports = CSSLoader
```

### loader 其他知识
- `loader` 有自己的上下文(`loaderContext`), `loader` 和 `loader.pitch` 函数都可以通过 `this` 拿到一些关键属性
  - `this.cacheable(Boolean)` 是否开启缓存，默认开启。
- 开发自定义 `loader` 常用第三方库:
  - `loader-utils`。提供一些 `loader` 常用的工具方法。
  - `schema-utils`。提供 `loader options` 的 `schema` 定义以及校验方法

### 常用 loader 原理

#### style-loader

主要功能是生成 `style` 标签并将所依赖的样式文件的内容填充至标签中，随后插入至 `head` 中，并内置热更新功能。

#### css-loader

将 `css` 文件中的 `url` (`url(xxx), import 'xxx'` 中的 xxx) 都替换成打包后正确的文件路径

#### file-loader

将引入的二进制文件写入文件系统，同时将该导入的返回值修改为打包后该文件的路径

#### url-loader

当文件小于指定大小时会转成 `Base64`, 否则交给 `file-loader` 处理。该导入返回值要么是 `Base64` 或者是生成文件后的文件路径

## plugin

 `webpack.config.js - plugins` 需传递一个含有 `apply` 方法的对象集合，此对象一般都是一个构造函数/类的实例，实例原型上含有 `apply` 方法，`webpack` 内部运行过程中 `compiler` 会调用此方法。

`plugin` 本质上是一个原型上含有 `apply` 方法的构造函数或者类, `apply` 方法接受一个参数,  值为 `compiler` 对象，可通过监听这个对象上的 `hooks` 或者其属性上其他对象的 `hook` 从而进行一些处理。

可以通过 `compiler.assets` 设置属性来添加要输出的资源，更多 `API` 需查看官方文档。

### Tapable

`Tapable` 是 `webpack` 源码使用的一个事件订阅、广播的库，用于 `Plugin` 中，插件可监听发射的不同事件并做出不同的行为。

`Tapable` 有如下几种事件类型:
- `SyncHook` 同步钩子
- `SyncBailHook` 同步保险丝钩子。根据监听顺序依次执行，一旦其中某个监听函数的返回值不为 `undefined`, 就代表错误，后续钩子不再执行。
- `SyncWaterfallHook` 同步瀑布式钩子。根据监听顺序依次执行，会将上一个监听函数的返回值传递给下一个监听函数。
- `SyncLoopHook` 同步循环式钩子。根据监听顺序依次执行，当其中一个监听函数返回值不为 `undefined`，则会循环调用，直至返回 `undefined` 再执行下一个监听函数
- `AsyncSeriesHook` 异步串行钩子。按监听顺序依次执行(如果存在错误对象，则直接结束执行回调)
- `AsyncSeriesBailHook` 异步串行保险丝钩子。根据监听顺序异步依次执行，一旦其中某个钩子返回值不为 `undefined`, 就代表错误，后续钩子不再执行，同时执行 `callAsync` 的回调函数。
- `AsyncSeriesWaterfallHook` 异步串行瀑布式钩子。根据监听顺序异步依次执行，会将上一个监听函数的返回值(`callback(err, ...args)` 的 `...args`)传递给下一个监听函数。
- `AsyncParallelHook` 异步并行钩子。所有监听函数同时执行，不关心监听顺序。
- `AsyncParallelBailHook` 异步并行保险丝钩子。所有监听函数同时执行，不关心监听顺序，一旦其中一个监听函数返回值不为 `undefined`，直接调用 `callAsync` 的回调。 类似 `Promise.all`

上述构造函数皆通过 `const hook = new xxxHook([arg1, arg2, ...args])` 初始化实例，注册参数个数。如果后续 `call` 或 `callAsync` 或 `promise` 调用的参数超过注册的参数个数，多余的会被忽略

同步钩子
  - 通过 `hook.tap(fnName, fn<arg1, arg2, ...args>)` 注册监听函数。监听函数通过 `return` 传递返回值
  - 通过 `hook.call(arg1, arg2, ...args)` 触发钩子

异步钩子
  - 注册监听函数有 `3` 种方式:
    - 通过 `hook.tap(fnName, fn<arg1, arg2, ...args)` 注册。无返回值
    - 通过 `hook.tapAsync(fnName, fn<arg1, arg2, ...args, callback>)` 注册。`fn` 内部调用 `callback(err, data)`传递返回值且表示当前监听函数结束
    - 通过 `hook.tapPromise(fnName, fn<arg1, arg2, ...args>)` 注册。`fn` 需返回一个 `Promise` 来传递返回值及状态。
  - 触发监听函数有 `2` 种方式:
    - 通过 `hook.callAsync(arg1, arg2, ...args, finalCallback)` 触发
    - 通过 `hook.promise(arg1, arg2, ...args)` 触发。此方法返回一个 `Promise`
  - 所有异步钩子的监听函数中 `callback` 回调函数的第一个参数都代表错误对象，只要传递就会忽略后续的监听的函数，直接执行最终的回调函数，其余的参数都表示返回参数，参数个数要跟注册钩子时的一致，否则多余的会被忽略。

## webpack chunk

### chunk 是什么？

`chunk` 是一个代码块，`module` 是一个模块，一个文件就是一个模块，一个 `chunk` 可能包含多个 `module`

产生 `chunk` 有如下三种方式:
- 增加入口 `entry`
- 异步导入 (`import(xxx)`)
- 配置 `splitChunks`


### chunk 代码运行时

`webpack` 打包后的入口 `chunk` 代码中自己实现了一套模块系统 `__webpack_require__`(类似 `Node` 的 `require`)来管理模块，所有源代码的导入方法(`import`, `import(xx)`, `require`)都会被转成这个，通过这个方法可以加载其余的 `module` 或者其他的 `chunk`。

如果存在异步 `chunk`, 入口 `chunk` 里的代码会增加一些如异步 `chunk` 的处理。

异步 `chunk` 的导入比较特殊, `import(xxx).then(module => {// do something})` 会被转成 `__webpack_require__.e(chunkId).then(__webpack_require__.bind(null, moduleName)).then(module => {// do something})`

当需要访问异步 `chunk` 代码块时，也就是执行了 `__webpack_require__.e(chunkId).then(__webpack_require__.bind(null, moduleName)).then(module => {// do something})` 这段代码

`__webpack_require__.e` 函数本质上就是通过 `JSONP` 发起请求加载对应的 `chunk` 文件，同时返回一个对应的 `Promise`。

`chunk` 文件是一段 `window['webpackJsonp'].push(chunkIds, modules)` 代码，`modules` 就是该 `chunk` 所包含的所有模块。当 `chunk` 文件下载完成会自动执行（`JSONP`的作用）。

而入口 `chunk` 文件重写了 `webpackJsonp` 数组的 `push` 方法，所以说 `window['webpackJsonp'].push(chunkIds, modules)` 本质上就是在执行 `webpackJsonpCallback(chunkIds, modules)` 语句。

`webpackJsonpCallback` 也是入口 `chunk` 声明的函数，其主要功能就是将新的 `modules` 合并到入口 `chunk` 里的 `modules` 中，随后将 `__webpack_require__.e` 返回的 `Promise` 置为完成状态 (调用其 `resolve` 方法)。

`__webpack_require__.e` 返回的 `Promise` 完成后自然会执行 `.then(__webpack_require__.bind(null, moduleName)`, 因为 `webpackJsonpCallback` 已经将新 `chunk` 里所有模块整合到入口模块里，所以此时 `__webpack_require__` 就可以直接加载到对应的模块代码并返回，最后就是执行业务代码的逻辑了(`.then(module => {// do something})`)。


## webpack-dev-server & Hot Module Replacement

### 模块热替换是什么?

在编辑代码按下保存后，页面不需要完全刷新就可以更新到最新状态。

### 用法

开启热加载的 `webpack.config.js` 关键配置如下:

```js
const webpack = require('webpack');

module.exports = {
  devServer: {
    contentBase: './dist',
    hot: true
  },
  plugins: [
     new webpack.HotModuleReplacementPlugin()
  ]
}
```

### 原理

`webpack-dev-server` 功能主要是如下两点:

- 初始化本地服务
  - 启动 `webpack`，生成 `compiler` 实例, 监听 `compiler` 的 `done` hook
  - 开启了一个 `express` 服务器，引入 `webpack-dev-middleware` 中间件。
    - `webpack-dev-middleware` 中间件有如下功能：
      - 调用 `compiler.watch` 监听本地文件变化并重新打包
      - 将 `webpack` 的文件系统改为内存文件系统(`memory-js`), 这也是为什么看不到产出文件且内存占用比较大的原因
      - 监听客户端发过来的 `http` 静态资源请求，并将编译好的内容从内存文件系统中读取返回
  - 启动 `websocket` 服务，在监听到 `webpack compiler done` 事件后，会向客户端推送一些事件
    - `hash` 事件会传递本次编译更新后的 hash 值
    - `ok` 事件代表编译成功
    - 还有一些其余不是很重要的事件(`errors`, `warnings`, ...)
- 修改 `entry` 配置，注入一些客户端代码，用于热更新处理
  - 声明一些热更新需要的函数
  - 与服务端建立 `websocket` 连接
    - 监听 `hash` 事件。触发时会将传递过来的 hash 值记录下来，维护一个 `lastHash` 和 `currentHash`
    - 监听 `ok` 事件。
      1. 向服务端发起一个请求获取 `[lastHash].hot-update.json` 的文件，此 `json` 文件记录了 `currentHash` 值以及需要更新的 `chunk`。
      2. 如果存在待更新的 `chunk`, 就会发起 `JSONP` 请求获取这些 `chunk` 的变更文件: `[chunkName].[lastHash].hot-update.js` 并执行。
      3. `[chunkName].[lastHash].hot-update.js` 内部是一段执行代码 `webpackHotUpdate(chunkName, updateModules)`，所以当 `script` 插入文档中，这段代码会立即执行。
      4. `webpackHotUpdate` 函数会遍历 `updateModules`，将新的模块替换掉缓存(`__webpack_require__.c`)中的旧模块, 并执行该更新后的模块代码，随即根据模块间的依赖关系冒泡式的将祖先模块中 `module.hot.accept(xx, callback)` 中的 `callback` 执行，`callback` 就是具体的更新逻辑，从而达到热更新的效果。
    - ...其余事件忽略

`HotModuleReplacementPlugin` 功能有如下几点:
- 监听 `webpack` 某个 hook 将 `chunk` 代码改写
- 修改模块系统。给 `__webpack_require__` 里的 `module` 增加了 `hot`、`parent`、`children` 属性
- 声明一些函数

以下函数分别在哪个文件声明？

TODO:
`hotDownloadManifest`, `hotDownloadUpdateChunk`, `webpackHotUpdate`, `hotAddUpdateChunk`, `hotUpdateDownloaded`

## webpack 与 gulp、rollup 区别

`Gulp` 是基于 `steam` 的前端自动化构建工具

`webpack` 和 `rollup` 是模块打包工具

`rollup` 是下一代 ES6 模块化工具，采用 `ES6 module` 设计，也是率先提出了 `Tree shaking` 的概念。

`rollup` 打包出的代码体积更小，适用于类库。但其不支持代码分割(`Code splitting`)和模块热更新(`HMR`)

`webpack` 生态丰富，适用于应用

- [Grunt / Gulp / Webpack / Rollup 比较](https://www.imooc.com/article/20603)
- [前端构建工具 Gulp / browserify/ webpack / npm ?](https://www.zhihu.com/question/37694275)


## 参考链接

- [深入浅出 webpack](https://webpack.wuhaolin.cn/5%E5%8E%9F%E7%90%86/5-1%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E6%A6%82%E6%8B%AC.html)
- [webpack原理](https://segmentfault.com/a/1190000015088834)
- [Webpack Hot Module Replacement 的原理解析](https://github.com/Jocs/jocs.github.io/issues/15)
- [轻松理解webpack热更新原理](https://juejin.im/post/5de0cfe46fb9a071665d3df0)

