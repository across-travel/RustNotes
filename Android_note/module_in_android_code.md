---
title: App中的模块化
date: 2016-11-19 10:55:01
category: Android_note
---

开发Android App时，需要调用蓝牙设备。

一开始是在Service中管理蓝牙设备的连接和业务。随着业务代码的增长，蓝牙的连接部分日趋复杂和混乱。
整个Service的代码也再变大。业务要求针对同一款蓝牙硬件设备开发出不同的App。各个App要实现的功能不同。
为避免大量的重复代码，需要将这个蓝牙设备管理模块化。

单例化一个BtDevice，相关信息都可以直接调用。参照Google的示例，蓝牙连接都在子线程中进行。因此
这个模块可以不依赖于Service。为取得蓝牙设备发回的数据，给它添加监听者，增加回调。App中有多个
activity，各个页面生命周期不同。增设一个消息管理Service，专门用于接受BtService的数据，然后
再用别的方法发送出去（EventBus，Broadcast...）。

在Android Studio中新建一个module，把蓝牙管理相关的代码放进模块，调整好API。编译后会有aar文件。
新的工程只需要引入aar文件即可。

模块化后，代码层级和逻辑更清晰，耦合度下降，更易维护。

### xml中找不到module中的自定义view
Android Studio 2.2.3 

模块的build和调用方的build的配置要一致，最好全部配置成一样的。

比如compileSdkVersion等等。然后重新make module和rebuild project
