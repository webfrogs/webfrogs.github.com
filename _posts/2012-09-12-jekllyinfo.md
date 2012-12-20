---
layout: post
title: "jekyll常用命令总结"
description: "jekyll常用命令总结"
category: jekyll
tags: [jekyll]
---


本文记录一些jekyll使用中常用的命令，这些命令均需在jekyll所在根目录下执行

###本地启动jekyll命令


	// 本地启动jeklly，默认4000端口
	jekyll --pygments --safe --server

命令执行后，会在本地开一个4000端口的http服务，接下来可以写文章了，可以访问http://localhost:4000/ 来随时查看编辑结果。

	
###新建文章命令

	// 新建名为“Hello World”的文章（如有同名，不会覆盖）
	rake post title="Hello World"

执行该命令后，将会在“_post”的文件夹下生成形式为“YYYY-MM-dd-titlename.md”的文件。其中，titlename就是命令中的title后引号中的值。**注意：命令中的title不能使用中文名**
