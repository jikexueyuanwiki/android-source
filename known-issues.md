# 已知的问题

即使是在我们最好的照看下，也会偶尔出现小的问题。这篇文章会持续追踪在使用 Android 源码的过程中出现的问题。

## 构建问题

### 在 toro 构建中丢失 CellBroadcastReceiver

#### 症状

在 AOSP 给 toro的构建中（Jelly Bean 4.2.1 之后的版本），CellBroadcastReceiver 并不会包括在系统中。

#### 原因：

在 *vendor/samsung/toro/device-partial.mk*中的 *PRODUCT_PACKAGES* 有一个排版错误：有一个 H 替代了 K。

#### 修复：

使用 4.2.2 的最后一个版本包，或者手工的修改排版。

### 丢失 CTS 原生 XML 生成器

#### 症状:

在有些 IceCreamSandwich 或之后的版本构造中，在构造中会先打印下面的警告：*/bin/bash: line 0: cd: cts/tools/cts-native-xml-generator/src/res: No such file or directory*

#### 原因：

有些 makefile 依赖的路径并不存在。

#### 修复：

无。这是一个无害的警告。

### Black Gingerbread 模拟器

#### 症状:

这个模拟器直接由 gingerbread 分支构造而且没有启动，一直卡死在绿屏状态下。

#### 原因：

gringerbread 分支使用的是 R7 版本的模拟器，该模拟器并没有所有必要的特征来运行最近版本的 gingerbread。

#### 修复：

使用 R12 版本的模拟器，以及一个新的匹配工具的核心。不需要清理 build。

```
$ repo forall platform/external/qemu -c git checkout aosp/tools_r12
$ make
$ emulator -kernel prebuilt/android-arm/kernel/kernel-qemu-armv7
```

### 构建在 MacOS 10.7 Lion 上的模拟器不工作

#### 症状：

构建在 MacOS 10.7 Lion 或者 XCode 4.x 上的模拟器（任何版本）不工作。

#### 原因：

一些在开发环境的更改导致了模拟器不能从工作环境中编译。

#### 修复：

使用一个 SDK 中的二进制文件，该文件可以构建在用 XCode 6 的 MacOS 10.6 上，并且可以在 MacOS 10.7 上工作。

### *WITH_DEXPREOPT=true* 以及模拟器构建。

#### 症状：

在模拟器的构建中，当部分行为的构建以及同步（使系统没有依赖）时，结果构建没有生效。

#### 原因：

默认情况下，现在所有模拟器构建都会进行 Dex 优化，这就会导致每次框架改变都会，请求跟随所有的依赖用来重新优化应用。

#### 修复：

用 *export WITH_DEXPREOPT=false* 使本地的 Dex 优化失效，用 *make installclean* 删除已经存在的优化版本，然后运行一个完整的构造，重新生成没有优化的版本。在进行完上述操作之后，部分的构造将会开始工作。

### 构造过程中出现 *Permission Denied*

#### 症状：

所有的构建失败都伴随着 “Permission Denied”，可能伴随着反病毒警告。

#### 原因：

有些反病毒程序会将一些 Android 源码树中的文件错误的识别成包含病毒。

#### 修复：

经过确认没有病毒存在后，让反病毒程序在 Android 源码树中失效。这样做对减少构建次数有好处。

### 构建错误与使用错误的编译器

#### 特征：

构建错误有许多的特征。其中一个特征是： *cc1: error: unrecognized command line option "-m32"*

#### 原因：

在 PATH 中 Android 构建系统使用的是默认的编译器，假设在 host 中有合适的编译器生成的二进制文件。其他情况（比如使用 Android NDK 或者构建内核）将会导致默认的编译器不是主编译器。

#### 修复：

使用 “clean” 脚本，确保没有别的操作会更改默认的编译器。

### 由于无默认工具设置导致的构造错误

#### 特征：

构造失败有很多特征，可能是由于文件丢失或者文件格式错误。其中一个的特征是 *member [...] in archive is not an object*。

#### 原因：

安卓构建系统往往使用很多 host 工具并依赖于他们默认的行为。有些设置更改了工具的行为并使他们的行为干扰到了系统构建。已知会导致问题的变量是 *CDPATH* 和 *GREP_OPTIONS*

#### 修复：

在尽可能少的自定义环境中构建 Android。

### 在 MacOS 10.7 上构建4.0.x 以及之前版本的错误

#### 特征：

在 MacOS 10.7 上构建 IceCreamSandwich 4.0.x（和更老的版本）失败，并提示下面的错误信息： *Undefined symbols for architecture i386: "_SDL_Init"*

#### 原因：

在 MacOS 10.7 上不能适配 4.0.x

#### 修复：

降级电脑系统到 MacOS 10.6 或者是在 MacOS 10.7 上构造当前版本。

```
$ repo init -b master
$ repo sync
```

### 在 MacOS 上用 XCode 4.3 的构建错误

#### 特征：

当使用 XCode 4.3 时所有的构造都失败

#### 原因：

XCode 4.3 的默认编译器从 gcc 改成了 llvm，而 llvm 拒绝过去可以在 gcc 上通过的代码。

#### 修复：

使用 XCode 4.2

### 在 Ubuntu 11.10 上构造 4.0.x 或更早版本的错误

#### 特征：

在 Ubuntu 11.10 或之后的版本中构建 IceCreamSandwich 4.0.x（或者其之前的版本）会出现类似 *<command-line>:0:0: warning: "_FORTIFY_SOURCE" redefined [enabled by default]* 的错误

#### 原因：

Ubuntu 11.10 使用 gcc 标记是默认的，而 Android 也会定义一个默认的标记，这样导致了冲突。

#### 修复：

可以选择降级到 Ubuntu 10.04 或者 使用当前分支，这样就可以在 Ubuntu 11.10 或者之后的版本中使用。

```
$ repo init -b master
$ repo sync
```

## 源文件同步问题

### 复杂同步源代码（代理问题）

#### 特征：

在http 错误的时候 *repo* 或者是 *repo sync* 失败，通常是 403 或者是 500。

#### 原因

有很多引起错误的原因，其中最常见的是关联到了 http 代理，这回是传输大数据的时候变得很困难。

#### 修复：

暂时还没有通用解决方法，使用 python 2.7 以及明确的使用 *repo sync -jl*来改善一些用户的使用情况。

### 复杂同步源代码（VituralBox 以太网问题）

#### 特征：

当在一些 VituralBox 安装中运行 *repo sync*，进程伴随着可能的特征挂起或者失败。其中一种特征是：*DownloadError: HTTP 500 (Internal Server Error: Server got itself in trouble)*

#### 原因：

默认的 VirualBox 的网络行为是使用 NAT（Network Addrss Translation）连接客户系统来连接网络。繁重的网络行为将会引发 NAT 中的某些代码。

#### 修复：

配置 VirtualBox 让其使用桥连接来代替 NAT。

### 复杂同步源代码（NDS 问题）

#### 特征：

当运行 *repo sync* 时，由于无法识别 hostname 伴随着一些错误导致进程失败。其中一个错误是： *<urlopen error [Errno -2] Name or service not known>*

#### 原因：

有些 DNS 系统在应对大数量的请求时需要同步源码树而导致运行缓慢（在最坏的假设情况下可能有好几百请求）

#### 修复：

手工移除相关的 hostname ，并且在本地硬编码那些结果。

你可以用 **命令来移除，这可以给你一些列的数字 IP 地址（通常是在输出中的 “Address” 部分）。

```
$ nslookup googlesource.com
$ nslookup android.googlesource.com
```

之后你可以在本地硬编码他们到 */etc/hosts*，再在这个文件中以下面的格式添加两行：

```
aaa.bbb.ccc.ddd googlesource.com
eee.fff.ggg.hhh android.googlesource.com
```

需要注意的是这只在服务的地址没有改变的情况下，所以如果他们更改了地址而你又无法连接，这时候你就要重新的获取 hostname ，相应的你还要更改 *etc/hosts*。

### 复杂同步源代码（TCP 问题）

#### 特征：

在同步时 *repo sync* 挂起，通常都是在已经同步了 99% 的情况下。

#### 原因：

在 TCP/IP 堆的设置会使一些网络环境变的很糟糕，比如 *repo sync* 会既不编译，也不失败。

#### 修复：

在 Linux 上执行 *sysctl -w net.ipv4.tcp_window_scaling=0*。在 MacOS 上，使 rfc 1323 扩展失效。

## 运行问题

### 照相机和 GSP 在 Galaxy Nexus 上失效

#### 特征：

照相机和 GSP 在 Galaxy Nexus 上失效。举个例子，照相机应用已启动就崩溃。

#### 原因：

在Android开源项目中布提供这些硬件外设需要专有库。

#### 修复：

无。