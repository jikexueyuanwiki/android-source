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

1.17 版本，repo 的 SHA-1 校验值是ddd79b6d5a7807e911b524cb223bc3544b661c28           
1.19 版本，repo 的 SHA-1 校验值是 92cbad8c880f697b58ed83e348d06619f8098e6c            
1.20 版本，repo 的 SHA-1 校验值是 e197cb48ff4ddda4d11f23940d316e323b29671c           
1.21 版本，repo 的 SHA-1 校验值是 b8bd1804f432ecf1bab730949c82b93b0fc5fede            

## 初始化一个 Repo 客户端

安装完 Repo 之后，建起一个客户端来访问 Android 的源码仓库：         

1. 创建一个空目录来存放你的工作文件。如果你使用的是 MacOS，则这个目录需要在一个区分大小写的文件系统上。可以以任何你喜欢的名称命名：                 

 ```
 $ mkdir WORKING_DIRECTORY
 $ cd WORKING_DIRECTORY
 ```               

2. 运行`repo init`更新最新版本的 Repo 该版本已经修复了大量已知 Bug。你必须为 manifest 指定一个 URL 、，它指定了 Android 源码树里的各个仓库都会被存放在你工作的目录下。                  

 ```
 $ repo init -u https://android.googlesource.com/platform/manifest
 ```         

 要查看 "master" 以外的分支，用 -b 来指定。想查看分支列表，参阅 [Source Code Tags and Builds](https://source.android.com/source/build-numbers.html#source-code-tags-and-builds) 。         

 ```
 $ repo init -u https://android.googlesource.com/platform/manifest -b android-4.0.1_r1
 ```

3. 出现提示时，在 Repo 中配置你的真实姓名和 email 地址。要使用 Gerrit 代码审查工具，你可能会需要一个注册过 Google 账户的邮箱地址。请确保你能通过这个邮箱地址收到消息。这将作为你贡献代码的署名出现。                  

成功初始化的时候会显示这样的消息状态－ Repo 已经在你的工作目录下完成了初始化。你的客户端目录下应该会包含一个 .repo 目录用来存放像 manifest 一类的文件。                    

## 下载 Android 源代码树

要将 manifest 中默认指定的 Android 代码树拉取到你的工作目录下，请运行：            

```
$ repo sync
```         

你的工作目录下会有对应的工程名并存储了 Android 源码文件。这个初始化同步操作大概会需要一个小时或更多的时间才能完成。想了解更多关于 Repo sync 的信息和 Repo 的其它指令，请参阅 [Developing](https://source.android.com/source/developing.html) 章节。                        

## 使用认证

通常情况下，访问 Android 源码资源都是匿名的。为了防止服务器负荷过大，每个 IP 地址都关联一个 quota。                     

当和他们共享同一个 IP 地址时（比如访问代码仓库时越过 NAT 防火墙），即使在正常模式下 quotas 也会被触发（例如许多用户在较短时段里从同一个 IP 地址下创建新客户端并发起同步请求）。               

在这种情况下，可以使用授权来访问，每个用户将会使用一个独立的与 IP 地址无关的 quota。            

第一步首先是使用[密码生成器](https://android.googlesource.com/new-password)，然后按照页面上的说明进行操作。            

第二步是通过使用 https://android.googlesource.com/a/platform/manifest 这个 manifest URL 来进行强制授权访问。注意 /a/ 目录如何进行前缀强制触发认证。你可以使用下面的指令进行强制认证来转化你的客户端： 
            
```
$ repo init -u https://android.googlesource.com/a/platform/manifest
```              

## 排除网络问题

当通过代理下载的时候（通常企业经常使用），需要去用 repo 明确地指定代理：                   

```
$ export HTTP_PROXY=http://<proxy_user_id>:<proxy_password>@<proxy_server>:<proxy_port>
$ export HTTPS_PROXY=http://<proxy_user_id>:<proxy_password>@<proxy_server>:<proxy_port>
```                   

更少见的是，Linux 客户端遇到连接问题，下载到一半的时候中止（典型的就是在“正在接受数据”过程中）。据报道调整 TCP/IP 堆的设置并使用非平行的命令可以改善这个问题。你需要 root 权限访问并修改 TCP 设定：                    

```
$ sudo sysctl -w net.ipv4.tcp_window_scaling=0
$ repo sync -j1
```             

## 使用一个本地镜像

当使用多个客户端时，尤其是在带宽有限的情况下，最好在本地对整个服务器内容创建一个镜像，然后从这个镜像同步到客户端（这样就不需要网络访问权限）。当包含更大量内容的时候，下载一个完整镜像和两个客户端分别同时进行下载是大致相同的。                       

这些说明假设镜像在 /usr/local/aosp/mirror 里被创建。第一步是创建一个镜像然后对自己进行同步。注意 --mirror 标识只可以在创建一个新客户端的时候被指定：                       

```
$ mkdir -p /usr/local/aosp/mirror
$ cd /usr/local/aosp/mirror
$ repo init -u https://android.googlesource.com/mirror/manifest --mirror
$ repo sync
```                      

一旦镜像被同步，新的客户端就可以从中创建，注意一定要指定一个绝对路径：                    

```
$ mkdir -p /usr/local/aosp/master
$ cd /usr/local/aosp/master
$ repo init -u /usr/local/aosp/mirror/platform/manifest.git
$ repo sync
```                         

最后，客户端要对服务器进行同步，镜像需要对客户端进行同步，然后客户端同步镜像：             

```
$ cd /usr/local/aosp/mirror
$ repo sync
$ cd /usr/local/aosp/master
$ repo sync
```                       

可以将镜像存储在一个局域网服务器上，然后通过 NFS，SSH 或者 Git 访问它。同样也可以将它存储在可移动设备上，然后在用户或者机器直接传递它。                 

## 验证 Git 标记

加载下面的公钥到你的 GunPG 密钥库。这个密钥用来进行标签注释并签署发布。                 

```
$ gpg --import
```                      

复制粘贴下面的密钥，然后键入 EOF（或者 Ctrl-D）来结束输入病保存这个密钥。               

```
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v1.4.2.2 (GNU/Linux)

mQGiBEnnWD4RBACt9/h4v9xnnGDou13y3dvOx6/t43LPPIxeJ8eX9WB+8LLuROSV
lFhpHawsVAcFlmi7f7jdSRF+OvtZL9ShPKdLfwBJMNkU66/TZmPewS4m782ndtw7
8tR1cXb197Ob8kOfQB3A9yk2XZ4ei4ZC3i6wVdqHLRxABdncwu5hOF9KXwCgkxMD
u4PVgChaAJzTYJ1EG+UYBIUEAJmfearb0qRAN7dEoff0FeXsEaUA6U90sEoVks0Z
wNj96SA8BL+a1OoEUUfpMhiHyLuQSftxisJxTh+2QclzDviDyaTrkANjdYY7p2cq
/HMdOY7LJlHaqtXmZxXjjtw5Uc2QG8UY8aziU3IE9nTjSwCXeJnuyvoizl9/I1S5
jU5SA/9WwIps4SC84ielIXiGWEqq6i6/sk4I9q1YemZF2XVVKnmI1F4iCMtNKsR4
MGSa1gA8s4iQbsKNWPgp7M3a51JCVCu6l/8zTpA+uUGapw4tWCp4o0dpIvDPBEa9
b/aF/ygcR8mh5hgUfpF9IpXdknOsbKCvM9lSSfRciETykZc4wrRCVGhlIEFuZHJv
aWQgT3BlbiBTb3VyY2UgUHJvamVjdCA8aW5pdGlhbC1jb250cmlidXRpb25AYW5k
cm9pZC5jb20+iGAEExECACAFAknnWD4CGwMGCwkIBwMCBBUCCAMEFgIDAQIeAQIX
gAAKCRDorT+BmrEOeNr+AJ42Xy6tEW7r3KzrJxnRX8mij9z8tgCdFfQYiHpYngkI
2t09Ed+9Bm4gmEO5Ag0ESedYRBAIAKVW1JcMBWvV/0Bo9WiByJ9WJ5swMN36/vAl
QN4mWRhfzDOk/Rosdb0csAO/l8Kz0gKQPOfObtyYjvI8JMC3rmi+LIvSUT9806Up
hisyEmmHv6U8gUb/xHLIanXGxwhYzjgeuAXVCsv+EvoPIHbY4L/KvP5x+oCJIDbk
C2b1TvVk9PryzmE4BPIQL/NtgR1oLWm/uWR9zRUFtBnE411aMAN3qnAHBBMZzKMX
LWBGWE0znfRrnczI5p49i2YZJAjyX1P2WzmScK49CV82dzLo71MnrF6fj+Udtb5+
OgTg7Cow+8PRaTkJEW5Y2JIZpnRUq0CYxAmHYX79EMKHDSThf/8AAwUIAJPWsB/M
pK+KMs/s3r6nJrnYLTfdZhtmQXimpoDMJg1zxmL8UfNUKiQZ6esoAWtDgpqt7Y7s
KZ8laHRARonte394hidZzM5nb6hQvpPjt2OlPRsyqVxw4c/KsjADtAuKW9/d8phb
N8bTyOJo856qg4oOEzKG9eeF7oaZTYBy33BTL0408sEBxiMior6b8LrZrAhkqDjA
vUXRwm/fFKgpsOysxC6xi553CxBUCH2omNV6Ka1LNMwzSp9ILz8jEGqmUtkBszwo
G1S8fXgE0Lq3cdDM/GJ4QXP/p6LiwNF99faDMTV3+2SAOGvytOX6KjKVzKOSsfJQ
hN0DlsIw8hqJc0WISQQYEQIACQUCSedYRAIbDAAKCRDorT+BmrEOeCUOAJ9qmR0l
EXzeoxcdoafxqf6gZlJZlACgkWF7wi2YLW3Oa+jv2QSTlrx4KLM=
=Wi5D
-----END PGP PUBLIC KEY BLOCK-----
```                        

输入完密钥，你可以使用下面的指令校验任何标记                      

```
$ git tag -v TAG_NAME
```                  

如果你还没有[搭建 ccache](https://source.android.com/source/initializing.html#ccache)，现在是完成他的最好时机。