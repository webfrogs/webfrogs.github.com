---
layout: post
title: "xcode自动打ipa包脚本"
description: "xcode自动打ipa包脚本"
category: iOS
tags: [iOS, xcode]
---


#前言
----
使用xcode进行IOS开发的时候，很多时候我们需要将工程打包成ipa文件，而xcode本身并没有这些功能。但是通过安装xcode的“Command Line Tools”这个工具，我们可以使用xcodebuild这个命令来对工程进行打包。然而这么打包出来的文件是以".app"后缀的。其实将其做成ipa文件也非常的简单，只要新建一个名为“Payload”的文件夹，将这个app文件放到里面，并将Payload文件夹压缩，将其后缀名改为ipa即可。借助shell脚本，我们可以让其自动完成这一过程。

#编写
----


下面是shell脚本内容：

	#!/bin/bash

	#--------------------------------------------
	# 功能：为xcode工程打ipa包
	# 作者：ccf
	# E-mail:ccf.developer@gmail.com
	# 创建日期：2012/09/24
	#--------------------------------------------


	#参数判断
	if [ $# != 2 ] && [ $# != 1 ];then
		echo "Number of params error! Need one or two params!"
		echo "1.path of project(necessary) 2.name of ipa file(optional)"
		exit	
	elif [ ! -d $1 ];then
		echo "Params Error!! The first param must be a dictionary."
		exit	
	fi

	#工程绝对路径
	cd $1
	project_path=$(pwd)
	#build文件夹路径
	build_path=${project_path}/build

	#工程配置文件路径
	project_name=$(ls | grep xcodeproj | awk -F.xcodeproj '{print $1}')
	project_infoplist_path=${project_path}/${project_name}/${project_name}-Info.plist
	#取版本号
	bundleShortVersion=$(/usr/libexec/PlistBuddy -c "print CFBundleShortVersionString" ${project_infoplist_path})
	#取build值
	bundleVersion=$(/usr/libexec/PlistBuddy -c "print CFBundleVersion" ${project_infoplist_path})
	#取bundle Identifier前缀
	#bundlePrefix=$(/usr/libexec/PlistBuddy -c "print CFBundleIdentifier" `find . -name "*-Info.plist"` | awk -F$ '{print $1}')


	#IPA名称
	if [ $# = 2 ];then
	ipa_name=$2
	fi


	#编译工程
	cd $project_path
	xcodebuild || exit

	#打包
	cd $build_path
	target_name=$(basename ./Release-iphoneos/*.app | awk -F. '{print $1}')
	if [ $# = 1 ];then
	ipa_name="${target_name}_${bundleShortVersion}_build${bundleVersion}_$(date +"%Y%m%d")"
	fi

	if [ -d ./ipa-build ];then
		rm -rf ipa-build
	fi
	mkdir -p ipa-build/Payload
	cp -r ./Release-iphoneos/*.app ./ipa-build/Payload/
	cd ipa-build
	zip -r ${ipa_name}.ipa *
	rm -rf Payload


将shell脚本保存，命名为“ipa-build”，然后为其添加可执行权限，命令如下：

	chmod +x ipa-build
	
这里提供shell脚本下载，下载完后，只需要为其增加可执行权限即可。[下载点这里](/assets/download/ipa-build)

#使用
----
使用非常简单，ipa-build执行需要两个参数，第一个参数是IOS工程所在的根目录(必须)，第二个参数就是生成ipa文件的文件名（不需要带后缀名ipa），第二个参数是可选的，如果不输入，则ipa文件生成默认格式：*(targetname)\_(version)\_build(buildversion)_yyyyMMdd.ipa* 括号里分别代表所对应的值，yyyyMMdd是当前日期格式。如果未将ipa-build加入环境变量中，使用时需要使用绝对路径。例如：需要打包的工程根路径为：~/iphone/HuaRongDao，而ipa-build放在了 ~/shell 路径下，要生成名为“HuaRongDao.ipa”的文件，则使用如下命令：

	~/shell/ipa-build ~/iphone/HuaRongDao HuaRongDao
	

命令执行完成后，会在工程目录下生成一个名为“build”的文件夹，打包好的ipa就放在build文件夹下的“ipa-build”文件夹中。

#总结
----
写这么一个脚本，主要是受到了一篇文章的启发，是一篇为IOS增加DailyBuild的文章，敏捷开发目前还没有接触，不过那篇文章的博客很不错。文章地址：[《给iOS工程增加Daily Build》](http://blog.devtang.com/blog/2012/02/16/apply-daily-build-in-ios-project/)。 文章中还有一个很有意思的地方，就是通过itms-services协议来安装ipa，已经尝试了，很有用，可以方便测试人员测试，并且可以给越狱手机做自动更新软件功能。待整理后，会写一篇文章来总结。


通过使用shell脚本，确实可以减少很多的工作量。脚本目前还有很多可以完善的地方，同时在做这个脚本的时候，也发现自己的linux知识上的不足，看来要多充充电了。