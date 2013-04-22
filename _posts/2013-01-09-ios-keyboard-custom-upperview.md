---
layout: post
title: "IOS开发之自定义系统弹出键盘上方的view"
categories:
- iOS
tags:
- iOS


---

这篇文章解决的一个开发中的实际问题就是：当弹出键盘时，自定义键盘上方的view。目前就我的经验来看，有两种解决方法。一个就是利用UITextField或者UITextView的inputAccessoryView属性，另一种，就是监听键盘弹出的notification来自己解决相关视图的位置问题。

第一种解决方法相对比较简单，第二种的方法中有一个难题就是当键盘的输入方式，也就是中英文切换时，键盘的高度是会发生变化的。需要动态来调整相关视图的位置。下面开始详细介绍解决方法。 

###设定inputAccessoryView属性
---
UITextField或者UITextView有一个inputAccessoryView的属性，其类型是UIView。使用中，可以自定义一个view，并将这个view传递给inputAccessoryView的属性即可。这种实现方式相对简单，可以满足很多情况的需求了。下面给出一些示例代码。

{% highlight objc %}
// 新建一个UITextField，位置及背景颜色随意写的。
UITextField *textField = [[UITextField alloc] initWithFrame:CGRectMake(50, 10, 200, 20)];
textField.backgroundColor = [UIColor grayColor];
[self.view addSubview:textField];
    
// 自定义的view
UIView *customView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 320, 70)];
customView.backgroundColor = [UIColor lightGrayColor];
textField.inputAccessoryView = customView;
    
// 往自定义view中添加各种UI控件(以UIButton为例)
UIButton *btn = [[UIButton alloc] initWithFrame:CGRectMake(100, 5, 60, 20)];
btn.backgroundColor = [UIColor greenColor];
[btn addTarget:self action:@selector(btnClicked) forControlEvents:UIControlEventTouchUpInside];
[customView addSubview:btn];

{% endhighlight %}

上面代码很简单，一看就明白了。这里的键盘时通过UITextField的becomeFirstResponder后弹出的。而我在开发中就碰到了一种情况，就是需要通过点击一个按钮来弹出键盘，同时键盘上方的自定义视图中需要包含一个UITextView。这时，这种情况就不适用了。需要用到第二种方法。

###监听键盘事件动态改变自定义view位置

这种方法的思路就是首先自己写一个view，然后监听键盘的事件，得到键盘的位置后调整自己写的view的位置，保证这个view的下边界与键盘的上边界相接。在自定义view中包含一个UITextField或者UITextView。通过代码调用其becomeFirstResponder方法来弹出键盘。

下面写一些关键代码，其中自定义的view名为_mainView，全局变量。

监听键盘事件代码:

{% highlight objc %}
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(changeContentViewPoint:) name:UIKeyboardWillShowNotification object:nil];
{% endhighlight %}

相关的函数代码:

{% highlight objc %}
// 根据键盘状态，调整_mainView的位置
- (void) changeContentViewPoint:(NSNotification *)notification{
    NSDictionary *userInfo = [notification userInfo];
    NSValue *value = [userInfo objectForKey:UIKeyboardFrameEndUserInfoKey];
    CGFloat keyBoardEndY = value.CGRectValue.origin.y;  // 得到键盘弹出后的键盘视图所在y坐标
    
    NSNumber *duration = [userInfo objectForKey:UIKeyboardAnimationDurationUserInfoKey];
    NSNumber *curve = [userInfo objectForKey:UIKeyboardAnimationCurveUserInfoKey];
    
    // 添加移动动画，使视图跟随键盘移动
    [UIView animateWithDuration:duration.doubleValue animations:^{
        [UIView setAnimationBeginsFromCurrentState:YES];
        [UIView setAnimationCurve:[curve intValue]];
        
        _mainView.center = CGPointMake(_mainView.center.x, keyBoardEndY - STATUS_BAR_HEIGHT - _mainView.bounds.size.height/2.0);   // keyBoardEndY的坐标包括了状态栏的高度，要减去
        
    }];
    
    
    
}
{% endhighlight %}

其中添加了一个动画，使得过渡效果好一点。
_mainView中即可添加自定义的UI控件。注意，这个_mainView中控件要从最下面开始布局，因为上述代码是以下方为准的。


