# 提交 Bugs

感谢你对 Android 的喜爱。帮助我们改善 Android 的最好方式就是告诉我们你发现关于它的任何问题。

## 报告问题

> 注意：安全漏洞的问题，请查看 [Reporting Security Issues](https://source.android.com/devices/tech/security/overview/updates-resources.html#reporting-security-issues)。如果你发现一个安全漏洞，*请不要使用下面的形式*。使用一个普通的形式，让大家都可以看到你的报告，这样能够在 bug 修复之前让其他用户不再冒险。还有，请发送 email 到 security@android.com 描述详情。

这里将讲述**非安全**问题的 bug：

- 搜索你的 bug，看看是否已经有人报告了。记得要搜索全部 issue，不仅仅是 open 状态下的，因为这个 issue 可能已经报告过了并且已经关闭了。可以通过星星的数量分类可以找到最受欢迎的结果。

- 如果你发现你的问题很重要，请标记它。这样就可以让我们知道这个 bug 修复的重要性。

- 如果没有人报告这个 bug，你就可以提交这个 bug。你可以使用下面的其中一种模版：

	- [设备中的 bug](https://code.google.com/p/android/issues/entry?template=User%20bug%20report) - 如果你是使用你自己的设备的用户，请使用这种。
	- [软件中的 bug](https://code.google.com/p/android/issues/entry?template=Developer%20bug%20report) - 如果你是在开发一个 APP 的过程中遇到的，请使用这种。
	- [功能请求](https://code.google.com/p/android/issues/entry?template=Feature%20request) - 如果你想在将来的版本中看到这个功能，请使用这种。

请记住 issue 跟踪器不是一个用户帮助论坛。这是一个待定的技术任务列表，以及和这些任务相关的信息，还有这些任务的进展信息，包括哪些任务可能在短期内有进展。

这个 issue 跟踪器紧紧专注于 Android 开源项目。零售设备的问题需要通过这些设备的支持渠道来报告，尤其是和 Nexus 不同的设备。开发者的应用程序的问题并不是 AOSP 需要报告的部分；这个也是 Google 应用的状况。

请注意我们不能保证特定的 bug 将会在特定的版本中解决。查看[Life of a Bug](https://source.android.com/source/life-of-a-bug.html)可以知道提交 bug 之后的情况。

一般而言，请尽可能的提交多一点的的 bug 信息，仅仅用一行内容告诉我们这里有问题是没用的，这样维护者很可能不采取任何行动就把它关闭了。你提交的详情越多，你的问题可能更好的解决。下面介绍一些好的 bug 提交报告和劣质的报告。

## 劣质的报告

> 标题：当运行 Eclipse 的时候，错误消息提示 Internal Error，并且需要打开 .log 查看详情。重现步骤：Object o = null 的时候发生，变成 Object o 的时候不发生。预期的结果：我没有获取到错误信息 -- Object o = null 不产生作用。观察结果：查看上文。

这个一个劣质的报告因为没有提供任何上下文相关的信息。这个是 ART 运行时的问题？ framework 的问题？ 或者其他问题？这个也没有提供任何关于重现错误的代码或者线索。换句话说，这个 bug 报告没有提供足够的信息让维护者来采取行动，所以它将被忽略。

## 优质的报告

标题：执行到 Object o = null 的时候，引起 Eclipse 的 Internal Error bug，使用的 Eclipse 版本 3.3.1.1，m37a android，代码如下：

```
package com.saville.android;

import android.app.Activity;
import android.os.Bundle;
import android.util.Log;

public class TestObjectNull extends Activity {
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle icicle) {
        super.onCreate(icicle);
        setContentView(R.layout.main);

        Object o = null;

        o = "hi";

        Log.v(TAG, "o=" + o);
    }

    static final String TAG = "TestObjectNull";
}
```

Eclipse 表明是 Internal Error，并且在 .log 中可以查看更多信息，然后询问是否退出工作台。在 setContentView(R.layout.main) 这里设置一个断点，然后单步执行 Object o = null 就会发生错误。如果我把它变成 Object o 就没什么问题。.log 文件的最后一行是：

```
!ENTRY org.eclipse.core.jobs 4 2 2008-01-01 13:04:15.825
!MESSAGE An internal error occurred during: "has children update".
!STACK 0
java.lang.InternalError: Invalid signature: "<null>"
        at
org.eclipse.jdi.internal.TypeImpl.signatureToTag(TypeImpl.java:307)
        at
org.eclipse.jdi.internal.LocalVariableImpl.tag(LocalVariableImpl.java:185)
        at
org.eclipse.jdi.internal.StackFrameImpl.getValues(StackFrameImpl.java:128)
        at
org.eclipse.jdi.internal.StackFrameImpl.getValue(StackFrameImpl.java:73)
        at
org.eclipse.jdt.internal.debug.core.model.JDILocalVariable.retrieveValue(JDILocalVariable.java:57)
        at
org.eclipse.jdt.internal.debug.core.model.JDIVariable.getCurrentValue(JDIVariable.java:66)
        at
org.eclipse.jdt.internal.debug.core.model.JDIVariable.getValue(JDIVariable.java:88)
        at
org.eclipse.debug.internal.ui.model.elements.VariableContentProvider.hasChildren(VariableContentProvider.java:62)
        at
org.eclipse.jdt.internal.debug.ui.variables.JavaVariableContentProvider.hasChildren(JavaVariableContentProvider.java:73)
        at
org.eclipse.debug.internal.ui.model.elements.ElementContentProvider.updateHasChildren(ElementContentProvider.java:223)
        at
org.eclipse.debug.internal.ui.model.elements.ElementContentProvider$3.run(ElementContentProvider.java:200)
        at org.eclipse.core.internal.jobs.Worker.run(Worker.java:55)
```

