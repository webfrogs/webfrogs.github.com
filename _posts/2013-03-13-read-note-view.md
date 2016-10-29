---
layout: post
title: "《View Programming Guide for iOS》阅读笔记"
categories:
- iOS
tags:
- iOS


---



文档地址: [《View Programming Guide for iOS》](http://developer.apple.com/library/ios/#documentation/windowsviews/conceptual/viewpg_iphoneos/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009503-CH1-SW2)



## View and Window Architecture
----
-	#### 视图绘制周期
UIView类使用了请求式绘制模型来展示内容。当一个视图第一次出现在屏幕上时，系统要求它绘制自己的内容。系统截取视图内容的一个快照，并且将这个快照用于视图的可视化呈现。如果视图内容永远不改变，那么这个视图的绘图代码可能永远都不会再次调用。这个快照的图片在大部分涉及到该视图的操作中被重复使用。如果改变了视图内容，则需要通知系统视图发生了改变。之后视图会重复绘制过程并且为新的绘制结果截取一个快照。

当视图内容发生改变时，不需要直接重绘这些改变。相反，通过调用函数*setNeedsDisplay* 或者*setNeedsDisplayInRect:*来使当前视图无效。这些函数会告诉系统视图的内容发生改变并且需要在下次时机到来时重绘。系统会一直等到当前的run loop结束后，才会开始任何绘制操作。这个延迟，给了你一个机会去废止多个视图，从当前视图层级中添加或者删除视图，隐藏视图，重设视图大小，和重定位视图。所有的这些改变稍后会再同一时间呈现。

备注：改变视图的几何结构并不会让系统自动重绘视图内容。视图的*contentMode*属性决定了视图几何结构的改变该如何解析。大部分的content modes只是在视图的边界中拉伸或者重定位已经存在的视图快照而不需要重新创建一个快照。

当绘制视图内容的时刻到来时，真正的绘制过程会根据视图和它的配置而有所不同。系统视图通常是实现自己的私有绘图函数来重绘内容。这些一样的系统视图通常会暴露一些接口，以便能用来配置视图实际的外观。对于自定义的UIView的子类，典型的应该重写视图的*drawRect:*函数，使用它来绘制视图内容。当然也存在一些其他的方法去提供视图的内容，比如直接设置内容下的图层。但是重写*drawRect:*函数是使用最多的技术。

-	UIKit框架的坐标原点位于左上角，x轴向右延伸，y轴向下延伸。而Core Graphics和OpenGL ES的坐标系统原点则在左下角，y轴向上延伸，x轴向右延伸。

-	UIView属性中的frame和center是相对于父视图的坐标系统的。而bounds属性相对于自身的坐标系统，故bounds默认的point位置是(0,0)，大小与frame相同

-	改变视图的transform属性时，所有变形都是相对于视图中心点也就是center属性的。
-	在视图的drawRect:方法中，可以使用仿射变换来定位和确定需要绘制的元素。相比于在视图的某个地点固定一个对象的位置，相对于一个固定点（通常是(0,0)）来创建每个对象是更为简单的。在绘制之前使用transform就能做到这点。在这种情况下，如果视图中的对象位置发生改变，只需要修改这个transform即可，这比在新的位置重新创建对象要快速并且花销更小。可以使用CGContextGetCTM函数来检索图形上下文的仿射变换矩阵，在绘制过程中也可以使用Core Graphics的相关函数来设置CTM.

	CTM(current transformation matrix)是任何时候都被使用的仿射变换，当操作的是整个视图时，CTM就是视图的transform属性。在drawRect:方法中，CTM与当前活动的图形上下文有关

-	当一个视图的transform属性不是identity transform时，这个视图的frame属性就是未定义并且必须被忽视的。此时，你必须使用视图的bounds和center属性来获得视图的大小和位置。该视图的任何子视图的frame矩形依然是有效的，因为它们是基于父视图的bounds属性的。

-	一个点并不一定对应着屏幕上的一个像素
-	对于显式定义了drawRect:方法的视图来说，UIKit负责调用这个方法。这个方法中的实现应该尽可能快地重绘视图的指定区域并且不应该做别的任何事情。不要在这里做额外的布局，也不要改变应用的数据模型。这个方法的唯一目的就是更新视图的可视内容。
-	自定义视图需要重写的事件处理函数有*touchesBegan:withEvent:*， *touchesMoved:withEvent:*， *touchesEnded:withEvent:*， *touchesCancelled:withEvent:* 如果使用了手势识别来处理事件，则不需要重写这些函数。如果视图不包含任何子视图或者它的尺寸不发生改变，也不需要重写*layoutSubviews*函数。最后，当视图内容在运行时发生改变，同时使用了UIKit或者Core Graphics来绘制图形，则需要重写*drawRect:*函数。

## Windows
---
-	每个iOS应用程序至少包含一个窗口。窗口通常座位一个或者多个视图的空白容器。同时，应用程序也不通过展示新的窗口来改变内容。如果想要这么做，改变窗口最前面的视图来完成。

-	当创建窗口时，应该总是将窗口的大小设置为充满屏幕的边界。不应该为了容纳状态栏或者其他元素而减去窗口大小。无论何时，状态栏总是浮在窗口的上面的。所以应该是放入到窗口中的视图来缩减大小去适应状态栏。如果是使用视图控制器，则视图控制器应该自动处理视图大小。

-	窗口有等级概念，每个*UIWindow*对象都有一个可配置的*windowLevel*属性。通常不需要改变应用程序的窗口等级。新的窗口在创建时，会自动指派为正常窗口等级。高窗口等级是出现在应用程序内容之上的必要信息，比如系统状态栏和alert消息。虽然可以手动将窗口设置为这样的等级，但当使用到特殊接口时，通常系统会做好这些事情。举例来说，当显示隐藏状态栏，或者显示一个alert视图时，系统会自动创建必要的窗口去显示这些内容。

-	当应用程序进入到后台时，窗口改变通知并不会被传递。因为当程序进入后台时尽管窗口不在屏幕上显示了，但在应用程序环境中，窗口依然被认为是可见的。

-	retina屏的iOS设备可以外接显示设备。

## Views
----

-	使用编程方法来创建视图时，视图创建代码一般放在视图控制器的*loadView*函数中。无论是使用编程或者nib文件来创建视图，都可以在*viewDidLoad*函数中添加视图的配置代码。
-	父视图会自动retain子视图，所以当添加了一个子视图后，release子视图的操作是安全的。事实上，推荐这么做，因为它能防止应用程序保持太多的视图而导致的内存泄露。记住，如果从父视图中移除了子视图后，还想继续使用子视图，必须对子视图做retain操作。*removeFromSuperview*函数会在子视图从父视图中移除后，自动释放子视图。如果没有在下一个时间循环周期前做retain操作，这个视图将会被释放。
-	UIView的*window*属性代表当前正在显示的视图所在的窗口。对于当前在屏幕上显示的视图来说，窗口对象就是它们所在视图层次的根视图。
-	如果隐藏的视图是first responder，这个视图不会自动的取消自己first responder的状态。以first responder为目标的事件依然会被传递到这个隐藏的视图。为了防止这种情况发生，应该在隐藏视图时，强制使其取消first responder状态。
-	当包含了旋转因子的视图做矩形转换时，看如下的图：

![图片](http://developer.apple.com/library/ios/documentation/windowsviews/conceptual/viewpg_iphoneos/Art/uiview_convert_rotated.jpg)

-	如果一个视图的*transform*属性不是identity transform，那么它的frame和autoresizing行为结果都是未定义的。

-	视图在初始化过程之前调用UIView类方法*layerClass*，并且使用返回的类来创建layer对象。此外，视图总是会指定自己本身作为layer对象的代理。在这点上，视图拥有着图层，并且视图和图层之前的关系必须不能改变。也就是说，不能指定相同的视图作为另一个图层对象的代理。改变这种所属关系或者代理关系，都可能会导致视图绘制的错误，并且应用程序存在潜在的crash问题。

-	通过创建UIView的子类，重写*layerClass*类函数可以改变创建图层时的默认的CALayer类。
-	自定义的layer不接收事件，也不参与到responder chain中，但是它们绘制自身，并且根据Core Animation的规则，在他们的父视图或者父图层中响应大小变化。
-	CGRectGetMidX(),CGRectGetMidY()两个函数可以分别得到一个frame的中心点的x坐标和y坐标。
-	CALayer的属性*position*就是中心点，和UIView的属性*center*效果相同。
-	自定义的视图类，如果是通过代码来创建，则需要重写*initWithFrame:*初始化函数。而若是从nib文件中加载，则需要重写*initWithCoder:*函数。**注意：**nib加载的视图并不会调用*initWithFrame:*函数。
-	视图默认的行为是一次只响应一个touch。如果用户按下了第二个手指，系统会忽视这个touch事件并且不会将它报告给视图。如果希望在视图的事件处理函数中跟踪多手指手势，需要设置视图的*multipleTouchEnabled*属性为YES来使多点触摸事件生效。
