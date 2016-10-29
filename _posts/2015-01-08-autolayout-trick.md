---
layout: post
title: "AutoLayout 布局技巧－等宽子视图"
categories:
- iOS
tags:
- iOS


---

所谓等宽子视图，也就是对一个有 n 个子视图的父视图来说，无论父视图的宽度怎么变化，所有子视图的宽度是相等的。这里给出一种等宽子视图布局的情况（n = 3），如下图：

![图片](/assets/images/20150108-1.png)

分析一下这种情况的视图约束规则， view1，view2，view3 这三个子视图具有相同的宽度，子视图之间的 padding 为 10，view1 的左边与父视图距离为 20 ，view3 的右边与父视图的距离也为 20。简单起见，所有子视图的高度都为 50（不考虑高度的影响）。

接下来将上面的约束一一实现，其中用到的关键的约束条件就是’等宽约束’。

下图是在 Storyboard 中的约束实现:

![图片](/assets/images/20150108-2.png)

下面是 Swift 代码实现的约束:

```swift
let view1 = UILabel()
view1.backgroundColor = UIColor.redColor()
view1.textAlignment = .Center
view1.text = "view1"
view1.textColor = UIColor.whiteColor()
view1.setTranslatesAutoresizingMaskIntoConstraints(false)
self.view.addSubview(view1)

let view2 = UILabel()
view2.backgroundColor = UIColor.blueColor()
view2.textAlignment = .Center
view2.text = "view2"
view2.textColor = UIColor.whiteColor()
view2.setTranslatesAutoresizingMaskIntoConstraints(false)
self.view.addSubview(view2)

let view3 = UILabel()
view3.backgroundColor = UIColor.purpleColor()
view3.textAlignment = .Center
view3.text = "view3"
view3.textColor = UIColor.whiteColor()
view3.setTranslatesAutoresizingMaskIntoConstraints(false)
self.view.addSubview(view3)

let viewDic = [
"view1": view1,
"view2": view2,
"view3": view3,

]

var constraints = NSLayoutConstraint.constraintsWithVisualFormat("H:|-20-[view1]-10-[view2(==view1)]-10-[view3(==view2)]-20-|", options: NSLayoutFormatOptions(0), metrics: nil, views: viewDic)
self.view.addConstraints(constraints)

constraints = NSLayoutConstraint.constraintsWithVisualFormat("V:|-50-[view1(50)]", options: NSLayoutFormatOptions(0), metrics: nil, views: viewDic)
self.view.addConstraints(constraints)

constraints = NSLayoutConstraint.constraintsWithVisualFormat("V:|-50-[view2(50)]", options: NSLayoutFormatOptions(0), metrics: nil, views: viewDic)
self.view.addConstraints(constraints)

constraints = NSLayoutConstraint.constraintsWithVisualFormat("V:|-50-[view3(50)]", options: NSLayoutFormatOptions(0), metrics: nil, views: viewDic)
self.view.addConstraints(constraints)
```

如果将例子中的所有的水平 padding 都去掉的话，则三个子视图就将父视图的宽度三等分了。
