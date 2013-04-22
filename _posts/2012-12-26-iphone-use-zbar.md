---
layout: post
title: IOS开发之ZBarReaderView的使用
categories:
- iOS
tags:
- iOS
- Zbar

---


当开发IOS程序中需要用到二维码识别功能的时候，zbar这个开源库估计会被不少人选择。但是关于zbar的用法，网上的资料大部分都集中在ZBarReaderViewController这个类的使用上。本人在使用中，发现ZBarReaderViewController这个类使用很不灵活，比如，如果需要对界面做一些自定义的定制时会变得很麻烦。在zbar的头文件中，我发现了ZBarReaderView这个类，直觉告诉我这个类的使用应该是比较灵活。google之后发现针对这个类的使用说明比较少，几乎没有，只能自己动手了，在下载了zbar的源码稍作研究后，终于搞定了ZBarReaderView的用法。

##用法
---
ZBarReaderView是UIView的子类，所以我们可以将其当做一个view来设置大小并放置到我们自己界面的任何地方。初始化ZBarReaderView的代码如下：

{% highlight objc %}
ZBarReaderView *readview = [ZBarReaderView new]; // 初始化
readview.frame = CGRectMake(0, 0, 320, 460);	// 改变frame
readview.readerDelegate = self;		// 设置delegate
readview.allowsPinchZoom = NO;		// 不使用Pinch手势变焦
[self.view addSubview:readview];	
{% endhighlight %}

其中第四行的默认值是YES。使用ZBarReaderView的类要实现  *ZBarReaderViewDelegate*代理。

添加上述代码后，只是将ZBarReaderView添加到了我们的控制器视图中，摄像头并没有启动，readview也不会显示视频流。ZBarReaderView中有两个方法可以很方便的开启和关闭摄像头。
{% highlight objc %}
[readview start];	// 开始扫描
[readview stop];	// 停止扫描
{% endhighlight %}

你可以在需要的时候调用这两个方法来控制摄像头的开启和关闭。这样，如果摄像头在开启状态并且扫描到二维码或者条形码以后，*ZBarReaderViewDelegate*的以下代理函数就会被调用。并可以在其中做一些处理。
{% highlight objc %}
- (void)readerView:(ZBarReaderView *)readerView didReadSymbols:(ZBarSymbolSet *)symbols fromImage:(UIImage *)image{
	// 得到扫描的条码内容
	const zbar_symbol_t *symbol = zbar_symbol_set_first_symbol(symbols.zbarSymbolSet);
	NSString *symbolStr = [NSString stringWithUTF8String: zbar_symbol_get_data(symbol)];
	
	
	if (zbar_symbol_get_type(symbol) == ZBAR_QRCODE) {  // 是否QR二维码
	}

}
{% endhighlight %}

你可能已经注意到*ZBarReaderViewDelegate*代理函数中的fromImage:(UIImage *)image这个参数了。没错，ZBarReaderView可以调用摄像头来完成拍照功能。你需要按以下方法调用。
{% highlight objc %}
[readview.captureReader captureFrame];
{% endhighlight %}

上述代码执行后，*ZBarReaderViewDelegate*的代理函数同样会被调用，其中的fromImage:(UIImage *)image就是方法调用时摄像头捕获的图像。

##总结
---
我本身对zbar这个开源的库也没有做深入研究，只是在实际使用中总结了一些用法。有感于这方面的资料比较少，特此做一个总结。

