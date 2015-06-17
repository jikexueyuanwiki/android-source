# Bug 的生命周期

Android 开源项目维护了一个公共的 issue 跟踪管理，在这里你可以报告 bugs 并给核心 Android 软件请求功能（可以到 [Reporting Bugs](https://source.android.com/source/report-bugs.html) 页面查看 issure 跟踪器的详细信息）。提交 Bugs 是一件很棒的事情（谢谢！），但是当你提交完之后会发生什么呢？这个页面就是描述 Bug 的生命周期。

**请注意**：Android 开源项目（AOSP）issue 跟踪器仅仅是针对 bugs 和与请求核心软件相关的功能，同时，它也是开源社区的一个技术工具。

这不是一个客户支持论坛。你可以在 [Google 的 Nexus 帮助页面](http://support.google.com/nexus) 得到 Nexus 设备的支持信息。其他设备的话，可以找设备厂商或者设备销售者。

寻找 Google 应用的帮助可以到 [Google 的支持页面](http://support.google.com/)。第三方应用的帮助可以找这个应用的开发者，比如，通过 Google Play 提供的联系方式。

下面是 bug 的生命周期，简短的概括：

1. bug 被提出后，就会在 New 状态。

2. AOSP 的维护者定期检查并且将 bug 分类。Bugs 将被分为四个“buckets” 中的其中一个：New，Open，No-Action，和 Resolved。
3. 每一个 bucket 都会包括一些状态用来提供 issue 生命期的更多信息。
4. Resoloves 状态下的 Bug 最终会发布在未来版本的 Android 软件中。

## Bucket 详情

这里是每个 bucket 的一些额外的信息，它意味着什么，以及它是怎么处理的。

### New 状态

New 状态的 issue 包括了没有采取行动的 bug 的报告。这两个状态是：

- *New：*这个 bug 报告还没有被分类（就是还没有被 AOSP 维护者检查）。

- *NeedsInfo：*提交的 bug 要有足够的信息来采取行动（给维护者）。在分类之前，提交 bug 的用户需要提供额外的细节信息。如果过了很长时间，新的信息还没有提供，这个 bug 可能默认会关闭，变成 No-Action 状态。

### Open 状态

这个状态的 bugs 包括需要采取行动的，但仍未解决，在等待修改的源码。

- *未分配：*这个 bug 报告已经被认为是一个有充分详情的正当的 issue 报告，但是它还没有被分配给 AOSP 奉献者解决。

- *已分配：*像未分配状态一样，但是这个 bug 实际上已经被分配给一个特定的奉献者来修改。

通常情况下，一个给定的 bug 开始是未分配状态的，直到有人打算解决它，此时它就会变成分配状态。然而，请注意这并不是绝对的，一个 bug 从未分配状态到解决状态也不奇怪。

一般而言，如果这个 bug 是 Open 状态的其中一种，AOSP 团队已经认为它是一个合理的 issue，并且这个 bug 将会接受到高质量的奉献修复。但是，不能保证及时修复到任何特定的发布版本中。

### No-Action 状态

这个状态下的 bugs 因为这样或那样的原因决定不在采取任何行动。

- *Spam：*一些好心的人提供的不错的建议，但是很遗憾，这个不是我们需要的。

- *Duplicate：*issue 跟踪器中已经有一个相同的问题了。任何目前的操作都会更新在那个报告中。

- *Unreproducible：*一个 AOSP 奉献者试图重现所描述的行为，但是没有重现出来。这个有时意味着 bug 虽然合理但是不常见或者很难重现，有时意味着这个 bug 已经在之后的版本中修复了。

- *Obsolete：*和 Unreproducible 状态相似，但是合理确定的是这个 bug 在提交的那个版本中存在，但已经在之后的版本中解决了。

- *WorkingAsIntended：*AOSP 维护者已经确定描述的行为不是 bug，但是是一个期望的行为。这种状态通常也被称为 WAI。

- *Declined：*这个像 WorkingAsIntended 一样，除了 WorkingAsIntended 通常是功能请求，不是 bugs。这个意味着 AOSP 维护者决定这个请求不在 Android 中实现。

- *NotEnoughInformation：*这个报告没有足够的信息让维护者采取行动。

- *UserError：*这个报告结果是用户使用 Android 的时候自己造成了错误。例如，输入错误的密码导致无法正常连接服务器。

- *WrongForum：*AOSP 无法处理个错误，通常来说是因为这个 bug 是定制的设备或者外部的应用程序导致的。

- *Question：*有人误以为 issue 跟踪是一个帮助论坛。

### Resolved 状态

这个状态的 bug 包括已经采取行动的，并且现在被认为是解决了的。

- *Released：*这个 bug 已经修复了并且发布在正式的版本中。在设置这个状态的同时，我们也会尽力发布一个适当的说明表示这个 bug 是在哪个版本中解决的。

- *FutureRelease：*这个 bug 在源码中已经解决（或者功能已经实现），但是还没有发布在正式的版本中。

## 其他特性

上面提到的状态和生命周期是我们通常跟踪软件的方式。然而，Android 包含了很多软件，相应的得到大量的 bugs。因此，一些 bugs 并没有经过正常流程中的所有的状态。我们会尽力让系统保持更新，但是我们倾向于周期性的进行 bug 清理，同时检查数据库并更新。