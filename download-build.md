# 下载与构建

通常内部测试 Android 的构建是在最近一个版本的 Ubuntu LTS（14.04）中， 但是大多数的发行版本都具有构建所需的工具。我们非常欢迎收到在其它版本上尝试的结果反馈，无论是成功，还是失败。            

在你下载并构建 Android 源码之前，请确定你的系统满足以下需求：                     

- 请选择 Linux 或 Mac OS 系统， 或者也可以在像 windows 等暂时不支持的系统中使用虚拟机去构建 Android，如果你在虚拟机中运行 Linux ，你至少需要 16GB 的 RAM/swap 和 100GB 以上的存储空间来构建 Android tree。具体磁盘大小需求请参阅下文。            

- 对于 Gingerbread （2.3.x） 及以上版本，包括 master 分支都需要一个64位的环境，你可以在32位系统上编译它之前的版本、       

- 至少有 100GB 的磁盘空间用来检查，150GB 用来单编译，还需要 200GB 以上的空间用来进行交叉编译。如果你采用 ccache ，你可能会需要更多的空间。                        

- Python 2.6-2.7 ，你可以从 [python.org](https://www.python.org/downloads/) 上下载。               

- GNU Make 3.81-3.82，你可以从 [gnu.org](http://ftp.gnu.org/gnu/make/) 上下载。                                      

- JDK7 用来构建 Android Open Source Project（AOSP） 的 master 分支；JDK6 用来构建 Gingerbread 到 Kitkat 之间的版本，JDK5 用来构建 Cupcake 到 Froyo 之间的版本，操作系统安装说明参见 [初始化编译环境](installation instructions by operating system)。                   

- Git 1.7 或更新版本，你可以在 [git-scm.com](http://git-scm.com/download) 中找到它。                       


