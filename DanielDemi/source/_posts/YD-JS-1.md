---
title: 你不知道的JS系列（一）
date: 2018-05-12 13:37:39
categories: js
tags: js
---
## 在函数中为什么这样调用[].slice.call(arguments)?

先来看一段代码

``` bash
function test(params) {
    console.log(arguments);
}
test(1) // 输出为[1]
```

从这边看出arguments不就是一个数组吗？
那么我们来看一下下面代码，一般来如果参数不是arguments,
而是普通的数组的话，那么下面这段代码arguments参数上。
    
``` bash
[].slice.call(arguments) === arguments.slice() ？ // false
```

js中的arguments参数是一个类数组对象，它并不具有数组的方法，所以需要通过call来引用其方法。
至此问题已经解决了。

// ps 可以通过测试测试arguments的type是否为Array来验证下
另外补充一个小知识点arguments类数组对象里有个方法叫callee，该方法就是为该函数本身，可用于迭代循环调用