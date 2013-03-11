---
layout: post
title: "IOS官方文档阅读之动画"
categories:
- IOS
tags:
- IOS


---

###写于开头
----
最近开始阅读苹果IOS开发文档，发现果然还是官方文档里有价值的东西多。之后会随着阅读的进度，陆续写一些的阅读文章，大部分是文档内容的摘录（英文不太好，有些地方翻译会有些生硬）。将所看到的有帮助的文档中的内容记录下来，也算是一种总结。

本文所对应的官方文档：[《View Programming Guide for iOS》](http://developer.apple.com/library/ios/#documentation/windowsviews/conceptual/viewpg_iphoneos/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009503-CH1-SW2)
其中的Animations部分。

###动画
----

#####1、什么能够使用动画

UIKit框架和Core Animation都提供了对动画的支持，但是这两种技术对动画的支持程度有所不同。UIKit框架是通过UIView对象来支持动画的。UIView包含了一些支持动画的属性，但这并不意味着动画会自动产生。改变这些属性通常只是立即更新UIView对象的属性值，并不会有动画产生。想要使这些改变产生动画，你必须在一个动画块中改变这些属性值。

以下列出UIView中支持动画的属性:

-	frame
-	bounds
-	center
-	transform
-	alpha
-	backgroundColor
-	contentStretch

在一些需要展示更复杂动画或者不被UIView类所支持的动画的场合，能够使用Core Animation和视图下面的图层来创建动画。由于视图和图层对象是错综联系在一起的，所以改变视图的涂层会影响到视图本身。使用Core Animation，你能为一下视图图层的改变添加动画：

-	图层的大小和位置
-	执行变形时的中心点
-	3D空间中图层以及其子图层的变形
-	图层从其所在图层层级中的添加或者删除
-	图层与其兄弟图层间的Z-order关系
-	图层的阴影
-	图层的边框(包括图层是否圆角)
-	改变大小操作中图层部分的拉伸
-	图层的不透明度
-	子图层在父图层边框以外部分的裁剪行为
-	图层当前的内容
-	图层的光栅化行为

**注意：**如果视图包含了自定义的图层对象——也就是说，图层对象没有关联的视图——任何该图层的变化动画，都必须使用Core Animation来完成。

#####2、视图动画属性的改变

UIView类对象属性改变所产生动画的代码写法有两种：如果在iOS4以后的系统，可以使用代码块对象，更早的系统中可以使用Begin/Commit函数。

代码块可以使用的函数：

-	animateWithDuration:animations:
-	animateWithDuration:animations:completion:
-	animateWithDuration:delay:options:animations:completion:

以上这些函数，都是UIView的类函数。正是由于是类函数，在其中的动画块代码不仅仅只展示单一的视图，可以包含多个视图的变化。



