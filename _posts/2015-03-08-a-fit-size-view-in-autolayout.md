---
layout: post
title: "Auto Layout 布局技巧－自适应内容大小的视图控件"
categories:
- iOS
tags:
- iOS


---

这是第二篇 Auto Layout 技巧，主要介绍下可以自适应内容大小的控件，主要介绍的是 UILabel 这个控件。

回想下不使用 Auto Layout 时，如果要写一个多行 UILabel 控件时需要做的工作。每当这个多行控件的文本内容发生改变时，我们都需要自己计算需要改变的文本所占的大小，然后去改变这个 UILabel 的 frame。