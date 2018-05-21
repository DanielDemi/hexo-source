---
title: 你不知道的JS系列（五）
comments: true
toc: true
date: 2018-05-21 22:58:34
categories: js
tags: js
---
1.js本身是不支持异步的机制的，只是浏览器解析制作的
2.异步最常见的就是回调函数，这边就需要理解一下事件轮询和队列；
所谓的事件轮询就是把到该运行该语句的时候，把该语句push到事件轮询机制里，
事件轮询按照先后顺序就行运行,所以使用settimeout()并不能十分精确地计时，
因为如果时间到了，事件轮询队列里还有其他待运行的，那么运行时间就会出现偏差。
3.generator(迭代器) 一个可以按照想要的顺序进行执行语句
迭代器写法：function *name() {}
通过在迭代器内部写yield来做语句运行的截止点
通过next来启用程序
以下为demo: *foo迭代器，it1和it2是迭代器实例（一开始是不会运行的，只有当next才会运行）,
yield可以喝语句进行互动，yeild后面的字段会被next()运行停止后解析;(形成互动)
```
function *foo() {
	var x = yield 2;
	z++;
	var y = yield (x * z);
	console.log( x, y, z );
}

var z = 1;

var it1 = foo();
var it2 = foo();

var val1 = it1.next().value;			// 2 <-- 让出2
var val2 = it2.next().value;			// 2 <-- 让出2

val1 = it1.next( val2 * 10 ).value;		// 40  <-- x:20,  z:2
val2 = it2.next( val1 * 5 ).value;		// 600 <-- x:200, z:3

it1.next( val2 / 2 );					// y:300
										// 20 300 3
it2.next( val1 / 4 );					// y:10
										// 200 10 3
```
另外可通过[Symbol.iterator]制造迭代器
var a = [1,3,5,7,9];

var it = a[Symbol.iterator]();

it.next().value;	// 1
it.next().value;	// 3
it.next().value;	// 5
..
