---
sidebar:
  title: apply,call,bind
#   step: 95
isTimeLine: true
title: apply,call,bind
date: 2024-06-09
tags:
  - 技术笔记
categories:
  - 技术笔记
recommend: 7
---

# apply,call,bind

## 相同点

1. 更改 this 指向
   :::tip
   MDN:bind() 方法会创建一个新函数。当这个新函数被调用时，bind() 的第一个参数将作为它运行时的 this，之后的一序列参数将会在传递的实参前传入作为它的参数
   :::

```js
let obj = {
  a: 1,
  b: 2,
  test() {
    console.log(this.a + this.b);
  },
};

obj.test(); // 3
obj.test.apply({ a: 2, b: 2 }); // 4
obj.test.call({ a: 3, b: 3 }); // 6
obj.test.bind({ a: 4, b: 4 })(); // 8
```

## 不同点

传参方式不一样

- bind(this,...argv)
- call(this,...argv)
- apply(this,[...argv])

```js
function test(a, b) {
  console.log(this, a + b);
}

test.call("call", 1, 2); // [String: 'call'] 3
test.apply("apply", [2, 4]); // [String: 'apply'] 6
test.bind("bind", 3, 6)(); // [String: 'bind'] 9
```

## 手写 call

> call() ：在使用一个指定的 this 值和若干个指定的参数值的前提下调用某个函数或方法。

```js
let foo = {
  value: 1,
};

function bar() {
  console.log(this.value);
}

bar.call(foo); // 1
```

注意两点：

1. call 改变了 this 的指向，指向到 foo；
2. bar 函数执行了；

### 版本一

上述方式等同于：

```js
let foo = {
  value: 1,
  bar: function () {
    console.log(this.value);
  },
};

foo.bar(); // 1
```

这个时候 this 就`指向`了 foo，但是这样却给 foo 对象本身`添加`了一个属性，所以们用 `delete` 再删除它即可。
所以我们模拟的步骤可以分为：

```
// 第一步
// fn 是对象的属性名，反正最后也要删除它，所以起什么都可以。
foo.fn = bar
// 第二步
foo.fn()
// 第三步
delete foo.fn
```

根据上述思路，提供一版：

```js
// 第一版本
Function.prototype.myCall = function (context) {
  context.fn = this;
  context.fn();
  delete context.fn;
};

// 测试一下
let foo = {
  value: 1,
};

function bar() {
  console.log(this.value);
}

bar.call2(foo); // 1
```

### 版本二

call 除了可以指定 this，还可以指定参数 -> bar.call(foo, 'kevin', 18);

```js
// 第二版本
Function.prototype.myCall = function (context) {
  context.fn = this;
  const args = [...arguments].slice(1);
  context.fn(...args);
  delete context.fn;
};
```

### 版本三

this 参数可以传 null，当为 null 的时候，视为指向 window

```js
// 第三版本
Function.prototype.myCall = function (context) {
  context = context || window;
  context.fn = this;
  const args = [...arguments].slice(1);
  context.fn(...args);
  delete context.fn;
};
```

### 版本四

或者 是个 return 值

```js
let foo = {
  value: 1,
};

function bar(name, age) {
  return {
    value: this.value,
    name,
    age,
  };
}

bar.call(foo, "zjy", 18);

// 第四版本 带入参  arguments / null  或者 是个 return 值
Function.prototype.myCall = function (context) {
  context = context || window;
  let args = [...arguments].slice(1); // ['zjy', '18']
  context.fn = this;
  const result = context.fn(...args);

  delete context.fn;

  return result;
};
```

## 手写 apply

apply 的实现跟 call 类似，只是入参不一样，apply 为数组

```js
Function.prototype.myApply = function (context) {
  context = context || window;
  context.fn = this;
  const args = [...arguments].slice(1);
  const result = context.fn(args);
  delete context.fn;

  return result;
};
```

## 手写 bind

关于指定 this 的指向，我们可以使用 call 或者 apply 实现

### 版本一

```js
Function.prototype.myBind = function (context) {
  var self = this;

  return function () {
    return self.apply(context);
  };
};
```

### 版本二

传参的模拟实现

```js
let foo = {
    value: 1
}

function bar(name, age) {
  console.log(this.value)
  console.log(name)
  console.log(age)
}

var barFoo = bar.bind(foo, 'zjy')
barFoo(18) -> 1 zjy 18
```

```js
Function.prototype.myBind = function (context) {
  if (typeof this !== "function") {
    return;
  }

  var self = this; // 此时的this 指向 bar
  var args = [...arguments].slice(1);

  return function () {
    var bindArgs = [...arguments];
    return self.apply(context, [...args, ...bindArgs]);
  };
};
```

### 版本三

bind 还有一个特点，就是

> 一个绑定函数也能使用 new 操作符创建对象：这种行为就像把原函数当成构造器。提供的 this 值被忽略，同时调用时的参数被提供给模拟函数。

```js
var value = 2;

var foo = {
  value: 1,
};

function bar(name, age) {
  this.habit = "shopping";
  console.log(this.value);
  console.log(name);
  console.log(age);
}

bar.prototype.friend = "zjy";

var bindFoo = bar.bind(foo, "aaaa");

var obj = new bindFoo("18");
// undefined
// daisy
// 18
console.log(obj.habit);
console.log(obj.friend);
// shopping
// zjy
```

所以由如下

```js
Function.prototype.MyBind = function (context) {
  if (typeof this !== "function") {
    return;
  }

  // 当时 null 的时候 应该是window
  context = context || window;
  let args = [...arguments].slice(1);
  let self = this;

  var fn = function () {
    var bindArgs = [...arguments];

    /**  解释下面 三目为什么这么做
         *   var bindFoo = bar.bind(foo, 'aaaaaa')
             var obj = new bindFoo(18)

            在new的情况下 fn -> bindFoo  this -> fn  反正就判断是不是 new 一个构造函数

         * 
    */

    return self.apply(this instanceof fn ? this : context, [
      ...args,
      ...bindArgs,
    ]);
  };

  /*  解释 下面   fn.prototype = Object.create(self.prototype)


    bar.prototype.friend = 'zjy' === this.prototype 
    var bindFoo = bar.bind(foo, 'aaaaaa') ===  fn
    var obj = new bindFoo(18) obj的属性没有friend -> 实例的原型找 -> bindFoo.prototype === fn.prototype （要他有 ）
    bar.prototype(他有) 所以赋值
  */
  fn.prototype = Object.create(self.prototype);

  return fn;
};
```
