---
layout: post
title: "NSOperation And NSOperationQueue "
date: 2014-03-02 14:05:08 +0800
comments: true
categories: 
---

# 一、Operation Obeject和 Operation Queue 概述
## 1、Operation Object 概述
Operation Object 是 NSOperation 类的实例，封装了应用需要执行的任务和执行任务需要的数据。 NSOperation 本身是一个抽象基类，我们必须实现其子类。Foundation framework 提供了两个具体子类，可以直接使用：
<!--more-->
1.  **`NSInvocationOperation`**   
基于应用的一个对象和selector来创建Operation Object，比如用现有的方法来执行需要的任务可以使用NSInvocationOperation。
2.  **`NSBlockOperation`**  
用来并发的执行一个或多个block对象。  

除了上述 NSInvocationOperation 和 NSBlockOperation 两种 operation onject 可以直接使用之外，我们还可以自定义 Operation object。


## 2、Operation Queue 概述
Operation Queue 是 Cocoa 版本的并发dispatch queue， 由 NSOperationQueue 类实现，

## 3、Operation 的优势
1. 支持基于图的Operation Objects`依赖关系`，可以阻止某个 operation 运行，直到它依赖的所有 operation 都已经完成。
2. 支持可选的 `completion block`，在 operation 的主任务完成后调用。
3. 支持应用使用 `KVO` 通知来监控 operation 的执行状态。
4. 支持 operation 的`优先级`。
5. 支持`取消`，允许终止正在执行的任务。

## 4、并发和非并发 Operation
可以将 Operation 添加到 operation queue 中来执行操作，也可以手动调用 start 方法来执行一个 operation object，但这样不保证 operation 会并发执行（由 operation 的 isConcurrent 属性决定，而 isConcurrent 默认是 NO，表示 operation 与调用线程同步执行）。  
**当提交非并发 operation 到 operation queue 时，queue 会创建线程来运行 operation，因此也能达到异步执行的目的。只有不希望用 operation queue 来执行 operation 时，才需要定义并发 operation。**


# 二、Operation Obeject和 Operation Queue 使用

## 1、使用 Operation 同步运行任务
### 1) 下面例子显示了如何使用 NSInvocationOperation 和 NSBlockOperation 以及自定义的 Operation 同步运行任务

源码文件清单如下（4个文件）：  
`第1个文件：CountingOperation.h`  

	@interface CountingOperation : NSOperation
	/* Designated Initializer */
	- (instancetype) initWithStartingCount:(NSUInteger)paramStartingCount endingCount:(NSUInteger)paramEndingCount;
	@end
	
	
`第二个文件：CountingOperation.m`  

	#import "CountingOperation.h"

    @interface CountingOperation ()
    @property (nonatomic, unsafe_unretained) NSUInteger startingCount;
    @property (nonatomic, unsafe_unretained) NSUInteger endingCount;
    @property (nonatomic, unsafe_unretained, getter=isFinished) BOOL finished;
    @property (nonatomic, unsafe_unretained, getter=isExecuting) BOOL executing;
    @end

    @implementation CountingOperation
    - (instancetype) init {
        return([self initWithStartingCount:0
                               endingCount:1000]);
    }

    - (instancetype) initWithStartingCount:(NSUInteger)paramStartingCount
                               endingCount:(NSUInteger)paramEndingCount
    {
        self = [super init];
        
        if (self != nil){
            
            /* Keep these values for the main method */
            _startingCount = paramStartingCount;
            _endingCount = paramEndingCount;
        }
        return(self);
    }

    - (void) main {
        @try {
            /* Here is our autorelease pool */
            @autoreleasepool {
                /* Keep a local variable here that must get set to YES
                 whenever we are done with the task */
                BOOL taskIsFinished = NO;
                
                /* Create a while loop here that only exists
                 if the taskIsFinished variable is set to YES or
                 the operation has been cancelled */
                while (taskIsFinished == NO && [self isCancelled] == NO){
                    
                    /* Perform the task here */
                    NSLog(@"Main Thread = %@", [NSThread mainThread]);
                    NSLog(@"Current Thread = %@", [NSThread currentThread]);
                    NSUInteger counter = _startingCount;
                    
                    for (counter = _startingCount;
                         counter < _endingCount;
                         counter++){
                        NSLog(@"Count = %lu", (unsigned long)counter);
                    }
                    /* Very important. This way we can get out of the
                     loop and we are still complying with the cancellation
                     rules of operations */
                    taskIsFinished = YES;
                }
                
                /* KVO compliance. Generate the
                 required KVO notifications */
                [self willChangeValueForKey:@"isFinished"];
                [self willChangeValueForKey:@"isExecuting"];
                _finished = YES;
                _executing = NO;
                [self didChangeValueForKey:@"isFinished"];
                [self didChangeValueForKey:@"isExecuting"];
            }
        }
        @catch (NSException * e) {
            NSLog(@"Exception %@", e);
        }
    }
    @end


`第三个文件：AppDelegate.h`  


	@interface AppDelegate : UIResponder <UIApplicationDelegate>

	@property (strong, nonatomic) UIWindow *window;

	@property (nonatomic, strong) NSInvocationOperation *simpleInvocationOperation;
	@property (nonatomic, strong) NSBlockOperation *simpleBlockOperation;

	@end
  
  
`第4个文件：AppDelegate.m`	
	
	#import "AppDelegate.h"

	#import "CountingOperation.h"

	@implementation AppDelegate

	- (void) simpleOperationEntry:(id)paramObject{
		NSLog(@"Parameter Object = %@", paramObject);
		NSLog(@"Main Thread = %@", [NSThread mainThread]);
		NSLog(@"Current Thread = %@", [NSThread currentThread]);
	}

	- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
	{
		NSNumber *simpleObject = [NSNumber numberWithInteger:123];
		self.simpleInvocationOperation = [[NSInvocationOperation alloc]
										  initWithTarget:self
										  selector:@selector(simpleOperationEntry:) object:simpleObject];
		[self.simpleInvocationOperation start];
		
		
		
		self.simpleBlockOperation = [NSBlockOperation blockOperationWithBlock:^{
			NSLog(@"Main Thread = %@", [NSThread mainThread]);
			NSLog(@"Current Thread = %@", [NSThread currentThread]);
			NSUInteger counter = 0;
			for (counter = 0; counter < 100;
				 counter++){
				NSLog(@"Count = %lu", (unsigned long)counter);
			} }];
		
		/* Start the operation */
		[self.simpleBlockOperation start];
		
		/* Print something out just to test if we have to wait
		 for the block to execute its code or not */
		NSLog(@"Main thread is here--after simpleBlockOperation");
		
		
		self.simpleCountingOperation = [[CountingOperation alloc]
								initWithStartingCount:0
								endingCount:100];
		[self.simpleCountingOperation start];
		NSLog(@"Main thread is here--after simpleCountingOperation");
		
		self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
		// Override point for customization after application launch.
		self.window.backgroundColor = [UIColor whiteColor];
		[self.window makeKeyAndVisible];
		return YES;
	}



`输出结果如下：`  
2014-03-07 19:04:09.108 OperationSample[1497:70b] Parameter Object = 123  
2014-03-07 19:04:09.111 OperationSample[1497:70b] Main Thread = <NSThread: 0x8a43800>{name = (null), num = 1}  
2014-03-07 19:04:09.112 OperationSample[1497:70b] Current Thread = <NSThread: 0x8a43800>{name = (null), num = 1}  
2014-03-07 19:04:09.113 OperationSample[1497:70b] Main Thread = <NSThread: 0x8a43800>{name = (null), num = 1}  
2014-03-07 19:04:09.114 OperationSample[1497:70b] Current Thread = <NSThread: 0x8a43800>{name = (null), num = 1}  
2014-03-07 19:04:09.115 OperationSample[1497:70b] Count = 0  
2014-03-07 19:04:09.116 OperationSample[1497:70b] Count = 1  
......  
2014-03-07 19:04:09.235 OperationSample[1497:70b] Count = 98  
2014-03-07 19:04:09.236 OperationSample[1497:70b] Count = 99  
2014-03-07 19:04:09.237 OperationSample[1497:70b] Main thread is here--after simpleBlockOperation  
2014-03-07 19:04:09.239 OperationSample[1497:70b] Main Thread = <NSThread: 0x8a43800>{name = (null), num = 1}  
2014-03-07 19:04:09.240 OperationSample[1497:70b] Current Thread = <NSThread: 0x8a43800>{name = (null), num = 1}  
2014-03-07 19:04:09.241 OperationSample[1497:70b] Count = 0  
2014-03-07 19:04:09.241 OperationSample[1497:70b] Count = 1  
......  
2014-03-07 19:04:09.401 OperationSample[1497:70b] Count = 98  
2014-03-07 19:04:09.402 OperationSample[1497:70b] Count = 99  
2014-03-07 19:04:09.404 OperationSample[1497:70b] Main thread is here--after simpleCountingOperation  

### 2) 也可以继承 NSOperation 并在子类中执行同步任务，示例在上面。  
关于继承 NSOperation 需要注意的事情：  

1. **如果不打算使用 operation 队列,必须在 operation 的 start 方法中自己新建一个线程。`如果不想使用 operation 队列并且不想让你的 operation 与其他你手工 start 的 operation 进行异步操作，你可以简单的在 operation 的 start 方法里面调用 main 方法。`**
2. **在你自己实现的 operation 中，NSOperation 实例有`两个重要的方法必须重载:isExecuting 和 isFinished`。他们可以被其它的任意对象调用。在这两个方法中,必须返回一个线程安全值,这个值可以在 operation 中进行操作。一旦你的 operation 开始了,必须通过 KVO 告诉所有的监听者,你改变了这这两个方法返回的值。**
3. **在 `operation 的 main 方法里面,必须提供 autorelease pool`,因为你的 operation 在未来的某个时刻会被 添加到一个 operation 队列中。你必须确保你的 operation 运行,这里有两种方法:手动启动 operation 或者通过 operation 队列启动。**
4. **`必须为你的 operation 提供一个初始化方法。所有 operation 的其它初始化方法,包括默认的 init 方法,必须调用指定的初始化方法`,指定的初始化方法一般具有 多个参数。其它的初始化方法必须确保传递的任何参数符合指定的初始化方法。**


## 2、使用 Operation 异步运行任务
我们希望 operations 运行在它们自己的线程中,而不是占用主线程的时间片，有两种方式异步运行任务：  

1. 最好的解决方案是`使用 operation 队列`。  
2. 不过,如果你想手动管理 operations(不建议),可以`继承 NSOperation,并在 main 方法中新启一个线程`。

**`注意`： operation 队列可能会也可能不会立即通过 nvocation operation 的 start 方法启动 invocation operation。但是,需要牢记重要的一点:添加 operations 至 operation 队列后,你不能手动 启动 operations,必须交由 operation 队列负责。**



### 1）下面的例子展示了将 NSInvocationOperation 加入 NSOperationQueue 的使用。
源码清单如下：

`AppDelegate.h:`

	@interface AppDelegate : UIResponder <UIApplicationDelegate>

	@property (strong, nonatomic) UIWindow *window;
	@property (nonatomic, strong) NSOperationQueue *operationQueue;
	@property (nonatomic, strong) NSInvocationOperation *firstOperation;
	@property (nonatomic, strong) NSInvocationOperation *secondOperation;

	@end
	
	
`AppDelegate.m:`

	#import "AppDelegate.h"

	@implementation AppDelegate

	- (void) firstOperationEntry:(id)paramObject{
		NSLog(@"%s", __FUNCTION__);
		NSLog(@"Parameter Object = %@", paramObject);
		NSLog(@"Main Thread = %@", [NSThread mainThread]);
		NSLog(@"Current Thread = %@", [NSThread currentThread]);
	}
    - (void) secondOperationEntry:(id)paramObject{
        NSLog(@"%s", __FUNCTION__);
		NSLog(@"Parameter Object = %@", paramObject);
		NSLog(@"Main Thread = %@", [NSThread mainThread]);
		NSLog(@"Current Thread = %@", [NSThread currentThread]);    } 

    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
    {
        NSNumber *firstNumber = @111;
		NSNumber *secondNumber = @222;
		self.firstOperation =[[NSInvocationOperation alloc]
		                      initWithTarget:self
		                      selector:@selector(firstOperationEntry:) object:firstNumber];
		self.secondOperation = [[NSInvocationOperation alloc]
		                        initWithTarget:self
		                        selector:@selector(secondOperationEntry:) object:secondNumber];
		self.operationQueue = [[NSOperationQueue alloc] init];
		
        /* Add the operations to the queue */
		[self.operationQueue addOperation:self.firstOperation];
		[self.operationQueue addOperation:self.secondOperation];
		NSLog(@"Main thread is here");
        
        // Override point for customization after application launch.
        return YES;
    }
	
`输出结果如下：`  
2014-03-07 20:31:22.300 OperationAsyncSample[1961:3207] -[AppDelegate secondOperationEntry:]  
2014-03-07 20:31:22.300 OperationAsyncSample[1961:1303] -[AppDelegate firstOperationEntry:]  
2014-03-07 20:31:22.300 OperationAsyncSample[1961:70b] Main thread is here  
2014-03-07 20:31:22.306 OperationAsyncSample[1961:1303] Parameter Object = 111  
2014-03-07 20:31:22.305 OperationAsyncSample[1961:3207] Parameter Object = 222  
2014-03-07 20:31:22.307 OperationAsyncSample[1961:3207] Main Thread = <NSThread: 0x8a3fa30>{name = (null), num = 1}  
2014-03-07 20:31:22.307 OperationAsyncSample[1961:1303] Main Thread = <NSThread: 0x8a3fa30>{name = (null), num = 1}  
2014-03-07 20:31:22.308 OperationAsyncSample[1961:3207] Current Thread = <NSThread: 0x8d32180>{name = (null), num = 2}  
2014-03-07 20:31:22.309 OperationAsyncSample[1961:1303] Current Thread = <NSThread: 0x8b2f050>{name = (null), num = 3}  

### 2）下面的例子展示了将自定义的 Operation 加入 NSOperationQueue 的使用。
如果我们子类化了一个 NSOperation 类,并且把这个子类的实例对象添加到了 operation 队列,我们需要做 稍微的改动。记住以下几点：

1. 由于把 NSOperation 的子类对象添加到一个 operation 队列中,该对象会异步运行。由此必须重载 NSOperation 的实例方法 isConcurrent,在该方法中返回 YES.
2. 在 start 方法里面执行 main 任务之前,需要定期的调用 isCancelled 方法来检测该函数的返回值,以确定是退出 start 方法还是开始运行 operation。在这里,当 operation 添加到队列中后,operation 的 start 方法将会被 operation 队列调用,start 方法中,调用 isCanelled 方法确定 operation 是否被取消。如果 operation 被取消了,只需要从 start 方法中简单的返回即可。如果没被取消,会在 start 方法中调用 main 方法。
3. 在 main task 实现部分中 override main 函数,main 函数将被 operation 执行。在这个函数里面确保分配和初始化 autorelease pool,并且在返回之前释放这个 pool。
4. 重载 operation 的 isFinished 和 isExecuting 方法,这两个函数返回对应的 BOOL 值,代表 operation 是执行完毕还是在执行中。  

源码清单如下：  
`SimpleOperation.h:`

	@interface SimpleOperation : NSOperation
	/* Designated Initializer */
	- (instancetype) initWithObject:(NSObject *)paramObject;
	@end
	
`SimpleOperation.m:`

	#import "SimpleOperation.h"

	@interface SimpleOperation ()
	@property (nonatomic, strong) NSObject *givenObject;
	@property (nonatomic, unsafe_unretained, getter=isFinished) BOOL finished;
	@property (nonatomic, unsafe_unretained, getter=isExecuting) BOOL executing;
	@end


	@implementation SimpleOperation
	- (instancetype) init {
		return([self initWithObject:@123]);
		}
		- (instancetype) initWithObject:(NSObject *)paramObject{
		self = [super init];
		if (self != nil){
			/* Keep these values for the main method */
			_givenObject = paramObject;
		}
		return(self);
	}

		- (void) main {
		
		@try {
			
			@autoreleasepool {
				/* Keep a local variable here that must get set to YES
				 whenever we are done with the task */
				BOOL taskIsFinished = NO;
				
				/* Create a while loop here that only exists
				 if the taskIsFinished variable is set to YES or
				 the operation has been cancelled */
				while (taskIsFinished == NO && [self isCancelled] == NO){
					
					/* Perform the task here */
					NSLog(@"%s", __FUNCTION__);
					NSLog(@"Parameter Object = %@", _givenObject);
					NSLog(@"Main Thread = %@", [NSThread mainThread]);
					NSLog(@"Current Thread = %@", [NSThread currentThread]);
					
					/* Very important. This way we can get out of the
					 loop and we are still complying with the cancellation
					 rules of operations */
					taskIsFinished = YES;
				}
				
				/* KVO compliance. Generate the
				 required KVO notifications */
				[self willChangeValueForKey:@"isFinished"];
				[self willChangeValueForKey:@"isExecuting"];
				_finished = YES;
				_executing = NO;
				[self didChangeValueForKey:@"isFinished"];
				[self didChangeValueForKey:@"isExecuting"];
			}
		}
		@catch (NSException * e) {
			NSLog(@"Exception %@", e);
		}
		}
		- (BOOL) isConcurrent{
		return YES;
		}
	@end
	
	
`AppDelegate.h:`

	@class SimpleOperation;
	@interface AppDelegate : UIResponder <UIApplicationDelegate>

	@property (strong, nonatomic) UIWindow *window;
	@property (nonatomic, strong) NSOperationQueue *operationQueue;
	@property (nonatomic, strong) SimpleOperation *firstOperation;
	@property (nonatomic, strong) SimpleOperation *secondOperation;

	@end
	
`AppDelegate.m:`

	#import "AppDelegate.h"
	#import "SimpleOperation.h"

	@implementation AppDelegate

	- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
	{
		NSNumber *firstNumber = @111;
		NSNumber *secondNumber = @222;
		self.firstOperation = [[SimpleOperation alloc]
							   initWithObject:firstNumber];
		self.secondOperation = [[SimpleOperation alloc]
								initWithObject:secondNumber];
		self.operationQueue = [[NSOperationQueue alloc] init];
		/* Add the operations to the queue */
		[self.operationQueue addOperation:self.firstOperation];
		[self.operationQueue addOperation:self.secondOperation];
		NSLog(@"Main thread is here");
		
		// Override point for customization after application launch.
		return YES;
	}
	
	
`输出结果如下：`  
2014-03-07 21:03:12.923 OperationAsyncSample[2053:320b] -[SimpleOperation main]  
2014-03-07 21:03:12.923 OperationAsyncSample[2053:1303] -[SimpleOperation main]  
2014-03-07 21:03:12.923 OperationAsyncSample[2053:70b] Main thread is here  
2014-03-07 21:03:12.926 OperationAsyncSample[2053:1303] Parameter Object = 111  
2014-03-07 21:03:12.926 OperationAsyncSample[2053:320b] Parameter Object = 222  
2014-03-07 21:03:12.927 OperationAsyncSample[2053:1303] Main Thread = <NSThread: 0x8c4d6e0>{name = (null), num = 1}  
2014-03-07 21:03:12.927 OperationAsyncSample[2053:320b] Main Thread = <NSThread: 0x8c4d6e0>{name = (null), num = 1}  
2014-03-07 21:03:12.928 OperationAsyncSample[2053:1303] Current Thread = <NSThread: 0x8c66b80>{name = (null), num = 2}  
2014-03-07 21:03:12.928 OperationAsyncSample[2053:320b] Current Thread = <NSThread: 0x8d7bba0>{name = (null), num = 3}  


### 3）下面的例子展示了 Operation 创建依赖的使用。
`源码清单如下：`

	@interface AppDelegate ()
		@property (nonatomic, strong) NSInvocationOperation *firstOperation;
	@property (nonatomic, strong) NSInvocationOperation *secondOperation;
	@property (nonatomic, strong) NSOperationQueue *operationQueue;
		@end
		@implementation AppDelegate
		- (void) firstOperationEntry:(id)paramObject{
		NSLog(@"First Operation - Parameter Object = %@",
			  paramObject);
		NSLog(@"First Operation - Main Thread = %@",
			  [NSThread mainThread]);
		NSLog(@"First Operation - Current Thread = %@",
			  [NSThread currentThread]);
	}

		- (void) secondOperationEntry:(id)paramObject{
		NSLog(@"Second Operation - Parameter Object = %@",
			  paramObject);
		NSLog(@"Second Operation - Main Thread = %@",
			  [NSThread mainThread]);
		NSLog(@"Second Operation - Current Thread = %@",
			  [NSThread currentThread]);
		}

		- (BOOL) application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions{
		NSNumber *firstNumber = @111;
		NSNumber *secondNumber = @222;
		self.firstOperation = [[NSInvocationOperation alloc]
							   initWithTarget:self
							   selector:@selector(firstOperationEntry:) object:firstNumber];
		self.secondOperation = [[NSInvocationOperation alloc]
								initWithTarget:self
								selector:@selector(secondOperationEntry:) object:secondNumber];
		[self.firstOperation addDependency:self.secondOperation];
		self.operationQueue = [[NSOperationQueue alloc] init];
		
		/* Add the operations to the queue */
		[self.operationQueue addOperation:self.firstOperation];
		[self.operationQueue addOperation:self.secondOperation];
		NSLog(@"Main thread is here");
		
		self.window = [[UIWindow alloc] initWithFrame:
					   [[UIScreen mainScreen] bounds]];
		self.window.backgroundColor = [UIColor whiteColor]; [self.window makeKeyAndVisible];
		return YES;
		}

`输出结果如下：`  
2014-03-07 21:14:30.362 OperationAsyncSample[2093:1303] Second Operation - Parameter Object = 222  
2014-03-07 21:14:30.362 OperationAsyncSample[2093:70b] Main thread is here  
2014-03-07 21:14:30.364 OperationAsyncSample[2093:1303] Second Operation - Main Thread = <NSThread: 0x8d49760>{name = (null), num = 1}  
2014-03-07 21:14:30.365 OperationAsyncSample[2093:1303] Second Operation - Current Thread = <NSThread: 0x8a50fc0>{name = (null), num = 2}  
2014-03-07 21:14:30.366 OperationAsyncSample[2093:3607] First Operation - Parameter Object = 111  
2014-03-07 21:14:30.366 OperationAsyncSample[2093:3607] First Operation - Main Thread = <NSThread: 0x8d49760>{name = (null), num = 1}  
2014-03-07 21:14:30.367 OperationAsyncSample[2093:3607] First Operation - Current Thread = <NSThread: 0x8b48d20>{name = (null), num = 3}  