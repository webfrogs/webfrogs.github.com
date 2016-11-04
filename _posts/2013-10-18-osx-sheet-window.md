---
layout: post
title: "Mac OS X开发之Custom Sheets"
categories:
- OSX
tags:
- OSX


---

## 前言
----

最近在准备写一个Cocoa软件，便开始学习一些Mac开发的知识。相比于iOS开发，Mac开发的资料真的是很少，而中文就更少了。于是决定将自己开发中的碰到的一些问题记录下来，提供一些借鉴。这篇博文记录了我在实现Custom Sheets时碰到的一些问题和相关解决办法。

Mac开发和iOS开发还是有一定的区别的，如果有iOS开发基础，那么看完[参考链接](#link)里的《How to Make a Simple Mac App on OS X 10.7 Tutorial》系列文章你就已经可是上手开发Mac应用了。

## Sheets介绍
----
大家对sheet应该不会感到陌生，iOS中就有UIActionSheet这个控件。苹果官方文档中对sheet是这么定义的。

> A sheet is simply a dialog attached to a specific window, ensuring that a user never loses track of which window the dialog belongs to.*

Mac中的NSOpenPanel的*beginSheetModalForWindow:completionHandler:*方法和NSAlert的*beginSheetModalForWindow:modalDelegate:didEndSelector:contextInfo:*方法都提供了sheet的显示方式，Custom Sheets的显示效果如下图，这也正是我们要实现的。
![sheetDemo](/assets/images/sheetDemo_20131018.gif)

## 注意事项
----

首先提供上面这个图片对应的Demo工程的源码：[点击下载](/assets/download/TestCustomSheet_20131018.zip)。

其中的几个细节要特别注意下：

1. 作为自定义Sheet的NSWindow的nib中的“Visible At launch”选项一定要取消选择。（如下图）
2. "Title Bar"这个选项如果不选中的话，弹出的自定义Sheet里的NSTextField无法输入内容。我也搞不清楚是为什么，谁了解的话，望不吝赐教。

![](/assets/images/qq-cut-20131018.jpg)

<a id='link' name='link'> </a>
## 参考链接
---

* [苹果官方文档-Sheet Programming Topics](https://developer.apple.com/library/mac/documentation/cocoa/conceptual/Sheets/Tasks/UsingCustomSheets.html#//apple_ref/doc/uid/20001290-BABFIBIA)
* [How to Make a Simple Mac App on OS X 10.7 Tutorial: Part 1/3](http://www.raywenderlich.com/17811/how-to-make-a-simple-mac-app-on-os-x-10-7-tutorial-part-13)
* [How to Make a Simple Mac App on OS X 10.7 Tutorial: Part 2/3](http://www.raywenderlich.com/18319/how-to-make-a-simple-mac-app-on-os-x-10-7-tutorial-part-23)
* [How to Make a Simple Mac App on OS X 10.7 Tutorial: Part 3/3](http://www.raywenderlich.com/18413/how-to-make-a-simple-mac-app-on-os-x-10-7-tutorial-part-33)
* [Cocoa Programming L34 - Custom Sheet - YouTube
](http://www.youtube.com/watch?v=QBkO6TD-fWA)
