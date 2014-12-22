---
layout: post
title: "Swift 脚本编写"
categories:
- Swift
tags:
- OSX


---

作为苹果新一代的编程语言，Swift 不仅可以用来开发 iOS 应用，还可以用来编写脚本，来完成 OS X 下的一些自动化的工作。终于可以用我们熟悉的语言来写自动化脚本了，想想是不是就觉得心里有点小激动呢^_^。 让我们开始编写脚本吧。

## 编写脚本
---

老规矩，从 Hello World 开始。

新建一个 Swift 文件，就叫 `Hello.swift` 吧。使用文本编辑器打开输入以下内容，并保存：
	
	#!/usr/bin/env xcrun swift
	println("Hello World")

打开终端，切换到保存 `Hello.swift` 文件的路径下，为该文件添加可执行权限，命令如下：

	chmod +x Hello.swift
	
然后运行一下这个文件：

	./Hello.swift
	
终端上输出了熟悉的 `Hello World`。


## 执行 shell 脚本
---

在Swift 脚本里，你可以尽情使用 Swift 语法特性。但有些情况下还是需要调用一些 shell 脚本的。这里就需要用到 Foundation 库中的 NSTask 类了。

首先，在脚本中引入 Foundation 库。

	import Foundation

调用 shell 脚本的关键代码:

	let shell = "pwd"
	let task = NSTask()
	task.launchPath = "/bin/bash"
	task.arguments = ["-c", shell]
	
	task.launch()

字符串常量 `shell` 可以是任意能在 `bash` 中执行的 shell 脚本文本内容

## 优缺点
---

对我们 iOS 开发者来说，使用 Swift 来编写脚本，优势很明显，那就是不需要另外学习脚本语言。缺点就是不能够跨平台，还有就是目前 Xcode 对编写 Swift 脚本的支持不够好（好吧，Xcode 对 Swift 开发 iOS 的支持也是不怎么样）。不过，多一种选择总是好的嘛。