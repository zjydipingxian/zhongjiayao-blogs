---
sidebar:
  title: 浏览器事件循环
#   step: 95
isTimeLine: true
title: 浏览器事件循环
date: 2024-6-10 16:02:00
tags:
  - 技术笔记
categories:
  - 技术笔记
recommend: 9
---

# 事件循环

JavaScript 是单线程的，但是为了处理异步任务，它使用了事件循环（Event Loop）

## 浏览器进程

- 主线程（Main Thread）
- 插件线程（Plugin Thread）
- GPU 线程（GPU Thread）
- 渲染线程（v8）
- 网络线程（Network Thread）

## 浏览器事件循环

![alt text](image-7.png)

## 宏（普通）任务

可以将每次执行栈执行的代码当做是一个宏任务

- I/O（Input/Output）
- setTimeout
- setInterval
- setImmediate
- requestAnimationFrame

## 微任务

当 `宏任务` 执行完，会在渲染前，将执行期间所产生的所有微任务都执行完

- process.nextTick
- MutationObserver
- Promise.then catch finally

## 整体流程

- 执行一个当前执行栈同步任务（栈中没有就从事件队列中获取）
- 执行过程中如果遇到微任务，就将它添加到微任务的任务队列中
- 宏任务执行完毕后，立即执行当前微任务队列中的所有微任务（依次执行）
- 当前宏任务执行完毕，开始检查渲染，然后 GUI 线程接管渲染
- 渲染完毕后，JS 线程继续接管，开始下一个宏任务（从事件队列中获取）

## 自测

## case 1

```js
console.log(1);

queueMicrotask(() => {
  console.log(2);
});

Promise.resolve().then(() => console.log(3));

setTimeout(() => {
  console.log(4);
});
```

<details>
  <summary><mark><font color=darkred>点击查看答案</font></mark></summary>
  <p> 输出结果</p>
  <pre><code>  
  // 1 2 3 4
  </code></pre>

    执行栈（ECStack）： [1]

    微任务：[console.log(2), console.log(3)]

    宏任务：[console.log(4)]

</details>

## case 2

```js
console.log(1);

setTimeout(() => {
  console.log(2);
  Promise.resolve().then(() => {
    console.log(3);
  });
});

new Promise((resolve, reject) => {
  console.log(4);
  resolve(5);
}).then((data) => {
  console.log(data);
});

setTimeout(() => {
  console.log(6);
});

console.log(7);
```

<details>
  <summary><mark><font color=darkred>点击查看答案</font></mark></summary>
  <p> 输出结果</p>
  <pre><code>  
  // 1 4 7  5 2  3 6
  </code></pre>

    执行栈（ECStack）： [1，console.log(4)， 7]

    微任务：[console.log(data);] // 第一轮没有了 -> 5

    宏任务：[
        console.log(2);
        Promise.resolve().then(() => {
            console.log(3)
        });

        console.log(6);
    ]

    微任务2：[ console.log(3)]

</details>

## case 3

```js
console.log(1);

setTimeout(() => {
  console.log(2);
  Promise.resolve().then(() => {
    console.log(3);
  });
});

new Promise((resolve, reject) => {
  console.log(4);
  resolve(5);
}).then((data) => {
  console.log(data);

  Promise.resolve()
    .then(() => {
      console.log(6);
    })
    .then(() => {
      console.log(7);

      setTimeout(() => {
        console.log(8);
      }, 0);
    });
});

setTimeout(() => {
  console.log(9);
});

console.log(10);
```

<details>
  <summary><mark><font color=darkred>点击查看答案</font></mark></summary>
  <p> 输出结果</p>
  <pre><code>  
    // 1410  567  239 8
  </code></pre>

    执行栈（ECStack）： [console.log(1) ,  console.log(4) , console.log(10)]

    微任务：[
            then((data) => {
                console.log(data); ->5 -> 第一轮的微任务
                Promise.resolve().then(() => {
                    console.log(6) 第一轮的微任务
                }).then(() => {
                    console.log(7) 第一轮的微任务

                    setTimeout(() => {
                    console.log(8)
                    }, 0);
                });
            })
        ]
    宏任务：[
       console.log(2);
        Promise.resolve().then(() => {
            console.log(3) 第二轮的微任务
        });,
        console.log(9);
    ]

    微任务2：[
        Promise.resolve().then(() => {
            console.log(3)
        });,
     ]

</details>

## case 4

```js
console.log(1);

setTimeout(() => console.log(2));

Promise.resolve().then(() => console.log(3));

Promise.resolve().then(() => setTimeout(() => console.log(4)));

Promise.resolve().then(() => console.log(5));

setTimeout(() => console.log(6));

console.log(7);
```

<details>
  <summary><mark><font color=darkred>点击查看答案</font></mark></summary>
  <p> 输出结果</p>
  <pre><code>  
  // 17 35 26 4
  </code></pre>

    执行栈（ECStack）： [1，7]

    微任务：[3， setTimeout(() => console.log(4))， 5]
    宏任务：[
        2
        6
    ]

</details>
