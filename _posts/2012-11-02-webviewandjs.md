---
layout: post
title: "IOS中UIWebView和JavaScript交互"
description: ""
category: iOS
tags: [iOS]
---

当程序中使用到UIWebView控件的时候，难免会遇到需要与页面进行交互的情况。这种情况在android平台下比较容易处理，android平台下WebView控件的addJavascriptInterface()方法可以很轻松的完成交互，而IOS上就稍复杂一些。

页面与客户端的交互是通过JS来完成的，通常情况下与JS的交互可以分为两种：客户端传递给JS一些数据和JS向客户端请求一些本地操作。下面分别对这两种情况进行处理。

**JS向客户端请求本地操作**

这里的实现主要是通过对UIWebView的delegate方法  

	-(BOOL)webView: shouldStartLoadWithRequest: navigationType: 

进行处理来实现的。通过在得到webview所要加载的url来判断是否是需要处理的条件即可。
下面举个具体例子来完成这个操作。如果需要通过JS来通知客户端需要调用用户登录的相关操作。首先跟服务端定好相关的协议串。提供一个方案，协议串的格式如下， 

	XX::command:param1=value1&param2=value2…
	//XX是协议名，command是命令名，后面是参数表（0或多个，command后的冒号不可省略）    

那么对应本次登录的请求串就是

	test::login:   

那么页面上的相应JS的写法就是：

	function sendLoginCommand(){  
    	var url="test::login:";  
    	document.location = url;  
	} 
	
对应IOS端的代码是：

	
	- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType{ ~
        
    	// 处理事件
    	NSString *requestString = [[request URL] absoluteString];
    	NSArray *components = [requestString componentsSeparatedByString:@"::"];
    	if (components != nil && [components count] > 0) {
        	NSString *pocotol = [components objectAtIndex:0];
        	if ([pocotol isEqualToString:@"test"]) {
            	NSString *commandStr = [components objectAtIndex:1];
            	NSArray *commandArray = [commandStr componentsSeparatedByString:@":"];
            	if (commandArray != nil && [commandArray count] > 0) {
                	NSString *command = [commandArray objectAtIndex:0];
                	if ([command isEqualToString:@"login"]) {
                    	UIAlertView *alert = [[UIAlertView alloc]initWithTitle:@"消息" message:@"网页发出了登录请求" delegate:nil cancelButtonTitle:@"OK" otherButtonTitles: nil];
                    	[alert show];
                	}
            	}
            	return NO;
        	}
    	}
    
    
    	return YES;
	}	


**客户端向JS传递数据**

客户端向JS传递数据，通过插入JS方法来实现，UIWebView的方法

	- (NSString *)stringByEvaluatingJavaScriptFromString:(NSString *)script

可以完成这项功能。继续以实际例子来说明，若要通过客户端来实现向JS传递一个用户名参数的功能，可以由客户端向页面中插入一个 getUsername()的JS方法。具体实现代码：

	NSString *js = [NSString stringWithFormat:@"function getUsername(){ return '%@'; }", @"bill"];
    [webView stringByEvaluatingJavaScriptFromString:js];
然后JS在需要得到用户名的地方只要调用window.getUsername()即可。

另外，由于android的addJavascriptInterface()方法中有两个参数，若第二个参数传入字符串，比如test,则JS的调用需要更改为 window.test.getUsername()。为了使IOS与android的调用保持一致，则需要对js进行修改，具体修改形式：

	#define JsStr @"var test = {}; (function initialize() { test.getUsername = function () { return '%@';};})(); "
	NSString *js = [NSString stringWithFormat:JsSt, @"bill"];
    [webView stringByEvaluatingJavaScriptFromString:js];

这样在IOS环境下JS也可以通过调用 window.test.getUsername()来获取用户名了。

*EOF*

