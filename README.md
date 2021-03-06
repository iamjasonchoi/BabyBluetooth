
![](http://images.jumppo.com/uploads/BabyBluetooth_logo.png)

The easiest way to use Bluetooth (BLE )in ios,even bady can use. 简单易用的蓝牙库，基于CoreBluetooth的封装，并兼容ios和mac osx.


- 基于原生CoreBluetooth框架封装的轻量级的开源库，可以帮你更简单地使用CoreBluetooth API。
- CoreBluetooth所有方法都是通过委托完成，代码冗余且顺序凌乱。BabyBluetooth使用block方法，可以重新按照功能和顺序组织代码，并提供许多方法减少蓝牙开发过程中的代码量。
- 链式方法体，代码更简洁、优雅。
- 通过channel切换区分委托调用，并方便切换

当前版本v0.2

# [english readme link,please click it!](https://github.com/coolnameismy/BabyBluetooth/blob/master/README_en.md)

# Contents

* [用法示例](#用法示例)
    * [QuickExample](#user-content-QuickExample)
    * [初始化](#初始化)
    * [搜索设备](#搜索设备)
    * [搜索并连接设备](#搜索并连接设备)
    * [直接连接设备](#直接连接设备)
    * [断开连接和取消扫描](#断开连接和取消扫描)
    * [services_characteristic_description](#services_characteristic_description)
    * [获取一个characteristic的value和全部description及description的value](#获取一个characteristic的value和全部description及description的value)
    * [通知方式监听一个characteristic的值](#通知方式监听一个characteristic的值)
    * [取消通知](#取消通知)
* [委托设置](#委托设置)
  	* [示例](#示例)
  	* [支持的委托方法](#支持的委托方法)
  	* [委托方法的频道channel](#委托方法的频道channel)
  	* [默认频道](#默认频道)
* [链式方法](#链式方法) 
    * [谓词](#谓词)
    * [链式方法的顺序](#链式方法的顺序)
* [辅助方法](#辅助方法) 
    * [BabyRhythm](#user-content-BabyRhythm)
* [如何安装](#如何安装)
* [示例程序说明](#示例程序说明)
* [程序结构](#程序结构)
* [兼容性](#兼容性)
* [后期更新](#后期更新)
* [蓝牙学习资源](#蓝牙学习资源)
* [期待](#期待)

# 用法示例

## QuickExample
```objc

//导入.h文件和系统蓝牙库的头文件
#import "BabyBluetooth.h"

-(void)viewDidLoad {
    [super viewDidLoad];

    //初始化BabyBluetooth 蓝牙库
    baby = [BabyBluetooth shareBabyBluetooth];
    //设置蓝牙委托
    [self babyDelegate];
    //设置委托后直接可以使用，无需等待CBCentralManagerStatePoweredOn状态
    baby.scanForPeripherals().begin();
}

//设置蓝牙委托
-(void)babyDelegate{

    //设置扫描到设备的委托
    [baby setBlockOnDiscoverToPeripherals:^(CBCentralManager *central, CBPeripheral *peripheral, NSDictionary *advertisementData, NSNumber *RSSI) {
        NSLog(@"搜索到了设备:%@",peripheral.name);
    }];
   
    //过滤器
    //设置查找设备的过滤器
    [baby setDiscoverPeripheralsFilter:^BOOL(NSString *peripheralsFilter) {
        //设置查找规则是名称大于1 ， the search rule is peripheral.name length > 1
        if (peripheralsFilter.length >1) {
            return YES;
        }
        return NO;
    }];
    

}
  
```


## 初始化 

```objc
    //单例初始化 推荐
    BabyBluetooth *baby = [BabyBluetooth shareBabyBluetooth];
    //常规初始化
    BabyBluetooth *baby = [[BabyBluetooth alloc]init];
```

## 搜索设备

```objc

    //搜索设备   
    baby.scanForPeripherals().begin();
    
    //搜索设备10秒后停止   
    baby.scanForPeripherals().begin().stop(10);
    
    //设置搜索设备的过滤器
    [baby setFilterOnDiscoverPeripherals:^BOOL(NSString *peripheralName) {
        //设置查找规则是名称大于1 ， the search rule is peripheral.name length > 2
        if (peripheralName.length >2) {
            return YES;
        }
        return NO;
    }];

```

## 搜索并连接设备

```objc
    
    /*
    *搜索设备后连接设备：1:先设置连接的设备的filter 2进行连接
    */
    //1：设置连接的设备的过滤器
     __block BOOL isFirst = YES;
    [baby setFilterOnConnetToPeripherals:^BOOL(NSString *peripheralName) {
        //这里的规则是：连接第一个AAA打头的设备
        if(isFirst && [peripheralName hasPrefix:@"AAA"]){
            isFirst = NO;
            return YES;
        }
        return NO;
    }];

    //2 扫描、连接
    baby.scanForPeripherals().connectToPeripherals().begin()

```

## 直接连接设备

````objc
 baby.having(self.currPeripheral).connectToPeripherals().begin();
````

## 断开连接和取消扫描

````objc
  
  //断开单个peripheral的连接
  [baby cancelPeripheralConnection:peripheral];//peripheral是一个CBPeripheral的实例
  
  //断开所有peripheral的连
  [baby cancelAllPeripheralsConnection];
    
  //取消扫描
  [baby cancelScan];
  
````

## services_characteristic_description

获取设备的services、characteristic、description以及value
````objc
  //设置peripheral 然后读取services,然后读取characteristics名称和值和属性，获取characteristics对应的description的名称和值
  //self.peripheral是一个CBPeripheral实例
   baby.having(self.peripheral).connectToPeripherals().discoverServices().discoverCharacteristics()
   .readValueForCharacteristic().discoverDescriptorsForCharacteristic().readValueForDescriptors().begin();
   
````

## 获取一个characteristic的value和全部description及description的value

````objc
  //self.peripheral是一个CBPeripheral实例,self.characteristic是一个CBCharacteristic实例
  baby.characteristicDetails(self.peripheral,self.characteristic);
````

## 通知方式监听一个characteristic的值
````objc
      //self.peripheral是一个CBPeripheral实例,self.characteristic是一个CBCharacteristic实例
            [baby notify:self.currPeripheral
          characteristic:self.characteristic
                   block:^(CBPeripheral *peripheral, CBCharacteristic *characteristics, NSError *error) {
                 //接收到值会进入这个方法
                NSLog(@"new value %@",characteristics.value);
     }];
````

## 取消通知
````objc
  //self.peripheral是一个CBPeripheral实例,self.characteristic是一个CBCharacteristic实例
  [baby cancelNotify:self.peripheral characteristic:self.characteristic];
````


# 委托设置
> 每次调用方法后，都会进入到相应的委托方法。

## 示例

````objc

  //设置扫描到设备的委托
    [baby setBlockOnDiscoverToPeripherals:^(CBCentralManager *central, CBPeripheral *peripheral, NSDictionary *advertisementData, NSNumber *RSSI) {
        NSLog(@"搜索到了设备:%@",peripheral.name);
    }];
    //设置设备连接成功的委托
    [baby setBlockOnConnected:^(CBCentralManager *central, CBPeripheral *peripheral) {
        //设置连接成功的block
        NSLog(@"设备：%@--连接成功",peripheral.name);
    }];
    
````
设置委托方法后执行

````objc

baby.scanForPeripherals().begin();

````
每当发现perihperal和连接成功后就会进入相应的委托方法。


## 支持的委托方法

````objc

//====================================设置默认的委托方法======================================
//设备状态改变的委托
-(void)setBlockOnCentralManagerDidUpdateState:(void (^)(CBCentralManager *central))block;
//找到Peripherals的委托
-(void)setBlockOnDiscoverToPeripherals:(void (^)(CBCentralManager *central,CBPeripheral *peripheral,NSDictionary *advertisementData, NSNumber *RSSI))block;
//连接Peripherals成功的委托
-(void)setBlockOnConnected:(void (^)(CBCentralManager *central,CBPeripheral *peripheral))block;
//连接Peripherals失败的委托
-(void)setBlockOnFailToConnect:(void (^)(CBCentralManager *central,CBPeripheral *peripheral,NSError *error))block;
//断开Peripherals的连接
-(void)setBlockOnDisconnect:(void (^)(CBCentralManager *central,CBPeripheral *peripheral,NSError *error))block;
//设置查找服务回叫
-(void)setBlockOnDiscoverServices:(void (^)(CBPeripheral *peripheral,NSError *error))block;
//设置查找到Characteristics的block
-(void)setBlockOnDiscoverCharacteristics:(void (^)(CBPeripheral *peripheral,CBService *service,NSError *error))block;
//设置获取到最新Characteristics值的block
-(void)setBlockOnReadValueForCharacteristic:(void (^)(CBPeripheral *peripheral,CBCharacteristic *characteristic,NSError *error))block;
//设置查找到Descriptors名称的block
-(void)setBlockOnDiscoverDescriptorsForCharacteristic:(void (^)(CBPeripheral *peripheral,CBCharacteristic *characteristic,NSError *error))block;
//设置读取到Descriptors值的block
-(void)setBlockOnReadValueForDescriptors:(void (^)(CBPeripheral *peripheral,CBDescriptor *descriptorNSError,NSError *error))block;

//====================================设置对应频道的委托方法======================================
//channel
//设备状态改变的委托
-(void)setBlockOnCentralManagerDidUpdateStateAtChannel:(NSString *)channel
                                                        block:(void (^)(CBCentralManager *central))block;
//找到Peripherals的委托
-(void)setBlockOnDiscoverToPeripheralsAtChannel:(NSString *)channel
                                          block:(void (^)(CBCentralManager *central,CBPeripheral *peripheral,NSDictionary *advertisementData, NSNumber *RSSI))block;

//连接Peripherals成功的委托
-(void)setBlockOnConnectedAtChannel:(NSString *)channel
                              block:(void (^)(CBCentralManager *central,CBPeripheral *peripheral))block;

//连接Peripherals失败的委托
-(void)setBlockOnFailToConnectAtChannel:(NSString *)channel
                                       block:(void (^)(CBCentralManager *central,CBPeripheral *peripheral,NSError *error))block;

//断开Peripherals的连接
-(void)setBlockOnDisconnectAtChannel:(NSString *)channel
                                    block:(void (^)(CBCentralManager *central,CBPeripheral *peripheral,NSError *error))block;

//设置查找服务回叫
-(void)setBlockOnDiscoverServicesAtChannel:(NSString *)channel
                                     block:(void (^)(CBPeripheral *peripheral,NSError *error))block;

//设置查找到Characteristics的block
-(void)setBlockOnDiscoverCharacteristicsAtChannel:(NSString *)channel
                                            block:(void (^)(CBPeripheral *peripheral,CBService *service,NSError *error))block;
//设置获取到最新Characteristics值的block
-(void)setBlockOnReadValueForCharacteristicAtChannel:(NSString *)channel
                                               block:(void (^)(CBPeripheral *peripheral,CBCharacteristic *characteristic,NSError *error))block;
//设置查找到Characteristics描述的block
-(void)setBlockOnDiscoverDescriptorsForCharacteristicAtChannel:(NSString *)channel
                                                         block:(void (^)(CBPeripheral *peripheral,CBCharacteristic *service,NSError *error))block;
//设置读取到Characteristics描述的值的block
-(void)setBlockOnReadValueForDescriptorsAtChannel:(NSString *)channel
                                            block:(void (^)(CBPeripheral *peripheral,CBDescriptor *descriptorNSError,NSError *error))block;

//设置查找Peripherals的规则
-(void)setFilterOnDiscoverPeripheralsAtChannel:(NSString *)channel
                                      filter:(BOOL (^)(NSString *peripheralName))filter;

//设置连接Peripherals的规则
-(void)setFilterOnConnetToPeripheralsAtChannel:(NSString *)channel
                                     filter:(BOOL (^)(NSString *peripheralName))filter;
                                     


````

## 委托方法的频道channel
> 委托方法的设置可以通过channel区分开来.

对于项目中多个类需要使用蓝牙这种情况，常规做法是你实例化出多个CBCentralManager,并分别设置CBCentralManager和CBPeripheral的委托，这种用法超级啰嗦和麻烦。不过使用BabyBluetooth,可以仅使用一个BabyBluetooth的实例，通过setBlockOn...AtChannel的方法设置不同channel的委托方法，并使用```` baby.channel(channelName) ````的方法切换到这个channelName的委托。

示例：
````objc

    //设置channel “detailsView” 频道上 ， 扫描到设备的委托
    NSString *channelName = @"detailsChannel";
    [baby setBlockOnDiscoverToPeripheralsAtChannel:channelName
          block:^(CBCentralManager *central, CBPeripheral *peripheral, NSDictionary *advertisementData, NSNumber *RSSI) {
         NSLog(@"channel(%@) :搜索到了设备:%@",channelName,peripheral.name);
    }];
    
    //切换到委托频道
    baby.channel(channelName)
    //扫描设备并开始执行
    .scanForPeripherals().begin();
    
````

##默认频道
在 baby.().().()链式函数体中，channel()可以放在.begin()之前的任意位置。若链式函数体中不包含.channel()方法或是channelName为nil时，会切换到默认频道。

#辅助方法

## BabyRhythm 
程序心跳：可以给peripheral的具体操作加上心跳进行监控，从而达到一些复杂控制的需求。

### 方法

````objc
//心跳
-(void)beats;
//主动中断心跳
-(void)beatsBreak;
//结束心跳，结束后会进入BlockOnBeatOver，并且结束后再不会在触发BlockOnBeatBreak
-(void)beatsOver;
//恢复心跳，beatsOver操作后可以使用beatsRestart恢复心跳，恢复后又可以进入BlockOnBeatBreak方法
-(void)beatsRestart;

//心跳中断的委托
-(void)setBlockOnBeatBreak:(void(^)(BabyRhythm *bry))block;
//心跳结束的委托
-(void)setBlockOnBeatOver:(void(^)())block;

//心跳时间间隔，默认是3
@property int beatsInterval;

````

### example
因为central对peripheral的操作是离散的，没有完成状态的。有时候我们需要确认相关操作完成才进行下一步操作。例如需求：先读取c1的值，然后将c2的值设为c1的值，最后开启c3的notify获取数据
````objc

  @implementation xxxx{
      //建议新建一个class，用属性保存c1,c2,c3，不然可能会出现黄色的警告。
      CBCharacteristic *c1;
      CBCharacteristic *c2;
      CBCharacteristic *c3;
  }

  .... //省略中间操作和代码

 BabyRhythm *rhythm = [[BabyRhythm alloc]init];

 [baby setBlockOnDiscoverCharacteristicsAtChannel:channelOfLoadloadHistoryData block:^(CBPeripheral *peripheral, CBService *service, NSError *error) {
        //心跳
        [rhythm beats];
        for (CBCharacteristic *characteristic in service.characteristics) {
            //c1
            if ([characteristic.UUID.UUIDString isEqualToString:@"FFA4"]) {
                c1 = characteristic;
            }
            //c2
            if ([characteristic.UUID.UUIDString isEqualToString:@"FFA3"]) {
                c2 = characteristic;
            }
        }
    }];
    
    [baby setBlockOnReadValueForCharacteristicAtChannel:channelOfLoadloadHistoryData block:^(CBPeripheral *peripheral, CBCharacteristic *characteristic, NSError *error) {
        //心跳，每进入方法就跳一次，超过beatsInterval的时间就会进入setBlockOnBeatBreak，我们可以通过它判断数据是否全部读取完毕。
        [rhythm beats];
        //c2值写进c1
        if ([characteristic.UUID.UUIDString isEqualToString:@"FFA6"]) {
            [device.peripheral writeValue:c2.value forCharacteristic:c1 type:CBCharacteristicWriteWithResponse];
        }
    }];

    [rhythm setBlockOnBeatBreak:^(BabyRhythm *bry) {

        //[self check] 可以具体验证下c1和c2值是否符合条件
        //若不符合条件继续发送心跳[bry beats];
        //若符合条件，则关闭心跳，订阅c3
        if ([self check]) {
            //关闭beats           
            [baby notify:device.peripheral characteristic:c3 block:^(CBPeripheral *peripheral, CBCharacteristic *characteristics, NSError *error) {
               //业务代码
            }];
 
        }
        [bry beats];
    }];


````

#链式方法

## 谓词 
在 baby.().().()链式函数体中，可以使用谓词方法and,then,with 让你的方法使用更加语义明确和优雅
例如：

````objc

  baby.having(self.p).and.channel(channelOnPeropheralView).
  then.connectToPeripherals().discoverServices().discoverCharacteristics()
  .readValueForCharacteristic().discoverDescriptorsForCharacteristic().readValueForDescriptors().begin();
  
````

## 链式方法的顺序

链式baby.().().() 方法按照CoreBluetooth蓝牙的顺序和限制。ios中蓝牙使用的顺序为：**扫描->连接外设->发现服务->读取服务的characteristic->characteristic value、characteristic的descriptors和descriptor的value**

- begin()或者stop()为链式方法的结尾。若没有stop()，则begin()为结尾。
- having(),channel()方法可以在任意begin()之前的位置。
- 没有peripheral实例时，不能连接设备。在BabyBluetooth中就是不执行needScanForPeripherals()，那么执行connectToPeripheral()方法时必须用having(peripheral)传入peripheral实例。
- 没有连接设备，不能执行discoverServices()
- 没有执行discoverServices()不能执行discoverCharacteristics()、readValueForCharacteristic()、discoverDescriptorsForCharacteristic()、readValueForDescriptors()
- 不执行discoverCharacteristics()时，不能执行readValueForCharacteristic()或者是discoverDescriptorsForCharacteristic()
- 不执行discoverDescriptorsForCharacteristic()时，不能执行readValueForDescriptors()


# 如何安装

##1 手动安装
step1:将项目src文件夹中的文件直接拖入你的项目中即可

step2:导入.h文件

````objc
#import "BabyBluetooth.h"
````

##2 cocoapods
暂不支持，后续版本会去支持


# 示例程序说明

**BabyBluetoothExamples/BabyBluetoothAppDemo** :一个类似lightblue的程序，蓝牙操作全部使用BabyBluetooch完成。
功能：
- 1：扫描周围设备
- 2：连接设备，扫描设备的全部services和characteristic
- 3：显示characteristic，读取characteristic的value，和descriptors以及Descriptors对应的value
- 4：写0x01到characteristic
- 5：订阅/取消订阅 characteristic的notify

**BabyBluetoothExamples/BabyBluetoothOSDemo** :一个mac os程序，因为os和ios的蓝牙底层方法都一样，所以BabyBluetooth可以ios/os通用。但是os程序有个好处就是直接可以在mac上跑蓝牙设备，不像ios，必须要真机才能跑蓝牙设备。所以不能真机调试时可以使用os尝试蓝牙库的使用。

功能：
- 1：扫描周围设备、连接设备、显示characteristic，读取characteristic的value，和descriptors以及Descriptors对应的value的委托设置，并使用nslog打印信息。

- 
# 程序结构
- BabyBluetooth 链式函数实现类，BabyBluetooth库方法调用的入口
- Babysister 蓝牙操作的实现类，通过它处理了链式函数，已经调用各种filter和block
- BabySpeaker chanel切换的实现和处理设置characteristic通知的委托方法
- BabyCallback 回叫函数的block和filter的model
- BabyToy 一些工具方法
- [BabyRhythm](#user-content-BabyRhythm) 辅助方法 

# 兼容性
- 蓝牙4.0，也叫做ble，ios6以上可以自由使用。
- os和ios通用
- 蓝牙设备相关程序必须使用真机才能运行。如果不能使用真机调试的情况，可以使用os程序调试蓝牙。可以参考示例程序中的BabyBluetoothOSDemo
- 本项目和示例程序是使用ios 8.3开发，使用者可以自行降版本，但必须大于6.0 

# 后期更新
- 现在block的委托方法还没涉及到CoreBluetooth的全部方法，后续会把全部方法补充进去
- 增加对外设模式使用的支持（目前主要是中心模式）

已经更新的版本说明，请在wiki中查看


# 蓝牙学习资源
- [ios蓝牙开发（一）蓝牙相关基础知识](http://liuyanwei.jumppo.com/2015/07/17/ios-BLE-1.html)
- [ios蓝牙开发（二）蓝牙中心模式的ios代码实现](http://liuyanwei.jumppo.com/2015/08/14/ios-BLE-2.html)
- [ios蓝牙开发（三）app作为外设被连接的实现](http://liuyanwei.jumppo.com/2015/08/14/ios-BLE-3.html)
- [ios蓝牙开发（四）BabyBluetooth蓝牙库介绍](http://liuyanwei.jumppo.com/2015/09/07/ios-BLE-4.html)
- 暂未完成-ios蓝牙开发（五）BabyBluetooth实现原理
- 待定...

qq交流群：426603940

# 期待
  - 蓝牙库写起来很辛苦，希望大家可以多多支持，多多star
  - 如果在使用过程中遇到BUG，或发现功能不够用，希望你能Issues我，谢谢
  - 期待大家也能一起为BabyBluetooth输出代码，这里我只是给BabyBluetooth开了个头，他可以增加和优化的地方还是非常多。也期待和大家在Pull Requests一起学习，交流，成长。

 
