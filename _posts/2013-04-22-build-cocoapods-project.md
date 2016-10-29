---
layout: post
title: "基于CocoaPods的iOS工程自动打包脚本实现"
categories:
- iOS
tags:
- iOS


---

### 关于CocoaPods

CocoaPods是一个用于管理iOS工程中所使用第三方开源库的工具，可以大大节省我们在工程中添加和配置第三方库所用的时间。关于CocoaPods的使用，推荐下面两篇博客。   

[《使用CocoaPods来做iOS程序的包依赖管理》](http://blog.devtang.com/blog/2012/12/02/use-cocoapod-to-manage-ios-lib-dependency/)作者：唐巧       
[《CocoaPods进阶：本地包管理》](http://www.iwangke.me/2013/04/18/advanced-cocoapods/#jtss-tsina)作者：王轲

----
### 自动打包脚本
认识到CocoaPods优点后，我便开始在工程中使用了。但在使用了CocoaPods之后，我发现我之前所写的一套iOS的CI脚本中的打包脚本无法完成对工程的自动打包操作了。分析了使用CocoaPods的工程后，对打包脚本做了一番修改。

我所写的iOS工程的CI脚本项目地址：[点击这里](https://github.com/webfrogs/xcode_shell)      
使用的方法可以参见我的博文：[《IOS工程自动打包并发布脚本实现》](http://webfrogs.me/2013/02/18/ios-automation/)


其中的“cocoapods-build”脚本，便是针对CocoaPods工程的打包脚本。

----
### 脚本使用

	cocoapods-build脚本绝对路径 参数1 参数2

参数1：cocoapods工程的根路径    
参数2：工程configuration。默认为Release。

打包完成后所生成的ipa文件位于工程根路径的“build”文件夹下的“ipa-build”文件夹中。

----
### 脚本分析

使用了CocoaPods后，会把当前的工程包含在一个workspace也就是工作空间中，这个工作空间里包含了两个工程：一个是我们自己的工程，另外一个，就是名为“Pods”的工程。CocoaPods会自动管理“Pods”工程，添加我们所需要的三方库，自动处理相关框架依赖。

之前的“ipa-build”这个打包脚本，针对的是xcode工程来进行编译打包的。无法完成对工作空间的编译操作。于是便将脚本中使用xcodebuild编译的部分修改成如下的形式

	xcodebuild -workspace *.xcwork* -scheme ${scheme_name} -configuration ${build_config} CONFIGURATION_BUILD_DIR=${compiled_path} ONLY_ACTIVE_ARCH=NO || exit

其中，-workspace说明是对工作空间进行编译，-scheme指定工程中配置的scheme，这里所取的值就是我们自己工程所一定的scheme。编译工作空间和编译工程不同的地方就是，编译工程默认会在工程跟路径下生成名为“build”的文件夹，而编译工作空间则不会，所以使用CONFIGURATION_BUILD_DIR来显式指定输出编译后的文件路径。
