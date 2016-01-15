---
layout: post
title: "iOS 开发随笔记录"
date: 2014-07-09 22:43:04 +0800
comments: true
categories: 
---

# 说明
本文是我的 iOS 随笔记录，来源于我自己碰到的问题，也有看到的别人遇到的问题记录。本文会随时更新，逐渐增加......
<!--more-->
# 随笔记录
## 1、在 ARC 下，IBOutlets 到底应该定义成 strong 还是 weak ？ 
`答案`：  

在 ARC 中,一般 outlet 属性都推荐使用 weak。  
nib 的 File's Owner 连接到顶层对象的 outlet，应该使用 strong。   

nib 的 File's Owner 连接到顶层对象的 outlet：就是除 main view 里面的其他对象的 outlet。每个 nib 文件在 XCode 中打开后，左边有个列表，列表中几乎都有一个 UIView，它就是顶层对象，跟它平级的其他对象也是顶层对象。比如导航栏中的按钮 UIBarButtonItem。  

UIViewController 的 view 属性是 strong，因为 UIViewController 要直接拥有 view。而添加到view 上的 subviews IBOutlet 为 weak，因为他们不是 UIViewController 直接拥有的，直接拥有 subviews 的是 UIViewController 的 view，ARC 会帮助管理内存。
  
参考苹果官方文档：
[https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/LoadingResources/CocoaNibs/CocoaNibs.html#//apple_ref/doc/uid/10000051i-CH4-SW8](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/LoadingResources/CocoaNibs/CocoaNibs.html#//apple_ref/doc/uid/10000051i-CH4-SW8)  

里面有如下描述：  

>From a practical perspective, in iOS and OS X outlets should be defined as declared properties. Outlets should generally be weak, except for those from File’s Owner to top-level objects in a nib file (or, in iOS, a storyboard scene) which should be strong. Outlets that you create should therefore typically be weak, because:  
* Outlets that you create to subviews of a view controller’s view or a window controller’s window, for example, are arbitrary references between objects that do not imply ownership.  
* The strong outlets are frequently specified by framework classes (for example, UIViewController’s view outlet, or NSWindowController’s window outlet).  

	@property (weak) IBOutlet MyView *viewContainerSubview;
	@property (strong) IBOutlet MyOtherClass *topLevelObject;  
	
还有：  
[http://stackoverflow.com/questions/7678469/should-iboutlets-be-strong-or-weak-under-arc](http://stackoverflow.com/questions/7678469/should-iboutlets-be-strong-or-weak-under-arc)  
	

