# Biomodule 蓝牙SDK 

# 简介

本 SDK 包含回车生物电采集模块的蓝牙连接和生物电采集控制。通过此 SDK 可以在 Android app 里快速实现和我们的采集模块连接，并控制其进行数据的采集和停止等指令。

## 开发组件

SDK包含以下文件：

<img src="https://github.com/EnterTech/Flowtime-BLE-SDK-Android/blob/master/docimage/1.jpeg" width="40%">

1. Android Studio 工程Demo
2. 蓝牙功能模块源代码
3. 蓝牙功能模块对应的jar包

# Ble基础SDK [![Download](https://api.bintray.com/packages/hzentertech/maven/biomoduleble/images/download.svg?version=1.2.0)](https://bintray.com/hzentertech/maven/biomoduleble/1.2.0/link)

## 说明

基础的Ble SDK可以方便从硬件端采集到原始脑波、心率等数据.

## 集成

### gradle自动依赖

在所需的module中的build.gradle文件下添加以下依赖即可：

```groovy
implementation 'cn.entertech:biomoduleble:1.1.0'
```

### jar包集成

如果你在自动依赖遇到问题也可以手动添加依赖

集成的方式很简单，只需要将Demo中app/libs目录下的蓝牙jar文件拷贝到自己项目的libs文件夹下，如果没有，需要新建一个文件夹。拷贝完后需要在你项目的build.gradle文件中添加如下依赖：

```groovy
implementation fileTree(include: ['*.jar'], dir: 'libs')
implementation files('libs/enter-biomodule-ble-v1.0.8.jar')
```

### 注意事项

另外build.gradle文件中还需要添加额外的依赖：

```groovy
compile 'com.polidea.rxandroidble2:rxandroidble:1.8.0'
compile 'com.orhanobut:logger:1.15'
```

并且需要申明蓝牙相关权限：

```xml
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
```

## 快速接入

### 1.连接设备

**代码示例**

```kotlin
var biomoduleBleManager = BiomoduleBleManager.getInstance(context)
//根据信号强弱连接最近的设备，如果需要连接指定设备可调用scanMacAndConnect方法传入mac地址连接
biomoduleBleManager.scanNearDeviceAndConnect(fun() {
            Logger.d("扫描成功")
        }, fun(e: Exception) {
            Logger.d("扫描失败：$e")
        }, fun(mac: String) {
            Logger.d("连接成功$mac")
        }) { msg ->
            Logger.d("连接失败")
        }
```

### 2.添加数据监听

**代码示例**

```kotlin
//心率数据监听
var heartRateListener = fun(heartRate: Int) {
    Logger.d("心率数据" + heartRate)
}
biomoduleBleManager.addHeartRateListener(heartRateListener)
//原始脑波数据监听
var rawDataListener = fun(data:ByteArray){
    Logger.d(Arrays.toString(data))
}
biomoduleBleManager.addRawDataListener(rawDataListener)
```

> 注意：如果在界面退出时需要调用对象的移除监听方法，否则会出现内存溢出，如biomoduleBleManager.removeRawDataListener(rawDataListener)

### 3.采集脑波和心率数据

**代码示例**

```kotlin
//如果想要停止采集调用stopBrainCollection()
biomoduleBleManager.startBrainCollection()
```

更多详细的蓝牙ble方法可以参考[Ble详细API说明](<https://github.com/Entertech/Enter-Biomodule-BLE-Android-SDK/blob/master/Ble%E8%AF%A6%E7%BB%86API%E8%AF%B4%E6%98%8E.md>)

# 设备管理界面 SDK（按需接入）[![Download](https://api.bintray.com/packages/hzentertech/maven/biomodulebleui/images/download.svg?version=1.0.2)](https://bintray.com/hzentertech/maven/biomodulebleui/1.0.2/link)

## 说明

设备管理界面SDK，包含了基础ble功能，另外还包含设备管理界面，如果对您的APP对界面没有特殊的定制化需要可直接接入设备管理界面

## 集成

在module的build.gradle文件中加入以下依赖：

```groovy
implementation 'cn.entertech:biomodulebleui:1.0.2'
```

在项目根目录的build.gradle文件下添加以下依赖地址

```groovy
allprojects {
    repositories {
        maven {
            url "https://dl.bintray.com/hzentertech/maven"
        }
    }
}
```

## 权限申请

```xml
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.INTERNET" />
```

## 初始化

如果对设备管理界面没有特殊要求可以直接我们提供的设备管理界面SDK，可以设置DeviceUIConfig这个类，进行相关属性的配置。

**代码示例**

```kotlin
var deviceUIConfig = DeviceUIConfig.getInstance(this)
deviceUIConfig.init(isDeviceBind, isMultipleDevice, deviceCount)
```

**参数说明**

| 参数             | 类型    | 说明                                                     |
| ---------------- | ------- | -------------------------------------------------------- |
| isDeviceBind     | Boolean | 是否绑定设备，如果是则每次连接设备时会自动连接之前的设备 |
| isMultipleDevice | Boolean | 是否支持多连接                                           |
| deviceCount      | Int     | 设备连接个数，最多可设备4个                              |

## 设备管理入口

设备管理界面对外提供的入口：DeviceManagerActivity

## 固件更新配置

**代码示例**

```kotlin
deviceUIConfig.updateFirmware(isUpdate,oldVersion,newVersion,path)
```

**参数说明**

| 参数       | 类型    | 说明                       |
| ---------- | ------- | -------------------------- |
| isUpdate   | Boolean | 是否开启固件更新           |
| oldVersion | String  | 当前固件版本号 格式：a.b.c |
| newVersion | String  | 新固件版本号 格式：a.b.c   |
| path       | String  | 固件升级包的路径           |

## 界面效果

<center class="half">
<img src="https://github.com/Entertech/Enter-Biomodule-BLE-Android-SDK/blob/master/docimage/%E8%AE%BE%E5%A4%87%E8%BF%9E%E6%8E%A5.jpeg" width="200"/><img src="https://github.com/Entertech/Enter-Biomodule-BLE-Android-SDK/blob/master/docimage/%E8%AE%BE%E5%A4%87%E8%BF%9E%E6%8E%A5%E6%88%90%E5%8A%9F.jpeg" width="200"/><img src="https://github.com/Entertech/Enter-Biomodule-BLE-Android-SDK/blob/master/docimage/%E8%AE%BE%E5%A4%87%E8%BF%9E%E6%8E%A5%E5%A4%B1%E8%B4%A5.jpeg" width="200"/>
</center>



<center class="half">
<img src="https://github.com/Entertech/Enter-Biomodule-BLE-Android-SDK/blob/master/docimage/%E5%9B%BA%E4%BB%B6%E5%8D%87%E7%BA%A71.jpeg" width="200"/><img src="https://github.com/Entertech/Enter-Biomodule-BLE-Android-SDK/blob/master/docimage/%E5%9B%BA%E4%BB%B6%E5%8D%87%E7%BA%A72.jpeg" width="200"/><img src="https://github.com/Entertech/Enter-Biomodule-BLE-Android-SDK/blob/master/docimage/%E5%9B%BA%E4%BB%B6%E5%8D%87%E7%BA%A73.jpeg" width="200"/>
</center>

# 更新日志

[biomoduleble 更新日志](https://github.com/Entertech/Enter-Biomodule-BLE-Android-SDK/wiki/biomoduleble--%E6%9B%B4%E6%96%B0%E6%97%A5%E5%BF%97)

[biomodulebleui 更新日志](https://github.com/Entertech/Enter-Biomodule-BLE-Android-SDK/wiki/biomodulebleui--%E6%9B%B4%E6%96%B0%E6%97%A5%E5%BF%97)