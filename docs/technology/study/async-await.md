---
sidebar:
  title: async-await
isTimeLine: true
title: async-await
tags:
  - 技术笔记
categories:
  - 技术笔记
recommend: 13
---

# async-await

> async/await 是 ES20717 引入的，主要是简化 Promise 调用操作，实现了以异步操作像同步的方式去执行，async 外部是异步执行的，同步是 await 的作用。

## async

首先，先了解 async 函数返回的是什么？以下面代码为例

> async 声明的函数，返回的结果是一个 Promise 对象。如果在函数中 return 一个直接量，async 会把这个直接量通过 Promise.resolve() 封装成 Promise 对象。

```js
async function testAsync() {
  return 'hello world'
}

// testAsync() 返回的是 Promise 对象

{
 Promise :{<fulfilled>: 'hello world'}
}
```

## await

一般来说，认为 await 在等待一个 async 函数完成。不过按照语法来看，await 等待的是一个表达式。这个表达式的计算结果是 Promise 对象或者其他值。

所以： await 等待的是右侧表达式的返回值！！！

- 等待异步操作：await 关键字用于等待一个异步操作完成。它会暂停方法的执行，直到异步操作完成，然后继续执行方法的剩余部分。
- 释放线程：在等待异步操作完成时，await 会释放当前线程，使其可以执行其他任务，从而提高应用程序的并发性能。

## async/await 入门

```js
Promise.resolve('绿').then((res) => {
  console.log(res)

  Promise.resolve('黄').then((res) => {
    console.log(res)

    Promise.resolve('红').then((res) => {
      console.log(res)
    })
  })
})
```

我们可以用 async/await 来实现一下：

```js
;(async () => {
  await Promise.resolve('绿').then((res) => console.log(res))
  await Promise.resolve('黄').then((res) => console.log(res))
  await Promise.resolve('红').then((res) => console.log(res))
})()
```

## async/await 实现原理

> async/await 是由 `generator` 函数 来实现的，该函数属于 ES6 新特性

```js
async function getResult() {
  await new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(1)
      console.log(1)
    }, 1000)
  })

  await new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(2)
      console.log(2)
    }, 500)
  })

  await new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(3)
      console.log(3)
    }, 100)
  })
}

getResult()

//  1，2，3.
```

那用 `generator` 函数专门来实现这个效果呢

```js
function* getResult(params) {
  yield new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(1)
      console.log(1)
    }, 1000)
  })

  yield new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(2)
      console.log(2)
    }, 500)
  })

  yield new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(3)
      console.log(3)
    }, 100)
  })
}
const gen = getResult()

gen.next()
gen.next()
gen.next()

//  3 2 1
```

是因为 三个 new Promise 几乎是同一时刻执行了。才会出现这种问题，所以需要等第一个 promise 执行完 resolve 之再执行下一个，所以要这么实现

```js
const gen = getResult()

gen.next().value.then((res) => {
  gen.next().value.then(() => {
    gen.next()
  })
})

// 1 2  3
```

但是呢，总不能有多少个 await，就要自己写多少个嵌套吧，所以还是需要封装一个函数，显然，递归实现最简单

```js
const gen = getResult()

function co(g) {
  const nextObj = g.next()
  if (nextObj.done) {
    return
  }
  nextObj.value.then(() => {
    co(g)
  })
}

co(gen)
```
