# 初始化编译环境

这个章节阐述如何建立起你的本地工作环境来构建 Android 源文件。你需要使用 Linux 或 Mac OS 操作系统。目前暂不支持在 windows 下编译。                   

想对整个代码审查和代码更新过程有个整体的了解，请参阅 [Life of a Patch](https://source.android.com/source/life-of-a-patch.html)。          

## 选择一个分支

部分构建环境的需求取决于你打算编译的源码版本，查看 [Build Numbers](https://source.android.com/source/build-numbers.html) 包含一个完整的分支列表供你选择。你也许想选择下载最新的源码（分支）进行构建，在这种情况下你可以在初始化仓库的时候简单的忽略分支规范。           

一旦你选择了一个分支，参考下面相应的说明来搭建你的编译环境。         

## 搭建一个 Linux 下的编译环境

这些指令适用于所有分支，包括 master 分支。             

通常内部测试 Android 的构建是在最近一个版本的 Ubuntu LTS （14.04）中，但是大多数发行版本都应该具备构建所需的工具，欢迎反馈您在其他版本上的尝试结果，无论成功还是失败。                

Gingerbread （2.3.x） 或以上版本，包括 master 分支，都需要一个64位的环境，它之前的版本可以在32位系统下进行编译。            

`提示： 参阅 [下载与构建](https://source.android.com/source/building.html)了解对硬件环境和软件环境的需求。然后参考下面对 Ubuntu 和 Mac OS 下构建的详细说明。`                             

### 安装JDK

[Android Open Source Project](https://android.googlesource.com/)（AOSP）的 master 分支需要 Java 7。在 Ubuntu 下可以使用 [OpenJDK](http://openjdk.java.net/install/)。              

Java 7：适用于最新版本的 Android                

`                          
 $ sudo apt-get update                           
 $ sudo apt-get install openjdk-7-jdk                    
`                       

也可以运行如下指令升级默认的 Java 版本：           

`                   
 sudo update-alternatives --config java                  
 sudo update-alternatives --config javac                 
`                   

如果你遇到 Java 版本错误，参考 [wrong-java-version](https://source.android.com/source/building-running.html#wrong-java-version) 中描述的来设置它的路径。                  

开发较早版本的 Android ，下载安装相应的 [Java JDK](http://www.oracle.com/technetwork/java/javase/archive-139210.html)：              
Java 6：适用 Gingerbread 到 KitKat 的版本                      
Java 5：适用 Cupcake 到 Froyo 的版本                          

`提示: 在构建过程中的命令会确定使用的是 Sun JDK 而不是先前安装的 JDK。`                  

### 安装需要的包（Ubuntu 14.04）

你将需要一个64位版本的 Ubuntu 。推荐 Ubuntu 14.04。                

`$ sudo apt-get install bison g++-multilib git gperf libxml2-utils make zlib1g-dev:i386 zip`               

### 安装需要的包（Ubuntu 12.04）

你也许会使用 Ubuntu 12.04 来构建较早版本的 Android 。 master 分支和最近的发行都不支持 12.04 版本。             

```                       
$ sudo apt-get install git gnupg flex bison gperf build-essential \                   
  zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \                  
  libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 \                   
  libgl1-mesa-dev g++-multilib mingw32 tofrodos \                          
  python-markdown libxml2-utils xsltproc zlib1g-dev:i386                              
$ sudo ln -s /usr/lib/i386-linux-gnu/mesa/libGL.so.1 /usr/lib/i386-linux-gnu/libGL.so                 
```                             

### 安装需要的包（Ubuntu 10.04 -- 11.10）

在 Ubuntu 10.04-11.10 上构建已不在被支持，但仍可以用来构建较早的 AOSP。            

```
 $ sudo apt-get install git gnupg flex bison gperf build-essential \                        
   zip curl zlib1g-dev libc6-dev lib32ncurses5-dev ia32-libs \                   
   x11proto-core-dev libx11-dev lib32readline5-dev lib32z-dev \                     
   libgl1-mesa-dev g++-multilib mingw32 tofrodos python-markdown \                        
   libxml2-utils xsltproc
```                   

在 Ubuntu 10.10 中：

```
$ sudo ln -s /usr/lib32/mesa/libGL.so.1 /usr/lib32/mesa/libGL.so
```            

在 Ubuntu 11.10 中：

```
$ sudo apt-get install libx11-dev:i386
```                 

### 配置 USB 接口 

在 GNU/Linux systems 系统下，（尤其是 Ubuntu 系统），通常用户在默认情况下不能直接读取 USB 设备。我们要对系统进行配置才能允许这种操作。                     

推荐的做法是创建一个文件 /etc/udev/rules.d/51-android.rules （作为 root 用户），接着将下面对内容复制上去。 <username> 必须被替换成实际被授权通过 USB 访问手机的用户名。                      

```                  
# adb protocol on passion (Nexus One)               
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e12", MODE="0600", OWNER="<username>"           
# fastboot protocol on passion (Nexus One)                 
SUBSYSTEM=="usb", ATTR{idVendor}=="0bb4", ATTR{idProduct}=="0fff", MODE="0600", OWNER="<username>"              
# adb protocol on crespo/crespo4g (Nexus S)               
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e22", MODE="0600", OWNER="<username>"            
# fastboot protocol on crespo/crespo4g (Nexus S)          
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e20", MODE="0600", OWNER="<username>"            
# adb protocol on stingray/wingray (Xoom)                
SUBSYSTEM=="usb", ATTR{idVendor}=="22b8", ATTR{idProduct}=="70a9", MODE="0600", OWNER="<username>"            
# fastboot protocol on stingray/wingray (Xoom)                
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="708c", MODE="0600", OWNER="<username>"              
# adb protocol on maguro/toro (Galaxy Nexus)              
SUBSYSTEM=="usb", ATTR{idVendor}=="04e8", ATTR{idProduct}=="6860", MODE="0600", OWNER="<username>"            
# fastboot protocol on maguro/toro (Galaxy Nexus)            
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e30", MODE="0600", OWNER="<username>"              
# adb protocol on panda (PandaBoard)                
SUBSYSTEM=="usb", ATTR{idVendor}=="0451", ATTR{idProduct}=="d101", MODE="0600", OWNER="<username>"             
# adb protocol on panda (PandaBoard ES)                  
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="d002", MODE="0600", OWNER="<username>"              
# fastboot protocol on panda (PandaBoard)                 
SUBSYSTEM=="usb", ATTR{idVendor}=="0451", ATTR{idProduct}=="d022", MODE="0600", OWNER="<username>"             
# usbboot protocol on panda (PandaBoard)                   
SUBSYSTEM=="usb", ATTR{idVendor}=="0451", ATTR{idProduct}=="d00f", MODE="0600", OWNER="<username>"              
# usbboot protocol on panda (PandaBoard ES)                
SUBSYSTEM=="usb", ATTR{idVendor}=="0451", ATTR{idProduct}=="d010", MODE="0600", OWNER="<username>"              
# adb protocol on grouper/tilapia (Nexus 7)                  
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e42", MODE="0600", OWNER="<username>"               
# fastboot protocol on grouper/tilapia (Nexus 7)                  
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e40", MODE="0600", OWNER="<username>"            
# adb protocol on manta (Nexus 10)                    
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4ee2", MODE="0600", OWNER="<username>"              
# fastboot protocol on manta (Nexus 10)                
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4ee0", MODE="0600", OWNER="<username>"          
```                            

这些规则会在下次插入设备的时候生效。因此可能需要先拔出设备再插回去。               

知道这些就可以在 Ubuntu Hardy Heron (8.04.x LTS) 和 Lucid Lynx (10.04.x LTS) 上工作。其他版本的 Ubuntu 或其他 GNU/Linux 的发行版本需要不同的设置。               

### 使用一个单独的输出目录

默认情况下，每个构建出来的结果都存储在系统资源目录的子文件夹 out/ 目录下。               

在一些拥有多个存储设备的机器上。构建在存储源文件并输出到单独卷的时候会进行的较快。还有其他一些性能，比如可以将输出存储到一个经过速度优化的文件系统中并且不会造成稳健性崩溃，因为所有文件都可以在文件损坏的情况下被重新生成。                   

要使用这种功能，配置输出 OUT_DIR_COMMON_BASE 变量指向到你存储输出数据的地方。          
```
export OUT_DIR_COMMON_BASE=<path-to-your-out-directory>
```    

每个源代码树的输出目录都将在文件夹创建后以源码树命名。             

举个例子，如果你现在有一些源代码树，比如 /source/master1 和 /source/master2 并且 OUT_DIR_COMMON_BASE 设置为 /output ，输出目录就会是 /output/master1 和 /output/master2。                 

有一个重要的情况就是不能将多个源代码树输出到同一个目录并且具有相同的名称，否则他们会在同时使用同一个输出目录的时候出现不可预知的结果，然后终止。             

这些仅支持 Jelly Bean (4.1) 及以上版本，包括 master 分支。          

## 在 Mac OS 下搭建编译环境

在默认安装下， Mac OS 运行在一个保留但不区分大小写的文件系统上。这种文件系统并不被 git 支持而且一些 git 指令 （比如 git status ）表现异常。出于此原因，我们推荐你在一个区分大小写的文件系统上工作使用 AOSP 源代码树。 使用磁盘映像可以非常容易地达到目的，讨论如下。              

一旦合适的文件系统准备好，在时尚的 Mac OS 环境下构建 master 分支会变得非常容易。较早的一些分支，包括 ICS ,需要一些额外的工具和 SDK 。            

### 创建一个区分大小写的磁盘映像

你可以用你的 Mac OS 环境下正在使用对磁盘映像创建一个区分大小写的文件系统。为了创建这个映像，启动磁盘工具并选择 " New Image " 。完成构建最少需要 25GB ；以后可能会需要更多。使用分散的映像可以节省空间并且以后可以根据需要增大。请确定选择 “区分大小写，日志”作为卷格式。                 

你也可以在 shell 里输入下面的命令来创建它。                

```
# hdiutil create -type SPARSE -fs 'Case-sensitive Journaled HFS+' -size 40g ~/android.dmg
```               

这会创建一个 .dmg （或者 .dmg.sparsefile ）文件，一旦被安装，将成为具有安卓开发所需格式的驱动器。           

如果你以后需要更大的空间，你可以使用下面的指令来重新设置分散映像的大小。             
```
# hdiutil resize -size <new-size-you-want>g ~/android.dmg.sparseimage
```                    

一个名为 android.dmg 的地盘映像被安装在你的主目录下，你可以在 ~/.bash_profile 文件中添加一个辅助功能模块。            

- 执行 mountAndroid 的时候安装这个镜像文件：          

```
# mount the android file image               
  function mountAndroid { hdiutil attach ~/android.dmg -mountpoint /Volumes/android; }
```                     

  注意：如果你的系统创建了一个 .dmg.sparsefile 文件，请将 -/android.dmg 替换为 -/android.dmg.sparsefile 。               

- 执行 umountAndroid 的时候卸载它： 
```             
  # unmount the android file image                     
    function umountAndroid() { hdiutil detach /Volumes/android; }
```             

一旦你安装了 android 卷，你可以在这里进行你所有的工作。也同样可以像一个外部驱动器一样直接弹出（卸载）它。                   

### 安装 JDK

[ Android 开源工程 (AOSP)](https://android.googlesource.com/) 中 Android 的 master 和 5.0.x 分支需要 Java 7。在 Mac OS 中，使用 [jdk-7u71-macosx-x64.dmg](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html#jdk-7u71-oth-JPR)。               

要开发 Android Gingerbread 到 KitKat 之间的版本，下载并安装 [Java JDK](http://support.apple.com/kb/dl1572) 的 Java 6 版本。                

### master 分支

要在 Mac OS 环境下构建最新版本的源代码，你需要一台具备 Intel/x86 的机器并且系统在 Mac OS X v10.8 （Mountain Lion）或以上， Xcode 版本在4.5.2以上并包含命令行工具。               

### 5.0.x 及其之前的分支

要在 Mac OS 环境下构建 5.0.x 及其之前的源码，你需要一台具备 Intel/x86 的机器并且系统在 Mac OS X v10.8 （Mountain Lion）或以上， Xcode 版本在4.5.2以上并包含命令行工具。                  

### 4.4.x 及其之前的分支

要在 Mac OS 环境下构建 4.2.x 及其之前的源代码， 你需要一台 Intel/x86 的机器并且系统为 Mac OS X v10.6 (Snow leopard) 或 Mac OS X v10.7 (Lion)，具备 Xcode 4.2 （苹果开发者工具）。尽管 Lion 系统并不事先具备 JDK，但它会在你尝试构建源码的时候自动安装。                    

剩下的关于 Mac OS 的内容只针对那些希望构建较早分支的用户。              

### 4.0.x 及其之前的分支

要在 Mac OS 环境下构建 android-4.0.x 及其之前的版本分支，你需要一台 Intel/x86 的机器并且运行 Mac OS X v10.5 (Leopard) 系统或 Mac OS X v10.6 (Snow Leopard) 系统，同时你将需要 Mac OS X v10.5 的 SDK 。              

#### 安装需要的包

- 从 [ Apple 开发者平台](https://developer.apple.com/)获取安装 Xcode。我们推荐 3.1.4 或更新版本 （比如 gcc 4.2）。4.x 的版本可能会造成困难。如果你还没有准备好注册成为 Apple 开发者，则需要创建一个 Apple ID 来进行下载。            

- 从 [macports.org](http://www.macports.org/install.php)获取 MacPorts 的安装。                 
  注意：确保在你的 /usr/bin 路径之前存在 /opt/local/bin 路径。如果没有，请将下面的内容添加到你的 ~/.bash_profile 文件中：                

```
export PATH=/opt/local/bin:$PATH
```                 

  注意：如果在你的主目录下没有 .bash_profile 文件，那就创建一个。              

- 从 MacPorts 上获取 make , git 和 GPG 包的安装：               

```
$ POSIXLY_CORRECT=1 sudo port install gmake libsdl git gnupg
```                

如果使用的是 Mac OS X v10.4 ，还需要安装 bison：             

```
$ POSIXLY_CORRECT=1 sudo port install bison
```                        

#### 从 make 3.82 中恢复

在 Android ICS 之前的版本，存在一个 bug 阻止了 android 的构建。你可以按照下面的步骤使用 MacPorts 安装 3.81 版本：                 

- 编辑 /opt/local/etc/macports/sources.conf 并添加下面一行内容   

```
file:///Users/Shared/dports
```            

添加完上面一行。之后创建这样一个目录： 

```
$ mkdir /Users/Shared/dports
```                 

- 在新的 dports 目录中，运行     

```
$ svn co --revision 50980 http://svn.macports.org/repository/macports/trunk/dports/devel/gmake/ devel/gmake/
```               

- 为你的本地仓库创建一个端口：         

```
$ portindex /Users/Shared/dports
```            

- 最后，输入下面的指令安装旧版的 gmake         

```
$ sudo port install gmake @3.81
```               

#### 设置文件描述限制符

在 Mac OS 上，默认的文件同时打开数量值太低，所以一个高并发的构建会超出这个限制。          

为了增加这个上限，将下面的一行添加到你的 -/.bash_profile 文件中： 

```
# set the number of open files to be 1024
ulimit -S -n 1024
```                  

## 优化构建环境（可选）

### 建立 ccache

你可以选择性的让构建使用 ccache 汇编工具。 Ccache 作为一个编译缓存器可以用来为重构建提速。如果你经常使用 make clean ，或者经常切换不同的工程进行构建的话，这将会非常好用。                  

在你的 .bashrc （或其它同类文件）中添加下面一行：   

```
export USE_CCACHE=1
```                    

默认情况下 cache 会被存储在 ~/.ccache 下。如果你的主目录在 NFS 活着其它非本地系统上，你也同样需要在你的 .bashrc 文件中指定目录。         

```
export CCACHE_DIR=<path-to-your-cache-directory>
```                     

建议缓存大小设为 50 － 100 GB 之间。在你下载好源代码之后运行下面的指令：               

```
prebuilts/misc/linux-x86/ccache/ccache -M 50G
```                   

在 Mac OS 中，你应该将 linux-x86 替换为 darwin-x86 ：               

```
prebuilts/misc/darwin-x86/ccache/ccache -M 50G
```                    

在构建 Ice Cream Sandwich (4.0.x) 或更早版本的时候，ccache 会在一个不同的位置下：                

```
prebuilt/linux-x86/ccache/ccache -M 50G
```               

这些设置被存储在 CCACHE_DIR 并且一直生效。                   

你的编译环境已经准备好了，接下来[下载源代码吧](https://source.android.com/source/downloading.html)。