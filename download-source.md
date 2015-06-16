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

 