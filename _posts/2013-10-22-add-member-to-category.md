---
layout: post
title: "objc之利用runtime为category添加成员变量"
categories:
- objc
tags:
- objc


---

## 背景
----

我们知道利用category可以在不知道某个类内部实现的情况下，为改类增加方法。但默认的情况下是不能添加成员变量的。但作为一门动态语言，我们可以利用objc的runtime来实现为其添加成员变量的功能，并且完成对KVO的支持。

## 实现
----

选取iOS下比较知名的下拉刷新开源库SVPullToRefresh库为例，在其对UIScrollView的扩展中，就利用了这种技术为UIScrollView添加了成员变量。

关键代码如下：

{% highlight objc %}
- (void)setPullToRefreshView:(SVPullToRefreshView *)pullToRefreshView {
    [self willChangeValueForKey:@"SVPullToRefreshView"];
    objc_setAssociatedObject(self, &UIScrollViewPullToRefreshView,
                             pullToRefreshView,
                             OBJC_ASSOCIATION_ASSIGN);
    [self didChangeValueForKey:@"SVPullToRefreshView"];
}

- (SVPullToRefreshView *)pullToRefreshView {
    return objc_getAssociatedObject(self, &UIScrollViewPullToRefreshView);
}
{% endhighlight %}

其中UIScrollViewPullToRefreshView是一个静态的char变量。在文件的头部有如下的定义：

{% highlight objc %}
static char UIScrollViewPullToRefreshView;
{% endhighlight %}

从上面代码可以看出，通过objc_setAssociatedObject和objc_getAssociatedObject这两个函数，完成了为UIScrollView添加SVPullToRefreshView类型变量的功能。同时，*willChangeValueForKey:*和*didChangeValueForKey:*这两个方法完成了KVO的功能。
