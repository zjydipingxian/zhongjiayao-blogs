---
sidebar:
  title: webpack5 基本使用
isTimeLine: true
title: webpack5 基本使用
# date: 2024-06-11 09:00:00
tags:
  - webpack5
categories:
  - webpack5
recommend: 3
---

# 基本使用

> Webpack 是一个静态资源打包工具。
> 它会以一个或多个文件作为打包的入口，将我们整个项目所有文件编译组合成一个或多个文件输出出去。
> 输出的文件就是编译好的文件，就可以在浏览器段运行了。
> 我们将 Webpack 输出的文件叫做 bundle

## 前言

### 1. 创建目录

```
webpack_code # 项目根目录（所有指令必须在这个目录运行）
    └── src # 项目源码目录
        ├── js # js文件目录
        │   ├── count.js
        │   └── sum.js
        └── main.js # 项目主文件
```

### 2. 创建文件

count.js

```js
export default function count(x, y) {
  return x - y
}
```

sum.js

```js
export default function count(x, y) {
  return x - y
}
```

main.js

```js
import count from './js/count'
import sum from './js/sum'

console.log(count(2, 1))
console.log(sum(1, 2, 3, 4))
```

### 3. 下载依赖

```sh
npm init -y

```

**如果喜欢尝鲜，想体验更快的启动和构建速度，推荐使用 pnpm**

其它情况推荐使用 pnpm，yarn

:::code-group

```sh [安装 PNPM]
pnpm install webpack webpack-cli -D
```

```sh [安装 npm]
npm install webpack webpack-cli -D

```

:::

### 4. 启动

:::code-group

```sh [ PNPM]
pnpm webpack ./src/main.js --mode=development

```

```sh [ npm]
npx  install webpack webpack-cli -D

```

:::

## 基本配置

在项目根目录下新建文件：webpack.config.js

```js
module.exports = {
  // 入口
  entry: '',
  // 输出
  output: {},
  // 加载器
  module: {
    rules: [],
  },
  // 插件
  plugins: [],
  // 模式
  mode: '',
}
```

## 1. entry（入口） 丶 output（输出）

```js
// Node.js的核心模块，专门用来处理文件路径
const path = require('path')

module.exports = {
  // 入口
  // 相对路径和绝对路径都行
  entry: './src/main.js',
  // 输出
  output: {
    // path: 文件输出目录，必须是绝对路径
    // path.resolve()方法返回一个绝对路径
    // __dirname 当前文件的文件夹绝对路径
    path: path.resolve(__dirname, 'dist'),
    // filename: 输出文件名
    filename: 'main.js',
  },
  // 模式
  mode: 'development', // 开发模式
}
```

2. 运行指令

```sh
npx webpack
或
pnpm webpack
```

## 2. loader（加载器）

> Loader 就像是一个翻译员，能把源文件经过转化后输出新的结果，并且一个文件还可以链式的经过多个翻译员翻译。

::: tip
在更高层面，在 webpack 的配置中，loader 有两个属性：

1. test 属性，识别出哪些文件会被转换。
2. use 属性，定义出在进行转换时，应该使用哪个 loader。
   :::

### 1. 处理 css 资源

:::code-group

```sh [ PNPM]
pnpm npm i css-loader style-loader -D

```

```sh [ npm]

npm i css-loader style-loader -D

```

:::

### 2. 功能

- css-loader：负责将 Css 文件编译成 Webpack 能识别的模块
- style-loader：会动态创建一个 Style 标签，里面放置 Webpack 中 Css 模块内容

### 3. 配置

```js
const path = require('path')

module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'main.js',
  },
  // [!code focus:11]
  module: {
    rules: [
      {
        // 用来匹配 .css 结尾的文件
        test: /\.css$/,
        // use 数组里面 Loader 执行顺序是从右到左
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
  plugins: [],
  mode: 'development',
}
```

### 4. 添加 css 资源

- src/css/index.css

```css
.box1 {
  width: 100px;
  height: 100px;
  background-color: pink;
}
```

- src/main.js

```js
import count from './js/count'
import sum from './js/sum'
// [!code focus:3]
// 引入 Css 资源，Webpack 才会对其打包
import './css/index.css'

console.log(count(2, 1))
console.log(sum(1, 2, 3, 4))
```

- public/index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0"
    />
    <title>Document</title>
  </head>
  <body>
    <h1>Hello Webpack5</h1>
    <div class="box1">123123</div>
  </body>
</html>
```

### 5. 处理图片资源

```js
{
    test: /\.(png|jpe?g|gif|webp)$/,
    type: "asset",
},
```
