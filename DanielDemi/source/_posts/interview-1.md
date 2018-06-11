---
title: 前端面试经典题目
comments: true
toc: true
date: 2018-06-11 22:29:06
categories: 前端面试
tags: 前端面试
---

### 这边列举了5个前端经典的面试题

首先我们来看一下题目，都各自思考一下，自己是否能够做出来。然后再跟后面的答案对比一下
#### 题目1
```
var a = {n:1}
var b = {n:2}
b.x = a = {n:3}
console.log(b.x)
a.x = a = {n:3}
console.log(a.x)
```

#### 题目2
```
（1）
var start = new Date()
setTimeout(function() {
    console.log(new Date() -start)
}, 500)
（2）
var start = new Date()
setTimeout(function() {
    console.log(new Date() -start)
}, 500)

while((new Date() - start <= 1000) {}
```

#### 题目3
```
var log = console.log
var hint = window.alert
var write = document.write
log('123')
hint('123')
wirte('123')

```

#### 题目4
```
var name = 'A'
function getName() {
    return this.name
}
var obj = {
    name: 'B',
    getName: function() {
        return this.name
    },
    showName:function(a) {
        console.log(getName())
        console.log(a())
        console.log(a === arguments[0])
        console.log(arguments[0]())
    } 
}
obj .showName(getName, 1)
```

#### 题目5
```
async function async1() {
    console.log('async1 start')
    await async2()
    console.log('async1 end')
}
async function async2() {
    console.log(async2())
}
console.log('script start')
setTimeout(function() {
    console.log('setTimeout')
}, 0)
async1()
new Promise(function (resolve) {
    console.log('promise1')
    resolve()
}).then(function() {
    console.log('promise2')
})
console.log('script end')
```



#### 答案1
```
console.log(b.x)   // {n:3}
console.log(a.x)   // undefined
这是跟js的优先级有关的 .的优先级最高，因此先计算左边的b.x,a.x
```

#### 答案2
```
(1)
console.log(new Date() -start)  // 不是一定500,而是大于等于500
主要是由于定时器的时间不一定准，这个跟异步的机制有关
当定时器的时间到了之后，执行函数会进入栈中，会根据栈中的执行任务，
依次执行（不懂的可以学习一下异步机制）
(2)
console.log(new Date() -start)  // 大于等于1000
由于是单线程
```

#### 答案3
```
log('123') // 正常
hint('123') // 正常
write('123') // 报错
这是由于write是相当于window.write；而document.write下面应该会用到this对象，
所以会报错
应当改成window.write(document, '123')
```

#### 答案4
```
console.log(getName()) // 'A'  this === window
console.log(a()) // 'A' this === window
console.log(a === arguments[0]) // true
console.log(arguments[0]()) // undefined this 为argument对象
```

#### 答案5
```
script start
async1 start
async2
promise1
async1 end
script end
promise2
setTimeout
这一块跟异步有关
先同步，后异步，在回调。还有promise机制
```