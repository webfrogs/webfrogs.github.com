---
layout: post
title: "IOS开发之零碎问题解决汇总"
categories:
- IOS
tags:
- IOS


---

单独写一篇文章，用于记录在IOS开发中碰到的一些细节上的零碎问题的解决方法。

---
* ####使用MFMailComposeViewController来发送邮件导致程序crash
不能直接将其初始化后使用，当系统中没有设置邮件账户时，会引起使用的应用程序崩溃。具体解决方法，参见链接：[http://blog.csdn.net/fxj281314/article/details/6010831](http://blog.csdn.net/fxj281314/article/details/6010831)

* ####UIImageView中图片按原比例裁剪放置
设置两个属性即可：

{% highlight objc %}
imageView.clipsToBounds = YES;		// 不显示子视图超出部分
imageView.contentMode = UIViewContentModeScaleAspectFill;	// 保持原比例裁剪
{% endhighlight %}