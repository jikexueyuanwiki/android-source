# 根据设备构建

这一篇补充说明主页上有关[构建和运行](https://source.android.com/source/building-running.html)中运行在特殊设备上的信息。

通过当前已经发放的版本，可以在 Nexus 4，Nexus 7 以及一些 Galaxy Nexus 的变异版本的手机上构建。每一个设备的具体实用性水平，取决于硬件所属的二进制文件。

给 Nexus 4 和 Nexus 7，所有的配置都可以使用，并且所有的硬件都可以工作。由于硬件的不同，不要在 Nexus 7 上使用 4.1.1，它已经被 4.1.2 以及后续的版本所取代。

所有的 Nexus 10 “manta” 配置可以使用 4.2.2。在这些设备上，图像，音频，Wi-Fi，蓝牙，照相机，NFC，GPS 和定向的感应器都可以运行。

Galaxy Nexus 变异版本可以使用的是 GSM/HSPA+ “maguro” 配置(只有在它已经被 “yakju” 或者 “takju” 操作系统所替代)，以及 VZW CDMA/LTE "toro" 配置。在这些设备上，图形和视频可以生效，和 Wi-Fi，蓝牙一样，它们都通过各自的细胞网络连接。NFC 和方向感应器也可以工作。

Galaxy Nexus 在 jb-mr1-dev-plus-aosp 分支上实验性的通过 Sprint CDMA/LTE 配置信息 “toroplus”。在此配置中，细胞网络并不能正常工作，其他的功能像他们在 “toro” 中一样的工作。

使用 Android 4.1.2，Motorola Xoom 可以用在产于美国的 Wi-Fi 配置 “wingray“。图形，音频， Wi-Fi 和蓝牙以及方向感应都可以正常工作。

所有 Nexus S 和 Nexus S 4G 的配置都可以用于 Android 4.1.2 上。在这些设备上，所有的硬件都可以正常的工作。

此外，[PandaBoard](http://pandaboard.org/) a.k.a ”panda“ 可以被用在 jb-mr1-dev-plus-aosp 分支上，但是仅仅是理论上可行。具体如何通过安卓开源工程来使用 PandaBoard，在资源树中的 *device/ti/panda/README* 中查看。

## 创建 fastboot 和 adb

如果你还没有安装好这些工具，fastboot 和 adb 可以通过一般的 build 系统来构建。按照网页中有关[Build and Running](https://source.android.com/source/building-running.html)的信息，替换掉主要的 *make* 命令，改为一下的命令：

```
$ make fastboot adb
```

## 启动进入 fastboot 模式

通过冷启动，下面核心的组合可以被用来进入 fastboot 模式，这是一个在 bootloader 中的一种模式，可以被用来启动设备：

|   Devices | Keys   
|-----------|--------
|   shamu   |Press and hold Volume Down, then press and hold Power
|   fugu    | Press and hold Power
| volantis  | Press and hold Volume Down, then press and hold Power
| hammerhead| Press and hold both Volume Up and Volume Down, then press and hold Power
| flo  | Press and hold Volume Down, then press and hold Power
| deb | Press and hold Volume Down, then press and hold Power
| manta | Press and hold both Volume Up and Volume Down, then press and hold Power
| mako  | Press and hold Volume Down, then press and hold Power
| grouper | Press and hold Volume Down, then press and hold Power
| tilapia | Press and hold Volume Down, then press and hold Power
| phantasm | Power the device, cover it with one hand after the LEDs light up and until they turn red
| maguro | Press and hold both Volume Up and Volume Down, then press and hold Power
| toro | Press and hold both Volume Up and Volume Down, then press and hold Power
| toroplus | Press and hold both Volume Up and Volume Down, then press and hold Power
| panda | Press and hold Input, then press Power
| wingray | Press and hold Volume Down, then press and hold Power
| crespo | Press and hold Volume Up, then press and hold Power
| crespo4g | Press and hold Volume Up, then press and hold Power

同样的，命令 *adb reboot bootloader* 可以被用于直接重启 Android 进入 bootloader 模式，并且不需要核心组合。

## 解锁 bootloader

只有 bootloader 允许才能刷入客户端。

默认情况下 bootloader 是上锁的。在设备处于 fastboot 模式下，bootloader 可以这么解锁：

```
$ fastboot oem unlock
```

程序必须确认在屏幕上，并且出于隐私原因将会删除用户数据。这只需要执行一次即可。

所有数据都被删除，也就是说不仅仅是应用程序的私有数据，还包括那些可以通过 USB 轻易获取的公共数据，其中包含照片和电影，都会被删除。请在解锁 bootloader 之前，确认重要的数据是否都做了备份。

在 Nexus 10 上，解锁 bootloader 之后内置的存储被置于未格式化的状态，所以必须通过下面的命令格式化：

```
$ fastboot format cache
$ fastboot format userdata
```

bootloader 可以通过下面的命令进行加锁

```
$ fastboot oem lock
```

注意，这会清除用户在 Xoom 上的数据（包括 USB 共享数据）

## 获取特定的二进制数据

安卓开源工程不能单单从纯粹的源码中使用，而且还必须通过硬件厂商的连接库才能运行，特别是那些具有硬件图像加速功能的手机。

官方提供给支持的设备的二进制文件被标以 AOSP 标签免费版本，是可以从 [Google's Nexus driver page](https://developers.google.com/android/nexus/drivers)中下载的。这些用不开源的代码，增加了额外的硬件能力。安装构建 AOSP 当前分支，请使用 [Binaries Preview for Nexus Devices](https://developers.google.com/android/nexus/blobs-preview)。

当给一个设备创建 master 分支时，大多数被标记发放或者多数不久之前的二进制数据应该是可以被使用的。

### 取出特定的二进制数据

每一个特定的二进制数据，都会通过一个自取脚本，从压缩的文档中获取。解压缩每一个文件，从资源树的根目录中，运行包括自取脚本，确认你同意封闭许可协议，之后二进制文件以及他们的匹配文件将会在资源树的 *vendor/* 层中安装。

### 添加特定二进制数据时的清理

为了确保取出后的二进制数据正确的添加进账号，之前已经存在对外输出的构建必须像这样删除：

```
$ make clobber
```

## 选取和构建匹配设备的配置

匹配和构建 Android 开源工程的步骤被写在了 [Building](https://source.android.com/source/building.html)上。

通过登录菜单，给大多数设备推荐的 builds，当运行无参数 *lunch* 命令时是可以运行的。适配 Nexus 的工厂图片和二进制数据可以在这里下载：

[https://developers.google.com/android/nexus/images](https://developers.google.com/android/nexus/images)

[https://developers.google.com/android/nexus/drivers](https://developers.google.com/android/nexus/drivers)

| Device | Code name | Build configuration
|--------|-----------|--------------------|
|Nexus 6      | shamu               | aosp_shamu-userdebug
|Nexus Player	 |fugu                 |aosp_fugu-userdebug
|Nexus 9	    |volantis (flounder)	|aosp_flounder-userdebug
|Nexus 5 (GSM/LTE)	| hammerhead| aosp_hammerhead-userdebug
|Nexus 7 (Wi-Fi)	|razor (flo)	|aosp_flo-userdebug
|Nexus 7 (Mobile)	|razorg (deb)	|aosp_deb-userdebug
|Nexus 10	|mantaray (manta)	|full_manta-userdebug
|Nexus 4	|occam (mako)	|full_mako-userdebug
|Nexus 7 (Wi-Fi)	|nakasi (grouper)	|full_grouper-userdebug
|Nexus 7 (Mobile)	|nakasig (tilapia)	|full_tilapia-userdebug
|Galaxy Nexus (GSM/HSPA+)	|yakju (maguro)|full_maguro-userdebug
|Galaxy Nexus (Verizon)	|mysid (toro)	|aosp_toro-userdebug
|Galaxy Nexus (Experimental)	|mysidspr (toroplus)|aosp_toroplus-userdebug
|PandaBoard (Archived)	| panda|aosp_panda-userdebug
|Motorola Xoom (U.S. Wi-Fi)	|wingray	|full_wingray-userdebug
|Nexus S	|soju (crespo)	|full_crespo-userdebug
|Nexus S 4G	|sojus (crespo4g)	|full_crespo4g-userdebug

不要在 Nexus 7 上使用 4.1.1，此机型只能使用 4.1.2 或更高版本。

## 设备刷入系统

如果有必要请将设备设置成 fastboot 模式（详情请看上文）。

一个完整的 Android 系统可以用一条命令来给设备刷入一个系统：在经过核对被写入的系统已经成功的和已经安装过的 bootloader，广播之间可以相互协作后，会将启动，修复，系统三个部分写在一块，最后重启系统。这样的操作也会清楚用户数据，就跟之前在 *fastboot oem unlock* 中提到的差不多。

```
$ fastboot -w flashall
```

需要注意的是，在 Motorola Xoom 上，文件系统创建出的 via fastboot 并不会良好的工作，并且强烈推荐用下面的命令重建它：

```
$ adb reboot recovery
```

一旦进入回复，打开菜单（按下电源和音量+），清除缓存部分，之后清除数据。

## 恢复设备出厂值

Nexus 5,Nexus 10,Nexus 4,Nexus Q,Nexus 7,Galaxy Nexus (GSM/HSPA+ "yakju" 和 "takju", 和 CDMA/LTE "mysid" 以及 "mysidspr")，Nexus S 和 Nexus S 4G 的出厂图片，都可以在[Google's factory image](https://developers.google.com/android/nexus/images)页面下载。

Motorola Xoom的工厂图片则直接通过 Motorola 发布。
