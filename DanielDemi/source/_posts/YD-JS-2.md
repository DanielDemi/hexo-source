---
title: 你不知道的JS系列（二）
date: 2018-05-12 13:37:39
comments: true
toc: true
categories: js
tags: js
---
## 1.toSring()/1.toFixed(3)等这些为什么执行会报错？

想要知道为什么，那么首先我们就必须清楚number这一类

``` bash
a = 4
b = 4.0
c = 4.
```

这三个都可以表示为数字4
所以1.toString()的问题出在1.的解析上面;
再如下代码应该会清楚点了

``` bash
1.toSring() // 报错
a = 1
a.toString() // "1"
1.1.toString() // "1.1"
```

从上可以看出并不是数字就无法.toString的。仅仅是因为整数的原因。因为4.就可以代码4
所以在1.toString的时候首先会解析1.为1因此就少了操作符号.（其实就是js解析方式的问题）。因此就会报错
那么可以通过如下方式来解决这个问题

``` bash
1..toString() // "1"
(1).toString() // "1"
1 .tiString() // "1"
等
```

综上已解释了1.toString()为什么会报错的原因