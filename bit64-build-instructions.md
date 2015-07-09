# Android 平台64位构建指导

## 概述

从构建系统的观点看，最显著的更改是用同一个 build 就可以生成出可以支持在2个目标 CPU 架构上（64 位和 32 位）的构建文件。这就是被人们所知的*多库构建*。

为了构建本地静态库和共享库，构建系统启动规则去给两个架构的 CPU 生成二进制文件。产品配置（PRODUCT_PACKAGE），和图形依赖，然后寻找到使用的是哪一个二进制文件，最后安装进系统镜像中。

构建系统默认情况下只构建 64 位版本给可执行文件和应用程序，但是你可以重写这个设置，通过一个全局的 *BoardConfig.mk* 参数或者一个局部的参数。

> **注意：**如果一个应用要开放一个 API 给其他可能是32位或是64位的应用时，这个应用必须在 manifest 文件中有 *android:multiarch* 属性，并且将其值设为 *true* ，这样可以避免潜在的错误。


## 产品配置

在 *BoardConfig.mk* 文件中，我们添加下面的参数来配置第二个 CPU 架构和接口：

```
TARGET_2ND_ARCH
TARGET_2ND_ARCH_VARIANT
TARGET_2ND_CPU_VARIANT
TARGET_2ND_CPU_ABI
TARGET_2ND_CPU_ABI2
```

你可以在 *build/target/board/generic_arm64/BoardConfig.mk* 中查看事例。

如果你想默认构建给 32 位的可执行文件和应用，设置下面的参数：

```
TARGET_PREFER_32_BIT := true
```

然而，你也可以在 *Android.mk* 文件中通过重写 module-specific 参数来进行设置。

在一个多库构建中，只要他们被构造系统定义了，命名在 *PRODUCT_PACKAGES* 中的模块就可以同时覆盖了 32 位和 64 位这两种二进制文件。在依赖库的获取中，只有被另外一个 32 位的可执行文件或库所请求时才会安装 32 位库。在 64 位库中也是一样的情况。

但是，命名在 *make* 命令行中的模块仅仅只覆盖 64 位的版本。举个例子，在执行完 *lunch aosp_arm64-eng,make libc* 后，只构建了 64 位的库。要构建 32 位的库，你需要运行 *make libc_32。*

## 在 Android.mk 中描述模块

你可以使用 *LOCAL_MULTILIB* 变量中配置你要构建的 32 位或者 64位以及重写全局的变量 *TARGET_PREFER_32_BIT*。

按照下列中的一个设置你的 *LOCAL_MULTILIB：*

* “both”：同时构建 32 位和 64 位
* “32”：只构建 32 位
* “64”：只构建 64 位
* “first”：只给第一个架构构建（ 在 32 位中优先 32 位，在 64位中优先 64 位）
* “”：默认参数；构建系统构建哪个架构取决于模块类以及其他*LOCAL_* 参数，比如 *LOCAL_MODULE_TARGET_ARCH*，*LOCAL_32_BIT_ONLY* 等等。

在多库构建中，像 *ifeq $(TARGET_ARCH)* 这种附件条件将不再起作用。

如果你想给一些特殊的架构上构建你的模块，下面的参数可能会帮助你:

* LOCAL\_MODULE\_TARGET\_ARCH：这个可以设置成一个架构的列表，有些像 *arm x86 arm64*.只有被构建的架构在列表中才会被构建系统添加进入当前模块。
* LOCAL\_MODULE\_UNSUPPORTED\_TARGET\_ARCH：这是 LOCAL\_MODULE\_TARGET\_ARCH 的对立选项。只有被构建的架构不在列表中才会被构建系统添加进入当前模块。

这里有上面两个变量的缩小版：

* LOCAL\_MODULE\_TARGET\_ARCH\_WARN 
* LOCAL\_MODULE\_UNSUPPORTED\_TARGET\_ARCH\_WARN

如果当前模块跳出了应该有的架构限制，那么构建系统将会提出警告。

启动 arch-specific 构建标识，使用 arch-specific LOCAL_ 变量。一个 arch-specific LOACL_ 变量一般情况是 LOCAL_ 跟上一个架构的后缀，比如:

* LOCAL\_SRC\_FILES\_arm, LOCAL\_SRC\_FILES\_x86，
* LOCAL\_CFLAGS\_arm, LOCAL\_CFLAGS\_arm64，
* LOCAL\_LDFLAGS\_arm, LOCAL\_LDFLAGS\_arm64，

需要注意的是：所有的 LOCAL_ 变量支持 arch-specific 变体。查看最新的相关列表，请看 *build/core/clear_vars.mk*。

## 安装路径

在前面，你应该使用 LOCAL_MODULE_PATH 来替代默认的那个用于安装一个库到一个路径。比如 *LOCAL\_MODULE\_PATH := $(TARGET\_OUT\_SHARED\_LIBRARIES)/hw*

在多库构建中，使用 LOCAL\_MODULE\_RELATIVE\_PATH 来替代：

```
LOCAL_MODULE_RELATIVE_PATH := hw
```

这样就可以让 64 位和 32 位都可以安装在正确的路径下了。

如果你想构建一个同时在 32 位和 64 位下都可以运行的可执行文件，你会都需要使用下面中的一个变量来区分你的安装路径：

* LOCAL\_MODULE\_STEM\_32, LOCA\L\_MODULE\_STEM\_64 具体说明安装文件的名字
* LOCAL\_MODULE\_PATH\_32, LOCAL\_MODULE\_PATH\_64 具体说明安装路径

## 生成源码

在多库构建中，如果你在 *$(local-intermediates-dir)*（或者含有参数的 *$(intermediates-dir-for)*）生成源文件，那么它将不会正常工作。这是因为32 位和 64 位会同时请求中间产生的源文件，但是 *$(local-intermediates-dir)* 只指向其中这两个中的其中一个路径。

值得高兴的是，构建系统现在为生成源文件提供可共享，对多库友好的中间文件路径。你可以使用 *$(local-generated-sources-dir)* 或者 *$(generated-sources-dir-for)* 来获取一个文件路径。他们的用法跟 *$(local-intermediates-dir)* 和 *$(intermediates-dir-for)* 类似。

如果源文件已经在新的中间文件路径中生成，并且被 *LOCAL_GENERATED_SOURCES* 所接管，那么它将在多库构建中生成 32 位和 64 位文件。

## 预构建

在多库中，你不能用 *TARGET_ARCH*（或者连同 *TARGET_2ND_ARCH* 一块使用）来告诉构建系统要预构建二进制文件的目标是什么。用之前提到的 LOCAL_ 变量 LOCAL\_MODULE\_TARGET\_ARCH 或者是 LOCAL\_MODULE\_UNSUPPORTED\_TARGET\_ARCH 来替换。

用这些变量，构建系统即使是在一个 64 位的多库构建中也能选择匹配 32 位的预构建二进制文件。

如果你想使用选择的架构来给预构建的二进制文件计算出源文件路径，你可以使用 *$(get-prebuilt-src-arch)*。

## Dex 预选择

在 64 位设备上，默认情况下我们同时给系统镜像生成 32 位和 64 位的 odex 文件以及一些 Java 库。默认我们只给 64 位架构生成 odex 文件给 APK 文件。如果某个应用可能会同时在 32 位和 64 位上登录，请使用 *LOCAL_MULTILIB := both* 来确保 32 位和 64 位的 odex 文件都被生成。这个标识同样也会告诉构建系统，如果有应用需要，就包括 32 位和 64 位的 JNI 库。

