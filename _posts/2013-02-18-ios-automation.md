---
layout: post
title: "IOS工程自动打包并发布脚本实现"
categories:
- iOS
tags:
- iOS
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

打开工程后，会发现本套脚本中包含好几个shell文件。下面对其功能做说明：

	ipa-build: 	编译xcode工程并生成ipa文件
	ipa-publish: 生成符合itms-services协议的文件，并发布到服务器。
	sendEmail: 	stmp发送email的脚本。（别人写的）
	sftpDownloadFile: 通过sftp协议下载文件
	sftpUploadFile: 通过sftp协议上传文件
	updateLocalIndexHtml:	对索引文件进行处理(二进制文件，非shell脚本)
	uploadItemsServicesFiles:	将itms-services协议文件上传到服务器
	
	
实际使用的脚本，只有"ipa-build"和"ipa-publish"这两个。其他文件会被ipa-publish这个脚本调用的依赖文件。其中出了"updateLocalIndexHtml"是我用objc写的一个用来进行文本处理的编译后的二进制文件，其余均为shell脚本。

shell脚本实现，大家可以打开脚本来看一下，里面的注释算是很详细了。不需要太多说明。

其中值得一提的就是我在写sftp协议上传功能的时候，碰到了一个问题就是使用脚本来自动输入密码，也就是交互式脚本的编写。最后选择了expect来完成，因为我发现mac系统里自带了这个expect命令。

###使用

----

在编写脚本时，我已经考虑到，要尽量使这个脚本使用起来简单方便。如果只需要打包，那么只使用ipa-build脚本即可。如果需要用itms-services协议来发布，则再运行ipa-publish脚本即可。在ipa-publish脚本中调用了upload脚本，所以upload脚本不需要单独使用。

ipa-build脚本使用方法：

	ipa-build脚本绝对路径 参数1 参数2

其中，参数1是IOS工程的根路径,是必输项。参数2可以不输入，是可选的，含义是编译时的工程configuration类型，有4种类型可选：Debug, AdHoc,Release， Distribution。默认是Release。

ipa-build脚本运行后，会在IOS工程根路径下生成名为“build”的文件夹，在这个文件夹中又有一个名为“ipa-build”的文件夹，打包所生成的最新ipa包就在其中。

ipa-publish脚本使用方法:

	ipa-publish脚本绝对路径 参数1 参数2
	
参数1是IOS工程的根路径,是必输项。参数2是可选的，含义是当上传文件成功后是否发送email通知，y为发送，n为不发送，默认的值是不发送。

ipa-publish脚本运行后，会在“build”文件夹中生成一个以工程的targetname为名字的文件夹。其中，存放了itms-services协议所需的所有文件。脚本会将里面内容全部上传到服务器中。

###注意事项
----

1、运行脚本需要绝对路径，不能使用相对路径。

2、脚本下载后,若要使用，有些脚本需要一些改动。

其中ipa-build脚本无须更改。可以直接使用。ipa-publish脚本需要配置一些信息后方能正常使用。

用文本打开ipa-publish脚本后，在shell开始的地方，有一段需要配置的地方，如下：

	#须配置内容  start

	#sftp参数设置
	sftp_server=192.168.xx.xx
	sftp_username=xx
	sftp_password=xx
	sftp_workpath="/usr/share/xx/xx/xx"

	#发布应用的url地址
	pulish_url="http://xx.com/xx"

	#以下是邮箱的相关设置
	#收件人
	email_reciver=xx@xx.com
	#发送者邮箱
	email_sender=xx@xx.com
	#邮箱用户名
	email_username=xx
	#邮箱密码
	email_password=xx
	#smtp服务器地址
	email_smtphost=smtp.exmail.qq.com


	#可配置内容  end
	
	
根据实际情况配置即可。

