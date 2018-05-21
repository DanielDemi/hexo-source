---
title: 你不知道的JS系列（三）
date: 2018-05-12 13:37:39
comments: true
toc: true
categories: js
tags: js
---
## parseInt(1/0, 19) 答案是18；是不是很神奇？

想要了解这个答案，那么就需要了解parseInt这个函数了。
parseInt如果传入的是非字符串，那么首先会解析为字符串。
1/0被解析为‘Infinity’.
那么parseInt(1/0, 19)就变成了parseInt(‘Infinity’, 19)。
那么答案就是18了，是不是很神奇