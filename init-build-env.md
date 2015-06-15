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

`Note： 参阅 [下载与构建](https://source.android.com/source/building.html)了解对硬件环境和软件环境的需求。然后参考下面对 Ubuntu 和 Mac OS 下构建的详细说明。`                             

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

`Note: 在构建过程中的命令会确定使用的是 Sun JDK 而不是先前安装的 JDK。`                  

### 安装需要的包（Ubuntu 14.04）

你将需要一个64位版本的 Ubuntu 。推荐 Ubuntu 14.04。                

`$ sudo apt-get install bison g++-multilib git gperf libxml2-utils make zlib1g-dev:i386 zip`               

### 安装需要的包（Ubuntu 12.04）

你也许会使用 Ubuntu 12.04 来构建较早版本的 Android 。 master 分支和最近的发行都不支持 12.04 版本。             

`                       
$ sudo apt-get install git gnupg flex bison gperf build-essential \                   
  zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \                  
  libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 \                   
  libgl1-mesa-dev g++-multilib mingw32 tofrodos \                          
  python-markdown libxml2-utils xsltproc zlib1g-dev:i386                              
$ sudo ln -s /usr/lib/i386-linux-gnu/mesa/libGL.so.1 /usr/lib/i386-linux-gnu/libGL.so                 
`                  

### 安装需要的包（Ubuntu 10.04 -- 11.10）

在 Ubuntu 10.04-11.10 上构建已不在被支持，但仍可以用来构建较早的 AOSP。            

`$ sudo apt-get install git gnupg flex bison gperf build-essential \                        
   zip curl zlib1g-dev libc6-dev lib32ncurses5-dev ia32-libs \                   
   x11proto-core-dev libx11-dev lib32readline5-dev lib32z-dev \                     
   libgl1-mesa-dev g++-multilib mingw32 tofrodos python-markdown \                        
   libxml2-utils xsltproc`                    

在 Ubuntu 10.10 中：

`$ sudo ln -s /usr/lib32/mesa/libGL.so.1 /usr/lib32/mesa/libGL.so`            

在 Ubuntu 11.10 中：

`$ sudo apt-get install libx11-dev:i386`                  

### 配置 USB 接口 
(Congiguring USB Access)