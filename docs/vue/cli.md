---
sidebar:
  title: 使用 rollup 搭建 mini_vue2版本 来分析源码
#   step: 95
isTimeLine: true
title: 使用 rollup 搭建 mini_vue2版本 来分析源码
# date: 2024-06-11 09:00:00
tags:
  - Vue2
categories:
  - Vue2
recommend: 2
---

# 使用 rollup 搭建 mini_vue2 版本

**如果喜欢尝鲜，想体验更快的启动和构建速度，推荐使用 pnpm**

其它情况推荐使用 pnpm，yarn

:::code-group

```sh [安装 PNPM]
npm install -g pnpm
```

```sh [安装 yarn]
npm install -g yarn
```

:::

## 快速创建项目

在自己本地创建一个文件夹，然后执行以下命令
:::code-group

```sh [npm]
npm init
```

:::

## 安装依赖

:::code-group

```sh [npm]
npm install @babel/core @babel/preset-env rollup rollup-plugin-babel rollup-plugin-serve -D
```

```sh [yarn]
yarn add @babel/core @babel/preset-env rollup rollup-plugin-babel rollup-plugin-serve -D
```

```sh [pnpm]
pnpm add @babel/core @babel/preset-env rollup rollup-plugin-babel rollup-plugin-serve -D
```

:::

## 配置 .babelrc

```js
{
    "presets": [
        [
            "@babel/preset-env"
        ]
    ]
}
```

## 配置 rollup.config.js

```js
import babel from 'rollup-plugin-babel'
import serve from 'rollup-plugin-serve'

export default {
  input: 'src/index.js', //入口文件
  output: {
    file: 'dist/vue.js',
    name: 'Vue', //全局变量名
    format: 'umd', //输出格式
    sourcemap: true, //生成map文件
  },
  plugins: [
    babel({
      exclude: 'node_modules/**',
    }),
    serve({
      open: true,
      port: 3000,
      contentBase: '',
      openPage: '/index.html',
    }),
  ],
}
```

## 配置 package.json

在 scripts 下面 添加 多一个命令

```js{3}
 "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "rollup -c -w" // [!code focus]
  },
```

## 最后

在当前项目 新建 `src` 目标 文件夹，然后新建 `index.js` 文件，然后运行
