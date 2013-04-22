---
layout: post
title: "IOS开发之自定义键盘"
categories:
- iOS
tags:
- iOS


---

实际开发过程中，会有自定义键盘的需求，比如，需要添加一个表情键盘。本文提供一种解决方法，思路就是通过获取系统键盘所在的view，然后自定义一个view覆盖在系统键盘view上，接下来的事情就非常简单了，就是在自定义的view里做任何自己想做的事情。

这个方法的关键在于获取系统键盘所在的view。要完成这个，需要监听UIKeyboardDidShowNotification这个系统通知（注意：如果在UIKeyboardWillShowNotification这个系统通知里处理是不会得到键盘所在view的）。代码如下：

{% highlight objc %}
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardDidShow:) name:UIKeyboardDidShowNotification object:nil];
{% endhighlight %}

keyboardDidShow函数实现：

{% highlight objc %}
- (void)keyboardDidShow:(NSNotification *)notification{
    UIView *keyboardView = [self getKeyboardView];
}
{% endhighlight %}

关键函数getKeyboardView的实现，该函数返回键盘所在view：
{% highlight objc %}
- (UIView *)getKeyboardView{
    UIView *result = nil;
    NSArray *windowsArray = [UIApplication sharedApplication].windows;
    for (UIView *tmpWindow in windowsArray) {
        NSArray *viewArray = [tmpWindow subviews];
        for (UIView *tmpView  in viewArray) {
            if ([[NSString stringWithUTF8String:object_getClassName(tmpView)] isEqualToString:@"UIPeripheralHostView"]) {
                result = tmpView;
                break;
            }
        }
        
        if (result != nil) {
            break;
        }
    }
    
    return result;
}
{% endhighlight %}

下面的事情就简单了，只要自定义一个view，并调用上面得到的keyboardView的addSubView函数，即可将自定义view覆盖在键盘view之上。然后，做自己想做的事情吧。


提供一个DEMO，链接：[https://github.com/webfrogs/CustomKeyboardDemo](https://github.com/webfrogs/CustomKeyboardDemo)