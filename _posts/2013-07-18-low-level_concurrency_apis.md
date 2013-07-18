---
layout: post
title: "Objc的底层并发API"
categories:
- objc
- iOS
- translation
tags:
- 


---

本文由[webfrogs](http://webfrogs.me/)译自[objc.io](http://www.objc.io/issue-2/low-level-concurrency-apis.html)，原文作者[Daniel Eggert](https://twitter.com/danielboedewadt)。转载请注明出处！

##小引
----
本篇英文原文所发布的站点[objc.io](http://www.objc.io/)是一个专门为iOS和OS X开发者提供的深入讨论技术的平台，文章含金量很高。这个平台每月发布一次，每次都会有数篇文章针对同一个特殊的主题的不同方面来深入讨论。本月的主题是“并发编程”，本文翻译的正是其中的第4篇文章。

翻译此文是受到了[破船](http://weibo.com/beyondvincent)的启发。他已经将objc.io本月主题的第二篇文章翻译完成了。    
[《OC中并发编程的相关API和面临的挑战(1)》](http://beyondvincent.com/2013/07/16/concurrent-programming-apis-and-challenges/)     
[《OC中并发编程的相关API和面临的挑战(2)》](http://beyondvincent.com/2013/07/17/concurrent-programming-apis-and-challenges-2/)


首次翻译文章，水平有限，欢迎指正。

##目录
----
1、[从前。。。](#once_upon_a_time)    
2、[延后执行](#delaying_after)    
3、[队列](#queues)    
&nbsp;&nbsp;&nbsp;&nbsp;3.1、[标记队列](#target_queue)   
&nbsp;&nbsp;&nbsp;&nbsp;3.2、[优先级](#priorities)    
4、[孤立队列](#isolation)    
&nbsp;&nbsp;&nbsp;&nbsp;4.1、[资源保护](#protecting_a_resource)    
&nbsp;&nbsp;&nbsp;&nbsp;4.2、单一资源的多读单写    
&nbsp;&nbsp;&nbsp;&nbsp;4.3、论点    
&nbsp;&nbsp;&nbsp;&nbsp;4.4、全都使用异步    
&nbsp;&nbsp;&nbsp;&nbsp;4.5、如何写出好的异步API    
5、迭代执行    
6、组    
&nbsp;&nbsp;&nbsp;&nbsp;6.1、对现有API使用*dispatch_group_t*    
7、Sources    
&nbsp;&nbsp;&nbsp;&nbsp;7.1、Watching Processes    
&nbsp;&nbsp;&nbsp;&nbsp;7.2、Watching Files    
&nbsp;&nbsp;&nbsp;&nbsp;7.3、Timers    
&nbsp;&nbsp;&nbsp;&nbsp;7.4、Canceling    
8、[输入输出](#input_output)    
&nbsp;&nbsp;&nbsp;&nbsp;8.1、GCD and Buffers    
&nbsp;&nbsp;&nbsp;&nbsp;8.2、Reading and Writing    
9、Benchmarking    
10、[原子操作](#atomic_operations)    
&nbsp;&nbsp;&nbsp;&nbsp;10.1、Counters    
&nbsp;&nbsp;&nbsp;&nbsp;10.2、Compare and Swap    
&nbsp;&nbsp;&nbsp;&nbsp;10.3、Atomic Queues    
&nbsp;&nbsp;&nbsp;&nbsp;10.4、Spin Locks    

  

##正文
----

这篇文章里，我们将会讨论一些iOS和OS X都可以使用的底层API。除了*dispatch_once*，我们一般不鼓励使用其中的任何一种技术。

但是我们想要揭示出表面之下深层次的一些可利用的方面。这些底层的API提供了大量的灵活性，但是伴随着灵活性而来的却是程序复杂度的提升和我们对代码的更多责任。在我们的文章[《common background practices》](http://www.objc.io/issue-2/common-background-practices.html)中提到的高层次的API和模式能够让你专注与手头的任务并且免于大量的问题。并且通常来说，高层次的API会提供更好的性能，除非你能负担的起使用底层API带来的纠结和调试代码的时间和努力。

尽管如此，了解深层次下的软件堆栈工作原理还是有很有帮助的。我们希望这篇文章能够让你更好的了解这个平台，同时，让你更加感谢这些高层的API。

首先，我们将会分析大多数组成*Grand Central Dispatch*的部分。苹果公司持续添加并且改善它已经有数年了。并且苹果已经将其开源，这意味着它对其他平台也是可用的了。最后，我们将会看一下[原子操作](#atomic_operations)——另外的一种底层构建块的集合。

也许，关于并发编程最好的书是*M. Ben-Ari*写的《Principles of Concurrent Programming》[ISBN 0-13-701078-8](https://en.wikipedia.org/wiki/Special:BookSources/0-13-701078-8)。如果你正在做任何有关并发编程的事情，你需要读一下这本书。这本书已经写了超过30年了，但仍然是无法超越。简洁的写法，优秀的例子和练习，带领你构建并发编程中的基本代码块。这本书现在已经绝版了，但是仍然有一些零散的复印本。有一个新版书，名字叫《Principles of Concurrent and Distributed Programming》[ISBN 0-321-31283-X](https://en.wikipedia.org/wiki/Special:BookSources/0-321-31283-X),好像有很多相同的地方，不过我还没有读过。

<a id='once_upon_a_time' name='once_upon_a_time'> </a>
###1、从前。。。

也许GCD中使用最多并且被滥用的就是*dispatch_once*了。正确的用法看起来是这样的：

{% highlight objc %}
+ (UIColor *)boringColor;
{
    static UIColor *color;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        color = [UIColor colorWithRed:0.380f green:0.376f blue:0.376f alpha:1.000f];
    });
    return color;
}
{% endhighlight %}

这段代码将会仅仅运行一次。并且在连续调用这段代码的期间，唯一性的检查是高效的。你能使用它来初始化全局的数据比如单例。要注意到的是，使用*dispatch_once_t*会使得测试变得非常困难——单例和测试只能任取其一。

要确保*onceToken*被声明为**static**，或者有全局作用域。任何其他的情况都会导致无法预知的行为。换句话说，不要把*dispatch_once_t*作为一个对象的成员变量，或者类似的情形。

退回到远古时代（其实也就是几年前），人们会使用*pthread_once*，因为*dispatch_once_t*更容易使用并且不易出错，所以你永远都不会再使用*pthread_once*了。

<a id='delaying_after' name='delaying_after'> </a>
###2、延后执行

另一个常见的朋友就是*dispatch_after*了。它使工作延后执行。它是很强大的，但是要注意：你很容易就陷入到一堆麻烦中。一般的使用是这样的：

{% highlight objc %}
- (void)foo
{
    double delayInSeconds = 2.0;
    dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, (int64_t) (delayInSeconds * NSEC_PER_SEC));
    dispatch_after(popTime, dispatch_get_main_queue(), ^(void){
        [self bar];
    });
}
{% endhighlight %}

咋一看，这段代码是极好的。但是这里也存在一些缺点。我们不能（直接）取消我们已经提交到*dispatch_after*的代码，它将会运行。

另外一个需要注意的事情就是，在人们使用*dispatch_after*去完成工作时，容易写出有时间bug的问题倾向的代码。举例来说，一些代码运行的太早并且很可能不知道为什么，这时你把它放到了*dispatch_after*中，现在，所有代码运行正常了。但是，几周以后，程序停止工作了，并且由于你没有明确指出代码是以什么样的次序运行的，调试代码就变成了一场噩梦。永远不要这样做，大多数的情况下，你最好把代码放到正确的位置。如果代码放到 *-viewWillAppear* 太早，那么或许 *-viewDidAppear* 就是正确的地方。

你将会为自己省去很多麻烦，通过在自己代码中创建直接调用（类似*-viewDidAppear*）而不是依赖与 *dispatch_after*。

如果你需要一些事情在某个特定的时刻及时的运行，那么 *dispatch_after* 或许会是个好的选择。确保同时考虑了*NSTimer*，这个API虽然有点笨重，但是它允许你取消这个定时。

<a id='queues' name='queues'> </a>
###3、队列

GCD的一个最基本的部分就是队列。下面我们会给出一些如何使用它的例子。当使用队列的时候，给它们一个好的标签会帮自己不少忙。当调试的时候，这个标签就是在Xcode(和lldb)中显示，这会帮助你了解应用程序当前是由谁负责的：

{% highlight objc %}
- (id)init;
{
    self = [super init];
    if (self != nil) {
        NSString *label = [NSString stringWithFormat:@"%@.isolation.%p", [self class], self];
        self.isolationQueue = dispatch_queue_create([label UTF8String], 0);
        
        label = [NSString stringWithFormat:@"%@.work.%p", [self class], self];
        self.workQueue = dispatch_queue_create([label UTF8String], 0);
    }
    return self;
}
{% endhighlight %}

队列可以是并行也可以是串行的。默认情况下，他们是串行的，也就是说，在任何给定的时间，只能有一块单独的代码运行。这就是孤立队列的运行方式。队列也可以是并行的，也就是同一时间内允许多段代码同时执行。

GCD队列的内部使用的是线程。GCD管理这些线程，并且使用GCD的时候，你不需要自己创建线程。重要的外在部分是GCD提供给你的用户API，一个不同的抽象层级的接口。当使用GCD来完成并发的工作时，你不必考虑线程方面的问题，代替的，只需考虑队列和工作项目（提交给队列的代码）。当然在这些之下的，依然是线程在工作。GCD的抽象层次为你通常的代码编写提供了更好的方式。

队列和工作项目的方式同时解决了一个普遍的会辐射出去的并发问题：如果我们直接使用线程，并且想要做一些并发的事情，我们可能将我们的工作分成100个小的工作项目，同时基于可以利用的CPU内核数量来创建线程，姑且是8线程。我们把这些工作项目送到这8个线程中。但是写这个函数的人同时也想要使用并发，因此当你调用这个函数的时候，也会创建8个线程。现在，你有了 8×8=64 个线程，尽管你只有8个CPU核心，也就是说任何时候只有12%的线程能够运行这时另外88%的线程什么事情都没做。使用GCD，你不会有这种问题，当操作系统关闭CPU核心以省电时，GCD甚至能够对应的调整线程数量。

GCD通过创建所谓的线程池来大致匹配CPU核心数量。要记住，线程的创建并不是无代价的。每个线程都需要占用内存和内核资源。这里也有一个问题：如果你提交了一个代码块给GCD，但是这个代码块阻塞了这个线程，那么这个线程在这段时间内就不是可用的，并且不能及时处理其他工作——它被阻塞了。为了确保工作项目在队列上一直是执行的，GCD不得不创建一个新的线程，并将新线程添加到线程池。

如果你的代码正在阻塞许多线程，这回带来很大的问题。最开始，线程消耗资源，更多的时，创建他们会变得代价高昂。这需要时间，而且在这段时间内，GCD无法以全速来运行工作项目。有许多能够导致线程阻塞的事情，但是最常见的时与I/O操作有关，也就是从文件或者网络中读写数据。正是因为这些原因，你不应该在GCD队列中以阻塞的方式来运行I/O操作。看一下下面的[输入输出](#input_output)段落以了解如何以GCD良好运行的方式来进行I/O操作。

<a id='target_queue' name='target_queue'> </a>
####3.1、标记队列

你能够为你创建的任何一个队列设置标记。这会是很强大的，并且有助于调试。

为每一个类创建自己的队列而不是使用全局的队列被认为是一种好的方式。这种放肆下，你可以设置队列的名字，这让调试变得轻松许多——Xcode可以让你在Debug Navigator中看到所有的队列名字，或者你可以直接使用lldb。*(lldb) thread list*命令将会在控制台打印出所有队列的名字。一旦你使用大量的异步内容，这是很有价值的帮助。

使用私有队列同样强调封装性。这时你自己的队列，你要自己决定如何使用它。

默认情况下，一个新创建的队列转发到默认优先级的全局队列中。我们就将会多讨论一点有关优先级的东西。

你可以改变你队列转发到的队列——也就是说你可以设置自己队列的目标队列。以这种方式，你可以将不同队列链接在一起。你的类*Foo*的队列转发到类*Bar*的队列，而类*Bar*的队列又转发到全局队列。

当你使用孤立队列（之后我们也会讨论）的时候，这会很有用。*Foo*有一个孤立队列，并且转发到*Bar*的孤立队列，考虑到*Bar*的孤立队列所保护的资源，它会自动变为线程安全的。

如果你希望多段代码同时运行，那要确保你自己的队列是并发的。同时需要注意，如果一个队列的目标队列使串行的（也就是非并发），那么实际上这个队列也会转换为一个串行队列。

<a id='priorities' name='priorities'> </a>
####3.2、优先级

你通过设置目标为全局队列中的一个来改变自己队列的优先级，但是你应该克制这么做的冲动。

在大多数情况下，改变优先级不会使事情照你预想的方向运行。一些看起简单的事情实际上是一个非常复杂的问题。你很容易会碰到一个叫做[优先级反转](http://en.wikipedia.org/wiki/Priority_inversion)的情况。我们的文章[《Concurrent Programming: APIs and Challenges》](http://www.objc.io/issue-2/concurrency-apis-and-pitfalls.html#priority_inversion)（已经由破船翻译了，详见[点击此处](http://beyondvincent.com/2013/07/16/concurrent-programming-apis-and-challenges/)）有更多关于这个问题的信息，这个问题几乎导致了NASA的探路者火星漫游器变成砖头。

在此基础上，使用*DISPATCH_QUEUE_PRIORITY_BACKGROUND*队列时，你需要格外小心。除非你理解了*throttled I/O and background status as per setpriority(2) *（抱歉，我也没理解，不知如何翻译了。）的意思，否则不要使用它。
不然，系统可能会以难以忍受的方式终止你的应用程序的运行。这可能集中在处理I/O操作上，这种操作以一种不与系统其他处理I/O操作的部分交互的方式运行。但是和优先级反转结合起来，这回变成一种危险的情况。

<a id='isolation' name='isolation'> </a>
###4、孤立队列

孤立队列是GCD队列使用中非常普遍的一种模式。这里有两个变种。

<a id='protecting_a_resource' name='protecting_a_resource'> </a>
####4.1、资源保护




{% highlight objc %}

{% endhighlight %}

<a id='input_output' name='input_output'> </a>
###8、输入输出



<a id='atomic_operations' name='atomic_operations'> </a>
###10、原子操作

