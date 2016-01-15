---
layout: post
title: "Toll-Free-Bridge"
date: 2014-02-13 23:40:09 +0800
comments: true
categories: 
---
# 一、Tool-Free-Bridge 概述
Toll-free bridging（TFB）是一种允许Core Foundation framework 类和 Foundation framework类之间可以互换使用的机制。  
Core Foundation framework 是用 C 实现的，Core Foundation framework 中得 CFRetain 和CFRelease 对应非ARC环境 Objective-C 的 retain 和 release。这两个框架的对象只有在创建方式上有很小的区别（即由哪个框架创建），创建之后就可以互换使用了。因为他们的转换是没有额外 CPU 开销的，所以称为Free Bridge。  
<!--more-->
但是，编译器不能自动管理 Core Foundation 对象的生命周期，所以必须通过转换或者 Core Foundation 宏的方式来告诉编译器管理对象所有权。  

引用苹果开发文档里面的描述如下：
>
1. \_\_bridge transfers a pointer between Objective-C and Core Foundation with no transfer of ownership.  
2. \_\_bridge_retained or CFBridgingRetain casts an Objective-C pointer to a Core Foundation pointer and also transfers ownership to you.
You are responsible for calling CFRelease or a related function to relinquish ownership of the object.
3. \_\_bridge_transfer or CFBridgingRelease moves a non-Objective-C pointer to Objective-C and also transfers ownership to ARC.   

翻译如下： 
 
1. **`__bridge`** 直接在 Objective-C 指针和 Core Foundation 指针之间转换，`不牵涉所有权`（即不影响引用计数）。
2. **`__bridge_retained`** 或 **`CFBridgingRetain`** 将 `Objective-C 指针`转换为 `Core Foundation 指针`，然后传递所有权。这样转换的对象需要调用 **`CFRelease`** 或者相关函数来释放。
3. **`__bridge_transfer`** 或 **`CFBridgingRelease`** 将`非 Objective-C 指针`转换为 `Objective-C 指针`并将所有权交给 ARC 管理。




# 二、Tool-Free-Bridge 示例
#### __bridge 使用示例：

	id obj = [[NSObject alloc] init];
	void *p = (__bridge void *)obj;
	id o = (__bridge id)p;


#### __bridge_retained 使用示例：  

ARC环境下的如下代码：  

	void *p = 0;	{
		id obj = [[NSObject alloc] init];
		p = (__bridge_retained void *)obj;
	}
	NSLog(@"class=%@", [(__bridge id)p class]);
	
对应非ARC环境下得如下代码：  

	/* non-ARC */ 
	void *p = 0;
	{
		id obj = [[NSObject alloc] init];
		/* [obj retainCount] -> 1 */
		
		p = [obj retain];
		/* [obj retainCount] -> 2 */
		
		[obj release];
		/* [obj retainCount] -> 1 */
	}

	/*
	[(id)p retainCount] -> 1 * which means,
	[obj retainCount] -> 1
	So, the object still exists.
	*/
	NSLog(@"class=%@", [(__bridge id)p class]);

#### __bridge_transfer 使用示例：
ARC 环境下如下代码：

	id obj = (__bridge_transfer id)p;
	
对应非 ARC 环境下如下代码：

	/* non-ARC */
	id obj = (id)p;
	[obj retain];
	[(id)p release];
	