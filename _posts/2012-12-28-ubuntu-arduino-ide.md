---
layout: post
title: "解决ubuntu下arduino IDE的Serial Port无法选择问题"
categories:
- arduino
tags:
- ubuntu
- arduino

---
听说arduino在国外很火，加上自己对硬件也蛮感兴趣，于是买了块arduino的板子，准备玩下。但是刚开始在ubuntu上使用arduino的IDE时就碰到了一个问题，就是在Tools菜单下的Serial Port无法选择。google了一下，问题解决。原因就是在ubuntu下，预置安装了一个叫**brltty**的程序与Arduino有冲突，卸载即可，命令如下：

	sudo apt-get remove brltty

**注意：**命令执行完成后，需要重启电脑。重启后，再打开Arduino的IDE就会发现Serial Port可以选择了。