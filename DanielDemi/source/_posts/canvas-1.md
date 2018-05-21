---
title: canvas注意事项
comments: true
toc: true
date: 2018-05-21 23:00:17
categories: canvas
tags: canvas
---
1. canvas设置宽度使用width和height，不要使用style属性
2. API介绍
   ctx.beginPath() ; 开始一个画布；通过该方法来画不同的内容  
   ctx.closePath() 表示是否绘成一个闭环区域
   stroke() 连线绘制  
   fill() 填充绘制
   这2个绘制都会检测是否有beginPath来确定绘制的内容（代码）
   如果同时使用fill和stroke的时候需要注意一下，先fill在stroke。这样描边的宽度不会被填充覆盖
   
   lineCap 设置线条开始和结束的样式 ,可设置超出后圆角或方形展示  
   lineJoin 线与线相交时的样式
   save()和restore() 绘制前保存之前的绘图状态，之后再还原
   （用于对canvas容器进行整体图形变换操作 translate,scale,rotate,transform, setTransform）
   [setTransform忽略会忽略之前的操作，其余几个都会叠加之前的操作(在一对svae和restore之间)]
   
   createPattern( 可重复画img,canvas，video)