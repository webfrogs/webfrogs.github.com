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

**注意：**该脚本需要与“ipa-build”脚本配合使用。“ipa-build”脚本下载：[点击这里](/assets/download/ipa-build)，相关文章[《xcode自动打ipa包脚本》](http://webfrogs.github.com/2012/09/19/buildipa/)

#实现
----

“ipa-publish”脚本内容：


	#!/bin/bash

	#--------------------------------------------
	# 功能：创建符合itms-services协议的发布文件
	# 作者：ccf
	# E-mail:ccf.developer@gmail.com
	# 创建日期：2012/09/24
	#--------------------------------------------
	# 修改日期：2012/09/27
	# 修改人：ccf
	# 修改内容：去掉打包的部分脚本，只保留生成协议文件部分，以后此脚本依赖ipa-build脚本
	#--------------------------------------------


	#须配置内容  start

	#发布应用的url地址
	pulish_url="http://webfrogs.github.com/urls/"

	#可配置内容  end

	#参数判断
	if [ $# != 2 ] && [ $# != 1 ];then
		echo "Number of params error! Need one or two params!"
		echo "1.root path of project(necessary) 2.name used to display when installing"
		exit
	elif [ ! -d $1 ];then
		echo "The param must be a dictionary."
		exit	
	
	fi


	#工程绝对路径
	cd $1
	project_path=$(pwd)

	#判断所输入路径是否是xcode工程的根路径
	ls | grep .xcodeproj > /dev/null
	rtnValue=$?
	if [ $rtnValue != 0 ];then
		echo "Error!! The param must be the root path of a xcode project."
		exit
	fi

	#判断是否执行过ipa-build脚本
	ls ./build/ipa-build/*.ipa &>/dev/null
	rtnValue=$?
	if [ $rtnValue != 0 ];then
		echo "Error!! No ipa files exists.Please run the \"ipa-build\" shell script first"
		exit
	fi


	#IPA名称
	if [ $# = 2 ];then
	ipa_name=$2
	fi

	#build文件夹路径
	build_path=${project_path}/build

	#工程配置文件路径
	project_name=$(ls | grep xcodeproj | awk -F.xcodeproj '{print $1}')
	project_infoplist_path=${project_path}/${project_name}/${project_name}-Info.plist
	#取build值
	bundleVersion=$(/usr/libexec/PlistBuddy -c "print CFBundleVersion" ${project_infoplist_path})
	#取bundle Identifier前缀
	bundlePrefix=$(/usr/libexec/PlistBuddy -c "print CFBundleIdentifier" `find . -name "*-Info.plist"` | awk -F$ '{print $1}')

	#生成发布文件

	#拷贝ipa
	cd $build_path
	target_name=$(basename ./Release-iphoneos/*.app | awk -F. '{print $1}')
	if [ $# = 1 ];then
	ipa_name="${target_name}"
	fi

	if [ -d ./$target_name ];then
		rm -rf $target_name
	fi
	mkdir $target_name
	cp ./ipa-build/*.ipa ./$target_name/${target_name}.ipa
	cp ../Icon@2x.png ./$target_name/${target_name}_logo.png
	cd $target_name
	#生成install.html文件
	cat << EOF > index.html
	<!DOCTYPE HTML>
	<html>
	  <head>
	    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
	    <title>安装此软件</title>
	  </head>
	  <body>
		<br>
		<br>
		<br>
		<br>
		<p align=center>
			<font size="8">
				<a href="itms-services://?action=download-manifest&url=${pulish_url}${target_name}.plist">点击这里安装</a>
			</font>
		</p>
	
 	   </div>
	  </body>
	</html>
	EOF
	#生成plist文件
	cat << EOF > ${target_name}.plist
	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
	<plist version="1.0">
	<dict>
	   <key>items</key>
	   <array>
	       <dict>
	           <key>assets</key>
	           <array>
 	              <dict>
 	                  <key>kind</key>
 	                  <string>software-package</string>
	                   <key>url</key>
 	                  <string>${pulish_url}/${target_name}/${target_name}.ipa</string>
 	              </dict>
	               <dict>
  	                 <key>kind</key>
  	                 <string>display-image</string>
  	                 <key>needs-shine</key>
   	                <true/>
   	                <key>url</key>
   	                <string>${pulish_url}/${target_name}/${target_name}_logo.png</string>
  	             </dict>
		       <dict>
 	                  <key>kind</key>
 	                  <string>full-size-image</string>
 	                  <key>needs-shine</key>
 	                  <true/>
	                   <key>url</key>
	                   <string>${pulish_url}/${target_name}/${target_name}_logo.png</string>
	               </dict>
	           </array><key>metadata</key>
	           <dict>
	               <key>bundle-identifier</key>
	               <string>${bundlePrefix}${target_name}</string>
	               <key>bundle-version</key>
 	              <string>${bundleVersion}</string>
	               <key>kind</key>
	               <string>software</string>
	               <key>subtitle</key>
	               <string>${ipa_name}</string>
	               <key>title</key>
	               <string>${ipa_name}</string>
	           </dict>
	       </dict>
	   </array>
	</dict>
	</plist>

	EOF

脚本说明：pulish_url="http://webfrogs.github.com/urls/" 这句话是需要配置的，对应的就是要将文件放到的http服务的url。这里是一定要配置正确的。

将shell脚本保存，命名为“ipa-publish”，然后为其添加可执行权限，命令如下：

	chmod +x ipa-publish
	
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
