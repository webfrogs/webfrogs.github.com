---
layout: post
title: "ios静态库制作中的注意事项"
description: ""
category: iOS
tags: [iOS]
---


在开发过程中，经常会碰到一些在不同工程中经常用到的部分，把这些部分抽取出来做成一个静态库往往是一个比较好的做法。xcode里就有制作静态库的模板，相关的制作步骤网上也有很多，但在实际的操作中，还是有不少细节方面需要注意。以下是我碰到的一些问题总结。


###1.编译release版本的库
在“Manage Schemes”中，将“Build Configuration”的选项改为“Release”即可。如图：

![release](/assets/images/QQ20121218-2.jpg)

###2.静态库中包含category
如果你在静态库工程中使用了category，那么你可能会碰到链接问题，解决的办法就是需要同时在生成静态库的工程和***使用静态库的工程中使用“-all_load”编译选项***，即在对应target的"Build Settings"中的“Other Linker Flags”选项添加“-all_load”。注意：使用静态库的工程中是一定要加该编译选项的！！至于生成静态库的工程中加不加没有试过，不过建议还是加上该编译选项。

###3.静态库支持的SDK版本
为了使自己的静态库尽可能多的支持IOS的系统版本，应该在"IOS Deployment Target"这个选项中选择自己所需的IOS版本。设置如下图，这个是我的静态库工程中的配置，红框框起来的是我修改过的选项。

![release](/assets/images/QQ20121218-3.jpg)

###4.自动拷贝头文件
在工程对应的target的“Build Phases”下添加“Copy Headers”的选项。该选项默认是没有的，添加方法是点击下方的“Add Build Phase”按钮后选择后即可添加。该选项下有3个子选项，分别是Public,Private,Project。通过点击下方的加号，可以将工程中的头文件添加到“Project”中，在其中的对应头文件点击右键，选择“Move to Public Group”，当头文件移到“Pulic”后，编译工程以后，在工程编译后.a文件所在的路径下，会同时出现一个"usr/local/include"的文件夹，其中的头文件就是public group中的头文件。这时只需将.a文件和这个路径下的头文件拷贝到所需工程文件即可。