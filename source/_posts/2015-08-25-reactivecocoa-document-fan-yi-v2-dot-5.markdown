---
layout: post
title: "ReactiveCocoa Document V2.5 翻译"
date: 2015-08-25 23:16:06 +0800
comments: true
categories: 
---

# 1. 基本操作（Basic Operators）

描述 ReactiveCocoa 最常用的一些操作以及使用范例。
主要是如何运用 序列（sequences） 和 信号（signals） 的流操作。

**用信号实现副作用（Performing side effects with signals）**

1. 订阅（Subscription）
2. 依赖注入（Injecting effects）

**流的传输（Transforming streams）**

1. 映射（Mapping）
2. 过滤（Filtering）

**流的结合（Combining streams）**

1. 串联（Concatenating）
2. 压缩（Flattening）
3. 映射和压缩（Mapping and flattening）

**信号的结合（Combining signals）**

1. 排序（Sequencing）
2. 合并（Merging）
3. 结合最新值（Combining latest values）
4. 切换（Switching）
<!--more-->
## 1.1 用信号实现副作用（Performing side effects with signals）

***译者注：什么是冷信号，什么是热信号？***

* 冷信号：被订阅的时候才激活。信号默认是冷类型的。
* 热信号：返回给调用者的时候已经被激活。

***译者注：什么是 _push-driven_ ? 什么是 _pull-driven_ ?***

* push-driven：在创建信号的时候，信号不会被立即赋值，之后才会被赋值（例如网络请求回来的结果或者是任意的用户输入的结果）。
* pull-driven：在创建信号的时候，序列中的值就会被确定下来，我们可以从流中一个个的查询值。



大多数信号（signals）初始化是冷（cold）信号，这意味着它们在被订阅（Subscription）之前不会做任何工作。  
订阅之后，信号或者它的订阅者能够完成边界效应，比如输出日志到控制台，做一个网络请求，修改用户界面等。  
副作用也可以被注入到一个信号，这样的副作用不会立刻完成，但是会被稍后的每一个订阅者触发副作用。  

***副作用***： 当调用函数时，除了返回函数值之外，还对主调用函数产生附加影响，这就叫函数的副作用。
ReactiveCocoa 的函数参数是 In/Out 作用的参数，即函数可能改变参数里面的的内容，把一些信息通过输入参数，夹带到外界。
这种情况严格来说也是副作用，是非纯函数。我们所讨论的函数式反应式编程中的函数式编程属于非纯函数，它是有副作用的。

###  1.1.1 订阅（Subscription）
`-subscribe...` 方法给你机会访问信号中的当前或者将来的值:

```objc
RACSignal *letters = [@"A B C D E F G H I" componentsSeparatedByString:@" "].rac_sequence.signal;

// 输出: A B C D E F G H I
[letters subscribeNext:^(NSString *x) {
    NSLog(@"%@", x);
}];
```

对于冷信号来说，副作用会在 _每一次订阅_ 时发生:

```objc
__block unsigned subscriptions = 0;

RACSignal *loggingSignal = [RACSignal createSignal:^ RACDisposable * (id<RACSubscriber> subscriber) {
    subscriptions++;
    [subscriber sendCompleted];
    return nil;
}];

// 输出:
// subscription 1
[loggingSignal subscribeCompleted:^{
    NSLog(@"subscription %u", subscriptions);
}];

// 输出:
// subscription 2
[loggingSignal subscribeCompleted:^{
    NSLog(@"subscription %u", subscriptions);
}];
```

行为可以被 `connection`（后面会讲到connection） 改变.

### 1.1.2 注入影响（Injecting effects）

`do...`方法给信号添加副作用而不需要实际订阅信号：

```objc
__block unsigned subscriptions = 0;

RACSignal *loggingSignal = [RACSignal createSignal:^ RACDisposable * (id<RACSubscriber> subscriber) {
    subscriptions++;
    [subscriber sendCompleted];
    return nil;
}];

// 没有任何输出
loggingSignal = [loggingSignal doCompleted:^{
    NSLog(@"about to complete subscription %u", subscriptions);
}];

// 输出:
// about to complete subscription 1
// subscription 1
[loggingSignal subscribeCompleted:^{
    NSLog(@"subscription %u", subscriptions);
}];
```

## 1.2 流的转换（Transforming streams）

下面的操作将流转换为一个新的流。

### 1.2.1 映射（Mapping）

`-map...` 方法被用来传递一个值给流，然后用该值创建一个新的流。

```objc
RACSequence *letters = [@"A B C D E F G H I" componentsSeparatedByString:@" "].rac_sequence;

// Contains: AA BB CC DD EE FF GG HH II
RACSequence *mapped = [letters map:^(NSString *value) {
    return [value stringByAppendingString:value];
}];
```

### 1.2.2 过滤（Filtering）

`-filter...` 方法用一个 block 测试每一个值，使得结果流中只包含测试通过的内容。

```objc
RACSequence *numbers = [@"1 2 3 4 5 6 7 8 9" componentsSeparatedByString:@" "].rac_sequence;

// Contains: 2 4 6 8
RACSequence *filtered = [numbers filter:^ BOOL (NSString *value) {
    return (value.intValue % 2) == 0;
}];
```

## 1.3 流的合并（Combining streams）

下面的操作合并多个流到一个单一的新流。

### 1.3.1 串联（Concatenating）

`-concat：`方法追加一个流的值到另外一个流后面。

```objc
RACSequence *letters = [@"A B C D E F G H I" componentsSeparatedByString:@" "].rac_sequence;
RACSequence *numbers = [@"1 2 3 4 5 6 7 8 9" componentsSeparatedByString:@" "].rac_sequence;

// Contains: A B C D E F G H I 1 2 3 4 5 6 7 8 9
RACSequence *concatenated = [letters concat:numbers];
```

### 1.3.2 扁平（Flattening，这个真不知道怎么翻译）

`-flatten` 操作适用于基于流的流，合并他们的值到一个新的流中。  

序列是串联（concatenated）的

```objc
RACSequence *letters = [@"A B C D E F G H I" componentsSeparatedByString:@" "].rac_sequence;
RACSequence *numbers = [@"1 2 3 4 5 6 7 8 9" componentsSeparatedByString:@" "].rac_sequence;
RACSequence *sequenceOfSequences = @[ letters, numbers ].rac_sequence;

// Contains: A B C D E F G H I 1 2 3 4 5 6 7 8 9
RACSequence *flattened = [sequenceOfSequences flatten];
```

信号是合并（merged）的：

```objc
RACSubject *letters = [RACSubject subject];
RACSubject *numbers = [RACSubject subject];
RACSignal *signalOfSignals = [RACSignal createSignal:^ RACDisposable * (id<RACSubscriber> subscriber) {
    [subscriber sendNext:letters];
    [subscriber sendNext:numbers];
    [subscriber sendCompleted];
    return nil;
}];

RACSignal *flattened = [signalOfSignals flatten];

// Outputs: A 1 B C 2
[flattened subscribeNext:^(NSString *x) {
    NSLog(@"%@", x);
}];

[letters sendNext:@"A"];
[numbers sendNext:@"1"];
[letters sendNext:@"B"];
[letters sendNext:@"C"];
[numbers sendNext:@"2"];
```

### 1.3.3 映射和扁平（Mapping and flattening）

`flattening` 本身并不有趣，但弄懂它的 `-flattenMap` 方法是如何工作的很重要。    

`-flattenMap:` 用来传递流的每一个值到 _一个新的流_ 。然后，所有返回的流会被扁平到一个新的流。 也就是说，它相当于 `-flatten` 操作后再 `-map:` 操作。

可用来扩展或编辑序列：

```objc
RACSequence *numbers = [@"1 2 3 4 5 6 7 8 9" componentsSeparatedByString:@" "].rac_sequence;

// Contains: 1 1 2 2 3 3 4 4 5 5 6 6 7 7 8 8 9 9
RACSequence *extended = [numbers flattenMap:^(NSString *num) {
    return @[ num, num ].rac_sequence;
}];

// Contains: 1_ 3_ 5_ 7_ 9_
RACSequence *edited = [numbers flattenMap:^(NSString *num) {
    if (num.intValue % 2 == 0) {
        return [RACSequence empty];
    } else {
        NSString *newNum = [num stringByAppendingString:@"_"];
        return [RACSequence return:newNum]; 
    }
}];
```

或者创建多个信号自动合并：

```objc
RACSignal *letters = [@"A B C D E F G H I" componentsSeparatedByString:@" "].rac_sequence.signal;

[[letters
    flattenMap:^(NSString *letter) {
        return [database saveEntriesForLetter:letter];
    }]
    subscribeCompleted:^{
        NSLog(@"All database entries saved successfully.");
    }];
```

## 1.4 信号组合（Combining signals）

下面的操作组合多个信号到一个新信号。

### 1.4.1 序列化（Sequencing）

`-then:` 启动原始的信号，等待它完成，然后只转发新信号中的值。

```objc
RACSignal *letters = [@"A B C D E F G H I" componentsSeparatedByString:@" "].rac_sequence.signal;

// The new signal only contains: 1 2 3 4 5 6 7 8 9
//
// But when subscribed to, it also outputs: A B C D E F G H I
RACSignal *sequenced = [[letters
    doNext:^(NSString *letter) {
        NSLog(@"%@", letter);
    }]
    then:^{
        return [@"1 2 3 4 5 6 7 8 9" componentsSeparatedByString:@" "].rac_sequence.signal;
    }];
```

这在某些情况下很有用，比如执行一个信号的所有副作用，然后开始另外一个信号，并且只返回第二个信号的值。

### 1.4.2 合并（Merging）

`+merge:` 方法会尽可能快的从很多信号中转发值到一个流中，在值到达的时候。

```objc
RACSubject *letters = [RACSubject subject];
RACSubject *numbers = [RACSubject subject];
RACSignal *merged = [RACSignal merge:@[ letters, numbers ]];

// Outputs: A 1 B C 2
[merged subscribeNext:^(NSString *x) {
    NSLog(@"%@", x);
}];

[letters sendNext:@"A"];
[numbers sendNext:@"1"];
[letters sendNext:@"B"];
[letters sendNext:@"C"];
[numbers sendNext:@"2"];
```

### 1.4.3 组合最新值（Combining latest values）

`+ combineLatest:` 和 `+combineLatest:reduce:` 方法会观察多个信号的改变，然后从所有信号发送最新的值：

```objc
RACSubject *letters = [RACSubject subject];
RACSubject *numbers = [RACSubject subject];
RACSignal *combined = [RACSignal
    combineLatest:@[ letters, numbers ]
    reduce:^(NSString *letter, NSString *number) {
        return [letter stringByAppendingString:number];
    }];

// Outputs: B1 B2 C2 C3
[combined subscribeNext:^(id x) {
    NSLog(@"%@", x);
}];

[letters sendNext:@"A"];
[letters sendNext:@"B"];
[numbers sendNext:@"1"];
[numbers sendNext:@"2"];
[letters sendNext:@"C"];
[numbers sendNext:@"3"];
```

注意结合信号会只发送第一个值，当所有输入被发送至少一个的时候。上面的例子中，`@"A"` 不会被转发因为 `number` 没有被发送一个值。

### 1.4.4 切换（Switching）

`-switchToLatest` 操作适用于基于信号的信号，并且总是从最新信号传递值。

```objc
RACSubject *letters = [RACSubject subject];
RACSubject *numbers = [RACSubject subject];
RACSubject *signalOfSignals = [RACSubject subject];

RACSignal *switched = [signalOfSignals switchToLatest];

// Outputs: A B 1 D
[switched subscribeNext:^(NSString *x) {
    NSLog(@"%@", x);
}];

[signalOfSignals sendNext:letters];
[letters sendNext:@"A"];
[letters sendNext:@"B"];

[signalOfSignals sendNext:numbers];
[letters sendNext:@"C"];
[numbers sendNext:@"1"];

[signalOfSignals sendNext:letters];
[numbers sendNext:@"2"];
[letters sendNext:@"D"];
```

[Connections]: FrameworkOverview.md#connections
[RACSequence]: ../ReactiveCocoa/RACSequence.h
[RACSignal]: ../ReactiveCocoa/RACSignal.h
[RACSignal+Operations]: ../ReactiveCocoa/RACSignal+Operations.h
[RACStream]: ../ReactiveCocoa/RACStream.h
[Sequences]: FrameworkOverview.md#sequences
[Signals]: FrameworkOverview.md#signals
[Streams]: FrameworkOverview.md#streams
[Subscription]: FrameworkOverview.md#subscription



# 2 设计指南（Design Guidelines）

本文档包含如何在工程中使用 ReactiveCocoa 的设计指南。本章的内容重度参考了 [Rx Design
Guidelines](http://blogs.msdn.com/b/rxteam/archive/2010/10/28/rx-design-guidelines.aspx)。

本文假设读者熟悉 ReactiveCocoa 的基本功能。`Framework Overview` 是开始了解 RAC 的更好的资源。

**`RACSequence`的约定**

 1. 运算默认是懒执行模式。
 2. 运算会阻塞调用者。
 3. 副作用只发生一次。

**`RACSignal`的约定**

 1. 信号事件是串行的。
 2. 订阅都是发生在调度的时候。
 3. 错误会被立即传送出来。
 4. 副作用发生在每次订阅。
 5. 订阅会自动部署完成和错误。
 6. Disposal取消正在进行的工作并且清除资源。

**最佳实践**

 1. 为返回信号的方法或属性使用描述性的声明。
 2. 始终缩进流操作。
 3. 流的所有值都使用相同的类型。
 4. 不要retain流过长时间。
 5. 只处理需要数量的流。
 6. 分发信号事件到一个已知的调度。
 7. 较少的场合需要切换调度者。
 8. 明确信号的副作用。
 9. 通过multicasting共享信号的副作用。
 10. 通过指定的流的名字来调试。
 11. 避免明确的订阅（subscriptions）和释放（disposal）。
 12. 尽可能避免使用subjects。

**完成新operators**

 1. 优先使用基于 RACStream 方法。
 2. 尽可能组合已存在的operators。
 3. 避免引入并发。
 4. 在dispasable中取消任务和清理所有资源。
 5. 在operator中不要阻塞。
 6. 深度递归要避免栈溢出。

## 2.1 RACSequence的约定（The RACSequence contract）

`RACSequence` 是 _pull-driven_ 流。序列行为类似内置集合，但有些不一样的地方。

### 2.1.1 （运算默认是懒执行模式）Evaluation occurs lazily by default

序列运算默认是懒执行模式，如下面的序列:

```objc
NSArray *strings = @[ @"A", @"B", @"C" ];
RACSequence *sequence = [strings.rac_sequence map:^(NSString *str) {
    return [str stringByAppendingString:@"_"];
}];
```

没有字符串会被实际追加直到序列真正需要的时候。
访问 `sequence.head` 会完成 `A_` 的拼接，访问 `sequence.tail.head` 会完成 `B_` 的拼接，等等。

这通常能避免不必要的工作（因为不需要的值不会被计算），但意味着序列是要处理的。

一旦被计算，序列中的值就被存储不会被重新计算。访问 `sequence.head` 多次只会做一次字符串的拼接工作。

如果懒式运算模式不可取 - 例如，因为内存有限的时候，较少使用内存更重要 - eagerSequence 属性可能被强制转为饥渴模式。

### 2.1.2 运算会阻塞调用者（Evaluation blocks the caller）

不管序列是懒模式还是饥渴模式，运算序列的任何部分都会阻塞调用者线程直到任务完成。阻塞是必须的因为值必须从序列中同步返回。

如果运算序列序列的代价大到可能阻塞线程很明显的时间，考虑用 `-signalWithScheduler:` 创建一个信号然后用它来替代序列。

### 2.1.3 副作用只发生一次（Side effects occur only once）

当传递给序列操作的 block 引发了副作用，要明白副作用对每个值只会发生一次，就是在值被运算的时候。

```objc
NSArray *strings = @[ @"A", @"B", @"C" ];
RACSequence *sequence = [strings.rac_sequence map:^(NSString *str) {
    NSLog(@"%@", str);
    return [str stringByAppendingString:@"_"];
}];

// Logs "A" during this call.
NSString *concatA = sequence.head;

// Logs "B" during this call.
NSString *concatB = sequence.tail.head;

// Does not log anything.
NSString *concatB2 = sequence.tail.head;

RACSequence *derivedSequence = [sequence map:^(NSString *str) {
    return [@"_" stringByAppendingString:str];
}];

// Still does not log anything, because "B_" was already evaluated, and the log
// statement associated with it will never be re-executed.
NSString *concatB3 = derivedSequence.tail.head;
```

## 2.2 RACSignal 约定（The RACSignal contract）

`RACSignal` 是 _push-driven_ 流，专注于通过 _subscriptions_ 分发异步事件。
关于信号和订阅的更多内容，参见 `Framework Overview`。

### 2.2.1 信号事件是串行的（Signal events are serialized）

信号可以在任何线程中分发事件。连续的事件甚至被允许分发到不同的线程或者调度者，除非显示的指定了分发到特定的调度者。

然而，RAC 不会有两个信号并发到达。一个事件被处理时，不会有另外的事件被分发。其他事件的发送会被强制等待直到当前事件被处理完成。

特别注意，这意味着传递给 `-subscribeNext:error:competed:` 的 block 之间不需要考虑同步，因为他们永远不会被同时调用。

### 2.2.2 订阅总会在调度的时候发生（Subscription will always occur on a scheduler）

要保证 `+createSingnal` 和 `-subscribe:` 的行为一致，每一个 `RACSignal` 必须确保在合法的调度者上订阅。

如果的订阅者的线程已经有一个 `+currentScheduler` ,调度会立刻发生；否则，会在后台调度的时候立刻发生。
注意主线程总是与 `—mainThreadScheduler` 关联，所以主线程的订阅总是立刻发生。

参见文档 `-subscribe:` 获取更多信息。

### 2.2.3 错误会立即传送出来（Errors are propagated immediately）

在 RAC中，`error` 事件有特别的语义。当错误被发送给信号，会立即转发给所有依赖的信号，引发整个依赖链的终止。

`Operators` 的主要目的是改变错误处理行为，但像 `-catch:`, `-catchTo:`, 或者 `-materialize` 明显是不符合此规则的。

### 2.2.4 副作用发生在每次订阅时（Side effects occur for each subscription）

对 `RACSignal` 的每一个新的订阅都会触发信号的副作用。这意味着任何副作用发生的次数和信号订阅的次数一样多。

考虑如下代码：

```objc
__block int aNumber = 0;

// Signal that will have the side effect of incrementing `aNumber` block 
// variable for each subscription before sending it.
RACSignal *aSignal = [RACSignal createSignal:^ RACDisposable * (id<RACSubscriber> subscriber) {
	aNumber++;
	[subscriber sendNext:@(aNumber)];
	[subscriber sendCompleted];
	return nil;
}];

// This will print "subscriber one: 1"
[aSignal subscribeNext:^(id x) {
	NSLog(@"subscriber one: %@", x);
}];

// This will print "subscriber two: 2"
[aSignal subscribeNext:^(id x) {
	NSLog(@"subscriber two: %@", x);
}];
```

副作用会在每次订阅的时候重复发生。同样适用于 `stream` 和 `signal` 操作。

```objc
__block int missilesToLaunch = 0;

// Signal that will have the side effect of changing `missilesToLaunch` on
// subscription.
RACSignal *processedSignal = [[RACSignal
    return:@"missiles"]
	map:^(id x) {
		missilesToLaunch++;
		return [NSString stringWithFormat:@"will launch %d %@", missilesToLaunch, x];
	}];

// This will print "First will launch 1 missiles"
[processedSignal subscribeNext:^(id x) {
	NSLog(@"First %@", x);
}];

// This will print "Second will launch 2 missiles"
[processedSignal subscribeNext:^(id x) {
	NSLog(@"Second %@", x);
}];
```

要阻止上述行为，在多次订阅一个信号时只执行它的副作用一次，可以用信号的多播功能 `multicasted`。

### 2.2.5 订阅会在完成和错误的时候自动释放（Subscriptions are automatically disposed upon completion or error）

当一个 `subscriber` 被发送给 `completed` 或者 `error` 事件，相关的订阅会自动被释放。这种行为通常无需手动去配置订阅。

参见文档 `Memory Management` 获取 `signal` 的更多信息。

### 2.2.6 Disposal取消正在进行的工作和清理资源（Disposal cancels in-progress work and cleans up resources）

订阅被释放的时候，不管手动或自动，任何正在处理或与订阅相关的工作会尽快被取消，订阅相关的资源会被释放。

## 2.3 Best practices

下面的建议有助于保证基于 RAC 的代码可预测，可理解和高效。

然而，仅仅只是指导。判断是否遵循了建议的标准是下面的代码片段。

### 2.3.1 为返回信号的属性和方法使用描述性的声明（Use descriptive declarations for methods and properties that return a signal）

方法或属性如果返回 `RACSignal` 类型，会很难看懂信号的语义。

有三个关键问题必须在声明中表达清楚：

1. 信号是 _热_ （返回给调用者的时候已经被激活）信号还是 _冷_ （被订阅的时候才激活）信号。
2. 信号包含0个，1个，还是多个值？
3. 信号是否有副作用？

**没有副作用的热信号** 应该典型的用属性来代替方法。使用属性意味着在订阅信号的时间之前不需要初始化，并且额外的订阅不会改变语义。
信号属性应该以事件命名（例如 `textChanged`）。

**没有副作用的冷信号** 应该从名词命名的方法中返回（例如：`-currentText`）。
这样的方法声明意味着信号不会被该方法保留，暗示着在订阅的时候任务已经完成了。
如果信号发送多个值，名词应该用复数（例如: `-currentModels`）。

**带副作用的信号** 应该被动词命名的方法返回（例如：`-logIn`）。
动词意味着该方法不是幂等的（幂等：意味着多次执行操作的结果和第一次执行的相同。应该就是函数的可重入性），
调用者必须小心只在副作用需要的时候才调用它。如果信号会发送一个或多个值，应该包含一个期望的名词
（例如：`-loadConfigration`， `-fecthLatestEvents`）。

### 2.3.2 始终缩进流操作（Indent stream operations consistently）

如果没有合适的格式化，流代码和容易变得密集和混乱。使用缩进能够清晰的看出来链式流操作的开始和结束。

调用流的简单的方法时，不需要要额外的缩进。：

```objc
RACStream *result = [stream startWith:@0];

RACStream *result2 = [stream map:^(NSNumber *value) {
    return @(value.integerValue + 1);
}];
```

如果传输同一个流多次，确保每一个步骤都是对齐的。
复杂的操作比如 `+zip:reduce:` 或 `+combineLatest:reduce:` 可以拆分成多行提高可读性。

```objc
RACStream *result = [[[RACStream
    zip:@[ firstStream, secondStream ]
    reduce:^(NSNumber *first, NSNumber *second) {
        return @(first.integerValue + second.integerValue);
    }]
    filter:^ BOOL (NSNumber *value) {
        return value.integerValue >= 0;
    }]
    map:^(NSNumber *value) {
        return @(value.integerValue + 1);
    }];
```

当然，带block参数的嵌套的流应该跟block一起自然缩进：

```objc
[[signal
    then:^{
        @strongify(self);

        return [[self
            doSomethingElse]
            catch:^(NSError *error) {
                @strongify(self);
                [self presentError:error];

                return [RACSignal empty];
            }];
    }]
    subscribeCompleted:^{
        NSLog(@"All done.");
    }];
```

### 2.3.3 对流的所有的值使用相同的类型（Use the same type for all the values of a stream）

`RACStream` （包括它的扩展，`RACSignal` 和 `RACSequence`）允许流由异质对象（即对象的类型不一致）组成，就像 Cocoa 集合那样。    
然而，在流中使用不同的类型使得操作复杂，还会给流的使用者带来额外的负担，使用者应该只关心如何调用支持的方法。

任何可能的时候，流都应该只包含相同类型的对象。

### 2.3.4 避免长时间持有流（Avoid retaining streams for too long）

保留 `RACStream` 超过必要的时间会引发关于保留的依赖问题，如内存使用过高等。

`RACSequence` 应该只被保留序列的 `head` 需要被保留的那么长时间。如果 head 不再被使用，保留节点的 tail 代替节点本身。 

参见 `Memory Management` 指引获取关于对象生命周期的更多信息。

### 2.3.5 只处理需要数量的流（Process only as much of a stream as needed）

让流或者 `RACSignal` 的订阅保持不必要的活跃状态会导致CPU使用增长。

如果流中只有特定数量的值需要被用到，`-take:` 操作可以用来返回这些值，然后返回值之后立即自动结束流。

类似 `-take:` 和 `-takeUntil` 等操作能自动释放栈。如果其余的值不再需要，任何依赖也会结束，这可以显著的减少潜在的开销。

### 2.3.6 分发信号事件到一个已知的调度（Deliver signal events onto a known scheduler）

当信号被一个方法返回，或者被信号组合，很难搞清楚是在哪个线程上事件被分发。
尽管事件被确保是串行的，但有时候需要更严格的情形，比如 UI 的刷新必须在主线程。

无论何时保证事件是串行的都很重要，`-deliverOn:` 操作应该被用来强制信号事件到达一个明确的 `RACScheduler`。

### 2.3.7 （较少的场合需要切换调度者）Switch schedulers in as few places as possible

在满足上面的情况下，事件还应该在必要的时候分发到明确的 `scheduler`。切换调度这会引入不必要的时延和 CPU 负担。

通常，使用 `-deliverOn:` 应该被限制在信号链的末端。例如，在订阅之前，或者在被绑定到一个属性之前。

### 2.3.8 明确信号的副作用（Make the side effects of a signal explicit）

`RACSignal` 的副作用应该尽可能避免，因为订阅可能出现行为副作用异常。

然而，有时候信号时间发生时，副作用是有用的。
尽管大多数 `RACStream` 和 `RACSignal` 操作接受任意的 block (有副作用的)，
使用 `-doNext:`, `-doError:`, `-doCpmpleted:` 能更明确和自解释副作用的发生。

```objc
NSMutableArray *nexts = [NSMutableArray array];
__block NSError *receivedError = nil;
__block BOOL success = NO;

RACSignal *bookkeepingSignal = [[[valueSignal
    doNext:^(id x) {
        [nexts addObject:x];
    }]
    doError:^(NSError *error) {
        receivedError = error;
    }]
    doCompleted:^{
        success = YES;
    }];

RAC(self, value) = bookkeepingSignal;
```

### 2.3.9 用多播共享信号副作用（Share the side effects of a signal by multicasting）

默认情况下，副作用在每次订阅的时候发生，但在某些特定的情况下副作用应够只发生一次--例如，
一个网络请求很明显不应该在添加新的订阅的时候重复调用。

`RACSignal` 的 `-publish` 和 `-multicast:` 操作允许一个单一的订阅通过使用 `RACMulticastConnection` 共享给多个订阅者。

```objc
// This signal starts a new request on each subscription.
RACSignal *networkRequest = [RACSignal createSignal:^(id<RACSubscriber> subscriber) {
    AFHTTPRequestOperation *operation = [client
        HTTPRequestOperationWithRequest:request
        success:^(AFHTTPRequestOperation *operation, id response) {
            [subscriber sendNext:response];
            [subscriber sendCompleted];
        }
        failure:^(AFHTTPRequestOperation *operation, NSError *error) {
            [subscriber sendError:error];
        }];

    [client enqueueHTTPRequestOperation:operation];
    return [RACDisposable disposableWithBlock:^{
        [operation cancel];
    }];
}];

// Starts a single request, no matter how many subscriptions `connection.signal`
// gets. This is equivalent to the -replay operator, or similar to
// +startEagerlyWithScheduler:block:.
RACMulticastConnection *connection = [networkRequest multicast:[RACReplaySubject subject]];
[connection connect];

[connection.signal subscribeNext:^(id response) {
    NSLog(@"subscriber one: %@", response);
}];

[connection.signal subscribeNext:^(id response) {
    NSLog(@"subscriber two: %@", response);
}];
```

### 2.3.10 通过给定的名字调试流（Debug streams by giving them names）

每一个 `RACStream` 有一个 `name` 属性用来协助调试。
流的 `description` 包含流的名称，并且 RAC 所有的操作都会添加这个名称。从名称可以很方便的标识出一个流。

例如如下代码片段：

```objc
RACSignal *signal = [[[RACObserve(self, username) 
    distinctUntilChanged] 
    take:3] 
    filter:^(NSString *newUsername) {
        return [newUsername isEqualToString:@"joshaber"];
    }];

NSLog(@"%@", signal);
```

上面的代码会记录一个类似 `[[[RACObserve(self, username)] -distinctUntilChanged]
-take: 3] -filter:` 的名称。

名称也可以通过 `-setNameWithFormat:` 手工添加。

`RACSignal` 也提供 `-logNext`, `-logError`, `-logCompleted` 和 `-logAll` 方法，
这些方法在事件发生时自动记录信号事件，包括信号名称和消息。这可以为实时观察信号提供便利。

### 2.3.11 避免明确的订阅和释放（Avoid explicit subscriptions and disposal）

尽管 `-subscribeNext:error:completed:` 和它的变体是处理信号的最基本的方式，但它们使用较少的声明导致代码复杂，
推荐使用副作用，尽量复用潜在的内建功能。

同样的，明确的使用 `RACDisposable` 类能快速导致老鼠窝一样（啥意思，一团糟的意思吗？）的资源管理和代码清除。

下面是几乎总是该遵循的高级模式，用来替换手动订阅和释放：

* `RAC` 或 `RACChannelTo` 宏能够用来绑定信号到一个属性，用来代替在改变发生时手动更新的机制。
* `-rac_liftSelector:withSignals:` 方法能够用来在信号触发时自动调用一个 selector 。
* `-takeUntil:` 之类的操作在时间发生时能够用来自动释放订阅（例如 UI 的‘取消’按钮被按下）。

通常，相比在订阅的回调中完成相同功能，使用 `stream` 和 `signal` 内建的操作只需更简单更少出错的代码。

### 2.3.12 尽可能避免使用 subjects（Avoid using subjects when possible）

`Subjects` 是信号用来桥接命令式代码和现实世界的一个强有力的工具。但是对于可变 RAC 来说，他们的过度使用很快会导致代码复杂。
 
 因为 Subjects 可以在任何地方任何时间使用，所以 subjects 经常打破 stream 的线性处理，导致逻辑复杂。
 Subjects 也不支持严格的 disposal，严格的 disposal 会引入不必要的任务。

Subjects 能够被 ReactiveCocoa 的下列其他模式替换：

* 考虑用 `+createSignal` block 生成值 来代替 提供初始化值到一个 subject 中。
* 考虑用 `+combineLatest:` 或 `+zip:` 等操作合并多个信号的输出 来代替 分发中间结果给 subject。
* 考虑用 `multicast` 多播基本的信号 来代替 使用subjects共享多个订阅的结果。
* 考虑用 `command` 或 `-rac_signalForSelector:` 来代替 实现多个动作方法来实现对 subject 的简单控制。

当 subject 必须使用时，他们几乎总是被使用在信号链的基本输入，而不是在信号链中间使用。

## 2.4 实现一个新的操作（Implementing new operators）

RAC 为 `stream` 和 `signal` 提供了大量内建操作，能够满足大部分应用场景；然而，RAC 不是一个封闭的系统。
ReactiveCocoa 考虑了为了特殊的用途实现一些额外的操作。

实现新的操作需要特别注意一些细节和简单化操作，避免在调用的代码中引入bug。

下面的指南包括一些通用的原则能够帮助编写符合预期的 API:

### 2.4.1 优先使用基于 RACStream 的方法（Prefer building on RACStream methods）

`RACStream` 提供的接口比 `RACSequence` 和 `RACSignal` 更简单，而且所有的 stream 操作也适用于 sequence 和 signal。

基于这个原因，无论何时新操作都应该基于 `RACStream` 的方法实现。
至少需要 `RACStream` 类的 `-bind:`, `-zipWith:`, 和 `-concat:` 方法，这些方法就已经很强大了，
不需要添加其他任何功能就可以完成很多任务。

如果一个新的 `RACSignal` 操作需要处理 `error` 和 `completed` 事件， 考虑使用 `-materialize` 方法给 stream 引入事件。
所有 materialized 的信号事件都能够被流操作修改，这能帮助最小化使用非 stream 的操作（？？？这句话没整明白）。

### 2.4.2 尽可能组合已存在的操作（Compose existing operators when possible）

RAC 经过了深思熟虑，通过了合法性测试，也在很多项目中被使用。重新改写操作的代码可能不会有很好的健壮性，或者不能处理一些内建操作已经考虑到的特殊情况。

为了最小的重复代码和尽可能少的引入 bug，在自定义的操作实现中，尽可能使用已提供的功能。通常只有很少的代码需要重写。

### 2.4.3 避免引入并发（Avoid introducing concurrency）

在编程中，并发是非常容易引入 bug 的。为了尽可能的避免死锁和竞态，不应该引入并发。

调用者可以在一个明确的 `RACScheduler` 上订阅和分发事件，RAC 提供强大的 `parallelize work` 方式而不会特别复杂。

### 2.4.4 在 disposable 中取消任务和清理所有资源（Cancel work and clean up all resources in a disposable）

 使用 `+createSignal:` 方法创建一个信号时，它提供的 block 需要返回一个 `RACDisposable`。
 该 disposable 可以：
 
 * 可以方便的取消信号开始的任务。
 * 立即 dispose 到其他信号的订阅，然后触发取消和清理。
 * 释放信号分配的内存和其它资源。

 这有助于实现 `RACSignal 的约定`

### 2.4.5 操作中不要阻塞（Do not block in an operator）

流操作应该立即返回一个新流。任何操作需要完成的工作应该是新流的运算的一部分，而 _不是_ 调用的流自身的一部分。

```objc
// WRONG!
- (RACSequence *)map:(id (^)(id))block {
    RACSequence *result = [RACSequence empty];
    for (id obj in self) {
        id mappedObj = block(obj);
        result = [result concat:[RACSequence return:mappedObj]];
    }

    return result;
}

// Right!
- (RACSequence *)map:(id (^)(id))block {
    return [self flattenMap:^(id obj) {
        id mappedObj = block(obj);
        return [RACSequence return:mappedObj];
    }];
}
```

如果要从流中返回一个或多个值（例如 `first`），该规则可以忽略掉。

### 2.4.6 避免深度递归导致栈溢出（Avoid stack overflow from deep recursion）

任何无限递归操作都需要使用 `RACScheduler` 的 `shceduleRecusiveBlock:` 方法。该方法会将递归转换为迭代操作，防止栈溢出。

例如，下面是 `-repeat` 的一个错误的实现，肯定会导致栈溢出和崩溃：

```objc
- (RACSignal *)repeat {
    return [RACSignal createSignal:^(id<RACSubscriber> subscriber) {
        RACCompoundDisposable *compoundDisposable = [RACCompoundDisposable compoundDisposable];

        __block void (^resubscribe)(void) = ^{
            RACDisposable *disposable = [self subscribeNext:^(id x) {
                [subscriber sendNext:x];
            } error:^(NSError *error) {
                [subscriber sendError:error];
            } completed:^{
                resubscribe();
            }];

            [compoundDisposable addDisposable:disposable];
        };

        return compoundDisposable;
    }];
}
```

而下面的版本就会避免栈溢出：

```objc
- (RACSignal *)repeat {
    return [RACSignal createSignal:^(id<RACSubscriber> subscriber) {
        RACCompoundDisposable *compoundDisposable = [RACCompoundDisposable compoundDisposable];

        RACScheduler *scheduler = RACScheduler.currentScheduler ?: [RACScheduler scheduler];
        RACDisposable *disposable = [scheduler scheduleRecursiveBlock:^(void (^reschedule)(void)) {
            RACDisposable *disposable = [self subscribeNext:^(id x) {
                [subscriber sendNext:x];
            } error:^(NSError *error) {
                [subscriber sendError:error];
            } completed:^{
                reschedule();
            }];

            [compoundDisposable addDisposable:disposable];
        }];

        [compoundDisposable addDisposable:disposable];
        return compoundDisposable;
    }];
}
```

[Framework Overview]: FrameworkOverview.md
[Memory Management]: MemoryManagement.md
[NSObject+RACLifting]: ../ReactiveCocoa/NSObject+RACLifting.h
[NSObject+RACSelectorSignal]: ../ReactiveCocoa/NSObject+RACSelectorSignal.h
[RAC]: ../ReactiveCocoa/RACSubscriptingAssignmentTrampoline.h
[RACChannelTo]: ../ReactiveCocoa/RACKVOChannel.h
[RACCommand]: ../ReactiveCocoa/RACCommand.h
[RACDisposable]: ../ReactiveCocoa/RACDisposable.h
[RACEvent]: ../ReactiveCocoa/RACEvent.h
[RACMulticastConnection]: ../ReactiveCocoa/RACMulticastConnection.h
[RACObserve]: ../ReactiveCocoa/NSObject+RACPropertySubscribing.h
[RACScheduler]: ../ReactiveCocoa/RACScheduler.h
[RACSequence]: ../ReactiveCocoa/RACSequence.h
[RACSignal]: ../ReactiveCocoa/RACSignal.h
[RACSignal+Operations]: ../ReactiveCocoa/RACSignal+Operations.h
[RACStream]: ../ReactiveCocoa/RACStream.h
[RACSubscriber]: ../ReactiveCocoa/RACSubscriber.h
[Subjects]: FrameworkOverview.md#subjects
[Parallelizing Independent Work]: ../README.md#parallelizing-independent-work


# 3 与 Rx 的差异（Differences from Rx）

ReactiveCocoa (RAC) 深受 .NET 的 `Reactive Extensions` （Rx）影响，但不是直接的移植。
RAC 的一些原则和接口对于已经熟悉 Rx 的开发者来说都会迷惑，但还是表达了相同的算法。

一些不同之处，像方法和类的命名，是为了符合 Cocoa 现有的风格。
还有一些是在 Rx 上的改进，或者从其他函数式响应式编程范式中借鉴的（例如 Elm 编程语言）。

下面，尝试讲解 RAC 和 Rx 的不同。
## 3.1 接口（Interfaces）

EAC 不提供类似 .NET 中的 `IEnumerable` 和 `IObserver` 接口。RAC 中主要用三个类来代替：

 * **RACStream**
   实现了基本流操作的抽象类。 实现了常用的 LINQ（Language Integrated Query，C# 术语，用于方便的执行一些增删查改等）操作。
 * **RACSignal**
   是 `RACStream` 的一个具体的子类，实现了 _pish-driven_ 流，跟 `IObserver` 很像。
   在该类或 `RACSignal+Operations` 类别中可以找到基于时间的操作，或者处理 `completed` 和 `error` 事件的方法。
 * **RACSequence**
   也是 `RACStream` 的一个具体的子类实现了 _pull-driven_ 流，跟 `IEnumerable` 很像。
 
## 3.2 流操作的名称（Names of Stream Operations）

RAC 通常使用 LINQ 风格命名流方法。大多数异常处理深受 Haskell 和 Elm 的影响。

注意如下不同：

* 用 `-map:` 代替 `Select`
* 用 `-filter:` 代替 `Where`
* 用 `-flatten ` 代替 `Merge`
* 用 `-flattenMap:` 代替 `SelectMany`

LINQ 操作在 RAC 中名称有变化（但行为或多或少相同），在文档中有记载，例如：

```objc
// Maps `block` across the values in the receiver.
//
// This corresponds to the `Select` method in Rx.
//
// Returns a new stream with the mapped values.
- (instancetype)map:(id (^)(id value))block;
```

# 4 框架概览（Framework Overview）

本文包含 ReactiveCocoa 框架不同组件的一些高层次的描述，并且尝试说明他们如何在一起工作，然后分别承担什么职责。
这意味着要先理解本文，然后才学习其他新模块和其他特定文档。

范例和如何使用 RAC，参见 `README` 或 `Design Guidelines`。

## 4.1 流（Streams）

`RACStream` 抽象类用来描述流，是存储对象值得所有序列。

值可以立即获得，也可以在未来某个时间获得，但必须是按顺序获取。在没有计算或等到第一个值之前是无法获取第二个值的。

流是游离的（monads，游牧的）？还允许基于一些基本操作构建复杂的操作（特别是 `-bind:`）。
RACStream 也从 `Haskell` 实现了 `Monoid` 和 `MonadZip` 类型类。？？

`RACStream` 自身并不是特别有用。大多数流被 `signal` 或 `sequences` 替代。

## 4.2 信号（Signals）

**signal**，由 `RACSignal` 表示，是 _push-dirven_ 类型的流。

信号通常用来表示未来可能会被分发的数据。当任务完成或者数据被接受，值会被 _发送_ 到信号，信号则推送他们到任何订阅者。
用户必须订阅(subscribe)信号才能访问信号的值。

信号提供给订阅者三种不同类型的事件：

* **next** 事件提供流中的一个新值。`RACStream` 方法只操作这种类型的事件。
* 不像 Cocoa 集合，它完全有效的信号是包括了 `nil`的.
* **error** 事件表明在信号完成之前发生了错误。该事件会包含一个 `NSError` 对象表明是什么错误。
* 错误必须特殊处理--错误不包含流中的值。
* **completed** 事件表明信号成功完成了，并且完成之后不会再有值会被添加到流中。完成操作必须特殊处理--完成不包含流的值。

信号的生命周期由任意数量的 `next` 组成，跟随者 `错误（error）` 和 `完成（completed）` 
(错误和完成二者只存在一个，不会同时存在)。

### 4.2.1 订阅（Subscription）

**订阅者** 是唯一能从信号中等待或者能够从信号中等待事件的主体。在 RAC 中，订阅是任何遵从 `RACSubscriber` 协议的对象。

**订阅** 在 `-subscribeNext:error:completed:` 或其他相应的方法中创建。
严格来说，大多数 `RACStream` 和 `RACSignal` 操作也能够创建订阅，
但这些中间状态的订阅通常只是一个实现的细节（不是很明白）？？

订阅会保留他们订阅的信号，不会自动释放除非信号完成或者出错。订阅也能够手动释放。

### 4.2.2 Subjects

**subject** 用 `RACSubject` 来描述，是可以被手动控制的信号类型。

Subjects 可以认为是可变的信号，类似 `NSMutableArray` 和 `NSArray` 的关系。在桥接非 RAC 代码和信号时很有用。

例如，处理应用逻辑的block，可以用发送事件到共享 subject 的 block 代替。
subject 随后返回一个 `RACSignal`，隐藏 block 的实现细节。

某些 subject 提供额外的功能。`RACReplaySubject` 能够用来为未来的订阅者缓存事件，
就像网络请求完成之前处理结果的东西都已准备好。

### 4.2.3 命令（Commands）

**command** 用 `RACCommand` 类来表示，为某些动作响应创建和订阅信号。命令让用户与 app 交互实现副作用变的很容易。

通常行为触发命令是由 UI 驱动的，例如一个按钮被按下。
基于信号的命令能够自动被禁用，这个禁用状态能够用 UI 中其他任何跟命令相关的控件来描述。

在 OS X 中，RAC 添加了一个 `rac_command` 属性到 `NSButton` 用来自动设置按钮的行为。

### 4.2.4 连接（Connections）

**连接** 用 `RACMulticastConnectiong` 类来描述，是可以在任意数量的订阅者之间共享的订阅。

信号默认是 _cold_ ,意味着他们在每一次新订阅者添加的时候开始工作。
这个行为通常是期望的，因为数据会为每个订阅者刷新和重新计算，但是如果信号有副作用或者任务代价很昂贵就会引入一些问题
（例如发送网络请求）。

连接通过 `RACSignal` 类的 `-publish` 或者 `-multicast:` 方法创建，
并且不管连接有多少次订阅，确保底层只有一个订阅被创建。一旦连接建立，连接的信号宣告是 _hot_ 类型，
在这底层的订阅会保留活动状态直到连接的 _所有_ 订阅都被释放。
 
## 4.3 序列（Sequences）

**序列** 用 `RACSequence` 类来描述，是 _pull-driven_ 类型的流。

序列是一个集合，累世完成 `NSArray` 的功能。
跟 array 不一样的是，序列的值默认是 _lazily_ 的，只在序列被需要的时候才会计算，提高了效率。
序列不能包含 `nil`。

序列类似 `闭包的序列` 或者  `Haskell` 中的 `List` 类型。

RAC 添加了 `-rac_qequence` 方法到大多数 Cocoa 的集合类，允许他们使用 `RACSequences` 来替代。

## 4.4 释放（Disposables）

**RACDisposable** 类用来取消任务和资源清理。

Disposables 常用于信号的取消订阅。
当订阅被释放，响应的订阅者不会再收到信号的任何未来事件。
并且任何订阅相关的工作（后台处理，网络请求等）都会取消，因为结果不再需要了。

关于取消的更多信息，参见 RAC `Design Guidelines`。

## 4.5 调度（Schedulers）

**调度** 用 `RACScheduler` 类来描述，是一个串行执行队列，信号在上面完成任务和分发结果。

调度类似 GCD 队列，但调度支持取消，并且总是串行执行。
`+immediateScheduler` 异常时，调度不会提供同步执行。这可以避免死锁，鼓励使用信号操作代替 block。

`RACScheduler` 有些方面也像 `NSOperatiaonQueue`，但调度不允许任务重新排序，也不支持依赖。

## 4.6 值类型（Value types）
   
 RAC 提供少量杂项类来方便表达流中的值：
 
 * **RACTuple** 是个小的，固定带笑傲的集合，能够包含 `nil`（用 `RACTupleNil` 表示）。经常用来表示多个流中合并的值。
 * **RACUnit** 是空值得单例。用来表示流中某些时候没有存在意义的数据。
 * **RACEvent** 表示任意信号事件。主要被 `RACSignal` 类的 `-materialize` 方法使用。

[Design Guidelines]: DesignGuidelines.md
[Haskell]: http://www.haskell.org
[lazy-seq]: http://clojure.github.com/clojure/clojure.core-api.html#clojure.core/lazy-seq
[List]: https://downloads.haskell.org/~ghc/latest/docs/html/libraries/base-4.7.0.2/Data-List.html
[Memory Management]: MemoryManagement.md
[monads]: http://en.wikipedia.org/wiki/Monad_(functional_programming)
[Monoid]: http://downloads.haskell.org/~ghc/latest/docs/html/libraries/base-4.7.0.2/Data-Monoid.html
[MonadZip]: http://downloads.haskell.org/~ghc/latest/docs/html/libraries/base-4.7.0.2/Control-Monad-Zip.html
[NSButton+RACCommandSupport]: ../ReactiveCocoa/NSButton+RACCommandSupport.h
[RACCommand]: ../ReactiveCocoa/RACCommand.h
[RACDisposable]: ../ReactiveCocoa/RACDisposable.h
[RACEvent]: ../ReactiveCocoa/RACEvent.h
[RACMulticastConnection]: ../ReactiveCocoa/RACMulticastConnection.h
[RACReplaySubject]: ../ReactiveCocoa/RACReplaySubject.h
[RACScheduler]: ../ReactiveCocoa/RACScheduler.h
[RACSequence]: ../ReactiveCocoa/RACSequence.h
[RACSignal]: ../ReactiveCocoa/RACSignal.h
[RACSignal+Operations]: ../ReactiveCocoa/RACSignal+Operations.h
[RACStream]: ../ReactiveCocoa/RACStream.h
[RACSubject]: ../ReactiveCocoa/RACSubject.h
[RACSubscriber]: ../ReactiveCocoa/RACSubscriber.h
[RACTuple]: ../ReactiveCocoa/RACTuple.h
[RACUnit]: ../ReactiveCocoa/RACUnit.h
[README]: ../README.md
[seq]: http://clojure.org/sequences


# 5. 内存管理（Memory Management）

ReactiveCocoa 的内存管理非常复杂，但最终结果是
**处理信号时，你并不需要保留他们**。

如果框架要求你保留每一个信号，那这个框架使用起来就太笨重，特别是一次性的信号用于未来某个时候。
你不需要保留任何长时间活跃的信号到属性中，然后确保用完之后再清除。这样可没劲了。

## 5.1 订阅者（Subscribers）

无论去哪之前， `subscribeNext:error:completed:` （还有所有其所有变量）创建一个 _隐式_ 的订阅者使用给定的block。
任何这些 block 引用的对象会被保留为订阅的一部分。就像其他对象，`self` 不会被保留除非有一个对它直接或间接的引用。

## 5.2 有限或短暂的信号（Finite or Short-Lived Signals）

RAC 内存管理最重要的原则是 **订阅在完成后错误时自动终止，订阅者会被移除**

例如，如果你的 view controller 中的代码如下：

```objc
self.disposable = [signal subscribeCompleted:^{
    doSomethingPossiblyInvolving(self);
}];
```

… the memory management will look something like the following:

那么内存管理会是下面的流程：

```
view controller -> RACDisposable -> RACSignal -> RACSubscriber -> view controller
```

然而， `RACSignal -> RACSubscriber` 的关系会在信号完成时立刻解除，打破保留环。

**这是你需要的**，因为 `RACSignal` 的生命周期会自然而然的匹配时间流的逻辑生命周期。

## 5.3 无限信号（Infinite Signals）

Infinite signals (or signals that live so long that they might as well be
infinite), however, will never tear down naturally. This is where disposables
shine.

**Disposing of a subscription will remove the associated subscriber**, and just

无限信号（或者说永远存活的信号），不会被自动清除。

**释放订阅会移除相关的订阅者**，并且会清除订阅相关的任何资源。

作为一个一般的经验法则，如果你需要手动管理订阅的生命周期，那可能存在更好的方式做到你想要的，请避免显示的订阅和释放。

## 5.4 从 `self` 分发的信号（Signals Derived from `self`）

还是有些比较棘手的情况的。任何时候一个信号的生命周期被绑在一个调用范围时，会比较难打破循环。

这种情况通常发生在关键路径上使用 `RACObserve()`，而关键路径又与 `self` 关联，然后应用的 block 有需要捕获 `self`。 

最简单的解决方案是 **捕获 self 弱引用**。

```objc
__weak id weakSelf = self;
[RACObserve(self, username) subscribeNext:^(NSString *username) {
    id strongSelf = weakSelf;
    [strongSelf validateUsername];
}];
```

或者，在导入 `EXTScope.h` 头文件之后：

```objc
@weakify(self);
[RACObserve(self, username) subscribeNext:^(NSString *username) {
    @strongify(self);
    [self validateUsername];
}];
```

*如果对象不支持若引用，那么用 `__unsafe_unretained` 或 `@unsafeify` 分别替换 `__weak` 或 `@weakify`*
例如，上面的示例可以像下面这么写：

```objc
[self rac_liftSelector:@selector(validateUsername:) withSignals:RACObserve(self, username), nil];
```

或者:

```objc
RACSignal *validated = [RACObserve(self, username) map:^(NSString *username) {
    // Put validation logic here.
    return @YES;
}];
```

无限的信号，通常可以从信号链的 block 避开引用 `self`（或任何对象）。

----

上面的信息是高效使用 ReactiveCocoa 所需要的一切。然而，还有很多只为技术上的好奇心或者对 RAC 感兴趣的声音存在。

“不需要保留”的设计目标引入如下问题：我们如何知道信号什么时候需要被释放？如果信号刚创建，没被自动释放池管理，没有被保留的话。

正确的回答是 `我们不需要`，但我们通常确保调用者如果想保留信号，会在当前运行循环迭代中保留它。

因此：

1. 一个被创建的信号自动被添加到激活信号集合中。
2. 信号会等一个主运行循环，然后如果没有订阅者订阅它就会从激活信号集中移除。除非信号被什么保留了，否则会被释放。
3. 如果在运行循环迭代中订阅发生了，信号会留在集合中。
4. 最后，如果所有订阅者已过去，步骤2就会再被触发。

如果运行循环是spun recursively（纺递归是什么鬼？就像 OS X中的模态事件）那就适得其反了。但是让框架的用户使用方便比任何其他事情都重要。
 

