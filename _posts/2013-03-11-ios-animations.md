---
layout: post
title: "IOS官方文档阅读之动画"
categories:
- iOS
tags:
- iOS


---

###写于开头
----
最近开始阅读苹果IOS开发文档，发现果然还是官方文档里有价值的东西多。之后会随着阅读的进度，陆续写一些的阅读文章，大部分是文档内容的摘录（英文不太好，有些地方翻译会有些生硬）。将所看到的有帮助的文档中的内容记录下来，也算是一种总结。

本文所对应的官方文档：[《View Programming Guide for iOS》](http://developer.apple.com/library/ios/#documentation/windowsviews/conceptual/viewpg_iphoneos/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009503-CH1-SW2)
其中的Animations部分。

###动画
----

#####什么能够使用动画

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

#####视图动画属性的改变

UIView类对象属性改变所产生动画的代码写法有两种：如果在iOS4以后的系统，可以使用代码块对象，更早的系统中可以使用Begin/Commit函数。

代码块可以使用的函数：

-	animateWithDuration:animations:
-	animateWithDuration:animations:completion:
-	animateWithDuration:delay:options:animations:completion:

以上这些函数，都是UIView的类函数。正是由于是类函数，在其中的动画块代码不仅仅只展示单一的视图，可以包含多个视图的变化。

如果应用程序是运行在iOS3.2及更早的版本，需要使用UIView的以下两个类函数：

	beginAnimations:context: 
	commitAnimations

来定义动画代码块。这两个函数标记了动画代码块的开始和结束位置。

**注意：**如果所写的应用程序是运行在iOS4及以上的环境中，应该使用基于块的函数

默认情况下，在一个动画块中所有动画属性的改变都会被展示。如果只希望其中一部分的变化以动画来表示，其余的不用。那么使用*setAnimationsEnabled:*函数可以暂时使动画失效，之后做任何不想使用动画的改变，然后再调用一次*setAnimationsEnabled:*函数来启用动画。还能通过调用*areAnimationsEnabled*函数来查看当前动画的可用状态。

#####为Begin/Commit函数配置参数

Begin/Commit函数有许多动画参数可以配置，需要通过使用UIView的一些类函数来完成。

配置参数的函数列表详见文档。


#####设置动画代理

如果希望在一段动画结束之后立即执行一段代码，需要为Begin/Commit函数关联一个代理对象和一个开始或者结束的选择器。通过UIView的类函数*setAnimationDelegate:*设置代理对象，类函数*setAnimationWillStartSelector:*和*setAnimationDidStopSelector:*设置开始结束选择器。

动画代理方法应该与以下形式类似：

	- (void)animationWillStart:(NSString *)animationID context:(void *)context;
	- (void)animationDidStop:(NSString *)animationID finished:(NSNumber *)finished context:(void *)context;

#####改变一个视图的子视图

当一个视图的子视图发生改变时，若需要动画来过渡，则需要使用函数*transitionWithView:duration:options:animations:completion:*来完成。

#####使用一个视图来替换另一个视图

当使用一个视图来替换另一个视图时，若需要动画过渡，则需要使用函数*transitionFromView:toView:duration:options:completion:*来完成

#####其他

可以使用完成块或者代理函数来完成多个动画的连接。

基于视图和基于图层的动画可以混合在一起使用。