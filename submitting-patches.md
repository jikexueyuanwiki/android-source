# 提交补丁

这个页面将描述提交补丁到 AOSP 的全过程，包括校验和 [Gerrit](https://android-review.googlesource.com/) 追踪更改。

## 前提条件

- 在按照说明学习本页面内容之前，你需要[初始化构建环境](https://source.android.com/source/initializing.html)，[下载源码](https://source.android.com/source/downloading.html)，[创建密码](https://android.googlesource.com/new-password)和按照密码生成页面的说明学习。

- 可以在 [developing](https://source.android.com/source/developing.html) 章节查看 Repo 和 Git 的详细信息。
- 可以在 [Project roles](https://source.android.com/source/roles.html) 章节查看你可以在开源社区扮演的各种角色信息。
- 如果你打算在 Android 平台奉献代码，应该先阅读 [AOSP 的权限声明信息](https://source.android.com/source/licenses.html)。
- 注意更改一些使用 Android 的上游项目，你应该给这个项目创建一个目录，像 [Upstream Projects](https://source.android.com/source/submit-patches.html#upstream-projects) 中所说的目录。

## 奉献者

### 与服务器进行身份验证

在你上传内容到 Gerrit 的时候，你需要[创建一个与服务器认证的密码](https://android.googlesource.com/new-password)。按照 password generator 页面的说明。你只要在第一次的使用的时候这样做。查看 [Using Authentication](https://source.android.com/source/downloading.html#using-authentication) 补充说明。

### 从一个 Repo 分支开始

在你有意向更改的地方，可以在相关联的 git 库中新建一个分支：

```
$ repo start NAME .
```

你可以在同一个库同一时间新建几个独立的分支。这个分支的名字是在你本地的工作空间，不包含在 gerrit 或者最终的源码分支树中。

### 更改

Once you have modified the source files (and validated them, please) commit the changes to your local repository:

一但你更改了源码里面的文件（请验证它们），在本地库中 commit 这个改变：

```
$ git add -A
$ git commit -s
``` 

在 commit 的消息中，写上你更改信息的详细描述。这个描述信息将会 push 到公共的 AOSP 库，所以请按照我们的指导来写更改列表的描述信息：



