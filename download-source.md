# 下载源码

 Android 的源代码树在 Google 托管的 Git 仓库中。本文主要阐述如何下载源码树中具体的 Android 代码。              

 ## 安装 Repo

 Repo 是一个方便在安卓中使用 Git 的工具。想了解更多关于 Repo 的信息，请参阅 [Developing](https://source.android.com/source/developing.html) 章节。             

 如何安装 Repo：            

 1. 确保在你的主目录下有一个 bin/ 目录并且它包含在你的路径中：         

 ```
 $ mkdir ~/bin
 $ PATH=~/bin:$PATH
 ```               

 2. 下载 Repo 工具并确保它是可执行的：           

 ```
 $ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
 $ chmod a+x ~/bin/repo
 ```              

 1.17 版本， repo 的 SHA-1 校验值是 ddd79b6d5a7807e911b524cb223bc3544b661c28           
 1.19 版本， repo 的 SHA-1 校验值是 92cbad8c880f697b58ed83e348d06619f8098e6c            
 1.20 版本， repo 的 SHA-1 校验值是 e197cb48ff4ddda4d11f23940d316e323b29671c           
 1.21 版本， repo 的 SHA-1 校验值是 b8bd1804f432ecf1bab730949c82b93b0fc5fede            

 ## 初始化一个 Repo 客户端

 安装完 Repo 之后，建起一个客户端来访问 Android 的源码仓库：         

 1. 创建一个空目录来存放你的工作文件。如果你使用的是 MacOS , 则这个目录需要在一个区分大小写的文件系统上。可以以任何你喜欢的名称命名：                 

 ```
 $ mkdir WORKING_DIRECTORY
 $ cd WORKING_DIRECTORY
 ```               

 2. 运行 ` repo init ` 更新最新版本的 Repo 该版本已经修复了大量已知 Bug 。你必须为 manifest 指定一个 URL ，它指定了 Android 源码树里的各个仓库都会被存放在你工作的目录下。                  

 ```
 $ repo init -u https://android.googlesource.com/platform/manifest
 ```         

    要查看 "master" 以外的分支，用 -b 来指定。想查看分支列表，参阅 [Source Code Tags and Builds](https://source.android.com/source/build-numbers.html#source-code-tags-and-builds) 。         

 ```
 $ repo init -u https://android.googlesource.com/platform/manifest -b android-4.0.1_r1
 ```

 3. 出现提示时，在 Repo 中配置你的真实姓名和 email 地址。要使用 Gerrit 代码审查工具，你可能会需要一个注册过 Google 账户的邮箱地址。请确保这个你能通过这个邮箱地址收到消息。这将作为你贡献代码的署名出现。                  

 成功初始化的时候会显示这样的消息状态－ Repo 已经在你的工作目录下完成了初始化。你的客户端目录下应该会包含一个 .repo 目录用来存放像 manifest 一类的文件。                    

 ## 下载 Android 源代码树

 要将 manifest 中默认指定的 Android 代码树拉取到你的工作目录下，请运行：            

 ```
 $ repo sync
 ```         

 你的工作目录下会有对应的工程名并存储了 Android 源码文件。这个初始化同步操作大概会需要一个小时或更多的时间才能完成。想了解更多关于 Repo sync 的信息和 Repo 的其它指令，请参阅 [Developing](https://source.android.com/source/developing.html) 章节。                        

 # 使用认证

 通常情况下，访问 Android 源码资源都是匿名的。为了防止服务器负荷过大，每个 IP 地址都关联一个 quota 。                     

 当和他们共享同一个 IP 地址时（比如访问代码仓库时越过以恶搞 NAT 防火墙），即使在正常模式下 quotas 也会被触发（例如许多用户在较短时段里从同一个 IP 地址下创建新客户端并发起同步请求）。               

 在这种情况下，可以使用授权来访问，每个用户将会使用一个独立的与 IP 地址无关的 quota 。            

 第一步首先是使用[密码生成器](https://android.googlesource.com/new-password)，然后按照页面上的说明进行操作。            

 第二步是通过使用 https://android.googlesource.com/a/platform/manifest 这个 manifest URL 来进行强制授权访问。注意 /a/ 目录如何进行前缀强制出发认证。你可以使用下面的指令进行强制认证来转化你的客户端：             
 ```
 $ repo init -u https://android.googlesource.com/a/platform/manifest
 ```              

## Troubleshooting network issues