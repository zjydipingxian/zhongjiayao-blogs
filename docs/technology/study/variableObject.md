---
sidebar:
  title: 变量对象
#   step: 97
isTimeLine: true
title: 变量对象
date: 2024-06-09
tags:
  - 技术笔记
recommend: 5
categories:
  - 技术笔记
---

# 变量对象

### 变量对象

当 JavaScript 代码执行一段可执行代码(executable code)时，会创建对应的执行上下文(execution context)。
对于每个执行上下文，都有三个重要属性：

1. variable object -> 作用域中定义的变量 Vo
2. 作用域链
3. this

在上下文 变量 或者 函数的声明 --

### 全局上下文

```js
console.log(this); // window -> 全局对象
```

### 函数上下文

activation object -> 活动(激活)对象 Ao
arguments -> 函数参数

function () {
arguments
}

function foo() {
console.log(a)
var a = 1
}
foo()

```js



var value = 1;

function foo() {
    console.log(value);
}

function bar() {
    var value = 2;
    foo();
}

bar();

// 函数执行上下文被压入执行上下文栈
ECStack = [
    fooContext
    barContext,
    globalContext
];


// 1. 我的猜想！！！！！！
globalContext = {
    VO: '全部的变量参数',
    Scope: [globalContext.VO],
    this: globalContext.VO
}

barScope.[[scope]] = [
    globalContext.VO
];

fooScope.[[scope]] = [
    barContext.AO,
    globalContext.VO
];


// 执行上下文都有三个属性，变量对象 VO/AO，作用域链 Scope  ，this
// 准备-初始化
fooContext = {
    AO: {
        arguments: {
            length: 0
        },
    },
    Scope: fooScope.[[scope]],
    this: undefined
}

//执行
fooContext = {
    AO: {
        arguments: {
            length: 0
        },
    },
    Scope: [AO ...fooScope.[[scope]] ],  // 合并关联
    this: undefined
}

// 初始化
barContext = {
    AO: {
        arguments: {
            length: 0
        },
        value: undefined,
        foo: reference to function f(){}
    },
    Scope: barScope.[[scope]],
    this: undefined,
}

// 执行
barContext = {
    AO: {
        arguments: {
            length: 0
        },
        value: 2,
        foo: reference to function f(){}
    },
    Scope:[AO ,...barScope.[[scope]]],
    this: undefined
}

```
