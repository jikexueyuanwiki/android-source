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

- 开始是一行摘要（最多60字），接着空一行。这样的格式在 git 和 gerrit 中用于各种显示。

	```
	short description on first line

	more detailed description of your patch,
	which is likely to take up multiple lines.
	```

- 这个描述信息应该着重说明解决了什么问题，怎么解决的。第二部分尽管是值得满意的，但是实现新功能的时候还是比较随意的。

- 写入一些简短的假定或者背景信息，这些内容可能对明年致力于这个功能的奉献者很重要。

在执行 repo init 后得到的独特的 change ID，名称和邮箱将会自动添加到你的 commit 信息中。

### 上传到 gerrit

一旦 commit 你的更改到个人历史，就可以上传到 gerrit。 

```
$ repo upload
```

如果你在同一个库中选择了多个分支，你将会得到提示，让你选择一个分支上传。

上传成功之后，repo 将会给你提供一个 Gerrit 新页面的 URL。预览这个链接并在评审服务器上查看你的补丁，添加评论，或者给你的补丁请求特定的评审员。

### 上传一个代替的补丁

假如一个评审员已经看了你的补丁，并且要求你做一些小的更改。你可以通过 git 修改 commit 信息，这样将会在 gerrit 中生成一个新的补丁，但是使用的是原先那个 change ID。

*注意，如果你在上传补丁之前添加了其他 commits，你将需要手动移动你的 git HEAD。*

```
$ git add -A
$ git commit --amend
```

当你上传了修改的补丁，它将在 gerrit 和本地 git 历史中代替之前的。

### 解决同步冲突

如果提交到源码中的其他补丁和你的有冲突，你将需要在源码库新 HEAD 的顶部重新定义你的补丁版本信息。一个简单的方法就是运行：

```
$ repo sync
```

这个命令将会从服务器先 fetches 最新的源码，然后会试图自动定义你的  HEAD 分支版本库状态到新的远程 HEAD 上面。

如果自动定义没有成功，你必须手动执行定义分支的版本库状态。

```
$ repo rebase
```

使用 git mergetool 可能帮助你处理分支的版本库状态冲突。一旦你成功地合并冲突文件，

```
$ git rebase --continue
```

手动或者自动定义分支的版本库状态完成之后，运行 repo upload 来提交你新定义的补丁。



