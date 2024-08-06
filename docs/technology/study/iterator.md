---
sidebar:
  title: 迭代器
#   step: 95
isTimeLine: true
title: 迭代器
tags:
  - 技术笔记
categories:
  - 技术笔记
recommend: 11
---

# 迭代器

在 ES6 中，迭代器（Iterator）是一种`接口`，为各种不同的数据结构提供统一访问的机制。一个`数据结构`只要部署 Iterator 接口，就可以完成遍历操作（即依次处理该数据结构的所有成员）

## 1. 什么是迭代器 [Symbol(Symbol.iterator)]

迭代器是一种特殊的对象，它具有一些为迭代过程设计的接口。迭代器在结构上具有如下特征：

::: tip :tada:

1. 有一个`next`方法，每次调用都会返回一个`结果对象`
2. 结果对象有两个属性：`value`和`done`。`value`表示下一个将要返回的值，而`done`是一个布尔状态，表达当前迭代是否已经结束。如果迭代还未完成，则`done`的值为`false`, 反之则为`true`
   :::

## 2. 什么是可迭代对象

::: tip :tada:

    具有 `Symbol.iterator` 属性的对象，即为可迭代对象。 如下：

    1. 数组
    2. 字符串
    3. Set集合
    4. Map 集合
    ...

这意味着这些数据结构的实例可以直接使用 for...of 或者扩展运算符进行遍历
:::

## 3. 迭代器与可迭代对象

迭代器对象：`Symbol.iterator` 属性是一个函数，调用该函数会返回一个迭代器对象

```js
let arr = [1, 2, 3]
let iterator = arr[Symbol.iterator]()

console.log(iterator.next()) // {value: 1, done: false}
console.log(iterator.next()) // {value: 2, done: false}
console.log(iterator.next()) // {value: 3, done: false}
console.log(iterator.next()) // {value: undefined, done: true}
console.log(iterator.next()) // {value: undefined, done: true}
```

可以看到，`Symbol.iterator`其实就是一个函数，它的返回值是一个特殊的对象，这个对象有两个属性：`value`和`done`。当迭代完成时，`done`的值为`true`，而其值为`undefined`。迭代完成后可以继续调用，没有次数的限制，但其返回的结果对象为`{value: undefined, done: true}`。

## 4. 模拟迭代器

::: tip :tada:

    模拟一个迭代器，需要实现以下两个接口：

1. `next()`方法，返回一个对象，包含两个属性：`value`和`done`。
2. `Symbol.iterator`方法，返回一个迭代器对象。

:::

```js
// 1. 模拟一个迭代器
function makeIterator(arr) {
  // 2. 实现 next() 方法
  return {
    next() {},
  }
}
```

而调用该方法会返回一个包含了`value`和`done`属性的结果对象。且该结果对象的`value`是`迭代`的，而`done`是表示当前迭代是否完成。因此我们需要一个能够记录当前迭代位置的指针，并且每一次的调用都去移动该指针。

```js
// 1. 模拟一个迭代器
function makeIterator(arr) {
  let i = 0

  // 2. 实现 next() 方法
  return {
    next() {
      // 当不超过数组长度时，返回一个对象，包含两个属性：value和done。
      if (i < arr.length) {
        return {
          value: arr[i++],
          done: false,
        }
      }

      return { value: undefined, done: true }
    },
  }
}

let arr = [1, 2, 3]
let iterator = makeIterator(arr)

console.log(iterator.next()) // {value: 1, done: false}
console.log(iterator.next()) // {value: 2, done: false}
console.log(iterator.next()) // {value: 3, done: false}
console.log(iterator.next()) // {value: undefined, done: true}
console.log(iterator.next()) // {value: undefined, done: true}
```

## 5. 自定义的迭代器

::: tip :tada:
如果希望自己创建的一个对象，默认也是可以迭代的，那么就可以在创建类的时候，给它加一个属性上添加方法[Symbol.iterator]，那么这个对象就也可以被 for...of 遍历了

:::

已知, 我们可以使用 for...of 循环数组, 但是不能循环 普通对象, 循环 普通对象 将会提示对象是不可迭代的, 如下代码: for...of 能够正常循环数组、但是不能循环普通对象 obj

```js
const arr = [1, 2, 3]

let obj = {
  key: 1,
}
// 循环数组
for (const value of arr) {
  console.log(value) // 1 2 3
}

// 循环普通对象
for (const value of obj) {
  console.log(value) // 报错: TypeError: obj is not iterable
}

// 那么我们可以这样

let obj = {
  a: 1,
  b: 2,
  c: 3,
  [Symbol.iterator]: function () {
    let i = 0

    let map = new Map()
    map.set('a', 1)
    map.set('b', 2)
    map.set('c', 3)

    // 2. 实现 next() 方法
    return {
      next() {
        let mapEntry = [...map.entries()]
        // 当不超过数组长度时，返回一个对象，包含两个属性：value和done。
        if (i < map.size) {
          return {
            value: mapEntry[i++],
            done: false,
          }
        }

        return { value: undefined, done: true }
      },
    }
  },
}

// 循环普通对象
for (const value of obj) {
  console.log(value) // 不报错了: TypeError: obj is not iterable
}

var [a, b, c] = obj
console.log('🚀 ~ key:', a, b, c)
```
