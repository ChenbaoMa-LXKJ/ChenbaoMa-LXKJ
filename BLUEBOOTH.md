
# <center>蓝牙相关内容整理</center> 

#### 既然是低功耗蓝牙，那它是怎么做到低功耗的呢？这里面用了哪些技术呢？


+ 第一，它的广播频段和广播时射频开启时间的减少：传统蓝牙使用16~32个频段进行广播，而BLE仅使用3个广播频段；每次广播时的射频开启时间由传统蓝牙的22ms减少为0.6 ~ 1.2ms。

+ 第二，它每次只传输少量数据。

+ 第三，它的传输速率是比较低的。

综合上面三个特点就决定了低功耗蓝牙相对传统蓝牙功耗更低，更节能。
举个例子，像视频传输、高质量音频传输或者传输大量数据的应用就不适合使用低功耗蓝牙。像传输小体量的数据，比如传输传感器的数据到我们的智能手机。
***

#### 低功耗蓝牙的架构层如下
<font color=Blue>架构层|<font color=Blue>功能</font>
---|:--:|
物理层|控制无线电收发
链路层 Link layer|定义数据包结构，包括状态机和无线电控制，并提供链路层级别的加密。
L2CAP|:逻辑链路控制与适配协议。L2CAP作为一个协议多路复用器，处理数据包的分段和分组。它还提供逻辑通道，这些通道通过一个或多个逻辑链路进行多路复用。用于蓝牙低功耗技术的L2CAP协议是在传统蓝牙L2CAP协议的基础上进行优化和简化的协议。
HIC|是控制器和主机之间通信的标准化接口。
ATT|属性协议。属性协议提供了在蓝牙低功耗设备之间传输数据的方法。它依赖于蓝牙低功耗连接，并提供通过该连接读取、写入、指示和通知属性值的过程。
GAP|通用访问配置文件。GAP层为蓝牙低功耗设备提供了宣传自己或其他设备、进行设备发现、打开和管理连接以及广播数据的手段。
SM|安全经理。提供绑定设备、加密和解密数据以及启用设备隐私的方法

***

## 关于BLE应用<font color=Blue>

- <font color=green>Broadcaster: </font>又称为Advertiser，周期性的向周围设备广播数据，
- <font color=green>Observer：</font>又称为Scanner，可以监听广播数据或者搜索周围设备,
- <font color=green>Central: </font>又称为master，负责扫描设备并发起建立请求，在建立连接后变成master
- <font color=green>Peripheral: </font>可称为slave,  负责广播的并接收连接请求的设备,在建立连接后称为slave

</font>


** <font size = 4>如果外设要发送的数据多于单个数据包所能容纳的数据，则连接事件将自动扩展，并且外设可以发送尽可能多的数据包，直到下一个连接间隔开始。这只能用于不需要确认的属性协议操作。**
###<font size =4>小总结：
- Master和Slave之间进行交互的间隔称为BLE蓝牙连接间隔，它是两个连续的连接事件之间的间隔，其中，连接间隔最小的值为7.5ms,最小的增量为1.25ms
- 为从机延迟提供跳过多个连接事件的能力。这种能力给外设设备更多的灵活性。如果外设没有要发送的数据，则可以跳过连接事件，保持睡眠并节省电量。 外设设备选择是否在每个连接事件时间点上唤醒。 虽然外设可以跳过连接事件，但不能超出从延迟参数允许的最大值
***

## 蓝牙switch (SL_BT_MSG_ID(evt->header)
```C
(&address, &address_type);获取地址及类型（公开身份/静态随机地址）然后移动反转地址得到System ID.
uint8_t system_id[8];
// Pad and reverse unique ID to get System ID.
 system_id[0] = address.addr[5];
system_id[1] = address.addr[4];
system_id[2] = address.addr[3];
ystem_id[4] = 0xFE;
system_id[5] = address.addr[2];
system_id[6] = address.addr[1];
system_id[7] = address.addr[0];
```

***
***
# 关于蓝牙的通讯协议
## GAP
<font size=5 color= blue> GPA 使设备可见，并决定了设备交互的方式和能力</font> 

GAP定义了：**问：主从设备 是否 能调换，主设备是否需要具备数据处理功能**
外围设备（Peripheral）-->穿戴设备--负责广播的并接收连接请求的设备--连接后为Slave
中心设备（Central）-->手机--负责扫描设备并发起建立请求--连接后为MASTER
- GAP 中外围设备通过两种方式向外广播数据： Advertising Data Payload（广播数据）和 Scan Response Data Payload（扫描回复)
 
## 广播数据单元
广播数据据单元：AD Structure
长度值Length |AD type |数据值data|
---|:--:|:--|
1字节|1字节|1字节
表示ADtpype+data的长度|表示观念广播数据的含义|表示物理状态的连接能力，对应的Bit取值为1为TURE为0为Fals

**注意：数据为16进制格式且LEN=Type长度+数据长度**
例如：0x0201190509646171690303F7FF07FF11000B16212C071617F10B16212C

Length |AD type |数据值data|
---|:--:|:--|
2|0X01|0X19|
5|0X09|0X64617169|
3|0X03|0XF7FF|
7|0XFF|0X11000B16212C|
7|0X16|0X17F10B16212C|


1. Type = 0x01 表示设备LE物理连接。
2. Type = 0x09 表示设备的全名
3. Type = 0x03 表示完整的16bit UUID。其值为0xFFF7。
4. Type = 0xFF 表示厂商数据。前两个字节表示厂商ID,即厂商ID为0x11。后面的为厂商数据，具体由用户自行定义。
5. Type = 0x16 表示16 bit UUID的数据，所以前两个字节为UUID,即UUID为0xF117，后续为UUID对应的数据，具体由用户自行定义。
   
**最后继承AdvertiseCallback自定义广播回调**


AD Type的类型如下：

- Flags：TYPE = 0x01。用来标识设备LE物理连接。
- bit 0: LE 有限发现模式
- bit 1: LE 普通发现模式
- bit 2: 不支持 BR/EDR
- bit 3: 对 Same Device Capable(Controller) 同时支持 BLE 和 BR/EDR
- bit 4: 对 Same Device Capable(Host) 同时支持 BLE 和 BR/EDR
- bit 5..7: 预留
​ 这bit 1~7分别代表着发送该广播的蓝牙芯片的物理连接状态。当bit的值为1时，表示支持该功能。


## GATT
定义两个 BLE 设备通过 Service 和 Characteristic 进行通信.
GATT的协议 需要先经过 GAP协议，且GATT的协议是独占的，也就是一个 BLE 外设同时只能被一个中心设备连接。一旦外设被连接，它就会马上停止广播，这样它就对其他设备不可见了。当设备断开，它又开始广播。

***
##软件调试部分
调试配置的主要内容：
```C++
private class daqiAdvertiseCallback extends AdvertiseCallback {
    //开启广播成功回调
    @Override
    public void onStartSuccess(AdvertiseSettings settingsInEffect){
        super.onStartSuccess(settingsInEffect);
        Log.d("daqi","开启服务成功");
    }

    //无法启动广播回调。
    @Override
    public void onStartFailure(int errorCode) {
        super.onStartFailure(errorCode);
        Log.d("daqi","开启服务失败，失败码 = " + errorCode);
    }
}
```
**初始化上述对象后可以进行广播：**
```C
//获取BLE广播的操作对象。
//如果蓝牙关闭或此设备不支持蓝牙LE广播，则返回null。
mBluetoothLeAdvertiser = mBluetoothAdapter.getBluetoothLeAdvertiser();
//mBluetoothLeAdvertiser不为空，且蓝牙已开打
if(mBluetoothAdapter.isEnabled()){
    if (mBluetoothLeAdvertiser != null){
         //开启广播
        mBluetoothLeAdvertiser.startAdvertising(mAdvertiseSettings,
            mAdvertiseData, mScanResponseData, mAdvertiseCallback);
    }else {
        Log.d("daqi","该手机不支持ble广播");
    }
}else{
    Log.d("daqi","手机蓝牙未开启");
}
```

**<font color=Blue>广播主要是通过`BluetoothLeAdvertiser#startAdvertising()`方法实现
但在之前需要先获取`BluetoothLeAdvertiser`对象。`BluetoothLeAdvertiser`对象存在两个情况获取为Null:
与广播成对出现就是`BluetoothLeAdvertiser.stopAdvertising()`停止广播了，传入开启广播时传递的广播回调对象，即可关闭广播：``mBluetoothLeAdvertiser.stopAdvertising(mAdvertiseCallback)``**






***
<font color=red>我是红色</font>
<font color=#008000>我是绿色</font>
<font color=Blue>我是蓝色</font>
<font size=6>我是尺寸</font>