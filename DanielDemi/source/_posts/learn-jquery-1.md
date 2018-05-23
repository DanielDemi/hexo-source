---
title: 学习jquery源码-1 （jquery-2.0.3）（不兼容ie6,7,8）
comments: true
toc: true
date: 2018-05-22 20:13:29
categories: jquery
tags: jquery
---
## 1. jquery 最外面有立即执行函数(防止污染，防止代码冲突)
```
(function( window, undefined ) {
    // 内容
})( window );
```
## 2. 如何对外暴露jquery呢？
```
if ( typeof window === "object" && typeof window.document === "object" ) {
    window.jQuery = window.$ = jQuery;
}
```
jquery通过把jQuery和$挂在到window上，那么就可以使用了。
## 3.如何避免冲突呢
```
_jQuery = window.jQuery,

// Map over the $ in case of overwrite
_$ = window.$,


noConflict: function( deep ) {
    if ( window.$ === jQuery ) {
        window.$ = _$;
    }

    if ( deep && window.jQuery === jQuery ) {
        window.jQuery = _jQuery;
    }

    return jQuery;
}
```
通过一开始保留全局变量，最后在通过noConflict来改变jquery命名

## 整理架构划分： （jquery是面向对象的）
1. (21-94)行：定义了一些变量和函数 jQuery = function() {}
2. (96-283)行: 定义了jquery的原型jQuery.fn = jQuery.prototype（fn即代表原型） 即给jQuery添加一些方法和属性
3. (285-347)行: extend: jQuery的继承方法（可用于插件和方法的扩展）
4. (349-817)行：jQuery.extend(): 扩展一些工具方法（静态方法和静态属性）
5. (877, 2856)行：sizzlejs选择器的实现（如$('ul li .div>sd')）,jQuery的核心，可单独使用（在官网直接下载sizzlejs）
6. (2880, 3042)行: Callbacks: 回调对象：对函数的统一管理
7. (3043, 3183)行：Deferred: 延迟对象，对异步的统一管理（定时器，dom加载结束，ajax等异步）
8. (3184, 3295)行：support: 功能检测
9. (3308, 3652)行：data(): 数据缓存 
10. (3653, 3797)行：queue(): 队列管理 (animate就使用了queue)
11. (3803, 4299)行：attr(), prop(), val(), addClass()等，对元素属性的操作
12. (4300, 5128)行：on(), trigger()： 事件操作相关方法 
13. (5140, 6057)行：dom操作： 添加 删除 获取 包装 等
14. (6058, 6620)行：css(): 样式操作 
15. (6621, 7854)行：提交数据和ajax()： ajax(), load(), getJson()等
16. (7855, 8584)行: animate(): 运动方法
17. (8585, 8792)行：offset(): 位置和尺寸的方法
18. (8804, 8821)行：jQ支持模块化的模式
19. (8826) window.jQuery = window.$ = jQuery