---
sidebar:
  title: 词法作用域和动态作用域
#   step: 99
isTimeLine: true
title: 词法作用域和动态作用域
# date: 2024-06-09
tags:
  - 技术笔记
categories:
  - 技术笔记
recommend: 3
---

# 词法作用域和动态作用域

:::tip 参考
[JavaScript 深入之词法作用域和动态作用域](https://github.com/mqyqingfeng/Blog/issues/3)
:::

### 1. 作用域

::: tip 大白话， 当前的执行环境 对这个模块的变量有访问权限
作用域是指程序源代码中定义变量的区域。
作用域规定了如何查找变量，也就是确定当前执行代码对变量的访问权限。
JavaScript 采用词法作用域(lexical scoping)，也就是静态作用域。

:::

### 2. 静态作用域和动态作用域

```js
var value = 1

function foo() {
  console.log(value)
}

function bar() {
  var value = 2
  foo()
}

bar()

// 结果是 ???   1

因为 JavaScript 采用的是词法作用域(静态作用域)，函数的作用域在函数定义的时候就决定了。
```

下面两段代码各自的执行结果是多少？

```js
// case 1
var scope = "global scope";
function checkscope() {
  var scope = "local scope";
  function f() {
    return scope;
  }
  return f();
}
checkscope(); // "local scope"

// case 2
var scope = "global scope";
function checkscope() {
  var scope = "local scope";
  function f() {
    return scope;
  }
  return f;
}
checkscope()(); // "local scope"

虽然两段代码执行的结果一样，但是两段代码究竟有哪些不同呢  -> 执行上下文解释
```
