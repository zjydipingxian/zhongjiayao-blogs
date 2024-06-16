---
sidebar:
  title: loader 基本使用 & 原理
isTimeLine: true
title: loader 基本使用 & 原理
# date: 2024-06-11 09:00:00
tags:
  - webpack5
categories:
  - webpack5
recommend: 4
---

# loader

> loader 用于对模块的源代码进行转换。loader 可以使你在 import 或 "load(加载)" 模块时预处理文件。因此，loader 类似于其他构建工具中“任务(task)”，并提供了处理前端构建步骤的得力方式。loader 可以将文件从不同的语言（如 TypeScript）转换为 JavaScript 或将内联图像转换为 data URL。loader 甚至允许你直接在 JavaScript 模块中 import CSS 文件！

::: tip
帮助 webpack 将不同类型的文件转换为 webpack 可识别的模块。
:::

## 基本使用

```js
module.exports = {
  module: {
    rules: [
      { test: /\.css$/, use: 'css-loader' },
      { test: /\.ts$/, use: 'ts-loader' },
    ],
  },
}
```

## 执行顺序

loader 从右到左（或从下到上）地取值(evaluate)/执行(execute)。  
 在下面的示例中，从 sass-loader 开始执行，然后继续执行 css-loader，最后以 style-loader 为结束

```js
module.exports = {
  module: {
    rules: [
      // 从下到上
      {
        test: /\.css$/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              modules: true,
            },
          },
          { loader: 'sass-loader' },
        ],
      },

      {
        test: /\.js/,
        use: ['bar-loader', 'foo-loader'], //  从右到左
      },
    ],
  },
}
```

## 编写 loader

> 由于 Webpack 是运行在 Node.js 之上的，一个 Loader 其实就是一个 Node.js 模块，这个模块需要导出一个函数。 这个导出的函数的工作就是获得处理前的原内容，对原内容执行处理后，返回处理后的内容。 (https://www.webpackjs.com/contribute/writing-a-loader/)

### 1. 最简单的 Loader 的源码如下：

```js
module.exports = function (source) {
  // source 为 compiler 传递给 Loader 的一个文件的原内容
  // 该函数需要返回处理后的内容，这里简单起见，直接把原内容返回了，相当于该 Loader 没有做任何转换
  return source
}
```

### 2. 获得 Loader 的 options

在最上面处理 SCSS 文件的 Webpack 配置中，给 css-loader 传了 options 参数，以控制 css-loader。 如何在自己编写的 Loader 中获取到用户传入的 options 呢？需要这样做：

```js
const loaderUtils = require('loader-utils')
module.exports = function (source) {
  // 获取到用户给当前 Loader 传入的 options
  const options = loaderUtils.getOptions(this)
  return source
}
```

### 3. 同步与异步

Loader 有同步和异步之分，上面介绍的 Loader 都是同步的 Loader，因为它们的转换流程都是同步的，转换完成后再返回结果。 但在有些场景下转换的步骤只能是异步完成的，例如你需要通过网络请求才能得出结果，如果采用同步的方式网络请求就会阻塞整个构建，导致构建非常缓慢。

- 在转换步骤是异步时，你可以这样：

```js
module.exports = function (source) {
  // 告诉 Webpack 本次转换是异步的，Loader 会在 callback 中回调结果
  var callback = this.async()
  someAsyncOperation(source, function (err, result, sourceMaps, ast) {
    // 通过 callback 返回异步执行后的结果
    callback(err, result, sourceMaps, ast)
  })
}
```

### 4. 其他 Loader API

| 方法名                  |                    含义                    |                                           用法 |
| ----------------------- | :----------------------------------------: | ---------------------------------------------: |
| this.async              |    异步回调 loader。返回 this.callback     |                  const callback = this.async() |
| this.callback           | 可以同步或者异步调用的并返回多个结果的函数 | this.callback(err, content, sourceMap?, meta?) |
| this.getOptions(schema) |           获取 loader 的 options           |                        this.getOptions(schema) |
| this.emitFile           |                产生一个文件                |        this.emitFile(name, content, sourceMap) |
| this.utils.contextify   |              返回一个相对路径              |        this.utils.contextify(context, request) |
| this.utils.absolutify   |              返回一个绝对路径              |        this.utils.absolutify(context, request) |

> 更多文档，请查阅 webpack 官方 loader api 文档 https://www.webpackjs.com/api/loaders

### 5. 加载本地 Loader

为了让 Webpack 加载放在本地项目中的 Loader 需要修改 resolveLoader.modules。
假如本地的 Loader 在项目目录中的 ./loaders/loader-name 中，则需要如下配置：

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          {
            loader: path.resolve('path/to/loader.js'),
            options: {
              /* ... */
            },
          },
        ],
      },
    ],
  },

  resolveLoader: {
    // 去哪些目录下寻找 Loader，有先后顺序之分
    modules: ['node_modules', './loaders/'],
  },
}
```

加上以上配置后， Webpack 会先去 node_modules 项目下寻找 Loader，如果找不到，会再去 `./loaders/` 目录下寻找。

## 实现 loader

### 1. 用来清理 js 代码中的 console.log

```js
module.exports = function cleanLogLoader(content) {
  // 将console.log替换为空
  return content.replace(/console\.log\(.*\);?/g, '')
}
```

### 2. 把 JavaScript 代码中的注释语法

```js
function replace(source) {
  // 使用正则把 // @require '../style/index.css' 转换成 require('../style/index.css');
  return source.replace(/(\/\/ *@require) +(('|").+('|")).*/, 'require($2);')
}

module.exports = function (content) {
  return replace(content)
}
```

### 3 将 ES6+语法编译成 ES5-语法。（. 简单版）

```js
const { validate } = require('schema-utils') // webpack 提供的校验工具
const babel = require('@babel/core')

module.exports = function (content) {
  const options = this.getOptions()

  //2.校验参数是否符合规则
  validate(loaderSchema, options)

  // 使用异步loader
  const callback = this.async()
  // 使用babel对js代码进行编译
  babel.transform(content, options, function (err, result) {
    callback(err, result.code)
  })
}
```
