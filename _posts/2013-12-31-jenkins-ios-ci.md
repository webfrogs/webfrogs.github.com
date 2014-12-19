---
layout: post
title: "使用Jenkins搭建iOS开发的CI服务器"
categories:
- iOS
tags:
- iOS
- CI


---

本文为[webfrogs](http://blog.nswebfrog.com/)原创，转载请注明作者及出处！

##目录
----
&nbsp;&nbsp;&nbsp;&nbsp;[**简介**](#introduce)     
&nbsp;&nbsp;&nbsp;&nbsp;[**下载并运行**](#download_and_run)    
&nbsp;&nbsp;&nbsp;&nbsp;[**Jenkins配置**](#jenkins_setting)        
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[安装git插件](#git_plugin_install)     
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[E-mail设置](#email_setting)     
&nbsp;&nbsp;&nbsp;&nbsp;[**自动化构建**](#autobuild)     
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[远程仓库设置](#git_setting)     
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[触发条件设置](#build_trigger)    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[编译设置](#build_setting)     
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[编译后行为设置](#post-build)     
&nbsp;&nbsp;&nbsp;&nbsp;[**单元测试**](#unit_test)     
&nbsp;&nbsp;&nbsp;&nbsp;[**最后**](#last)     

<a id='introduce' name='introduce'> </a>
##简介
----

持续集成CI（continuous integration）是一种可以增加项目可见性，降低项目失败风险的开发实践。iOS开发中CI的选择有很多，比如可以使用Apple提供的Bots来完成自动化构建和单元测试，其优点就是和Xcode深度集成，只需几步配置就可以完成，缺点就是不够灵活，可定制化程度不高。这篇文章主要讲解如何使用开源社区的一个CI工具Jenkins来搭建iOS开发的CI环境。如果是搭建单独CI服务器的话，就需要一台单独的mac机器了。

<a id='download_and_run' name='download_and_run'> </a>
##下载并运行
---
打开Jenkins的[官网](http://jenkins-ci.org/)，在页面的右侧，点击下载最新版本的Jenkins的war包。

下载完成后，打开terminal，进入到war包所在目录，执行命令：

	java -jar jenkins.war --httpPort=8888

httpPort指定的就是Jenkins所使用的http端口，这里指定8888，可根据具体情况修改。待Jenkins启动后，打开浏览器输入地址：

	http://localhost:8888/

便可以打开Jenkins的管理界面了。

<a id='jenkins_setting' name='jenkins_setting'> </a>
##Jenkins配置
---
<a id='git_plugin_install' name='git_plugin_install'> </a>
####安装git插件

Jenkins默认没有安装git插件，需要手动选择安装。进入Jenkins的管理界面，依次选择`Manage Jenkins->Manage Plugins`,选中“Available”选项，在页面的右上角的“Filter”中输入git过滤条件，在所有列出的结果中，选中“Git Client Plugin”和“Git Server Plugin”这两个选项，然后点击按钮“Download now and install after restart”。等待插件下载安装成功后，重启Jenkins。如下图所示：

![图片](/assets/images/QQ20140102-1.jpg)

<a id='email_setting' name='email_setting'> </a>
####E-mail设置
Jenkins可以在适当的时机发送邮件通知，比如自动化构建失败时。这就需要对E-mail的发送进行相关的设置。

发送邮件使用的是SMTP协议，首先要设置Jenkins的管理员邮箱，在`Manage Jenkins->Configure System`的“Jenkins Location”中设置“System Admin e-mail address”为需要的邮箱，也就是Jenkins发送邮件的发件人。     

接下来设置邮件SMTP的相关信息，在“E-mail Notification”区域中，点击“Advanced...”按钮，然后进行设置，首先填写SMTP服务器地址，选中“Use SMTP Authentication”的复选框，然后输入用户名和密码，最后在“Test configuration by sending test e-mail”中输入一个测试邮箱来测试邮件是否能发送成功。如果成功，会有相关提示，如下图所示。

![图片](/assets/images/QQ20140102-2.jpg)

**注意：**在设置邮箱时，Jenkins管理员邮箱要与SMTP中设置的发送邮箱为同一个邮箱，否则在使用比如qq邮箱或者是163邮箱时，就会报错。

<a id='autobuild' name='autobuild'> </a>
###自动化构建
----
在Jenkins中，所有的任务都是以“Job”为单位的。下面以新建一个iOS项目Daily Build的自动化构建Job为例来做一个演示。

在Jenkins管理的首页左侧，点击“New Job”，接下来输入Job的名字，这里输入“Dailybuild”，选择“Build a free-style software project”然后点击“OK”，进入下一个页面。

<a id='git_setting' name='git_setting'> </a>
####远程仓库设置

首先进行版本控制的相关设置，这里我们选择git。输入git的仓库地址，然后选择需要build的分支，另外，在“Additional Behaviours”中，还可以选择一些额外的git操作。如下图。

![图片](/assets/images/QQ20140102-3.jpg)

**提示：**Jenkins使用当前用户.ssh目录下的公私钥来进行git的相关操作。

<a id='build_trigger' name='build_trigger'> </a>
####触发条件设置

下一步，设置build的触发条件，由于是做Daily Build，所以在“Build Triggers”中，选择“Build periodically”，然后在输入框中输入build的规则，这里，我们的规则是每个工作日的下午6点25到30分之间进行build，所以在输入框中输入“H(25-30) 18 * * 1-5”（点击输入框右边的问号，会有详细的规则编写说明），如下图。
![图片](/assets/images/QQ20140102-4.jpg)

<a id='build_setting' name='build_setting'> </a>
####编译设置

然后，进行对工程编译的相关设置。这里，可以使用Jenkins自带的xcode插件（需要安装，参考上面git插件的安装方法）来完成，也可以自己编写脚本来完成。编写脚本时，可以直接使用Xcode的xcodebuild来写，也可以使用Facebook提供的xctool来做。但在本例中使用的是本人遍写的makefile来完成编译打包。这个makefile的功能有：指定Provisioning Profile打包编译，生成itms-services协议相关文件并以scp或者ftp方式上传到服务器来实现ota功能，发送邮件通知和iMessage通知。使用的makefile的github地址[在这里](https://github.com/webfrogs/CCMakefile4iOS)，里面有使用说明。

点击“Add build step”按钮，选择“Execute shell”，在command中输入一下内容：

	#!/bin/zsh
	make clean
	make
	make upload
	make sendEmail
	make sendIMsg

如图所示：
![图片](/assets/images/QQ20140102-5.jpg)

**说明：**如果不使用iMessage通知，可以去掉第一行和最后一行，否则，Jenkins默认的shell会导致iMessage通知不能正常发送。

<a id='post-build' name='post-build'> </a>
####编译后行为设置
工程成功编译以后，我们可以设置编译出来的ipa文件（甚至可以直接是ota文件），将其与本次build的相关结果放到一起，提供下载。也可以在build失败时，发送邮件提醒。设置如下。

点击“Add post-build action”选择“Archive the artifacts”，在输入框中输入“build/*.ipa”，就可以将编译打包后的ipa文件集成。点击“Add post-build action”选择“E-mail Notification”，在输入框中输入编译失败后邮件的通知者邮箱，如有多个，以空白字符分隔，如下图：
![图片](/assets/images/QQ20140102-6.jpg)

至此，一个Daily Build的Job基本设置完成，点击“Save”按钮保存设置。在Job中，点击“Build Now”，测试下我们刚才的配置。如果build失败，可以点击“Console Output”查看log来查找错误的地方。如果成功，在相应的build中，可以看到如下图的内容：
![图片](/assets/images/QQ20140102-7.jpg)

<a id='unit_test' name='unit_test'> </a>
###单元测试
----
在本例中，iOS工程的单元测试选择xcode自带的XCTest框架（Xcode5之前叫做OCUnit）。创建单元测试Job和自动化构建的Job过程一样，只在触发构建规则，build的脚本和编译后的规则有些不同。以下只说明不同的地方。

单元测试的触发规则应该在git仓库的每次有新提交时就触发执行，所以在"Build Triggers"中，选择“Poll SCM”，在规则中写入“H/10 * * * *”，意思是每十分钟轮询一次远程仓库，如果有新的提交，则开始构建。可以根据自己需求来设置轮询的时间间隔。

接下来是在build中输入单元测试脚本。这里需要有一些准备，首先，由于Jenkins只接收Junit的单元测试报告，这里要安装一个将脚本执行结果的ocunit格式的测试报告转化为JUnit报告格式的脚本，该项目名叫OCUnit2JUnit,项目地址[点这里](https://github.com/ciryon/OCUnit2JUnit)。安装非常简单，命令行下执行`gem install ocunit2junit`（可能需要sudo权限）。第二步，需要在当前项目工程中，将项目schemes共享，并上传到远程仓库。在工程中选择“Manage Schemes”在弹出的菜单中勾选“Shared”，然后在git中将相应的shared shceme添加到版本控制中并上传到远程仓库。如下图

![图片](/assets/images/QQ20140102-8.jpg)

“Build”配置中，依然选择“Execute shell”，shell的内容如下：

	xcodebuild test -scheme testCI -sdk iphonesimulator7.0 -destination OS=7.0,name="iPhone Retina (4-inch)" -configuration Debug  2>&1 | ocunit2junit
	
这里的单元测试是在模拟器中进行，如果测试服务器连接着iOS设备，也可以设置在iOS设备中进行，只需修改上述shell的参数即可。

最后是编译后行为的设置，这里要将测试报告加入。点击“Add post-build action”选择“Publish JUnit test result report”，输入内容`test-reports/*.xml`保存设置。

接下来在单元测试的Job中，点击“Build Now”来测试一下Job的配置，如果配置正确，则会看到模拟器启动，然后运行了一下程序，之后在build的结果里，可以看到相应的测试报告，点击进去会有详细的信息，如下图:

![图片](/assets/images/QQ20140102-9.jpg)

<a id='last' name='last'> </a>
##最后
----
芈峮在《豆瓣ios自动化测试实践和经验》([视频地址](http://v.youku.com/v_show/id_XNDM0NDg5MzIw.html)，[PPT地址](http://adc.alibabatech.org/ppts/up-1341913160-0.pdf))中提到Jenkins还可以集成UIAutomation来进行iOS的UI方面的自动化测试，并且还发布了他们自己封装的UI测试工具`ynm3k`，项目地址[点这里](https://github.com/douban/ynm3k)。待研究之后再写下相关的经验吧。
