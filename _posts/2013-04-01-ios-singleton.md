---
layout: post
title: "iOS单例的两种实现"
categories:
- iOS
tags:
- iOS


---

单例模式算是开发中比较常见的一种模式了。在iOS中，单例有两种实现方式（至少我目前只发现两种）。根据线程安全的实现来区分，一种是使用*@synchronized*，另一种是使用GCD的*dispatch_once*函数。

要实现单例，首先需要一个static的指向类本身的对象，其次需要一个初始化类函数。下面是两种实现的代码。

1、@synchronized

```objc
static InstanceClass *instance;
+ (InstanceClass *)defaultInstance{
    @synchronized (self){
        if (instance == nil) {
            instance = [[InstanceClass alloc] init];
        }
    }

    return instance;
}
```

2、GCD


{% highlight objc %}

static InstanceClass *instance;
+ (InstanceClass *)defaultInstance{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[InstanceClass alloc] init];
    });

    return instance;
}
{% endhighlight %}

总的来说，两种实现效果相同，但第二种GCD的实现方式写起来比较简单。如果不习惯GCD的方式，可以使用第一种方式。
