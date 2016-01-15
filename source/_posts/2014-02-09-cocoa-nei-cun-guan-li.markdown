---
layout: post
title: "Cocoa 内存管理"
date: 2014-02-09 15:35:50 +0800
comments: true
categories: 
---

# 一、Cocoa 内存管理的正确思考方式
内存管理是程序员必须掌握的基本技能，所以总结一下 Cocoa 内存管理。  
说到 Cocoa 的内存管理，必须先知道`引用计数（Reference Counting）`这个概念。而引用计数又分为iOS5之前的手工管理（非ARC）和iOS5之后自动管理（ARC）的。<!--more-->
## 1、规则
1. 自己生成的对象，自己所持有。
2. 非自己生成的对象也能持有。
3. 不再需要自己持象时释放。
4. 非自己持有的对象无法释放。

## 2、对象操作与 Objective-C 方法的对应关系
对象操作 											| Objective-C方法| 
:----------- 									| :----------- |
生成并持有对象(Create and have ownership of it) 	| alloc/new/copy/mutableCopy|
持有对象(Take ownership of it)         			| retain        |
释放对象(Relinquish it)							|release|
废弃对象(Dispose of it)							|dealloc|

## 3、详解 Cocoa 内存管理
### 1）自己生成的对象，自己所持有。

	/*
	You create an object and have ownership.
	*/
	id obj = [[NSObject alloc] init];

	/*
	Now, you have ownership of the object.
	*/

	
### 2）非自己生成的对象也能持有。

	/*
	Obtain an object without creating it yourself or having ownership
	*/
	id obj = [NSMutableArray array];
	
	/*
	The obtained object exists and you don’t have ownership of it.
	*/

### 3）不再需要自己持象时释放。

	/*
	You create an object and have ownership.
	*/
	id obj = [[NSObject alloc] init];

	/*
	Now you have ownership of the object.
	*/
	[obj release];

	/*
	The object is relinquished.
	Though the variable obj has the pointer to the object,
	you can’t access the object anymore.
	*/

取得存在的对象，但自己不持有。主要用于方法返回对象。
	
	- (id)object
	{
		id obj = [[NSObject alloc] init];

		
		/*
		At this moment, this method has ownership of the object. 
		*/

		[obj autorelease];

		/*
		The object exists, and you don’t have ownership of it. 
		*/
		return obj;
	}

### 4）非自己持有的对象无法释放。

	/*
	You create an object and have ownership. 
	*/

	id obj = [[NSObject alloc] init];

	/*
	Now you have ownership of the object. 
	*/

	[obj release];

	/*
	You relinquished the object, of which you don’t have ownership! 

	The application will crash!
	The applications will crash in these cases:
	When you call the release method to an already-disposed-of object. 
	When you access an already-disposed-of object.
	*/
	id obj1 = [obj0 object];
	/*
	The obtained object exists and you don’t have ownership of it. 
	*/

	[obj1 release];

	/*
	You relinquished the object of which you don’t have ownership!
	The application will crash sooner or later. 
	*/    

# 二、ARC 规则
## 1、概述
引用计数式内存管理的本质在ARC中并没有改变。就像`ARC`这个名表示的那样，ARC只是自动的帮助我们处理`引用计数`的相关部分。
下面的示例代码如果注释中有/\* non-ARC \*/，说明是ARC无效的，没有该注释说明是ARC有效的。
## 2、ARC所有权修饰符
* __strong
* __weak
* __unsafe_unretained
* __autoreleasing

**`__strong、__weak、__autoreleasing 保证附有这些修饰符的自动变量初始化为nil。`**
即：

	id __strong obj0;
	id __weak obj1;
	id __autoreleasing obj2;
		
等价于：

	id __strong obj0 = nil;
	id __weak obj1 = nil;
	id __autoreleasing obj2 = nil;

#### a) __strong
**`__strong是默认修饰符。表示对对象的强引用。持有强引用的变量在超出其作用域时被废弃，随着强引用的时效，引用的对象回随之释放。`**
  
##### Sample1
**`ARC有效`**时，下面的代码：  

	id  obj = [[NSObject alloc] init];
与**`ARC有效`**时的如下代码相同：

	id __strong obj = [[NSObject alloc] init];
与**`ARC无效`**时的如下代码相同：

	/* non-ARC */
	id obj = [[NSObject alloc] init];
##### Sample2
**`ARC有效`**时，下面的代码：

	{
		id __strong obj = [[NSObject alloc] init];
	}
与**`ARC无效`**时的如下代码相同：

	/* non-ARC */
	{

		id obj = [[NSObject alloc] init];
		[obj release];
	}

#### b) __weak
**`__weak与__strong相反，提供对对象的弱引用，可以用来避免循环引用。`**  
**`__weak弱引用不能持有对象实例，所以在超出其变量作用域时，对象即被释放。`**  
**`在__weak持有某对象的弱引用时，若该对象被废弃，则此弱引用将自动失效且弱引用修饰的变量值被赋值为nil。`**

##### Sample1
	id __weak obj1 = nil;
	{
		id __strong obj0 = [[NSObject alloc] init];
		obj1 = obj0;
		NSLog(@"A: %@", obj1);
	}

	NSLog(@"B: %@", obj1);
结果如下:  A: <NSObject: 0x753e180>   
B: (null)
#### c) __unsafe_unretained
**`__unsafe_unretained与__weak一样，但差别在于__unsafe_unretained持有某对象的弱引用时，若该对象被废弃，则此弱引用将自动失效，但是弱引用修饰的变量值不会被赋值为nil。`**  
**`__unsafe_unretained是iOS4的应用中需要用的，庆幸的是现在应用基本都不要支持iOS4了`**
##### Sample1
	id __weak obj1 = nil;
	{
		id __strong obj0 = [[NSObject alloc] init];
		obj1 = obj0;
		NSLog(@"A: %@", obj1);
	}

	NSLog(@"B: %@", obj1);
结果如下:  A: <NSObject: 0x753e180>   
B: <NSObject: 0x753e180> 

#### d) __autoreleasing
ARC有效时，autorelease功能是起作用的。  

##### __autoreleasing Sample1
**`ARC无效`**时的如下代码：

	/* non-ARC */
	NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
	id obj = [[NSObject alloc] init];
	[obj autorelease];
	[pool drain];

与**`ARC有效`**时的如下代码相同：
	@autoreleasepool {
		id __autoreleasing obj = [[NSObject alloc] init];
	}
##### 如下例子，因为**作为方法的返回值，编译器会自动注册到autoreleasepool**，所以不使用`__autorelease`修饰符也能使对象注册到autoreleasepool

	+ (id) array {
		return [[NSMutableArray alloc] init]; 
	}

	+ (id) array {
		id obj = [[NSMutableArray alloc] init];
		return obj; 

	}
##### **访问__weak修饰的变量，必然要访问注册到autoreleasepool的对象**

	id __weak obj1 = obj0; NSLog(@"class=%@", [obj1 class]);
与下面的代码相同：

	id __weak obj1 = obj0;
	id __autoreleasing tmp = obj1;
	NSLog(@"class=%@", [tmp class]);

##### 为什么访问__weak修饰的变量，必然要访问注册到autoreleasepool的对象？  
**因为\_\_weak修饰符支持有对象的弱引用，而在访问对象的过程中，该对象可能被废弃。而把要访问的\_\_weak修饰的对象注册到autoreleasepool中，那么在@autoreleasepool块结束之前都能确保该对象存在**

##### **id指针或者对象指针的指针在没有显式指定时会被附加上\_\_autorelease修饰符**  

	id *obj;
	NSObject **ob;

等价于：

	id __autorelease *obj;
	NSObject * __autorelease *ob;

所以：

	- (BOOL) performOperationWithError:(NSError **)error;

等价于：

	- (BOOL) performOperationWithError:(NSError * __autoreleasing *)error;
	
其实现应该如下所示：

	- (BOOL) performOperationWithError:(NSError * __autoreleasing *)error {
		/* Error occurred. Set errorCode */

		*error = [[NSError alloc] initWithDomain :MyAppDomain code: errorCode userInfo: nil];
		return NO;
	}
	
##### 赋值给对象指针时，所有权修饰符必须一致
如：
 
 	NSError *error = nil; 
 	NSError **pError = &error;
 	
编译器会报错：  
error: initializing 'NSError *__autoreleasing *' with an expressionof type 'NSError *__strong *' changes retain/release properties of pointerNSError **pError = &error;    下面是正确的形式：

	NSError *error = nil;
	NSError * __strong *pError = &error; 
	/* No compile error */
其他修饰符也是：

	NSError __weak *error = nil;
	NSError * __weak *pError = &error; 
	/* No compile error */

	NSError __unsafe_unretained *unsafeError = nil;
	NSError * __unsafe_unretained *pUnsafeError = &unsafeError;
	/* No compile error */
##### 虽然可以非显示的指定__autorelease修饰符，但也可以显式指定，不过显式指定时，对象变量要位自动变量（包括局部变量、函数、方法参数等）
## 3、ARC 详细规则
1. **不能使用retain, release, retainCount, and autorelease.**2. **不能使用NSAllocateObject and NSDeallocateObject.**
3. **必须遵守内存管理的方法命名规则.**
4. **不要显式调用dealloc.**
5. **使用 @autoreleasepool 代替 NSAutoreleasePool.**
6. **不能使用区域（NSZone）.**
7. **对象类型变量不能作为C语言结构体或者联合(struct/union)的成员.**
8. **显式转换"id"和"void *".**
