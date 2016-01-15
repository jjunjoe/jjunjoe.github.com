---
layout: post
title: "Core Bluetooth Programming Guide 中文翻译"
date: 2014-07-02 20:43:57 +0800
comments: true
categories: Apple Programming Guide
---
## 前言
本文是苹果《Core Bluetooth Programming Guide》的翻译。  
  
## 关于Core Bluetooth  
Core Bluetooth 框架提供了蓝牙低功耗无线设备与 iOS 应用或 Mac 应用通讯的必要的类。应用可以发现，探索低功耗外设，并与它交互，比如心率监控器和数字温度调节器。`从 OS X V10.9 和 iOS 6 之后，Mac 和 iOS 设备也能充当蓝牙低功耗外设给包括 Mac 和 iOS 在内的其他设备提供数据服务了。`  

![](http://jjunjoe.github.io/images/apple/corebluetooth/figure0-1.png)
  
<!--more-->
### 概览  
蓝牙低功耗无线技术（BLE）基于蓝牙4.0规范，在其基础上定义了大量低功耗设备之间通信的协议。Core Bluetooth 框架是蓝牙低功耗协议栈的抽象。也就是说，它对开发者隐藏了规范底层的许多细节，使得你开发与蓝牙低功耗设备通讯的应用变得容易。  
  
#### 主机（Central）和外设（Peripheral）是 Core Bluetooth 的关键角色  
在蓝药低功耗通讯中，有两个关键的角色：`主机`和`外设`。一个典型的外设拥有其他设备需要的数据。一个典型的主机使用外设提供的信息完成某些任务。例如，BLE数字温度调节器可以提供室内温度给一个 iOS 应用，然后应用将温度用用户友好的方式展现出来。    
外设在空气中广播数据。而主机可以扫描拥有它感兴趣的数据的外设。当主机发现这样的外设时，主机可以请求连接该外设然后探索和交互。然后外设以主机期望的方式响应主机的请求。  
  

  #### Core Bluetooth 简化了通用的蓝牙任务  
Core Bluetooth 框架抽象了蓝牙4.0规范的底层细节。这样应用中需要完成的许多常用的 BLE 任务都变得很简单了。如果你开发的应用是完成蓝牙主机功能，Core Bluetooth 让它很容易发现和与外设连接，容易探索外设和与外设交互。另外，Core Bluetooth 让你很容易使用本地设备来充当外设。  
  
#### iOS 应用状态影响蓝牙行为  
当 iOS 应用处于后台或者挂起状态，与蓝牙相关的功能会受到影响。默认情况下，应用在后台或者挂起状态下不能完成 BLE 任务。如果你的应用需要在后台完成 BLE 任务，你需要声明一个或者全部两个 Core Bluetooth 应用的执行模式（一个是主机，一个是外设）。即使你声明了他们的执行模式，但是当应用在后台时，某些特定的蓝牙任务操作还是会与前台有所区别。在设计应用时需要注意这些差别。     
支持后台处理的应用可能在任何时候被系统终止然后释放内存。自 iOS 7,Core Bluetooth 支持保存主机和外设管理的对象，也支持在应用重启时恢复状态。你可以使用这个特性来支持必要的长期蓝牙设备任务。  
  
#### 按照最佳实践加强用户体验  
Core Bluetooth 框架让你控制许多通用的 BLE 传输。你需要按照最佳实践负责任的完成这个层级的控制，增强用户体验。  
例如，许多任务需要使用设备无线电来发射信号，而设备无线电是与其他形式的无线通讯共享的，并且无线电使用对电池的使用寿命有很大的影响，所以设计应用时要尽量减少对无线电的使用。  
  
#### How to Use This Document  
未翻译。  
  
#### See Also  
未翻译。  
  
# Core Bluetooth 概览  
Core Bluetooth 框架让你的 iOS 和 Mac 应用与 BLE 设备通信。例如，你的应用发现，探索，并与 BLE 设备通信（心率监控器，数字温度调节器，甚至其他 iOS 设备）。    
Core Bluetooth 框架是对使用低功耗设备的蓝牙4.0规范的抽象。也就是说，它影藏了许多底层细节，开发者使用框架可以更容易的开发与 BLE 交互的应用。因为框架基于蓝牙4.0规范，所以采用了规范的一些原则和术语。本章介绍关键的术语和原则，这些都是你使用 Core Bluetooth 开发应用时必须清楚的。  
  
## 主机和外设以及他们在蓝牙通信中扮演的角色  
在所有的 BLE 通信中有两个主要的概念：`主机`和`外设`。基于一些传统的CS体系，`外设`典型的`拥有其他设备需要的数据`。`主机`典型的`使用外设提供的信息来完成特定的任务`。如图 1-1 所示，一个心率监控器有些有用的信息，你的 Mac 或 iOS 应用需要这些数据然后将它们友好的展现给用户。  

![](http://jjunjoe.github.io/images/apple/corebluetooth/figure1-1.png)
  
### 主机发现和连接发广播的外设  
外设向外广播一些广播包形式的数据。`广播包`是外设提供的包含有用信息的一束数据，例如外设的名称和主要功能。举个例子，数字温度调节器可以广播当前室温。在 BLE，广播是外设被感知存在的主要方式。    
主机能够扫描，侦听任何它感兴趣的正在广播的外设。如图 1-2 所示，主机能够请求连接它发现的外设。  

![](http://jjunjoe.github.io/images/apple/corebluetooth/figure1-2.png)
  
### 外设的数据如何组织  
连接外设的目的是为了探索外设并与外设交互数据。在做这个之前，有必要理解外设的数据是怎么组织的。    
外设可以包含一个或者多个服务来提供连接信号强度相关的有用的信息。`服务`是`设备完成某功能相关的数据集合或者设备某特征数据的集合`。例如，心率监控器可以展示来自心率传感器的心率数据。`服务`由`特征（characteristics）`或者`其他服务`组成。`特征`提供外设服务的进一步详情。例如，心率服务可能仅仅包含一个描述心率传感器预定位置的特征和另外一个发送心率测量数据的特征。图 1-3 列出了心率监控器服务和特征的数据结构。  

![](http://jjunjoe.github.io/images/apple/corebluetooth/figure1-3.png)
  
### 主机探索外设与外设交互数据  
在主机成功与外设建立连接之后，主机可以找出外设提供的全方位的服务和特性（而广播数据可能只包含部分可用的服务）。    
主机也可以通过读写服务的特征值来与外设交互。例如，应用可以向数字温度调节器请求当前室温，或者提供一个值让温度调节器设定室温。  
  
## 如何描述主机，外设，外设数据  
在 Core Bluetooth 框架中，BLE 的主要参与者和数据都被映射得简单而轻量。  
  
### 主机端相关对象  
使用本地主机与远程外设通信时，你在主机端完成操作。除非你设置一个本地外设，然后使用它响应主机的请求，否则大多数蓝牙传输发生在主机这一端。  
  
#### 本地主机和远程外设  
在主机端，本地主机设备用 `CBCentralManager` 对象表示。这些对象用来管理发现或连接远程外设（远程外设用 `CBPeripheral` 对象表示），包括扫描，发现，连接发广播的外设。图 1-4 显示了 Core Bluetooth 框架中本地主机和远程外设如何表示。  

![](http://jjunjoe.github.io/images/apple/corebluetooth/figure1-4.png)
  
#### 远程外设的数据用 CBService 和 CBCharacteristic 对象表示  
当你与远程外设交互数据的时候，实际上是处理它的服务和特征。在 Core Bluetooth 框架中，远程外设的服务用 CBservice 对象表示。类似的，远程外设的特征用 CBCharacteristic 对象表示。 图 1-5 阐明了远程外设的基服务和特征的基本结构。  

![](http://jjunjoe.github.io/images/apple/corebluetooth/figure1-5.png)
  
### 外设方相关对象  
自 OS X v10.9 和 iOS 6，Mac 和 iOS 设备（包括Mac，iPhone，iPad）能够提供 BLE 外设功能，给其他设备提供服务数据。设置你的设备充当外设的角色时，你要完成 BLE 通信中外设那一侧的功能行为。              
  
#### 本地外设和远程主机  
在外设一侧，本地外设用 CBPeripheralManager 对象表示。这些对象用来管理发布本地外设服务和特征数据库中的服务，广播到远程主机设备（用 CBCentral 对象表示）。外设管理对象也用来响应远程主机的读写请求。图 1-6 显示了本地外设和远程主机在 Core Bluetooth 中的表示。  

![](http://jjunjoe.github.io/images/apple/corebluetooth/figure1-6.png)
  
#### 本地外设数据用 CBMutableService 和 CBMutableCharacteristic 对象表示  
设置一个本地外设并与它交互数据时，你需要处理本地外设的多个服务和特征的版本。在 Core Bluetooth 框架，本地外设的服务用 CBMutableService 对象表示。类似的，本地外设的特征用 CBMuableCharacteristic 对象表示。 图 1-7 阐明了本地外设服务和特征的基本机构。  

![](http://jjunjoe.github.io/images/apple/corebluetooth/figure1-7.png)
  
  
# 主机通常要完成的任务  
BLE 中扮演主机角色的设备要完成大量通用的任务--例如，发现和连接可以用的外设，探索外设，与外设交互数据。相对而言，扮演外设的角色也需要完成一些与主机不同的通用的任务，例如发布和广播服务，响应读写数据，处理已连接的主机的请求。    
本章将学习如何使用 Core Bluetooth 完成 BLE 主机端的通用任务。基于代码的示例可以帮助你用本地设备扮演主机的角色。具体来说你可以学习到：  
  
* 启动主机管理对象  
* 发现和连接广播的外设  
* 连接之后探索外设数据  
* 给外设服务的一些特征值发送读写请求  
* 订阅特征值并在他们更新时得到通知  
  
下一章将会学习如何用本地设备扮演外设角色。    
本章的示例代码简单而抽象，在你实际的应用中需要对其做适当的修改来完善。许多跟主机相关的高级主题会再后面的章节讲述。  
  
## 启动主机管理  
CBCentralManager 对象时本地主机设备面向对象的表示，完成 BLE 传输前需要分配和初始化一个主机管理实例。你可以通过调用CBCentralManager类的 initWithDelegate:queue:options: 方法初始化主机管理对象，如下所示：    
	  
	myCentralManager =    [[CBCentralManager alloc] initWithDelegate:self queue:nil options:nil];  
  
在这个例子中，self 被设置为接收任何主机事件的委托。通过指定 dispatch queue 为 nil，主机管理对象会用 main queue 主队列分发主机事件。     
当你创建一个主机管理对象，它会调用它委托的 centralManagerDidUpdateState: 方法。你需要实现改委托方法确保 BLE 在主机设备上被支持和可用。关于如果完成该委托方法的更多信息，参见 CBCentralManagerDelegate Protocol Reference。  
  
## 发现在广播的外设  
主机端的第一个任务是发现应用需要连接的可用的外设。正如前面提到的，广播是外设让外界感知它存在的主要方式。你可以通过调用 CBCentralManager 类的 scanForPeripheralsWithServices:options: 方法来发现任何正在广播的外设，如下所示：  
  
	[myCentralManager scanForPeripheralsWithServices:nil options:nil];  
	  
注意：如果第一个参数填 nil，主机管理对象返回所有发现的外设，而不管他们提供什么服务。在实际的应用中，你需要指定一个 CBUUID 对象的数组，每一个 CBUUID 对象表示一个 UUID，代表外设正在广播的一个服务。当你指定了 UUID 数组，主机管理对象仅返回广播这些服务的外设，可以让你仅扫描你感兴趣的设备。    
在调用 scanForPeripheralsWithServices:options: 方法发现可用的外设后，主机管理对象每发现一个外设都调用一次它的委托方法 centralManager:didDiscoverPeripheral:advertisementData:RSSI: 。任何被发现的外设都返回一个 CBPeripheral 对象。如下面所示，你可以完成该委托方法来列出发现的任何外设：  
  
	- (void)centralManager:(CBCentralManager *)central  
	didDiscoverPeripheral:(CBPeripheral *)peripheral       advertisementData:(NSDictionary *)advertisementData                    RSSI:(NSNumber *)RSSI {      NSLog(@"Discovered %@", peripheral.name);      ...  
  
当你发现你想连接的外设后，停止扫描其它设备以节约电量。  
  
	[myCentralManager stopScan];	NSLog(@"Scanning stopped");  
## 发现外设后连接它  
在你发现你感兴趣的外设后，可以请求连接，通过调用 CBCentralManager 类的 connectPeripheral:options: 方法。简单的调用该方法到你需要连接的指定外设，如下所示：  
  
	[myCentralManager connectPeripheral:peripheral options:nil];  
	  
假设连接成功，主机管理对象会调用它的委托方法 centralManager:didConnectPeripheral: ，你可以在建立连接后记录日志，如下所示：  
  
	- (void)centralManager:(CBCentralManager *)central    didConnectPeripheral:(CBPeripheral *)peripheral {    NSLog(@"Peripheral connected");    ...  
在与外设交互之前，需要设置外设的委托以确保期望的回调方法能响应，如下所示：  
	peripheral.delegate = self;  
## 发现已连接外设的服务  
在与外设建立连接之后，你可以探索它的数据。探索外设的数据的第一步是发现它可用的服务。因为广播的数据量大小是有限制的，你需要探索外设的更多服务而不仅是广播。你可以调用 CBPeripheral 类的 discoverServices: 方法来发现外设提供的所有服务。  
  
	[peripheral discoverServices:nil];  
	  
注意：在实际的应用中，你不应该传递 nil 做参数，因为这样会返回外设的所有服务。外设可能会拥有很多服务，而不仅仅是你感兴趣的那些。发现服务会浪费电池和时间。更多情况下，你应该为你感兴趣的服务指定 UUID。    
  
指定服务被发现时，外设（就是你已经连接上的 CBPeripheral 对象）调用 peripheral:didDiscoverServices: 委托方法。Core Bluetooth 建立一个 CBService 对象的数组--数组的每一个元素都对应一个外设已发现的服务。如下所示，你可以实现委托方法来访问数组中的服务：  
  
	- (void)peripheral:(CBPeripheral *)peripheral	didDiscoverServices:(NSError *)error {    for (CBService *service in peripheral.services) {        NSLog(@"Discovered service %@", service);        ...	} ...  
  
## 发现服务的特征  
假设你已经发现了你感兴趣的服务，下一步探索就是发现服务提供的所有特征。简单的调用 CBPeripheral 类的 discoverCharacteristics:forService: 方法就可以发现一个服务的所有特征，对于一个服务来说，可以操作如下所示：  
  
	NSLog(@"Discovering characteristics for service %@", interestingService);	[peripheral discoverCharacteristics:nil forService:interestingService];  
   
   
注意：在实际的应用中，第一个参数不应该传递 nil，因为这样会返回一个外设服务的所有特征。因为外设的一个服务可能包含许多的特征，而不仅仅是你感兴趣的那些，发现所有特征会影响电池寿命和浪费时间。更多情况下，你应该为你感兴趣的特征指定 UUID。  
  
外设指定服务的特征被发现后会调用它委托对象的  peripheral:didDiscoverCharacteristicsForService:error: 方法。Core Bluetooth 建立一个 CBCharacteristic 对象的数组--数组的每一个元素都对应一个已发现的特征。下面的例子显示了如何实现委托方法来简单的打印已发现的每一个特征：  
  
	- (void)peripheral:(CBPeripheral *)peripheral	didDiscoverCharacteristicsForService:(CBService *)service             error:(NSError *)error {    for (CBCharacteristic *characteristic in service.characteristics) {        NSLog(@"Discovered characteristic %@", characteristic);        ...	} ...## 获取特征的值  
`特征`包含了简单的值，用来`表示外设服务的更多信息`。例如，健康体温计服务的温度测量特征可能有一个值用来表示摄氏度。你可以直接获取或者订阅这些值。  
  
### 读特征值  
在你找到你感兴趣的服务的特征后，你可以读取你想要的特征值，通过调用 CBPeripheral 类的 readValueForCharacteristic: 方法，如下所示：  
  
	NSLog(@"Reading value for characteristic %@", interestingCharacteristic);	[peripheral readValueForCharacteristic:interestingCharacteristic]; 当你尝试读取特征值时，外设调用它委托对象的 peripheral:didUpdateValueForCharacteristic:error: 方法获取值。如果获取成功，可以通过特征值的属性来访问这些值，如下所示：  
	- (void)peripheral:(CBPeripheral *)peripheral	didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic             error:(NSError *)error {    NSData *data = characteristic.value;    // parse the data as needed    ...  
  
`注意`：并非所有的特征都有可读的值。你可以通过访问 CBCharacteristicPropertyRead 来查明特征值是否可读，详情参阅 CBCharacteristic Class Reference。如果你尝试读取一个不可读的值，peripheral:didUpdateValueForCharacteristic:error: 会适当的返回一个错误。  
  
### 订阅特征值  
虽然使用 readValueForCharacteristic: 方法读取特征值能够高效满足大多数应用场景，但它不是最高效的获取值改变的方式。大多数特征值改变的时候--例如，任意时刻的心率--你应该通过订阅来获取特征值。当你订阅特征值，在值改变的时候，你能够收到外设的通知。    
你可以订阅感兴趣的特征值，通过调用 CBPeripheral 类的 setNotifyValue:forCharacteristic: 方法并指定第一个参数为 YES，如下所示：  
  
	[peripheral setNotifyValue:YES forCharacteristic:interestingCharacteristic];  
	  
当你尝试订阅（或取消订阅）一个特征值，外设会调用它委托对象的 peripheral:didUpdateNotificationStateForCharacteristic:error: 方法。如果订阅请求失败，你可以完成该委托来访问失败原因，如下所示：  
  
	- (void)peripheral:(CBPeripheral *)peripheral	didUpdateNotificationStateForCharacteristic:(CBCharacteristic *)characteristic             error:(NSError *)error {    if (error) {        NSLog(@"Error changing notification state: %@",           [error localizedDescription]);    ...  
`注意`：并非所有的特征都允许你订阅他们的值。你可以查明一个特征是否配置为可订阅的，通过访问 Characteristic Properties 枚举，详情参阅CBCharacteristic Class Reference。  
在成功订阅特征值之后，在值发生改变的时候外设会给应用发通知。每次值的改变，外设都会调用它委托对象的peripheral:didUpdateValueForCharacteristic:error: 方法。要获取更新后的值，你需要实现该委托方法。  
## 写特征值  
在很多用户场景下，需要用到写特征值。例如，如果你的应用与一个基于 BLE 的数字温度调节器交互，你需要提供一个值让它设定室温。如果特征值是可写的，你可以给它写入一些数据（NSData类型），通过调用 CBPeripheral 类的 writeValue:forCharacteristic:type: 方法，如下所示：  
  
	NSLog(@"Writing value for characteristic %@", interestingCharacteristic); [peripheral writeValue:dataToWrite forCharacteristic:interestingCharacteristic     type:CBCharacteristicWriteWithResponse];  
当尝试写一个特征值，你需要指定你需要完成的写类型。上面的例子指定的写类型是 CBCharacteristicWriteWithResponse，意味着外设会让你的应用知道写操作是否成功。Core Bluetooth 框架支持的更多写类型信息，参见 CBPeripheral Class Reference 的枚举 CBCharacteristicWriteType。  
如果指定了CBCharacteristicWriteWithResponse，外设通过调用委托对象的peripheral:didWriteValueForCharacteristic:error: 方法来响应一个写请求。如果写失败，你可以实现该委托方法来获取错误原因，如下所示：  
  
	- (void)peripheral:(CBPeripheral *)peripheral     didWriteValueForCharacteristic:(CBCharacteristic *)characteristic               error:(NSError *)error {	if (error) {          NSLog(@"Error writing characteristic value: %@",              [error localizedDescription]);	} ...  
  
注意：特征可能仅仅允许特定类型的写操作改变它的值。你可以查明特征值允许哪些写类型，通过访问 Characteristic Properties 的相关属性，详情参考  CBCharacteristic Class Reference。  
  
# 外设通常要完成的任务  
在上一章节，我们学习了如何完成 BLE 主机端的通用任务。在本章，我们学习如何完成 BLE 外设端的通用任务。基于代码的示例可以帮助你用本地设备扮演外设的角色。具体来说你可以学习到：  
  
* 启动外设管理对象  
* 启动外设的服务和特征  
* 发布服务和特征到设备的本地数据库  
* 广播服务  
* 响应已连接的主机的读写请求。   
* 发送更新的特征值给订阅了的主机  
  
你会发现本章的例子很简单和抽象，在你实际的应用中需要对其做适当的修改来完善。在本地设备完成外设功能的许多高级主题--包括提示，陷阱，最佳实践--在后面的章节中会覆盖到。  
  
## 启动外设管理  
用本地设备扮演外设角色的第一步就是分配和初始化一个外设管理实例（用 CBPeriphralManager 对象表示）。通过 调用 CBPeriphralManager 类的 initWithDelegate:queue:options: 方法启动外设管理，如下所示：  
  
	myPeripheralManager =	[[CBPeripheralManager alloc] initWithDelegate:self queue:nil options:nil];  
  
在这个例子中，self 被设置为委托对象来接受外设的事件。当你指定 dispatch queue 为 nil，外设管理对象分发外设事件到主队列。    
当你创建一个外设管理者，外设管理者调用 peripheralManagerDidUpdateState: 委托方法。你可以实现该委托方法来确认 BLE 是否在本地外设上可用。更多实现该委托方法的详情参考 CBPeripheralManagerDelegate Protocol Reference .  
  
## 设置服务和特征  
如图 1-7，本地外设的服务和特征数据库是树形结构。你必须以类似的树形结构来设置服务和特征。完成这些任务的第一步是弄懂服务和特征时如何被定义的。  
  
### 服务和特征用 UUID 标识  
外设的服务和特征用128位的 UUID 标识，在 Core Bluetooth 框架中使用 CBUUID 对象。然而不是所有服务和特征的UUID都被蓝牙 Special Interest Group（SIG）预定义出来了。蓝牙 SIG 已经预定义了大量 UUID 为短 16 位以方便使用。例如 ，SIG 已经预定义存储为 16 位的 UUID 180D 标识心率服务。这个 UUID 等同于 128 位的 UUID：0000180D-0000-1000-8000-00805F9B34FB，预定义在蓝牙 4.0 规范，第 3 卷，3.2.1 节。    
CBUUID 类提供了便捷的工厂方法来处理 UUID。例如，在代码中你无须传递心率服务的 128 位 UUID 字符，而是简单的使用 CBUUID 的 UUIDWithString 方法。如下所示：  
  
	CBUUID *heartRateServiceUUID = [CBUUID UUIDWithString: @"180D"];  
	  
当你用一个预定义的 16 位 UUID 使用 CBUUID 对象,Core Bluetooth 会填充其余的位。  
  
### 为自定义的服务和特征创建自定义 UUID  
你可以拥有非预定义的服务和特征。前提是你需要生成的你 128 位 UUID 来标识他们。  
使用命令行工具的 uuidgen 指令可以很方便的生成 128 位的 UUID。第一步，打开一个终端窗口，下一步，在终端输入 uuidgen，就可以得到一个唯一的 128 位ASCII 字符串用连字符（-）分隔的，下面是一个示例：  
  
	$ uuidgen	71DA3FD1-7E10-41C1-B16F-4430B506CDE7  
你可以使用这个 UUID 创建一个 CBUUID 对象，使用 UUIDWithString 方法，如下所示：  
	CBUUID *myCustomServiceUUID =    [CBUUID UUIDWithString:@"71DA3FD1-7E10-41C1-B16F-4430B506CDE7"];  
### 构建你的服务和特征树  
在你拥有自定义的 UUID 后，你可以建立可变的服务和特征，然后把他们像树形结构组织在一起。例如，如果你拥有一个特征的 UUID， 你可以建立一个可变的特征，通过调用 CBMutableCharacteristic 类的initWithType:properties:value:permissions: 方法，如下所示：  
  
	myCharacteristic =    [[CBMutableCharacteristic alloc] initWithType:myCharacteristicUUID     properties:CBCharacteristicPropertyRead     value:myValue permissions:CBAttributePermissionsReadable];  
当你创建一个可变的特征，设置它的属性，值，权限。你决定特征的属性和权限，是否特征值是可读写的，连接的主机是否能否订阅特征值。在这个例子中，特征值被设置为可被主机读的。关于支持可变特征的属性和权限的更多详情请参阅 CBMutableCharacteristic Class Reference 。  
`注意`：如果你指定特征值，它的值被缓存，属性和权限被设置为可读的。如果你需要特征值可以被写，或者你希望特征所属的已发布服务的生命周期内特征值可以改变，需要设置值为 nil。遵循这个原则可以确保值无论什么时候外设管理对象收到主机的读写请求，值都能被动态处理。  
到目前为止，你已经创建了一个可变的特征，你可以创建一个与特征关联的可变的服务。通过调用 CBMutableService 类的 initWithType:primary: 方法，如下所示：  
	myService = [[CBMutableService alloc] initWithType:myServiceUUID primary:YES];  
在该例中，第二个参数设置为 YES，意思是服务是主服务（相对次要服务而言的）。`主服务`描述设备的中主要功能，能被别的服务包含或引用。`次要服务`描述它引用的另外一个服务的上下文。例如，心率监控器主服务可以暴露心率传感器心率数据，而次要服务可以暴露传感器的电池数据。    
创建按服务之后，你可以通过设置服务的特征数组来关联一个特征，如下所示：  
	myService.characteristics = @[myCharacteristic];  
## 发布服务和特征  
在构建服务和特征树之后，外设下一步要完成的是发布他们到设备的服务和特征数据库。使用 Core Bluetooth 很容易就能完成该任务。调用 CBPeripheralManager 类的 addService: 方法，如下所示：  
  
	[myPeripheralManager addService:myService];  
  
当你调用这个方法来发布你的服务，外设管理对象调用委托方法 peripheralManager:didAddService:error:。如果有错误发生，服务将无法发布，可以实现该委托方法来获取错误原因，如下所示：  
  
	- (void)peripheralManager:(CBPeripheralManager *)peripheral            didAddService:(CBService *)service                    error:(NSError *)error {    if (error) {        NSLog(@"Error publishing service: %@", [error localizedDescription]);	} ...  
  
  
`注意`：在你发布服务和任何跟它关联的特征到外设的数据库之后，服务就会被存储并且不能对它再做修改。  
  
## 广播服务  
当你发布服务和特征到设备的服务特征数据库之后，你已经做好了广播的准备工作，广播可以被正在侦听的主机接收。如下例子所示，可以通过调用 CBPeripheralManager 类的 startAdvertising: 方法广播服务，并且可以传递一个 NSDictionary 对象作为广播数据，如下所示：  
  
	[myPeripheralManager startAdvertising:@{ CBAdvertisementDataServiceUUIDsKey :    @[myFirstService.UUID, mySecondService.UUID] }];  
  
在这个例子中，字典对象唯一的键 CBAdvertisementDataServiceUUIDsKey，它的值是一个 CBUUID 对象的数组，代表你需要广播的服务的 UUID。在字典中你可以指定的键的详细描述在 CBCentralManagerDelegate Protocol Reference 的 Advertisement Data Retrieval Keys 部分。`外设管理对象仅支持两个键：CBAdvertisementDataLocalNameKey 和 CBAdvertisementDataServiceUUIDsKey`。  
  
当你广播外设的数据，外设管理对象调用委托方法 peripheralManagerDidStartAdvertising:error:。如果有错误发生，服务没有被广播出去，实现该委托方法获取错误原因，如下所示：  
  
	- (void)peripheralManagerDidStartAdvertising:(CBPeripheralManager *)peripheral                                       error:(NSError *)error {    if (error) {        NSLog(@"Error advertising: %@", [error localizedDescription]);	} ...  
注意：数据广播基于“尽力而为”的原则来完成。因为空间是有限的，可能同时有多个应用的广播。更多详情，参见CBPeripheralManager Class Reference 的 startAdvertising: 方法。    
广播行为也受应用后台的影响，这个主题在下一章讲述。    
  
一旦外设开始广播数据，远程主机就能发现和初始化到外设的连接。  
  
## 响应主机的读写请求  
在被一个或多个远程主机连接后，你可以开始接收来自主机的读写请求。请使用适当的方式响应这些请求。下面的例子描述了如何处理这些请求。    
当一个已连接的主机请求读特征值，外设管理对象会调用 peripheralManager:didReceiveReadRequest: 委托方法。该委托方法用 CBAttRequest 对象分发请求，CBAttRequest 对象包含一些属性可以用来完成请求。    
例如，当你收到一个要读特征值的简单请求，从委托方法中接收到的 CBATTRequest 对象属性能够用来保证你设备数据库中的特征与远程主机指定的读请求的特征相符合。你可以实现该委托方法，如下所示：  
  
	- (void)peripheralManager:(CBPeripheralManager *)peripheral    didReceiveReadRequest:(CBATTRequest *)request {    if ([request.characteristic.UUID isEqual:myCharacteristic.UUID]) {        ...  
如果特征的 UUID 匹配，下一步就是确保读请求的特征值的偏移没有越界。如下面的代码所示，你可以使用 CBATTRequest 对象的 offset 属性来确保读请求没有越界：  
	if (request.offset > myCharacteristic.value.length) {    [myPeripheralManager respondToRequest:request        withResult:CBATTErrorInvalidOffset];    return;	}  
假设请求的 offset 通过检验，现在可以设置请求特征属性值（默认是nil）为你本地外设的特征值，同时需要考虑读请求的 offset：  
	request.value = [myCharacteristic.value subdataWithRange:NSMakeRange(request.offset, myCharacteristic.value.length - request.offset)];  
  
在你设置值之后，响应远程主机告知请求已经成功完成。通过调用 CBPeripheralManager 类的  respondToRequest:withResult: 方法，传回请求（已经更新值的请求）和请求的结果，如下所示：  
  
	[myPeripheralManager respondToRequest:request withResult:CBATTErrorSuccess];	...  
  
每次 peripheralManager:didReceiveReadRequest: 委托方法被调用时都调用 respondToRequest:withResult: 方法。  
  
`注意`:如果特征的 UUID 不匹配，或者读操作不能完成，你不应该尝试填充请求对象。你应该立即调用 respondToRequest:withResult: 方法并提供一个标识失败原因的结果。你可以指定的可能的结果列表，参见  Core Bluetooth Constants Reference 的 CBATTError Constants 枚举。  
  
处理已连接主机的写请求同样简单。当一个已连接主机发送写请求，外设管理对象调用 peripheralManager:didReceiveWriteRequests: 委托方法。这一次，委托方法用一个包含 CBATTRequest 对象的数组分发请求，数组每一个元素表示一个写请求。在你确认写请求完成后，你可以如下所示写特征值：  
  
	myCharacteristic.value = request.value;  
  
虽然上面没有完整的示范，但在写特征值时也要考虑请求的 offset 属性。    
  
像响应读请求一样，peripheralManager:didReceiveWriteRequests: 调用的时候都调用 respondToRequest:withResult: 方法。意思是说，尽管你从 peripheralManager:didReceiveWriteRequests: 方法中接收了一个包含 CBATTRequest 对象的数组，单respondToRequest:withResult: 第一个参数需要填一个 CBATTRequest 对象。你需要像如下代码所示传递数组的第一个请求：  
  
	[myPeripheralManager respondToRequest:[requests objectAtIndex:0]    withResult:CBATTErrorSuccess];  
`注意`：多个请求与单个请求的处理方式一致--如果多个请求的任一的请求不能被完成，你就不应该完成所有请求。取而代之的是立即调用 respondToRequest:withResult: 方法并提供一个描述失败原因的结果。  
## 发送更新特征到订阅的主机  
通常，连接的主机会订阅一个或多个特征值。这样在订阅的特征值改变的时候可以发送通知给主机。下面的示例描述如何实现.    
当一个连接的主机订阅特征值，外设管理对象调用peripheralManager:central:didSubscribeToCharacteristic: 委托方法。  
  
	- (void)peripheralManager:(CBPeripheralManager *)peripheral                  central:(CBCentral *)central	didSubscribeToCharacteristic:(CBCharacteristic *)characteristic {    NSLog(@"Central subscribed to characteristic %@", characteristic);    ...  
用上面的委托方法作为给主机发送更新值的开始步骤。  
接下来，获取更新的特征值然后发送给主机，通过调用 CBPeripheralManager 类的 updateValue:forCharacteristic:onSubscribedCentrals: 方法：  
	NSData *updatedValue = // fetch the characteristic's new value	BOOL didSendValue = [myPeripheralManager updateValue:updatedValue    forCharacteristic:characteristic onSubscribedCentrals:nil];当你调用上面的方法给订阅的主机发送特征值，你可以在最后一个参数指定你想更新的主机。在上面的例子中，如果你指定 nil，所有连接并且订阅的主机都会更新（仅仅连接但没有订阅的主机会被忽略）。    
updateValue:forCharacteristic:onSubscribedCentrals: 方法返回一个布尔值表明更新值是否正确发送到订阅的主机。如果用来传输更新值的底层队列已经满了，该方法返回 NO。在有传输队列可用时外设管理对象就会调用peripheralManagerIsReadyToUpdateSubscribers: 方法。你可以实现该委托方法重新发送，通过调用 updateValue:forCharacteristic:onSubscribedCentrals: 方法。    
`注意`：使用通知发送一个简单的数据包给订阅的主机。当你更新一个订阅的主机，你应该通过调用 updateValue:forCharacteristic:onSubscribedCentrals: 在通知中一次性发送全部更新的值。根据特征值的大小，并非全部的数据会被通知传输。如果发生了这种情况，应该在主机端通过调用 readValueForCharacteristic: 方法获取全部值。  
# iOS 应用 Core Bluetooth 后台处理   
在 iOS 应用中，搞清楚应用是运行在前台还是后台是非常重要的。应用后台和前台必须有不同的行为，因为 iOS 设备的资源是很有限的。完整的 iOS 多任务讨论，参见 “App States and Multitasking” in iOS App Programming Guide .    
默认情况下，Core Bluetooth 许多常用的任务--不论是主机端还是外设端--在后台或挂起状态下都是不可用的。你可以声明应用支持 Core Bluetooth 后台执行模式允许你的应用从挂起状态唤醒来处理特定的蓝牙相关事件。即使你的应用不支持全范围的后台处理，仍然可以要求系统在有重要事件发生时提示应用。    
即使你的应用支持 Core Bluetooth 后台执行模式，它也不会永远运行。在某些时候，系统会因为给当前位于的前台应用释放内存而终结你的应用--这将导致任何当前活动或者挂起的连接丢失。 自 iOS 7，Core Bluetooth 支持给主机端和外设端保存状态信息，也支持在应用启动时恢复状态信息。你可以在需要长期使用蓝牙设备的应用中使用该特征。  
  
## 仅支持前台的应用  
大多数 iOS 应用，除非你要求完成指定的后台任务，你的应用在转到后台状态后很快就会进入挂起状态。在挂起状态下，应用不能完成蓝牙相关任务，也不能感知到任何蓝牙相关事件。    
  
在主机端，仅支持前台的应用--即没有声明支持 Core Bluetooth 后台执行模式的应用--在后台状态或者挂起状态不能扫描和发现任何广播的设备。在外设端，仅支持前台的应用无法在后台状态或者挂起状态下广播，并且主机对已发布的特征值的访问也会接收到错误。    
  
依据使用方式不同，默认的行为也会以多种方式影响你的应用。例如，假设你正在跟一个已连接的外设交互数据，现在假设你的应用状态挂起状态（比如，由于用户打开了另外一个应用），当你的应用在挂起状态，如果到外设的连接丢失，你不会感知到任何断连的发生，直到你的应用重新调度到前台状态。  
  
### 利用外设连接选项  
在仅支持前台运行的应用处于挂起状态的时候，所有蓝牙相关的事件都被系统放在队列里面，直到应用调度为前台状态后才会分发给应用。也就是说，主机端特定的事件发生时，Core Bluetooth 提供了一个方式提示用户。    
你可以在连接远程外设的时候，使用下面的外设连接选项来调用 connectPeripheral:options: 方法来提示用户。  
  
* `CBConnectPeripheralOptionNotifyOnConnectionKey`—在成功连接之后，如果你希望应用在挂起状态下系统还能显示提示信息，请包含该key。* `CBConnectPeripheralOptionNotifyOnDisconnectionKey`—如果在断连的时候应用处于挂起状态，你希望系统显示指定外设的断连提示信息，请包含该key。* `CBConnectPeripheralOptionNotifyOnNotificationKey`—如果应用处于挂起状态，你希望显示从指定外设接收到的所有通知信息，请包含该key。    
更多关于外设连接选项的详情，参阅 CBCentralManager Class Reference 的 Peripheral Connection Options constants。  
## Core Bluetooth 后台执行模式  
如果应用需要在后台模式下完成某些蓝牙相关的任务，应用必须在其信息属性列表文件中（Info.plist）声明为 Core Bluetooth 后台执行模式。当你声明了后台模式，系统在将应用从挂起状态唤醒为前台状态来处理蓝牙相关的事件。这个功能对定期与 BLE 设备交互的应用来说非常重要，比如心率监控器。  有两种 Core Bluetooth 后台执行模式可以声明--一个用于完成主机端功能，一个用于完成外设端功能。如果应用要完成两个角色，需要同时声明两个后台执行模式。通过设置 `Info.plist` 文件的 `UIBackgroundModes` 键来声明Core Bluetooth 后台执行模式，键的值是一个数组，包含如下字符串：  
* `bluetooth-central` 应用与 BLE 外设通信（即应用作为主机端）。  
* `bluetooth-peripheral` 应用共享数据（即应用作为外设端）。  
  
`注意`：在 XCode 中，属性列表编辑器默认展示键位可读的字符串而不是实际的键名称。右键菜单选择 Show Raw Keys/Values 可以显示实际的键名称。  
  
更多如何配置 Info.plist 文件的详情，参阅 Property List Editor Help。  
  
### 蓝牙主机后台执行模式  
如果应用用来实现主机功能，在 Info.plist 文件中包含了 UIBackgroundModes 键，设置其值为 bluetooth-central，Core Bluetooth 框架允许应用在后台运行来处理特定的蓝牙相关的任务。当你的应用位于后台，你仍然可以发现和连接外设，探索外设并于外设交互数据。另外，系统会在 CBCentralManagerDelegate 或者 CBPeripheralDelegate 委托方法被调用时唤醒应用，允许你的应用处理重要的主机端的事件，比如连接建立或关闭，外设发过来特征值的更新，主机管理对象状态改变。    
虽然应用在后台时你也可以完成许多蓝牙相关的任务，但要记住，应用在后台时扫描外设与前台时扫描外设时很不一样的。在你应用处于后台时，扫描设备需要特别注意如下事项：  
  
* `CBCentralManagerScanOptionAllowDuplicatesKey 扫描选项键会被忽略，并且多个广播外设的发现会被合并成一个发现事件。`  
* `如果所有的应用都在后台扫描外设，你主机扫描广播外设的间隔时长会增加。这会导致发现广播设备的时间变长。`  
  
这些与前台不同的地方帮助我们最小化无线使用，增加 iOS 设备电池使用寿命。  
  
### 蓝牙外设后台执行模式  
如果应用用来实现外设功能，在 Inof.plist 文件中包含 UIBackgroundModes 键，设置其值为 bluetooth-peripheral。如果应用中包含了该键值对，系统会唤醒你的应用来处理读，写，订阅事件。    
除了允许你的应用唤醒来处理已连接主机的读，写，订阅请求外，Core Bluetooth 框架还允许你的应用在后台时发广播。但需要注意，应用在后台时广播和在前台时广播是不一样的，当应用后台广播时，需要注意如下事项：  
  
* `CBAdvertisementDataLocalNameKey 广播键被忽略掉，外设的本地名称不会被广播。`  
* `所有包含 CBAdvertisementDataServiceUUIDsKey 广播键的服务被放在一个特殊的“溢出”区，他们仅能被希望扫描他们的 iOS 设备发现。`  
* `如果所有广播的应用都位于后台，外设发送广播包的频率会降低。`  
  
## 小心谨慎的使用后台执行模式  
虽然在某些特定的用户场景下，声明应用支持后台执行模式是很有必要的，但你应该总是为后台处理负责。因为完成许多蓝牙相关的任务都需要使用设备主板上的无线电，而无线电的使用会严重影响电池使用寿命，所以要尽量少的使用后台。被唤醒的应用应该尽可能快得完成蓝牙相关事件的处理，然后再次进入挂起状态。    
  
声明为蓝牙后台执行模式的应用需要遵循如下的基本准则：  
  
* `应用必须提供一个基于会话的接口允许用户决定开始和停止分发蓝牙相关的事件。`  
* `一旦被唤醒，应用有大概10秒钟的时间来完成任务。理想情况下，应用需要尽可能快的完成任务然后进入挂起状态。应用在后台花太多时间可能会被系统杀掉。`  
* `应用不应该因为不相关的任务被系统唤醒来完成不相关的任务（实在翻译不出来它的本意，估摸着是这个意思吧）。`  
   
关于应用在后台状态下应有的更多详情，参见“Being a Responsible Background App” in iOS App Programming Guide 。  
  
## 在后台完成长时间的任务  
一些应用需要在后台状态下使用 Core Bluetooth 完成长期的任务。例如，假设你在开发一个管理 BLE 的智能门锁的家庭安全应用。当用户离开家的时候，门自动落锁，当用户回家的时候，门自动开锁--整个过程中，应用一直在后台。当用户离开家，iOS 设备会超出锁的连接范围，导致到门锁的连接断连。这个时候，应用能能够调用 CBCentralManager 类的 connectPeripheral:options: 方法，并且因为连接请求不会超时，iOS 设备会在用户回家的时候重连。    
现在假设用户离开家一些天。如果应用在用户离开的时候被系统终止了，当用户回家时应用就不能够重连门锁了，用户就开不了门锁了。类似的这些应用，需要持续使用 Core Bluetooth 实现长期任务，例如监控活动，挂起连接等。  
  
### 状态保存和恢复  
因为 Core Bluetooth 内建状态保存和恢复机制，你的应用可以选择性的要求系统代表应用的主机或外设管理对象完成他们特定的蓝牙相关的任务，即使应用没有在运行。当任务完成后，系统重启你的应用到后台然后给应用机会恢复应用的状态并合理的处理事件。在上面描述到的家庭安全应用中，系统可以在用户回家，连接请求完成后监控连接请求，重启应用来处理 centralManager:didConnectPeripheral: 回调。  
  
Core Bluetooth 既支持主机端，也支持外设端的状态保存和恢复。当你的应用作为主机端，添加了状态保存和恢复，系统会在终结你应用时保存你的主机管理对象状态（如果应用有多个主机管理对象，你可以选择其中的一个来让系统跟踪它）。对于一个给定的 CBCentralManager 对象，系统跟踪如下信息：  
  
* `主机管理对象扫描的服务（还有扫描开始后指定的任何扫描选项）。`  
* `主机正在连接正在尝试连接的外设和已经连接的外设。`  
* `主机订阅的特征。`  
  
应用作为外设端利用状态保存和恢复跟主机端类似。对于一个 CBPeripheralManager 对象，系统跟踪如下信息：  
  
* `正在广播的外设管理对象的数据。`  
* `外设发布到设备数据库的服务和特征。`  
* `订阅特征值的主机。`  
  
当应用被系统重启到后台模式（比如因为应用正在扫描的设备被发现），你可以重新实例化应用的主机和外设管理对象，恢复他们的状态。下面的一节详细描述如何利用状态存储和恢复。  
  
### 添加状态保存和恢复支持  
状态保存和恢复在 Core Bluetooth 中是可选的特性。你可以按照下面的处理添加保存和恢复功能：  
  
1. (必要）选择状态保存和恢复  
2. (必要）重新实例化主机和外设管理对象  
3. (必要) 实现相应的恢复委托方法  
4. (可选) 更新初始化处理  
  
#### 选择状态保存和恢复  
要实现状态保存和恢复，在初始化主机或者外设的管理对象时简单的提供一个恢复唯一标识符就可以了。`恢复标识符`是一个`表示主机或者外设管理对象的字符串`。字符串的值只能被你的代码鉴别，这个字符串的存在告诉 Core Bluetooth 需要保存标记对象的状态。Core Bluetooth 只会保存拥有恢复标识符的对象状态。    
  
例如，在主机端，如果使用 CBCentralManager 对象的一个实例来完成状态保存和恢复，需要在分配和初始化主机管理对象的时候，指定 CBCentralManagerOptionRestoreIdentifierKey 并提供一个恢复标识符，如下所示：  
  
	myCentralManager =    [[CBCentralManager alloc] initWithDelegate:self queue:nil     options:@{ CBCentralManagerOptionRestoreIdentifierKey:     @"myCentralManagerIdentifier" }];  
尽管上面的例子是关于主机的状态保存和恢复，但外设管理对象也是类似的实现方式。 在分配和初始化外设管理对象的时候，指定 CBPeripheralManagerOptionRestoreIdentifierKey 并提供一个恢复标识符。  
`注意`：因为应用可以有多个 CBCentralManager 实例和多个 CBPeripheralManager 实例对象，要确保恢复标识符是唯一的，那样系统才能正确区分不同的主机和外设管理对象。  
#### 重新实例化化主机和外设管理对象  
当应用被系统重启到后台，你要做的第一件事就是依据恢复标识符对相应的主机和外设重新实例化他们的状态。如果你的应用仅有一个主机或者外设管理对象，并且这个管理对象在应用的生命周期内都存在，那么这个步骤你不用做任何处理。  
  
如果你的应用不止使用一个主机或外设管理对象，或者如果你只使用一个管理对象但是它在应用的生命周期内不是一直存在，那么应用在被系统重启后需要知道重新实例化哪一个管理对象。你可以访问系统在你的应用终止时为你保存的一个包含所有恢复标识符的列表，通过在实现 application:didFinishLaunchingWithOptions: 委托方法时使用启动选项键（UIApplicationLaunchOptionsBluetoothCentralsKey or UIApplicationLaunchOptionsBluetoothPeripheralsKey）。  
  
例如，应用重启时，你可以获取系统为你应用保存的所有的主机管理对象的恢复标识符，如下所示：  
  
	- (BOOL)application:(UIApplication *)application	didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {    NSArray *centralManagerIdentifiers =        launchOptions[UIApplicationLaunchOptionsBluetoothCentralsKey];	...  
在你获得恢复标识符列表后，可以循环遍历然后重新实例化主机管理对象。    
  
`注意`：当应用重启，系统只会为正在完成某些蓝牙相关任务的主机和外设管理对象提供恢复标识符。  
  
#### 实现相应的恢复委托方法  
重新实例化相应的主机和外设管理对象后，用蓝牙系统的状态同步管理对象的状态并存储下来。为了在系统帮应用处理的任务的基础上（应用没有运行的时候系统代做的处理）加快应用处理，你需要实现一些恢复的委托方法。对于一个主机管理对象，需要实现 centralManager:willRestoreState: 委托方法，对已一个外设管理对象，需要实现 peripheralManager:willRestoreState: 委托方法。  
  
`重要`：***对于选择保存和恢复状态的应用，centralManager:willRestoreState: 和 peripheralManager:willRestoreState: 是应用重启到后台状态完成某些蓝牙相关任务的第一个方法。对于不选择保存和恢复状态的应用（或者启动时不存储任何东西的应用），centralManagerDidUpdateState: 和 peripheralManagerDidUpdateState: delegate 是应用重启后调用的第一个方法。***  
  
在上面的委托方法中，最后一个参数是一个字典，包含应用终止时保存的管理对象的信息。    
恢复 CBCentralManager 对象的状态，使用 centralManager:willRestoreState: 委托方法里提供的字典的键。举个例子，如果在应用终止时主机管理对象拥有任何活动的或挂起的连接，系统会代表应用继续监控他们。如下所示，你可以使用 CBCentralManagerRestoredStatePeripheralsKey 来获取主机管理对象已经连接或者正在尝试连接的所有外设的列表：  
  
	- (void)centralManager:(CBCentralManager *)central        willRestoreState:(NSDictionary *)state {      NSArray *peripherals =	state[CBCentralManagerRestoredStatePeripheralsKey];      ...  
如何处理上面的列表依赖于你的使用场景。例如，如果应用拥有主机管理对象已经发现的的外设列表，你可能想添加已存储的外设到列表中来保持他们的引用。要确保设置了外设的委托方法，以确保能接收想要的回调。    
你可以在 peripheralManager:willRestoreState: 委托方法中类似的存储 CBPeripheralManager 对象。  
#### 更新初始化处理  
在完成必要的三个步骤后，你可能想要更新主机和外设的管理对象的初始化处理。尽管这是一个可选的步骤，但对于应用的流畅运行是很重要的。举个例子，你的应用可能在探索已连接外设的数据的过程中被终止了，这个时候应用已经被这个外设存储，应用并不知道它被终止时发现处理完成了多少。你可能想在你在发现处理时被终止的那个点继续。   
例如，在 centralManagerDidUpdateState: 委托方法中初始化应用时，可以找出已恢复的外设（在应用终止前），如下所示：  
  
	NSUInteger serviceUUIDIndex =    [peripheral.services indexOfObjectPassingTest:^BOOL(CBService *obj,    NSUInteger index, BOOL *stop) {        return [obj.UUID isEqual:myServiceUUIDString];    }];	if (serviceUUIDIndex == NSNotFound) {    [peripheral discoverServices:@[myServiceUUIDString]];    ...如上面的例子所示，如果系统在完成发现服务之前终止应用，在这个时候通过调用 discoverServices 方法开始探索恢复的外设数据。如果应用成功发现了服务，你可以检查服务的特征（以及是否已经订阅他们）。通过这种方式更新初始化处理，你可以确保你在正确的时间调用正确的方法。  
# 与远程外设交互的最佳实践  
Core Bluetooth 框架提供了很多到主机端应用的透明传输。应用有控制权，并负责实现大多数主机端任务，例如设备发现和连接，探索，以及与远程外设交互数据。本章为你在这个层面开发 iOS 应用提供负责任的指导和最佳实践。  
  
## 重视无线电使用和电池消耗  
开发与 BLE 设备交互的应用时，记住 BLE 通信贡献设备无线电来传输信号。因为其他形式的无线通讯也需要使用设备的无线电--比如WIFI，经典蓝牙（应该是说MFI蓝牙之类吧），还有其他使用 BLE 蓝牙的应用--开发应用时要最小化无线电的使用。    
开发应用时最小化无线电使用是非常重要的，因为无线电使用对电池使用寿命影响很严重。下面的指导能帮助你很好的使用设备无线电。这样你的应用会更优美，设备电池也能使用更久。  
  
### 只有需要时才扫描设备  
当调用 CBCentralManager 类的 scanForPeripheralsWithServices:options: 方法发现正在广播服务的远程外设时，你的主机设备使用无线电来侦听广播设备直到你明确告诉它要停止。  
除非你要发现更多的设备，否则在发现一个你想要连接的设备后停止扫描其它设备。使用 CBCenteralManager 类的stopScan 方法停止扫描其它设备。  

### 只有需要时才指定 CBCentralManagerScanOptionAllowDuplicatesKey 选项  
远程外设可能每秒发送多个广播包来向侦听的主机宣告它的存在。当使用 scanForPeripheralsWithServices:options: 方法扫描时，`默认的行为是将一个广播外设的多个发现合并成一个发现事件`--意思是说，主机管理对象为每个发现的新外设调用 centralManager:didDiscoverPeripheral:advertisementData:RSSI: 方法，而不管它接到了多少广播包。已经发现的外设在更改它的广播包的时候，主机也会调用这个委托方法。  
如果你要改变默认的行为，你可以在调用 scanForPeripheralsWithServices:options: 时指定 CBCentralManagerScanOptionAllowDuplicatesKey 常量作为扫描选项。这样，主机每从外设收到一个广播包都回产生一个发现事件。关闭默认的行为在某些用用场景下是很有用的，例如初始化一个基于外设接近度的连接（使用外设接收信号强度指示器的值 RSSI）。需要重视，指定该选项严重影响电池使用寿命，所以只有在确实需要完成类似的应用场景时才指定该选项。  

### 明智的探索外设数据  
外设的服务和特征可能比应用感兴趣的多。发现外设所有的服务和特征对电池和应用体验都由负面的影响。所有你应该值查找和发现和你应用相关的服务和特征。  
例如，假设你已经连接到一个外设，外设有很多服务可用，但你的应用仅仅需要访问他们中间的两个。你可以仅仅查找和发现这两个服务，通过调用 CBPeripheral 类 discoverServices: 方法，并传递一个包含这两个服务的 UUID 的数组，如下所示：

	[peripheral discoverServices:@[firstServiceUUID, secondServiceUUID]];
	
在你发现这两个你感兴趣的服务之后，你可以类似的查找和发现你感兴趣这两个服务的特征。再次简单的传递一个代表你想发现的特征的 UUID 数组，调用 CBPeripheral 类的 discoverCharacteristics:forService: 方法。  

### 订阅经常变化的特征值  
如之前描述的，有两种方法获取特征值：

* 你可以明确的轮询特征值，在每次你需要获取值的时候调用 readValueForCharacteristic: 方法。
* 你可以订阅特征值，通过调用 setNotifyValue:forCharacteristic: 方法，这样每次特征值改变都可以收到来自外设的一个通知。  

最佳做法尽可能的订阅特征值，特别是值经常改变的情况下。  

### 从设备获取你需要的所有数据后断开连接  
连接不再需要时，断开连接可以降低无线电的使用。下面两种情况下你应该断开设备连接：  

* `你订阅的所有特征值已经停止发送通知了。（你可以确认特征值是否是能通知，通过访问特征的 isNotifying 属性）`
* `你已经从外设获取了你需要的所有数据。`

上面任意一种情况下，取消订阅然后断开与外设的连接。通过调用 setNotifyValue:forCharacteristic: 方法取消订阅特征值。通过调用 CBCentralManager类的 cancelPeripheralConnection: 方法取消与外设的连接，如下所示：

	[myCentralManager cancelPeripheralConnection:peripheral];
	
`注意`: cancelPeripheralConnection 方法时非阻塞的，任何到外设的 CBPeripheral 类的尝试断连的命令都是未决的，可能完成也可能没完成。因为其他应用可能仍然拥有一个到外设的连接，取消一个本地连接不保证底层物理链路能立即断连。然而从应用的角度来看，外设是被认为断连的，主机管理对象会调用 centralManager:didDisconnectPeripheral:error: 委托方法。

## 重连外设
使用 Core Bluetooth 框架，有三种方式重连外设：

* `获取已知外设的列表`--之前发现或者连接的外设--使用 retrievePeripheralsWithIdentifiers: 方法。如果你查找的外设在里面，尝试连接它。连接选项在后面的描述中。
* `获取当前连接的外设列表`，使用 retrieveConnectedPeripheralsWithServices: 方法。如果你查找的外设在里面，尝试连接它。连接选项在后面的描述中。
* `扫描并发现外设`，使用  scanForPeripheralsWithServices:options: 方法。如果你发现了外设，连接它。连接选项在后面的描述中。

依赖于具体的使用场景，连接设备时你可能不需要每次都扫描和发现相同的设备。取而代之的是，你可以先尝试其他方式重连。如图 5-1 所示，一个重连工作流可能像下面列出的按顺序尝试每一个选项。  

`注意`：依据不同的使用场景，重连选项的选择和次序是有很大差异的。例如，你可能决定不使用第一个连接选项，也可能决定同时尝试前面两个选项。

### 获取已知外设列表
你第一次发现外设时，系统生成一个 UUID 代表这个外设。你可以存储这个标识（比如用 NSUserDefault），稍候可以使用它来尝试重连外设，通过调用 CBCentralManager 类的 retrievePeripheralsWithIdentifiers: 方法。下面的例子描述了一种重连以前连接过的外设的方法。  
当应用启动时，调用 call the retrievePeripheralsWithIdentifiers: 方法，传递包含外设标识符的数组，如下所示：

	knownPeripherals =    [myCentralManager retrievePeripheralsWithIdentifiers:savedIdentifiers];
主机管理对象尝试匹配你提供的标识符并返回一个包含 CBPeripheral 对象的数组。如果没有任何匹配，数组是空的，你应该尝试其他的重连选项。如果数组不为空，让用户选择（在 UI ）需要重连哪个外设。  
当用户选择一个外设，调用CBCentralManager 类的 connectPeripheral:options: 方法重连外设。如果外设任然可用于连接，主机调用 centralManager:didConnectPeripheral: 委托方法，外设就被成功连接了。  
`注意`：有些情况下外设不能被连接。例如，外设不在主机附近。另外，一些 BLE 设备使用周期性改变的随机设备地址，这样尽管设备在附近，它的地址从上次被系统发现以来发生了改变，上面任意一种情况下你尝试连接 CBPeripheral 对象都不会符合实际的外设。如果你因为外设地址改变而无法重新连接它，你需要重新发现外设，通过调用 scanForPeripheralsWithServices:options: 方法。  
### 获取已连接外设列表  
另外一种重连的方式时通过检查外设是否已经连接到系统了（比如被其他的应用连接了）。你可以通过调用 CBCentralManager 类的 retrieveConnectedPeripheralsWithServices: ，该方法返回一个 CBPeripheral 对象的数组，表示当前已连接的外设。  
因为当前可能不止一个外设连接到系统，你需要传递一个 CBUUID 的数组（服务的 UUID）来获取当前连接到系统并且包含了指定 UUID 服务的外设。如果当前没有外设连接到系统，数组为空。你需要尝试其他重连选项。如果数组不为空，让用户选择（在 UI）需要重连哪个威外设。  
假设用户发现和选择了他想要的外设，调用 CBCentralManager 类的 connectPeripheral:options: 方法在本地连接它到你的应用。（尽管设备已经连接到系统，你仍然需要在本地连接它到你的应用才能开始探索和交互数据）。本地连接建立后，主机管理对象调用 centralManager:didConnectPeripheral: 方法，外设就被成功重连了。

# 设置本地设备为外设端的最佳实践  
和许多主机端传输一样，Core Bluetooth 框架给你控制权以全方位的控制外设端。本章在你在这个层面开发应用提供负责任的指导和最佳实践。

## 广播注意事项  
设置外设完成外设端功能的时候，广播外设数据是一个重要部分。下面的内容帮助你以适当的方式完成广播。  

### 考虑广播数据的限制  
你通过传递给 CBPeripheralManager 类 startAdvertising: 方法一个包含广播数据的字典来广播外设数据。当你创建一个广播的字典，需要记住它们有什么限制，有哪些限制。  
虽然普通的广播包可以携带外设的大量信息，但你只能广播设备的本地名称和你想广播出去的服务的 UUID。也就是说，当你创建广播字典，`你只能广播下面的两个键：CBAdvertisementDataLocalNameKey 和 CBAdvertisementDataServiceUUIDsKey`。如果你指定其他任何 key 都会收到一个错误。  
还有一个限制是关于广播数据可以使用多少空间的。当应用位于前台时，这两个键的任意组合最多能在初始化的广播数据中使用 28 字节空间。如果这个空间用完了，那么可以有额外的 10 个字节仅供扫描响应时本地名称使用。任何不适合分配空间的服务 UUID 都被添加到一个特殊的“溢出”区。他们只能被 iOS 设备明确的扫描来被发现。如果应用位于后台，本地名称不会被广播，所有的服务 UUID 都被放在溢出区。  

`注意`：空间大小不包含请求的每个新数据类型的 2 个字节的头部。  

## 仅广播必要的数据  
因为广播外设数据需要使用本地设备的无线电（还有电池电量），只有在你需要其他设备连接你的时候才广播。一旦连接后，设备可以与外设直接探索和交互数据，不再需要广播。所以在 BLE 传输不必要广播的时候停止广播，可以最小化无线使用，增强用户体验，节省电池。在外设端停止广播，简单的调用 CBperipheralManager 类的 stopAdvertsing 方法，如下所示：

	[myPeripheralManager stopAdvertising];
	
### 让用户决定什么时候广播  
往往只有用户才知道什么时候需要广播。例如，你知道附近没有任何 BLE 设备时，广播数据没有任何意义。因为应用总是不知道附近有什么设备，所以提供一个 UI 接口让用户决定什么时候广播。  

## 配置特征  
当你创建一个可变的特征，你可以设置它的属性和权限。这些设置决定了主机如何访问特征值，如何与特征交互。根据应用需求来决定如何配置属性和权限，下面提供了一些指导：

* `允许所有已连接的主机订阅特征。`
* `对来自未配对主机的访问，保护敏感的特征值。`

### 配置特征以支持通知  
强烈建议主机订阅经常变化的外设的特征值。  
当你创建一个可变的特征，通过设置 CBCharacteristicPropertyNotify 常量来配置特征的属性支持订阅，如下所示：

	myCharacteristic = [[CBMutableCharacteristic alloc]    initWithType:myCharacteristicUUID    properties:CBCharacteristicPropertyRead | CBCharacteristicPropertyNotify    value:nil permissions:CBAttributePermissionsReadable];
在这个例子中，特征值时可读并且可被已连接的主机订阅。
### 需要配对连接才能访问敏感数据  
依赖具体的使用场景，你可能要保证服务的某些特征值的安全（这句翻不出来，大概是不能随便被别人获取的意思吧）。例如，假设你需要提供一个社交媒体个人服务。这个服务有很多特征，他们的值表示会员的个人资料，如姓名，邮箱地址等。你更可能只希望新人的设备才能获取会员的邮箱地址。  
你可以通过设置特征属性和权限来保证只有信任的设备才能访问敏感的特征值。继续上面的例子，允许信任的设备获取会员邮件地址，可以像如下所示来完成功能：
	emailCharacteristic = [[CBMutableCharacteristic alloc]    initWithType:emailCharacteristicUUID    properties:CBCharacteristicPropertyRead    | CBCharacteristicPropertyNotifyEncryptionRequired    value:nil permissions:CBAttributePermissionsReadEncryptionRequired];

在这个例子中，特征被配置为允许信任的设备读和订阅它的值。  
如果主机和外设是 iOS 设备，其他设备想要与它配对时，iOS 设备都会收到一个提示，主机设备的提示包含一个编码，你必须在外设端输入这个编码才能完成配对。  
在配对完成后，外设认为配对的主机是可以信任的并允许主机访问它加密的特征值。

# 译者结束语
利用空余时间断断续续几天翻译完了，错误和遗漏之处在所难免，如果你愿意，可以通过 [jjunjoegg@gmail.com](jjunjoegg@gmail.com) 告诉我，谢谢。后面计划实现一个蓝牙主机和外设的例子程序。
