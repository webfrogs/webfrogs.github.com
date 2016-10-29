---
layout: post
title: "shell脚本实现ipa一键安装(itms-services协议)"
description: ""
category: iOS
tags: [iOS, xcode]
---


#简介
----
通过itms-services协议，可以通过safari浏览器直接在IOS设备上安装应用程序。具体效果可以看图。

![示例](/assets/images/QQ20120927-1.jpg)![示例](/assets/images/QQ20120927-2.jpg)![示例](/assets/images/QQ20120927-3.jpg)

利用这种方式，只要在内网布置一个服务器，测试人员只需要通过测试设备的safari浏览器访问特定的url既可以实现安装，然后测试了。（PS：越狱设备也可以）

itms-services协议需要一个plist配置文件。如果要实现上面图示的功能，需要的文件有：一个ipa文件，一个plist文件，一个html文件和一个图片文件。其中，最主要的，就是plist文件。通过shell脚本，我们可以让其自动为我们生成plist文件和html文件，并且在xcode工程中的ipa文件和程序图标文件复制一份，放到一起。下面，我们来实现这个名为“ipa-publish”的shell脚本。

**注意：** 该脚本需要与“ipa-build”脚本配合使用。“ipa-build”脚本下载：[点击这里](/assets/download/ipa-build)，相关文章[《xcode自动打ipa包脚本》](http://webfrogs.github.com/2012/09/19/buildipa/)

#实现
----

这里提供shell脚本下载，下载完后，只需要为其增加可执行权限即可。[下载点这里](/assets/download/ipa-publish)

#使用
----
该脚本依赖“ipa-build”脚本所产生的内容，所以在执行该脚本之前，首先需要执行“ipa-build”脚本。执行该脚本有两个参数：第一个为xcode工程根路径。第二个是在安装时的显示程序的名称，即第三幅图中下载安装时所显示的名字，第二个参数可选，默认的是xcode工程的target的名字。

比如：“ipa-publish”位于~/shell 路径，xcode工程根路径：~/iphone/HuaRongDao，安装时显示的名称为“华容道”,则命令如下：

	~/shell/ipa-publish ~/iphone/HuaRongDao 华容道

命令执行完后，在当前工程的build文件夹下，会有一个以target名字命名的文件夹，里面就是整个发布所需要的文件。

把这个文件夹直接上传到http服务上。之后就可以通过发布的url加上 /文件夹名 来访问了。


这里提供一个测试地址: [http://webfrogs.github.com/urls/HuaRongDao/](http://webfrogs.github.com/urls/HuaRongDao/) 直接通过IOS设备的safari浏览器访问该url，然后点击其中的链接，就可以出现开始时图示的信息了。

#总结
----
这篇文章写的比较乱，其实这么做的真正意义是，可以通过脚本，来实现一些CI的功能。
之后还可以用脚本来写自动上传的功能，完成xcode工程自动打包，生成文件，以及上传自动化。减轻我们开发时打包的工作量，同时也方便了测试人员安装测试。脚本还有很多可以优化的地方，比如，打包脚本的增加根据渠道来打不同包的功能。
