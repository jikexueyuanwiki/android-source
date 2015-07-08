# 使用 Eclipse

本文将帮助您为 Android 平台开发建立 Eclipse IDE 。

>注意：如果您正在寻找如何使用 Eclipse 开发 Android 上运行的应用程序的资料，那么这不是您要找的页面。你也许会发现 [the Eclipse page on developer.android.com](https://developer.android.com/tools/sdk/eclipse-adt.html) 页面更加有用。

## 基本设置

首先，重要的是要确保常规 Android 开发系统的设置。

    cd /path/to/android/root   
    make

**重要提示**：你将仍然使用 `make` 建立你的文件，实际运行（在模拟器上或者设备上）。你将使用 Eclipse 来编辑文件并且验证它们的编译，但是当你想要运行某些东西时你需要确保你的文件保存在 Eclipse 里，并且在命令行里运行 `make`。Eclipse 的建立只是为了检查错误。

Eclipse 需要一个目录列表去搜索 Java 文件。这个被称为“Java Build Path”，还能够设置到 `.classpath` 文件。我们有一个示例版本让你开始。

    cd /path/to/android/root   
    cp development/ide/eclipse/.classpath .
    chmod u+w .classpath
如果必要的话，现在可以编辑复制 `.classpath`。

## 增加 Eclipse 的内存设置

Android 项目（所占空间）非常的大以致有时 Eclipse 的 Java VM 在运行编译时内存不足。而编辑 `eclipse.ini` 文件就可以避免这个问题。在 Apple OSX 上，eclipse.ini 文件是位于

    /Applications/eclipse/Eclipse.app/Contents/MacOS/eclipse.ini

内存相关的默认设置（如 Eclipse 3.4）

     -Xms40m  
     -Xmx256m  
     -XX:MaxPermSize=256m

为 Android 开发的建议设置是：

     -Xms128m 
     -Xmx512m 
     -XX:MaxPermSize=256m

这些设置是将 Eclipse 的 Java 堆的最小值设为 128 MB，最大值设为 512 MB，并且默认永久代的最大值为 256 MB。

现在启动 Eclipse：

    eclipse

## 创建一个项目

现在为 Android 开发创建一个项目：

1. 如果 Eclipse 要求你选择一个工作区的位置，选择默认。
2. 如果你有一个“欢迎”屏幕，关闭它显示到 Java 视图。
3. 文件（File） > 新建（New） > Java 项目（Java Project）
4. 选择一个项目名称，“android”或者任何你喜欢的名称。
5. 取消使用默认位置，输入路径到 Android 根目录，然后点击 Finish。
6. 等待建立项目的时候。（你会看到右下角有一个微妙的进度表）

一旦项目工作区创建，Eclipse 也应当开始建立。从理论上讲，它的建立应该是没有错误的，你应该准备好开始启动。如果有必要，取消选择再重新选择项目自动构建以强制重建。

>注意：Eclipse 有时会添加一个 `import android.R` 声明在你文件的顶部以使用资源，特别是当你要求 Eclipse 分类或以其他方式管理导入包时。这将导致你的构造被破坏。要留心找到那些错误的导入声明并删除掉它们。

##当你同步时

每次你同步 Repo，或者以其他方式改变 Eclipse 外部的文件（特别是.classpath）时，你需要更新 Eclipse 视图的以下东西：

1. 窗口（Windows） > 显示视图（Show View）> 导航（Navigator）
2. 在导航（Navigator）中，右键单击项目名称
3. 点击右键菜单中的刷新（Refresh）。

## 添加 Apps（应用程序）到构建路径（Build Path）上

默认的 `.classpath` 包括核心系统的来源和一组 APP 的样本，但也许并不包括你想要的特定的 APP。添加一个 APP，你必须添加 APP 的源目录。需要在 Eclipse 里面完成这些：

1. 项目（Project）> 属性（Properties）
2. 选择左边菜单上的 “Java Build Path（Java 构建路径）”上。
3. 选择 “Source（源）” 选项卡。
4. 点击“Add Folder…（添加文件夹…）”
5. 添加你的 APP 的 `src` 目录下。
6. 点击 OK。

当你完后，你应该可以看到“source folder（源文件夹）” 路径在列表中如下显示：

    android/packages/apps/YOURAPP/src

根据你所包含的 APP，你可能也需要包含 `android/dalvik/libcore`  下的 `othersrc/main/java` 目录。如果你发现你不能在默认设置下构建的时候你就用这个方法。
    
## Eclipse 格式化

你可以导入文件到 `development/ide/eclipse` 里面使得 Eclipse 跟随 Android 的样式规则。

1. 选择窗口（Windows）> 参数选择（Preferences）> Java > 代码风格（Code Style）。
2. 使用格式化（Formatter）> 导入（Import）导入 `android-formatting.xml`。
3. 整理导入包 (Imports) > 导入（Import）来导入 `android.importorder`。

## 使用 Eclipse 调试仿真器

你也可以使用 Eclipse 通过单步调试代码去调试模拟器。首先，开始运行模拟器：

    cd /path/to/android/root 
    . build/envsetup.sh 
    lunch 1    
    make       
    emulator
如果模拟器正在运行，你可以看见一张手机的图片。  

在另一个命令行，启动 DDMS（Dalvik虚拟机调试管理器）：

    cd /path/to/android/root 
    ddms
你应该看到一个 splufty 调试控制台。

现在，在Eclipse中，您可以连接到模拟器：

1. 运行（Run）> 打开调试对话框…（Open Debug Dialog…）
2. 右键单击“Remote Java Application”，选择“New”。
3. 选择一个你喜欢的名字，例如“android-debug”或任何东西。
4. 将“Project”设置为你的项目名称。
5. 主机依旧设置为“localhost”，但端口更改为 8700。
6. 点击“Debug”按钮，这样就应该准备就绪了。

需要注意的是端口 8700 连接到任何的进程都是在目前 DDMS 控制台选择的，所以你需要确保 DDMS 已经选择你想要调试的进程。

你可能需要打开调试视图（在“Java”视图图标的右上角，点击的“Open Perspective” 的小图标然后选择“Debug”）。一旦你做了这些，你应该可以看到一个线程的列表；如果你选择一个（线程）并且中断它（通过点击“pause”图标），那么它应该显示堆栈跟踪，源文件，和执行路线。断点和诸如此类的东西都应该在工作。

## 扩展材料

Mac 系统的苹果键替换 Ctrl 键

<table>
<thead>
<tr>
<th>快捷键</th>
<th>功能</th>
</tr>
</thead>
<tbody>
<tr>
<td>Ctrl-Shift-o</td>
<td>导入所有需要的包</td>
</tr>
<tr>
<td>Ctrl-Shift-t</td>
<td>按名称加载类</td>
</tr>
<tr>
<td>Ctrl-Shift-r</td>
<td>按名称加载非资源类</td>
</tr>
<tr>
<td>Ctrl-1</td>
<td>快速修复</td>
</tr>
<tr>
<td>Ctrl-e</td>
<td>最近查看的文件</td>
</tr>
<tr>
<td>Ctrl-space</td>
<td>自动完成</td>
</tr>
<tr>
<td>Shift-Alt-r</td>
<td>重构：重命名</td>
</tr>
<tr>
<td>Shift-Alt-v</td>
<td>重构：移动</td>
</tr>
</tbody>
</table>

## Eclipse 不能正常运行时，我该怎么做？

首先确认：

- 你准确地按照这个网页的说明去执行。
- 你的问题视图没有显示任何错误。
- 你的应用程序遵从包/目录的构造。

如果你仍然有问题，请联系其中一个 [Android 社区电子邮箱列表或 IRC 通道](https://source.android.com/source/community.html)。
