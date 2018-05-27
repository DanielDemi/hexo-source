---
title: 学习jquery源码-3 （jquery-2.0.3）（不兼容ie6,7,8）
comments: true
toc: true
date: 2018-05-27 15:29:14
categories: jquery
tags: jquery
---
## jquery代码96-283行 目录罗列，主要是添加了其属性和方法
```
jQuery.fn = jQuery.prototype = {
    jquery: 版本号
    constructor: 修正指向问题
    init(): 初始化和参数管理
    selector: 存储选择字符串
    length: this对象的长度
    toArray(): 转数组
    get(): 转原生集合
    pushStack(): JQ对象的入栈
    each(): 遍历集合
    ready(): DOM加载的接口
    slice(): 集合的截取
    first(): 集合的第一项
    last(): 集合的最后一项
    eq(): 集合的指定项
    map(): 返回新集合
    end(): 返回集合前一个状态
    // 内部方法
    push(): 
    sort():
    splice():
}
```
## jquery: 版本号
```
// 源码：
// 这边就是表明了jquery的版本号
jquery: core_version,
```
## constructor: 修正指向问题
```
// 源码：
constructor: jQuery


// 构造器， 一般构造函数会自动生成
但是构造器是很容易被修改的。
A() {}
(1)A.prototype.aa = 's';
alert(A.constructor) // A()
(2)A.prototype = {
    aa: 's'
}
alert(A.constructor) // Object() 
这边（1）（2）2种方法都是实现了增加属性aa,效果是一样的，但是第二种
方法会覆盖原有的方法，既构造函数，因此指向是会被修改的。
因此jquery需要把指向就行修正
```
## init(): 初始化和参数管理
```
// 源码：
// 选择器selector 限制条件context 内部参数不对外暴露rootjQuery：$(document)
// 通常我们选择元素是这样的$('#div')  $('.box')  $('div')  $('#div div.box') 这种事各类选择权选择元素
// 另外一种是创建标签 $('<li>')  $('<li>1</li>')  这种都是创建标签
init: function( selector, context, rootjQuery ) {
    var match, elem;

    // 对不正确的选择器进行处理: $(""), $(null), $(undefined), $(false)
    if ( !selector ) {
        return this;
    }

    // 处理字符串 如 $('#div')  $('.box')  $('div')  $('#div div.box')或  $('<li>')  $('<li>1</li>')
    if ( typeof selector === "string" ) {
        // 处理$('<li>')
        if ( selector.charAt(0) === "<" && selector.charAt( selector.length - 1 ) === ">" && selector.length >= 3 ) {
            // 
            match = [ null, selector, null ];

        } else {
            // 处理选择器 $('#div')  $('.box')  $('div')  $('#div div.box');此外还有不规则的标签如$('<li>hello')
            // rquickExpr = /^(?:\s*(<[\w\W]+>)[^>]*|#([\w-]*))$/ 
            // 用于匹配 $('#id') 和 $('<li>hello') 会有结果 match = []
            // $('#id') match = ['#id', null, 'id']
            // $('<li>hello') match = ['<li>hello', '<li>', nill]
            // 而$('.box')  $('div')  $('#div div.box')则会是match = null
            match = rquickExpr.exec( selector );
        }

        // 根据上面的处理match有3种情况
        // 1.null
        // 2.['#id', null, 'id']
        // 3.['<li>hello', <li>, null]
        // 4.[null, <li>, null]

        // 2,4可以进入 ,为什么2可以进入，2的match[1]为null，因为id选择器不需要上下文限制，因此!context为真
        if ( match && (match[1] || !context) ) {

            // 4进入这里，就是标签 context只能是document，但是可以指定不同的document，如iframe下的
            if ( match[1] ) {
                // 可能填写 document 或 $(document) 最终得到都是原生document
                context = context instanceof jQuery ? context[0] : context;

                // scripts is true for back-compat
                // jQuery.parseHTML 把字符串转换成节点数组;见代码库init-1
                // jQuery.merge 合并数组,类数组
                // 这边返回this对象即为类数组
                // {
                //    0：xxx
                //    1: xxx
                //    ...
                //    length: n
                // }
                jQuery.merge( this, jQuery.parseHTML(
                    match[1],
                    context && context.nodeType ? context.ownerDocument || context : document,
                    true
                ) );

                // 针对标签带属性的处理，如 $('<li>', {title: 'tese', html:'abcd'})
                // rsingleTag匹配单标签  jQuery.isPlainObject判断是否为对象
                if ( rsingleTag.test( match[1] ) && jQuery.isPlainObject( context ) ) {
                    for ( match in context ) {
                        // jquery是否有对应的方法， 如html。直接函数调用
                        if ( jQuery.isFunction( this[ match ] ) ) {
                            this[ match ]( context[ match ] );

                        // 其他都作为属性
                        } else {
                            this.attr( match, context[ match ] );
                        }
                    }
                }

                return this;

            // id选择器进入这里
            } else {
                elem = document.getElementById( match[2] );
                // 确保节点存在，   
                // 在clone时,某种情况下即使删除了节点也会存在
                // 因此必须在判断下是否有父节点
                if ( elem && elem.parentNode ) {
                    this.length = 1;
                    this[0] = elem;
                }

                this.context = document;
                this.selector = selector;
                return this;
            }

        // 处理: $(expr, $(...))
        } else if ( !context || context.jquery ) {
            return ( context || rootjQuery ).find( selector );
        // 处理: $(expr, context)
        } else {
            return this.constructor( context ).find( selector );
        }

    // 处理节点 如$(this) $(document)
    } else if ( selector.nodeType ) {
        this.context = this[0] = selector;
        this.length = 1;
        return this;

    // 处理函数形式如 $(function() {})
    // Shortcut for document ready
    } else if ( jQuery.isFunction( selector ) ) {
        return rootjQuery.ready( selector );
    }
    // 处理数组和对象 $([]) $({})
    if ( selector.selector !== undefined ) {
        this.selector = selector.selector;
        this.context = selector.context;
    }

    return jQuery.makeArray( selector, this );
}
```
代码库init-1
```
jQuery.parseHTML：
var str = '<li>1</li><li>2</li><li>3</li>'
var arr = jQuery.parseHTML(str, document, true); // ['li', 'li', 'li']
// flag 表示script标签是否可以填进来
parseHTML(str, document, flag)
var src = '<script>alert(1)<\/script>'
jQuery.parseHTML(str, document, false) // 添加失败
jQuery.parseHTML(str, document, true) // 添加成功

jQuery.merge：
var arr = ['a', 'b']
var arr2 = ['c', 'd']

console.log($.merge(arr, arr2)) // ['a','b','c','d']
对我们对外就是这么使用的合并数组

在内部可以合并特殊的json形式，即类数组
var arr3 = {
    0: '1',
    1: 'v',
    length: 2
}
$.merge(arr3, arr2) 
//  {
    0: '1',
    1: 'v',
    2：'c',
    3: 'd',
    length: 4
}
```
## selector和length
```
selector: "", // 用于保存选择器
length: 0, // 记录节点长度
```
## 一些方法
```

```

