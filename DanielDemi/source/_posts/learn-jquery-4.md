---
title: 学习jquery源码-4 （jquery-2.0.3）（不兼容ie6,7,8）
comments: true
toc: true
date: 2018-05-27 15:29:14
categories: jquery
tags: jquery
---
## jquery代码285-347行 extend: jQuery的继承方法（可用于插件和方法的扩展）

首先我们来看一下如何使用extend的
```
$.extend()
$.fn.extend()

1. 扩展工具方法
$.extend({
    aaa: function() {
        console.log(111)
    },
    bbb: function() {
        console.log(222)
    }
})
调用：
$.aaa()
$.bbb()
2.扩展jq实例方法
$.fn.extend({
    aaa: function() {
        console.log(111)
    },
    bbb: function() {
        console.log(222)
    }
})
调用：
$().aaa()
$().bbb()
3.扩展属性
var a = {}
$.extend('a', {name: 'xxx'})

4.深拷贝和浅拷贝
var a = {}
var b = {name: 'hello', age: {t: 10}}

$.extend(a, b)

a.name = 'hi'
console.log(b.name) // hello 不受影响
a.age.t = 20;
console.log(b.age.t) // 20 受影响
这主要是由于$.extend默认是浅拷贝

如果需要深拷贝那么需要这么使用
$.extend(true, a, b)
a.age.t = 20;
console.log(b.age.t) // 10 不受影响
```

接下来开始看源码：先简单看下架构
```
jQuery.extend=jQuery.fn.extend = function() {
    // 定义一些变量
    if() {}  // 看是不是深拷贝情况

    if() {}  // 看参数正确不

    if() {}  // 看是不是插件情况

    for() { // 可能多个对象
        if() {}  // 防止循环引用
        if() {}  // 深拷贝
        else if () {} // 浅拷贝
    }
}
```
接下来是源码及解释
```
jQuery.extend = jQuery.fn.extend = function() {
	var options, name, src, copy, copyIsArray, clone,
		target = arguments[0] || {},
		i = 1,
		length = arguments.length,
		deep = false;

	// 判断第一个参数是否为boolean，用于检测是否为深拷贝
	if ( typeof target === "boolean" ) {
		deep = target;  // 是否深拷贝
		target = arguments[1] || {}; // 目标对象
		i = 2;
	}

	// 目标元素不是对象或不是函数时，默认变为空的json
	if ( typeof target !== "object" && !jQuery.isFunction(target) ) {
		target = {};
	}

	// 这边判断是不是插件
	if ( length === i ) {
        // $.extend和$.fn.extend的this是不同的，因此扩展使用方式也就不同了
		target = this;
		--i;
	}

	for ( ; i < length; i++ ) {
		// 判断后面参数是否有值
		if ( (options = arguments[ i ]) != null ) {
			// 遍历对象属性
			for ( name in options ) {
				src = target[ name ];
				copy = options[ name ];

				// 碰到循环引用时跳出；如a = {} b = {x: a} $.entend(a,b)
				if ( target === copy ) {
					continue;
				}

				// 深拷贝，拷贝的值是数组或对象
				if ( deep && copy && ( jQuery.isPlainObject(copy) || (copyIsArray = jQuery.isArray(copy)) ) ) {
					if ( copyIsArray ) {
						copyIsArray = false;
						clone = src && jQuery.isArray(src) ? src : []; // 这边判断下原对象是否包含该属性，如果有则在原属性上进行扩展
    				} else {
						clone = src && jQuery.isPlainObject(src) ? src : {};
					}

					// 在进行一次拷贝
					target[ name ] = jQuery.extend( deep, clone, copy );

				// 浅拷贝，直接赋值
				} else if ( copy !== undefined ) {
					target[ name ] = copy;
				}
			}
		}
	}

	return target;
};
```