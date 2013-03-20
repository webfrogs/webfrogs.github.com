---
layout: post
title: "git submodule的使用"
categories:
- git
tags:
- git


---

开发过程中，经常会有一些通用的部分希望抽取出来做成一个公共库来提供给别的工程来使用，而公共代码库的版本管理是个麻烦的事情。今天无意中发现了git的git submodule命令，之前的问题迎刃而解了。

###添加

为当前工程添加submodule，命令如下：

	git submodule add 仓库地址 路径
	
其中，仓库地址是指子模块仓库地址，路径指将子模块放置在当前工程下的路径。    
**注意：**路径不能以 / 结尾（会造成修改不生效）、不能是现有工程已有的目录（不能順利 Clone）

命令执行完成，会在当前工程根路径下生成一个名为“.gitmodules”的文件，其中记录了子模块的信息。添加完成以后，再将子模块所在的文件夹添加到工程中即可。

###删除

submodule的删除稍微麻烦点：首先，要在“.gitmodules”文件中删除相应配置信息。然后，执行“git rm –cached ”命令将子模块所在的文件从git中删除。

###下载的工程带有submodule

当使用git clone下来的工程中带有submodule时，初始的时候，submodule的内容并不会自动下载下来的，此时，只需执行如下命令：

	git submodule update --init --recursive
	
即可将子模块内容下载下来后工程才不会缺少相应的文件。

