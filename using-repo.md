# Repo 命令手册

Repo 的使用形式如下：

    repo <COMMAND> <OPTIONS>

可选元素显示在方括号[]里面。例如，许多命令接受项目列表作为参数。您可以指定项目列表作为名称列表或本地源目录的路径列表：

    repo sync [<PROJECT0> <PROJECT1> <PROJECTN>]
    repo sync [</PATH/TO/PROJECT0> ... </PATH/TO/PROJECTN>]

## 帮助

一旦 Repo 被安装，你可以找到总结所有命令的最新的文档，运行：

    repo help

在 Repo 目录中运行如下这个，你可以获得任何命令的信息：

    repo help <COMMAND>

## 初始化

    $ repo init -u <URL> [<OPTIONS>]

在当前目录下安装 Repo。这会产生一个 `.repo/` 目录，目录包括装 Repo 源代码和标准 Android 清单文件的 Git 仓库。`.repo/` 目录还包括 `manifest.xml`，是一个在 `.repo/manifests/` 目录选择清单的符号链接。

选项：

- `-u`：指定一个恢复清单仓库的地址（URL）。常见的清单可以在 `https://android.googlesource.com/platform/manifest` 找到。
- `-m`：在仓库里选择一个清单文件。如果没有清单名称，那么默认是 default.xml。
- `-b`：指定一个修正，例如，一个特殊的清单分支（manifest-branch）。

>注意：所有剩余的 Repo 命令，在当前工作目录下必须是 `.repo/` 的父目录或是一个父目录的子目录。

## 同步

    repo sync [<PROJECT_LIST>]

下载新的更改，更新在你本地环境中的工作文件。如果你不带参数运行 `repo sync` ，它将同步所有项目的文件。

当你运行 `repo sync` ，将会发生：

- 如果项目从来没有被同步过，那么 `repo sync` 相当于 `git clone`。远程仓库的所有分支都被复制到本地项目目录中。
- 如果项目曾经已经被同步过，那么 `repo sync` 相当于：   
    <code>
    git remote update   
    git rebase origin/<BRANCH>

`<BRANCH>` 是本地项目目录中的当前检查的分支。如果本地分支不跟踪远程仓库的分支，那么没有同步发生的项目。
- 如果 git rebase 操作导致合并冲突，那么你需要使用正规的 Git 命令（例如，`git rebase ——continue`）去解决这个冲突。

一个成功的 `repo sync` 之后，指定项目中的代码将和最新的代码留在远程仓库中。

选项：  

- `-d`：切换指定项目回到清单修正。如果该项目目前是一个主题分支那就有帮助，但清单修正是暂时需要。
- `-s`：同步到一个已知的构建 manifest-server 在当前清单指定的元素。`
- `-f`：继续同步其他项目，即使有项目同步失败。

## 上传

    repo upload [<PROJECT_LIST>]

在指定的项目中，Repo 把本地分支的更新比作远程分支在最后一次 Repo 同步。Repo 会提示你选择一个或多个尚未上传审查的分支。

你选择一个或多个分支时，选到的分支上的所有 commits 会通过 HTTPS 连接传送到 Gerrit。你将需要配置一个 HTTPS 密码去启用上传授权。查看 [密码生成器](https://www.googlesource.com/new-password?state=android&code=4) 生成新的用户名/密码配对去使用　HTTPS。

当 Gerrit 在它的服务器接收到对象数据时，它会把每个 commit 转变成更改，所以审阅者可以单独的评论每条 commit。将几条“checkpoint” commits 一起合并到一条单个的 commit 上，然后在你运行 repo upload 之前使用 git rebase -i 。

如果你不带参数运行 repo upload，它将搜索所有项目上传的更改。

在更改（changes）上传之后进行编辑，你应该使用一个类似 `git rebase -i` 或 `git commit --amend` 的工具去更新你的本地的 commits。你所有编辑完成之后：

- 确保更新的分支是当前已审查的分支。
- 在系列里的每个 commit 的括号内输入 Gerrit 改变 ID：
    
```
   
    # Replacing from branch foo 
    [ 3021 ] 35f2596c Refactor part of GetUploadableBranches to lookup one specific...
    [ 2829 ] ec18b4ba Update proto client to support patch set replacments 
    # Insert change numbers in the brackets to add a new patch set.
    # To create a new change record, leave the brackets empty.

``` 
 
上传完成后，更改将会有一个额外的补丁集（Patch Set）。

## 差异

    repo diff [<PROJECT_LIST>]

在 commit 和工作目录之间使用 `git diff` 表示明显差异的更改。

# 下载

    repo download <TARGET> <CHANGE>

从审查系统下载指定的更改，然后使它在你的项目的本地工作目录中可用。

例如，下载 [change 23823](https://android-review.googlesource.com/#/c/23823/) 到你的平台/框架/基本目录：

    $ repo download platform/build 23823

一个 `repo sync` 应该可以有效地移除任何通过 `repo download` 恢复的 commit。或者，你可以检查远程分支；例如，`git checkout m/master`。

>注意:当更改在 Gerrit 网络上可见时和 `repo download` 被所有用户找到时，期间，有一个轻微的镜像滞后，因为复制延迟存在于全世界所有的服务器。

## forall

    repo forall [<PROJECT_LIST>] -c <COMMAND>

在每个项目中执被给予的 shell 命令。如下的附加环境变量是通过 `repo forall` 才变得有效的：

- `REPO_PROJECT` 设置项目唯一的名称。
- `REPO_PATH` 是相对于客户端 root 的路径。
- `REPO_REMOTE` 是清单中远程系统的名称。
- `REPO_LREV` 是清单中修订本的名字，翻译成一个本地跟踪分支。如果你需要通过清单修正去一个在本地执行的 git 命令的时候可以使用。
- `REPO_RREV` 是清单中修订本的名字，正如在清单中所写的那样。

选项：

- `-c`：执行命令和参数。命令是通过 `/bin/sh` 评估的并且后面的任何参数就如 shell 位置的参数通过。
- `-p`：在指定命令的输出前显示项目标题。这是通过绑定管道到命令的stdin，stdout，和 sterr 流，并且用管道输送所有输出量到一个连续的流，显示在一个单一的页调器会话。
- `-v`：显示命令写到 sterr 的信息。

## 删减

    repo prune [<PROJECT_LIST>]

删减（删除）已经合并的标题。 

## 开始

    repo start <BRANCH_NAME> [<PROJECT_LIST>]

一个新分支的发展，从清单中指定的修正开始的。

`<BRANCH_NAME>`参数应该提供一个更改的简短说明给你正在尝试建立的项目。如果你不知道，那就考虑使用默认名称。

`<PROJECT_LIST>` 指定将要参与这个特性分支的项目。

>注意："." 是当前工作目录下的项目的一个方便的简写。

## 状态

    repo status [<PROJECT_LIST>]

比较工作目录和暂存区（索引），和这个分支（HEAD）在每个项目指定的最近一次提交。为每个不同于这三个状态的文件展示一个摘要行。

运行 `repo status` ，查看当前分支的状态。状态信息将会在项目中列表出来。项目中的每个文件，两个字母的代码是经常被使用的：

在第一列中，大写字母表明了暂存区和最后一次提交状态的差异。

<table>
<thead>
<tr>
<th>字母</th>
<th>含义</th>
<th>描述</th>
</tr>
</thead>
<tbody>
<tr>
<td>-</td>
<td>无变化</td>
<td>在 HEAD 和在索引中是一样的</td>
</tr>
<tr>
<td>A</td>
<td>增加</td>
<td>不在 HEAD, 在索引</td>
</tr>
<tr>
<td>M</td>
<td>修改</td>
<td>在 HEAD, 在索引表示修改</td>
</tr>
<tr>
<td>D</td>
<td>删除</td>
<td>在 HEAD, 不在索引</td>
</tr>
<tr>
<td>R</td>
<td>重命名</td>
<td>不在 HEAD, 在索引中改变路径</td>
</tr>
<tr>
<td>C</td>
<td>复制</td>
<td>不在 HEAD, 在索引中复制另一份</td>
</tr>
<tr>
<td>T</td>
<td>改变模式</td>
<td>在 HEAD 和在索引中是一样的内容, 改变模式</td>
</tr>
<tr>
<td>U</td>
<td>拆分</td>
<td> HEAD 和索引之间有冲突; 要求解决</td>
</tr>
</tbody>
</table>

在第二列中，一个小写字母表明了工作目录和索引的差异。
<table>
<thead>
<tr>
<th>字母</th>
<th>含义</th>
<th>描述</th>
</tr>
</thead>
<tbody>
<tr>
<td>-</td>
<td>新建/未知</td>
<td>不在索引, 在工作目录</td>
</tr>
<tr>
<td>m</td>
<td>修改</td>
<td>在索引, 在工作目录, 修改</td>
</tr>
<tr>
<td>d</td>
<td>删除</td>
<td>在索引, 不在工作目录</td>
</tr>
</tbody>
</table>