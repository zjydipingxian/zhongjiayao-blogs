---
sidebar:
  title: 生成器
isTimeLine: true
title: 生成器
tags:
  - 技术笔记
categories:
  - 技术笔记
recommend: 12
---

# 生成器

ES6 生成器（Generator）是 JavaScript 语言新增的一种特殊函数类型，它提供了一种优雅的解决方案来处理异步编程。生成器使得开发者可以用同步的线性代码风格来书写异步操作，代码更加简洁、可读性更高，同时还能够减少回调地狱和处理并发操作的复杂性。

## 1. 什么是生成器

生成器是一种特殊的函数，它可以暂停执行并返回一个中间结果，然后再次从暂停的地方继续执行。生成器函数通过关键字 `function*` 声明，并且使用 `yield` 关键字指定调用迭代器的`next`方法时的返回值。

- 简单地说，迭代器就是实现了 next()方法的一类特殊的对象。这个 next()方法的返回值是一个对象，包含了 value 和 done 两个属性。例如：

```js
function* fn() {
  yield 1
  yield 2
  yield 3
}

// 返回一个迭代器对象
let iterator = fn()

console.log(iterator.next()) // {value: 1, done: false}
console.log(iterator.next()) // {value: 2, done: false}
console.log(iterator.next()) // {value: 3, done: false}
console.log(iterator.next()) // {value: undefined, done: true}

// 或者   iterator 可被迭代
for (let item of iterator) {
  console.log(item)
}
```

## 2. 生成器的声明

生成器是一个函数的形式，通过在函数名称前加一个星号(\*)就表示它是一个生成器。所以只要是可以定义函数的地方，就可以定义生成器

```js
function* gFn() {}

const gFn = function* () {}

const o = {
  *gFn() {},
}
```

## 3. 生成器的使用

### 1. 执行

::: tip :100:
生成器一开始处于暂停执行的状态，需要调用 next 方法才能让生成器开始或恢复执行。
:::

return 会直接让生成器到达 done: true 状态:

```js
function* gFn() {
  console.log(111)

  return 1
}
const g = gFn()

g.next() // {value: 1, done: true}
```

生成器代码的执行可以被 yield 控制:

```js
function* gFn() {
  yield
  console.log(111)
  return 222
}
const g = gFn()
console.log('1', g.next())
console.log('2', g.next())
```

::: tip yield 关键字
提到生成器，自然不能忘记 yield 关键字。 yield 能让生成器停止，此时函数作用域的状态会被保留，只能通过在生成器对象上调用 next 方法来恢复执行。
上面我们已经说了， return 会直接让生成器到达 done: true 状态，而 yield 则是让生成器到达 done: false 状态，并停止
:::

很多时候不希望 next 返回的是一个 undefined，这个时候我们可以通过 yield 来返回结果:

```js
function* gFn() {
  yield 100
  console.log(111)
  return 222
}
const g = gFn()
console.log('2', g.next()) //   {value: 100, done: false}
console.log('3', g.next()) //  {value: 222, done: true}
```

### 2. 生成器传递参数

yield 语句后面可以跟一个表达式，这个表达式的值会作为 next 方法的参数，传递给 yield 语句。

```js
function* gFn(initial) {
  console.log(initial)
  console.log(yield)
  console.log(yield)
  console.log(yield)
}

const g = gFn('red')
g.next('white')
g.next('blue')
g.next('purple')
g.next('xxx')

// red
// {value: undefined, done: false}

// blue
// {value: undefined, done: false}

// purple
// {value: undefined, done: false}

// xxx
// {value: undefined, done: true}
```

::: warning

1. 生成生成器，此时处于暂停执行的状态
2. 调用 next，让生成器开始执行，输出 red，然后准备输出 yield，发现是 yield，暂停执行，出去外面一下。
3. 外面给 next 方法传参 blue，又恢复执行，然后之前暂停的地方(即 yield)就会接收到 blue。然后又遇到 yield 暂停。
4. 又恢复执行，输出 purple

第一次传的 white 怎么消失了?

它确实消失了，因为第一次调用 next 方法是为了开始执行生成器函数，而刚开始执行生成器函数时并没有 yield 接收参数，所以第一次调用 next 的值并不会被使用
:::

### 3. 生成器替代迭代器

生成器是一种特殊的迭代器，那么在某些情况下我们可以使用生成器来替代迭代器：

```js
const names = ['aaa', 'bbb', 'ccc']
function* gFn(arr) {
  for (let i = 0; i < arr.length; i++) {
    yield arr[i]
  }
}

const g = gFn(names)
console.log('🚀 ~ g:', g)

// for (let item of g) {
//   console.log('🚀 ~ item:', item) // aaa , bbb, ccc
// }
// 或者

console.log(g.next()) // {value: 'aaa', done: false}
console.log(g.next()) // {value: 'bbb', done: false}
console.log(g.next()) // {value: 'ccc', done: false}
console.log(g.next()) //   {value: 'undefined', done: true}
```
