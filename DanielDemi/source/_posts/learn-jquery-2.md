---
title: 学习jquery源码-2 （jquery-2.0.3）（不兼容ie6,7,8）
comments: true
toc: true
date: 2018-05-26 16:40:01
categories: jquery
tags: jquery
---
jquery 21至94行定义一些变了和函数
```
(function(window, undefined)) {
    // 这边传入undefined是因为，该词不是保留字符，在有些浏览器中是可以修改其值的
    var 
    rootjQuery,
    readyList,
    // 这边保存字符串的undefined,这是为了兼容ie6，7，8，9。见下面的代码库1
    core_strundefined = typeof undefined, 
    // 变量存储
    location = window.location,
    document = window.document,
    docElem = document.documentElement,
    // 这边是为了防冲突，见代码库2
	_jQuery = window.jQuery,
	_$ = window.$,
    // $.type()中所需变量
    class2type = {},
    // 老版本是缓存数据相关变量(2.0版本已经没有用处了)
    core_deletedIds = [],

    core_version = "2.0.3",

	// 方法存储
	core_concat = core_deletedIds.concat,
	core_push = core_deletedIds.push,
	core_slice = core_deletedIds.slice,
	core_indexOf = core_deletedIds.indexOf,
	core_toString = class2type.toString,
	core_hasOwn = class2type.hasOwnProperty,
	core_trim = core_version.trim, // 去除前后空格
    // 这边jquery是返回一个对象，见代码库3
    jQuery = function( selector, context ) {
		return new jQuery.fn.init( selector, context, rootjQuery );
	},
    // 正则
    // 匹配数字用的
    core_pnum = /[+-]?(?:\d*\.|)\d+(?:[eE][+-]?\d+|)/.source,
    // 匹配空格
    core_rnotwhite = /\S+/g,
    // 匹配标签<p>aaa 或 #div
    rquickExpr = /^(?:\s*(<[\w\W]+>)[^>]*|#([\w-]*))$/,
    // 独立的空标签<p></p>
    rsingleTag = /^<(\w+)\s*\/?>(?:<\/\1>|)$/,
    // css兼容
    // ie前缀
    rmsPrefix = /^-ms-/,
    // 大小写
	rdashAlpha = /-([\da-z])/gi,
    // dom驼峰处理
    fcamelCase = function( all, letter ) {
		return letter.toUpperCase();
	},
	completed = function() {
		document.removeEventListener( "DOMContentLoaded", completed, false );
		window.removeEventListener( "load", completed, false );
		jQuery.ready();
	};
}(window)
```
代码库1
```
var a;
一般情况下有2种判断a为underfined
1. a == undefined
2. typeof a == 'undefined'
注：但是在ie6789下，xmlNode.method使用第一种方式是无法判断出来的
```
代码库2
```
一般情况下，可能已经使用了$或jquery
    var $ = 10;
    var jquery = 'test';
在这种情况下，我们在引入jquery.js就可以把这些变量先存储起来
    <script src="jquery-2.0.3.js"></script>
```
代码库3
```
在jquery中我们都是这么使用的
    jQuery().css()
    $().css()
我们想要这么使用，这说明jquery()就是一个对象
我们平常创建一个对象都是这样的A
A() {

}
A.prototype.init = {}
A.prototype.css = {}
var x =new A()
x.init();
x.css();

那么在jQuery中他不像前面的，是直接调用方法
$().css()
他没有经过new，和init初始化，这说明在内部已经创建了对象，并且初始化了
因此它返回的 new jQuery.fn.init( selector, context, rootjQuery )
但是new 的是init相当于初始化了，但是对象是init的，它是如何使用jQuery下prototype属性方法呢
就是通过以下一段代码实现的
jQuery.prototype.init.prototype = jQuery.prototype
```



