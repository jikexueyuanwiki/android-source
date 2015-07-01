# 构建内核

如果你只对内核感兴趣，你可以利用这篇文章来帮助你下载并且构建出合适的内核。

下面的信息我们假定你还没有下载所有的 AOSP。如果你已经下载了所有的 AOSP ，你可以跳过 git clone 的部分到下载实际的内核源码。

在下面的例子中，我们将使用 Pandaboard 内核。

## 明确要创建的内核

这张表列出了内核源码的名字，位置以及二进制数据：

|Devices|Binary location   |Source location|Build configuration
|-------|------------------|---------------|-------------------
| shamu     |device/moto/shamu-kernel	 | kernel/msm  | shamu_defconfig|
| fugu      |device/asus/fugu-kernel	    | kernel/x86_64 | fugu_defconfig |
| volantis  |device/htc/flounder-kernel  |kernel/tegra   | flounder_defconfig|
| hammerhead|device/lge/hammerhead-kernel|kernel/msm|hammerhead_defconfig
| flo|device/asus/flo-kernel/kernel	|kernel/msm| flo_defconfig|
|deb|device/asus/flo-kernel/kernel	|kernel/msm| flo_defconfig|
|manta	|device/samsung/manta/kernel	|kernel/exynos|manta_defconfig|
|mako	|device/lge/mako-kernel/kernel	|kernel/msm|mako_defconfig|
|grouper	|device/asus/grouper/kernel	|kernel/tegra|tegra3_android_defconfig|
|tilapia	|device/asus/grouper/kernel	|kernel/tegra|tegra3_android_defconfig|
|maguro	|device/samsung/tuna/kernel	|kernel/omap|tuna_defconfig|
|toro	|device/samsung/tuna/kernel	|kernel/omap|tuna_defconfig|
|panda	|device/ti/panda/kernel	|kernel/omap|panda_defconfig|
|stingray	|device/moto/wingray/kernel	|kernel/tegra|stingray_defconfig|
|wingray	|device/moto/wingray/kernel	|kernel/tegra|stingray_defconfig|
|crespo	|device/samsung/crespo/kernel	|kernel/samsung|herring_defconfig|
|crespo4g	|device/samsung/crespo/kernel	|kernel/samsung|herring_defconfig|

你可能会想查看设备工程中核心组件里面你所感兴趣的 git 日志。

设备工程在当前格式的设备/<vendor>/<name>目录下。

```
$ git clone https://android.googlesource.com/device/ti/panda
$ cd panda
$ git log --max-count=1 kernel
```

这里的内核组件提交信息中包含了一部分有问题的内核源码的 git log 信息。第一进入的是最近一次的，也就是用于构建内核的信息。你在下面的步骤中会需要用到它。

## 辨别内核版本

为了查明在实际的系统映像中所使用的内核版本，在内核文件中执行下面的代码：

```
$ dd if=kernel bs=1 skip=$(LC_ALL=C grep -a -b -o $'\x1f\x8b\x08\x00\x00\x00\x00\x00' kernel | cut -d ':' -f 1) | zgrep -a 'Linux version'
```

在 Nexus 5 （hammerhead），你可以用这个命令：

```
$ dd if=zImage-dtb bs=1 skip=$(LC_ALL=C od -Ad -x -w2 zImage-dtb | grep 8b1f | cut -d ' ' -f1 | head -1) | zgrep -a 'Linux version'
```

## 下载资源

下载哪一个取决于你所想要的：

```
$ git clone https://android.googlesource.com/kernel/common.git
$ git clone https://android.googlesource.com/kernel/x86_64.git
$ git clone https://android.googlesource.com/kernel/exynos.git
$ git clone https://android.googlesource.com/kernel/goldfish.git
$ git clone https://android.googlesource.com/kernel/msm.git
$ git clone https://android.googlesource.com/kernel/omap.git
$ git clone https://android.googlesource.com/kernel/samsung.git
$ git clone https://android.googlesource.com/kernel/tegra.git
```

* *goldfish* 工程中包含为模拟器平台使用的内核源码。

* *msm* 工程有可供 ADP1，ADP2，Nexus One，Nexus 4，Nexus 5，Nexus 6，也可以被当做一个起始点给 Qualcomm MSM 芯片组使用。

* *omap* 工程被用于 PandaBoard 和 Galaxy Nexus，也可以被当做一个起始点给 TIOMAP 芯片组使用。

* *samsung* 工程是为了 Xoom，Nexus 7，Nexus 9，也可以被当做一个起始点给 Samsung Hummingbird 芯片组使用。

* *tegra* 工程是给 Xoom，Nexus 7， Nexus 9，也可以被当做一个起始点给 NVIDIA Tegra 芯片组使用。

* *exynos* 工程包含供 Nexus 10 的核心源码，也可以被当做一个起始点给 Samsung Exynos 芯片组使用。

* *x86_64* 工程包含有 Nexus Players 的核心源码，也可以被当做一个起始点给 Intel x86_64 芯片组。

## 下载预构建 gcc

确保预生成的工具链在你的环境变量中

```
$ export PATH=$(pwd)/prebuilts/gcc/linux-x86/arm/arm-eabi-4.6/bin:$PATH
```

或者

```
$ export PATH=$(pwd)/prebuilts/gcc/darwin-x86/arm/arm-eabi-4.6/bin:$PATH
```

在一个 Linux host 中，如果你没有 Android 源码树，你可以从下面的地址中下载预编译工具链：

```
$ git clone https://android.googlesource.com/platform/prebuilts/gcc/linux-x86/arm/arm-eabi-4.6
```

## 构建

作为一个例子，我们将会使用下面的命令来构建 panda 内核：

```
$ export ARCH=arm
$ export SUBARCH=arm
$ export CROSS_COMPILE=arm-eabi-
$ cd omap
$ git checkout <commit_from_first_step>
$ make panda_defconfig
$ make
```

构建 tuan 内核，你需要将之前的命令中所有 'panda' 改为 'tuna'。

核心组件将会输出：‘arch/arm/boot/zlmage’。它可以被拷贝进入 Android 源码树中，以便于构建适合的 boot 镜像。

或者你可以使用 *make bootimage* 命令，包含 *TARGET_PREBUILD_KERNEL* 变量，或者其他任何 make 命令行用来构建 boot 镜像。

```
$ export TARGET_PREBUILT_KERNEL=$your_kernel_path/arch/arm/boot/zImage
```

那个变量支持所有用过 device/common/populare-new-device.sh 命令启动的设备。