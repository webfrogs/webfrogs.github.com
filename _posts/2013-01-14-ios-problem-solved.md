---
layout: post
title: "IOS开发之细节知识点汇总"
categories:
- iOS
tags:
- iOS


---

单独写一篇文章，用于记录在IOS开发中碰到的一些细节上的零碎问题的解决方法。

---

* ####使用命令将模拟器所用静态库和真机所用静态库合并成为一个



	lipo -create XXX.a XXX.a -output XXX.a



* ####使用MFMailComposeViewController来发送邮件导致程序crash
不能直接将其初始化后使用，当系统中没有设置邮件账户时，会引起使用的应用程序崩溃。具体解决方法，参见链接：[http://blog.csdn.net/fxj281314/article/details/6010831](http://blog.csdn.net/fxj281314/article/details/6010831)

---

* ####UIImageView中图片按原比例裁剪放置
设置两个属性即可：

{% highlight objc %}
imageView.clipsToBounds = YES;		// 不显示子视图超出部分
imageView.contentMode = UIViewContentModeScaleAspectFill;	// 保持原比例裁剪
{% endhighlight %}

---

* ####系统键盘弹出时获取键盘的相关信息
监听键盘的系统通知：UIKeyboardWillHideNotification或者UIKeyboardDidShowNotification，在其处理函数中，可以得到键盘的相关信息.例如：

{% highlight objc %}
- (void)keyboardDidShow:(NSNotification *)noti{
    NSDictionary *userInfo = [noti userInfo];
    
    NSValue* aValue = [userInfo objectForKey:UIKeyboardFrameEndUserInfoKey];
    CGSize keyboardSize = [aValue CGRectValue].size;
    float keyboardHeight = keyboardSize.height;  // 得到键盘高度
    float keyBoardOriginY = aValue.CGRectValue.origin.y;  // 得到键盘弹出后的键盘视图所在y坐标
        
}
{% endhighlight %}

上面的函数中，就得到了键盘的高度和键盘view的y坐标。

*注意：*

1、如果UITextView或者UITextField有inputAccessoryView属性，则键盘高度包括inputAccessoryView属性的view的高度。

2、键盘所在的y坐标是以整个屏幕的坐标为参照的。在实际使用中，通常需要减去上方状态栏的高度，即减去20.

* ####将UIView转换为UIImage并保存到文件

转换函数：

{% highlight objc %}
//把UIView 转换成图片  
-(UIImage *)getImageFromView:(UIView *)view{  
         UIGraphicsBeginImageContext(view.bounds.size);  
         [view.layer renderInContext:UIGraphicsGetCurrentContext()];  
         UIImage *image = UIGraphicsGetImageFromCurrentImageContext();  
         UIGraphicsEndImageContext();  
         return image;  
}
{% endhighlight %}

将UIImage保存为文件（jpg和png格式）代码：
{% highlight objc %}
//把UIView 转换成图片  
// Create paths to output images
NSString  *pngPath = [NSHomeDirectory() stringByAppendingPathComponent:@"Documents/Test.png"];
NSString  *jpgPath = [NSHomeDirectory() stringByAppendingPathComponent:@"Documents/Test.jpg"];

// Write a UIImage to JPEG with minimum compression (best quality)
// The value 'image' must be a UIImage object
// The value '1.0' represents image compression quality as value from 0.0 to 1.0

[UIImageJPEGRepresentation(image, 1.0) writeToFile:jpgPath atomically:YES];

// Write image to PNG
[UIImagePNGRepresentation(image) writeToFile:pngPath atomically:YES];
{% endhighlight %}


