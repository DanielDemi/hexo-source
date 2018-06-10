---
title: vue采坑记录
comments: true
toc: true
date: 2018-05-30 22:15:22
categories: vue
tags: vue
---
#### 1.数据双向绑定
```
在vue中数据双向绑定是十分重要的，那么所有的参数都必要现有定义，
如果没有一开始没有定义参数的话，那么就不会双向实现
需要通过set来修改变量
<input v-model="test.val">

data() {
    return {
        test: {}
    }
},
created() {
    test.val = 1;
}
// 这种情况下是不会双向绑定的

需要这样
created() {
    this.$set(test, val, 1)
}
// 经过set设置，会将该参数进入watch，那么就会双向绑定了

当然一开始data里面直接有val参数就完全不是问题了，这类主要是针对动态创建的问题
``` 
#### 2.修改其他组件的样式
``` 
在vue文件中，为让文件样式独立，我们经常会在style加上scoped，这样
这里的样式只会作用于该文件，但是这样我们修改自组件的样式的时候，往往是不生效的
需要这样设置才行,在需要修改子组件样式前面加/deep/，如

parent /deep/ child {}
``` 