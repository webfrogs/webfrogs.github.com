---
layout: post
title: "IOS工程自动打包并发布脚本实现"
categories:
- IOS
tags:
- IOS
- xcode


---

###前言

-----

IOS的开发过程中，当需要给测试人员发布测试包的时候，直接使用xcode来做的效率是非常低下的。尤其是当有一点小改动需要重新出包时，那简直是个折磨的人的工作。通过一番研究后，遂决定写一系列脚本，以代替人工完成打包和发布的过程。

目前脚本已经完成，基本可以满足我目前的需求。现将其开源，托管在github上，项目地址：[点击这里](https://github.com/webfrogs/xcode_shell)


###思路

----

借助xcode所附带的“Command Line Tools”，可以通过命令行来完成IOS工程的编译和打包工作。脚本正是基于此完成的。

本套脚本分为三个部分：负责编译工程并打包的脚本ipa-build，负责生成itms-services协议文件的脚本ipa-publish，以及负责将ipa-publish脚本生成文件上传到服务器的脚本upload。

其中，由于我自己的情况是服务器端的同事给我了内部测试服务器的sftp的上传权限，所以这个upload脚本主要实现了使用sftp来上传的功能。具体可以实际情况来做修改。

关于itms-services协议的一些内容，可以参考我之前的文章:[《shell脚本实现ipa一键安装(itms-services协议)》](/2012/09/27/ipapublsh/)

**注意：**默认安装完的xcode并没有自带“Command Line Tools”，需要在xcode中选择后下载才能使用

###实现

----

关于脚本实现部分，大家可以打开脚本来看一下，里面的注释算是很详细了。不需要太多说明。其中值得一提的就是我在实现upload脚本时，碰到了一个问题就是使用脚本来自动输入密码，也就是交互式脚本的编写。最后选择了expect来完成，因为我发现mac系统里自带了这个expect命令。

###使用

----

在编写脚本时，我已经考虑到，要尽量使这个脚本使用起来简单方便。如果只需要打包，那么只使用ipa-build脚本即可。如果需要用itms-services协议来发布，则再运行ipa-publish脚本即可。在ipa-publish脚本中调用了upload脚本，所以upload脚本不需要单独使用。

ipa-build脚本使用方法：

	ipa-build脚本路径 参数1 参数2

其中，参数1是IOS工程的根路径,是必输项。参数2可以不输入，是可选的，含义是编译时的工程configuration类型，有4种类型可选：Debug, AdHoc,Release， Distribution。默认是Release。

ipa-build脚本运行后，会在IOS工程根路径下生成名为“build”的文件夹，在这个文件夹中又有一个名为“ipa-build”的文件夹，打包所生成的最新ipa包就在其中。

ipa-publish脚本使用方法:

	ipa-publish脚本路径 参数1 参数2
	
参数1是IOS工程的根路径,是必输项。参数2是可选的，意思是itms-services协议安装时提示弹出程序名称，无太大意义。只输入第一个参数即可。

ipa-publish脚本运行后，会在“build”文件夹中生成一个以targetname为名字的文件夹。其中，存放了itms-services协议所需的所有文件。并会将里面内容全部上传到服务器中。

###注意
----

脚本下载后,若要使用，有些脚本需要一些改动。

其中ipa-build脚本无须更改。若要使用ipa-publish脚本，需打开脚本做一些改动方可使用。

ipa-build脚本需要修改的地方：

	1、pulish_url修改为http方式能访问到的地址
	2、脚本最后一句命令，需要修改为本机存放upload脚本的绝对路径

如果你也是使用sftp协议来向服务器上传文件，只需修改upload脚本开始时的参数即可.其中参数hostfilepath指的是用来放置上次文件的服务器绝对路径。

