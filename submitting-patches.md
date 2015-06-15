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

如果提交到源码中的其他补丁和你的有冲突，你将需要在源码库新 HEAD 的顶部 rebase（重新定义分支版本库状态）你的补丁。一个简单的方法就是运行：

```
$ repo sync
```

这个命令将会从服务器先 fetches 最新的源码，然后会试图自动 rebase HEAD 到新的远程 HEAD 上面。

如果自动 rebase 没有成功，你必须手动执行 rebase。

```
$ repo rebase
```

使用 git mergetool 可能帮助你处理 rebase 冲突。一旦你成功地合并冲突文件，

```
$ git rebase --continue
```

手动或者自动 rebase 完成之后，运行 repo upload 来提交你 rebase 补丁。

### 提交被批准之后

提交的内容在通过检查和验证过程之后，Gerrit 会自动的 merges 这个更改到公共的库中。其他用户可以运行 repo sync 来 pull 这个更新到自己的本地。


## 检查员和审核者

### 检查一个更改

如果你被分配成这个更改内容的批准者，你需要决定接下来的内容：

- 这个更改是不是符合项目一开始的目的？
- 这个更改是不是在现存的构造中有效？
- 这个更改引入的设计缺陷是否会在将来出现问题？
- 这个更改是否遵循这个项目中已建立的最佳实践？
- 这个更改是执行描述方法的好方式？
- 这个更改是否引入任何安全或不稳定的风险？

如果你准许了这个改变，请在 Gerrit 中用 LGTM（Look Good to ME）标记它。

### 审核更改

如果你被分配成这个更改内容的审核者，你需要做接下来的内容：

- 使用其中一种下载命令将这个更改 Patch 到自己的本地客户端。
- 构建和测试这个更改。
- 在 Gerrit 中，使用发布评论的方式来标记这个 commit，用“Verified”或者“Fails”，并且添加一个消息来解释哪些问题是经过鉴定的。

### 从 Gerrit 下载更改的内容

已经审核和合并之后的提交将会在下一次 repo sync 的时候被下载。如果你希望下载一个特定的没有经过检验的更改，执行：

```
$ repo download TARGET CHANGE
```

这个 TARGET 就是你下载的更改将要放到本地目录的位置，CHANGE 就是在 Gerrit 中的更改列表的数字。想要知道更多信息，请查看[Repo reference](https://source.android.com/source/using-repo.html)。

### 怎么样才能成为一个审核者或者检验者？

简单来说，需要对一个或者多个 Android 项目贡献高质量代码。想要了解更多关于 Android 开源社区的不同角色和都有谁参与的信息，可以查看 [Project Roles](https://source.android.com/source/roles.html)。

### Diffs 和评论

在 Gerrit 中点击一个更改的 Id number 或者 Subject 可以打开这个更改的详细信息。想知道现有的代码和更新的代码之间的差异，可以点击文件名下的 Side-by-side diffs。

### 添加评论

在开源社区的任何一个人都可以使用 Gerrit 来给提交的源码添加内联的评论。一个好的评论将会关联行或者部分在 Gerrit 中的附加代码。这或许是关于如何改进一行代码，简短的或者建设性的意见，或者，这个也许是为什么作者这样写代码就有意义的解释。

想要添加内联评论，双击代码中相关联的行，并且在下一个打开的窗口写上你的评论。你点击保存之后，只有你可以看到你的评论。

想要发布你的评论，让其他 Gerrit 使用者看到评论，点击 Publish Comments 按钮。你的评论内容将会通过 email 发送给这个更改的所有当事人，包括更改的拥有者，补丁更新者（如果和拥有者不是同一个人），还有所有当前的检查者。

## Upstream 项目

Android 使用很多其他开源项目，比如，Linux 内核和 WebKit，像在 [Codelines](https://source.android.com/source/code-lines.html)，[Branches](https://source.android.com/source/code-lines.html)，[Releases](https://source.android.com/source/code-lines.html) 中描述的那样。在 external/ 下的大多数项目，更改应该被 upstream，然后 Android 维护者通知新的 upstream 版本将包括这些更改。让我们跟踪一个新的 upstream 版本，可能对上传补丁有好处。但是如果这些项目被广泛使用，像下面提到的大多数项目一样，将很难做出改变，对于这样的项目，我们倾向于每次发布版本都升级。

一个有趣的特殊情况是仿生（bionic）。很多代码都是来源于 BSD，所以除非改变的是新的仿生代码，我们宁愿看到一个 upstream 修复，然后从 适当的 BSD 上 pull 一个完整的新的文件（很可悲的是我们同时有很多不同的 BSD，但是我们希望在未来解决这个问题，并且进入一个我们更密切跟踪 upstream 的位置）。

### ICU4C

在 external/icu4c 目录下，ICU4C 项目的所有改变，都应该在  [icu-project.org/](http://site.icu-project.org/) 里被 upstream。查看 [Submitting ICU Bugs and Feature Requests](http://site.icu-project.org/bugs) 获取更多信息。

### LLVM/Clang/Compiler-rt

LLVM-related 项目（external/clang, external/compiler-rt, external/llvm）的所有更改都应该在 [llvm.org/](http://llvm.org/) upstream。

### mksh

external/mksh 目录下，MirBSD Korn Shell 项目的所有更改要么发送 email 到 mirbsd.org（不需要订阅提交）域名下的 miros-mksh，要么是 [Launchpad](https://launchpad.net/mksh) 来进行 upstream。

### OpenSSL

external/openssl 目录下的 OpenSSL 项目的所有更改都应该在 [openssl.org](http://www.openssl.org/) 中 upstream。

### V8

external/v8 目录下 V8 项目的所有更改都应该提交到 [code.google.com/p/v8](https://code.google.com/p/v8) 中 upstream。进入 [Contributing to V8](https://code.google.com/p/v8/wiki/Contributing) 查看详情。

### WebKit

external/webkit 目录下 WebKit 项目的所有更改都应该在 [webkit.org](http://www.webkit.org/) 中 upstream。这个过程首先是提出一个 Webkit bug。这个 bug 应该是使用 Android 平台和系统，并且这个 bug 仅仅是针对于 Android 的。当添加了修复提议并有测试，Bugs 将更容易引起检查员的注意。查看 [Contributing Code to WebKit](http://webkit.org/coding/contributing.html) 获取更多信息。

### zlib

All changes to the zlib project at external/zlib should be made upstream at zlib.net.

external/zlib 目录下 zlib 项目的所有更改狗应该在 [zlib.net](http://zlib.net/) 中 upstream。