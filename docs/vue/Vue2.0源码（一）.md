---
sidebar:
  title: 手写 Vue2.0 源码（一）响应式数据原理
#   step: 95
isTimeLine: true
title: 手写 Vue2.0 源码（一）响应式数据原理
# date: 2024-06-11 09:00:00
tags:
  - Vue2
categories:
  - Vue2
recommend: 3
---

# 手写 Vue2.0 源码（一）-响应式数据原理

> Vue 的一个核心特点是数据驱动 数据变化了想要同步到视图就必须要手动操作 dom 更新 但是 Vue 帮我们做到了数据变动自动更新视图的功能 那在 Vue 内部就一定有一个机制能监听到数据变化然后触发更新 本篇主要介绍响应式数据的原理

## 数据初始化

```html
<div id="app">{{ message }}</div>
```

```js
var vm = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!',
  },
})
```

这段代码 大家一定非常熟悉 这就是 Vue 实例化的过程 从 new 操作符 咱们可以看出 Vue 其实就是一个构造函数

```js
// src\core\instance\index.js   Vue2 官方源码目录

import { initMixin } from './instance/init'

function Vue(options) {
  // 初始化
  this._init(options)
}

// Vue2 官方源码目录: src\core\instance\index.js
initMixin(Vue)

export default Vue
```

initMixin 把 \_init 方法挂载在 Vue 原型 供 Vue 实例调用

```js{3}
// Vue2 官方源码目录: src\core\instance\init.js

import { initState } from './state'
export function initMixin(Vue) {
  Vue.prototype._init = function (options) { // [!code focus]
    const vm = this
    vm.$options = options
    // 初始化状态
    initState(vm)
  }
}
```

::: tip
initState 主要做 一些 `props` `methods` `data` `computed` `watch` 的初始化 主要关注 initData 里面的 observe 是响应式数据核心 所以另建 observer 文件夹来专注响应式逻辑 其次我们还做了一层数据代理 把 data 代理到实例对象 this 上 也就是 我们常习惯的书写方法 `vm
.msg` 或者 `this.msg`
:::

```js
// Vue2 官方源码目录: src\core\instance\init.js

import { initState } from './state'
export function initMixin(Vue) {
  Vue.prototype._init = function (options) {
    const vm = this
    vm.$options = options
    // 初始化状态
    initState(vm) // [!code focus]  // Vue2 官方源码目录:  src\core\instance\state.js
  }
}
```

## 对象的数据劫持

::: tip
数据劫持核心是 defineReactive 函数 主要使用 Object.defineProperty 来对数据 get 和 set 进行劫持
:::

```js
export function initState(vm) {
  const opts = vm.$options

  // 是否有 props
  if (opts.props) initProps(vm, opts.props)

  // 是否有方法
  if (opts.methods) initMethods(vm, opts.methods)

  // 是否有 data
  if (opts.data) {
    initData(vm) // [!code focus]
  } else {
    // 默认 data
    observe((vm._data = {}), true) // [!code focus]
  }

  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch) initWatch(vm, opts.watch)
}
```

### 1. initData 方法

```js
function initData(vm) {
  let data = vm.$options.data

  // 判断 data 是不是个函数
  //  这里要注意一下, data 是个函数, this的指向要转回去给 vue 实例
  data = vm._data = typeof data === 'function' ? data.call(vm, vm) : data

  // 将data 上的属性代理到 vm 上， 不是vue3中的 proxy
  const keys = Object.keys(data)
  let i = keys.length
  while (i--) {
    const key = keys[i]
    proxy(vm, `_data`, key)
  }

  observe(data)
}
```

### 2. observe (重点中的重点)

看看 observe 函数 是如何监听数据的变化 其核心是 源码 `defineReactive 函数 `中 `Object.defineProperty`

```js
// Vue2 官方源码目录: src\core\observer\index.js
export function observe(value) {
  if (typeof value !== 'object' || value === null) {
    return value
  }

  let ob
  if (value.__ob__ instanceof Observer) {
    ob = value.__ob__
  }

  // 用来进行数据 劫持观测
  ob = new Observer(value) // [!code focus]

  return ob
}

export class Observer {
  constructor(value) {
    this.value = value

    // 从vue2中源码上看到这个方法, 添加了一个属性, 添加到对象上
    def(value, '__ob__', this)

    // 判断是不是对象
    if (Object.prototype.toString.call(value) === '[object Object]') {
      this.walk(value)
    }
  }
  /**
   * 遍历给定对象的属性，将每个属性转换为响应式。
   *
   * 此函数的目的是为了初始化一个对象的属性，使这些属性成为响应式的。
   * 它通过定义每个属性的getter和setter来实现，这样当属性被访问或修改时，
   * 可以触发相应的反应机制。
   *
   * @param {Object} obj 要转换为响应式的目标对象。
   */
  walk(obj) {
    // 获取对象的所有键
    const keys = Object.keys(obj)
    // 遍历对象的所有键
    for (let i = 0; i < keys.length; i++) {
      let key = keys[i]
      let value = obj[key]
      // 通过defineReactive定义属性的getter和setter，使其响应式
      defineReactive(obj, key, value) // [!code focus]
    }
  }
}

export function  defineReactive(obj, key, value)  // [!code focus] {
  observe(value) // a: {b: 1}  // 处理对象嵌套问题  // [!code focus]
  Object.defineProperty(obj, key, {
    get() {
      // console.log("获取--get");
      return value
    },

    set(newVal) {
      // console.log("设置--set");
      if (newVal === value) {
        return
      }
      observe(newVal) // [!code focus]
    },
  })
}
```

## 数组数据劫持

Vue2 底层改写了 Array 原型上的方法

```js
// Vue2 官方源码目录: src\core\observer\array.js
const arrayProto = Array.prototype
export const arrayMethods = Object.create(arrayProto)

const methodsToPatch = ['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse']

methodsToPatch.forEach((method) => {
  arrayMethods[method] = function (...args) {
    // console.log("拦截数组");

    const result = arrayProto[method].apply(this, args)
    const ob = this.__ob__
    let inserted

    // 对数组的追加  push( args -> {a:2}) unshift(args -> {a:2}) splice

    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2) // slice(0,1,args)
        break
    }

    if (inserted) {
      ob.observeArray(inserted)
    }

    return result
  }
})
```

所以对数组的拦截监听

```js
export class Observer {
  constructor(value) {
    this.value = value

    // 从vue2中源码上看到这个方法, 添加了一个属性, 添加到对象上
    def(value, '__ob__', this)

    this.dep = new Dep() // 所有对象类型增加一个dep

    // 判断是不是数组
    if (Array.isArray(value)) {
      protoAugment(value, arrayMethods) // [!code ++]   // protoAugment 是  target.__proto__ = src
      this.observeArray(value) // [!code ++]
    }

    ....
  }

  observeArray(items) { // [!code ++]
    for (let i = 0, l = items.length; i < l; i++) { // [!code ++]
      observe(items[i]) // [!code ++]
    }  // [!code ++]
  } // [!code ++]
}
```
