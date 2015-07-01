# 关于代码风格的指导

以下的规则并不是普通的指导方针或者我们的推荐，而是严格的规范。不遵循这些风格而构建出来的安卓软件将不会被接受。

目前，并不是所有的代码都符合规范，但是以后的代码应该向规范靠拢。

## Java 语法规则

我们遵循标准的 Java 编码规范，此外我们加入了一些新的规范：

### 不能忽视异常

有时候，开发者会倾向于编写完全忽视了异常的代码，就像下面这样：

~~~

void setServerPort(String value) {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatExceptio e) { }
}

~~~

开发者绝不能像这样。当你觉得你的代码不会出现这个错误，或者处理这个错误并不重要的时候，像上面的代码那样忽略了异常处理就相当于在你的代码中给以后接手的开发者埋下了地雷，他们总有一天会在这里遇到问题。你必须将代码中所有的异常用一种规范化的方式处理好，具体的处理方式取决于情况的不同。

不管什么时候，如果代码中存在空的捕获语句，开发者应当感到毛骨悚然。不论如何，在捕获语句中，必然存在应当执行的操作，至少你应该多考虑一下。在 Java 中，这种恐慌感是无法逃避的。 -[James Gosling](http://www.artima.com/intv/solid4.html)

可接受的处理方式（可以参照个人的喜好选择）有：

- 由方法的调用者抛出异常

~~~

void setServerPort(String value) throws NumberFormatException {
    serverPort = Integer.parseInt(value);
}

~~~

- 抛出一个适合你当前抽象层次的新异常

~~~

void setServerPort(String value) throws ConfigurationException {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) {
        throws new ConfigurationException("Port " + value + " is not valid.");
    }

~~~

- 通过在 catch 代码块中使用一个合适的值替代异常值来处理异常

~~~

/** 设置端口号，当 value 是一个无效数字时， 使用80替代 */

void setServerPort(String value) {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) {
        serverPort = 80; // 服务器的默认端口
    }
}

~~~

- 捕获异常并抛出一个新的`运行时异常`。这种方式存在风险：只有当你确认出现这种错误的时，令程序崩溃是最合适的处理方式，才能采用该方式。

~~~

/** 设置端口号，如果 value 是一个无效值，程序中断。 */
void setServerPort(String value) {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) {
        throws new RuntimeException("port " + value + " is invalid, ", e);
    }
}

~~~

请注意，原来的异常传递给了 RuntimeException 的构造器。如果你的代码必须在 Java 1.3 以下版本编译，你需要省略掉异常发生的原因。

- 最后一招：如果你认为忽略屌异常处理是最合适的，并且对此相当有信心，那么你可以忽略异常。但是你必须在注释中说明为什么：

~~~

/** 如果 value 是无效的数字，将会使用原来的端口号。 */
void setServerPort(String value) {
    try {
        serverPort = Integer.parseInt(value); 
    } catch (NumberFormatException e) {
        // 文档中会注明该方法将忽略掉用户输入的无效值
        // serverPort 将不会发生改变
    }
}

~~~

### 不要捕获一般性异常（Generic Exception）

有些时候，为了偷懒，开发者将会像下面这样捕获异常并作出处理：

~~~

try {
    someComplicatedIOFunction();        // 可能抛出 IOException 
    someComplicatedParsingFunction();   // 可能抛出 ParsingException 
    someComplicatedSecurityFunction();  // 可能抛出 SecurityException 
    // phew, made it all the way 
} catch (Exception e) {                 // 我希望捕获所有的异常
    handleError();                      // with one generic handler!
}

~~~

你不能这样做。几乎所有情况下都不适合捕获一般性异常，因为其中包括还包括了错误异常，这是相当危险的。这意味着你并不期待的异常（包括运行时异常，就像 ClassCastException）最后却在应用级别的错误处理中被捕获。它掩盖了处理失败的代码，这意味着如果有人在您正在调用的代码中添加了一个新类型的异常，编译器无法告诉你需要以不同的方式处理该错误。并且在几乎全部情况下，你都不应该以同种方式处理不同种类的异常。

这一规则有极少数例外：在某些测试代码和顶级代码中，你希望捕捉所有类型的异常（以防止他们在界面中显示或者继续进行批处理作业）。在这种情况下，你需要捕获一般性异常并合适地处理错误。这么做之前你需要谨慎考虑，然后在注释中解释为什么在这里这么做是安全的。

替代捕获一般性异常的选项：

- 在同一个 try 语句之后设置多个分离的 catch 块来处理不同的异常。这个方法可能不太方便，但是要比捕获所有异常要好不少。需要注意的是，catch 块中应尽量避免代码重复过多。

- 通过多个 try 语句重构代码以使错误处理的粒度更细小，把 IO 从 parsing 分离出来，在不同的情况中单独处理错误。

- 重新抛出异常。很多情况下，你不需要在当前层次下捕获异常，此时只需在方法中抛出异常即可。

记住：异常是你的朋友！当编译器告诉你还有异常没有被捕获的时候，不要皱眉。此时应该微笑：编译器令你能够更加轻松地在自己的代码中捕获运行时异常，是值得高兴的事。

### 不要使用终结器（Finalizers）

终结器是一种可以在对象被垃圾回收器回收时，执行一大块代码的方法。

优点：方便清理，特别是针对外部资源的时候。

缺点：并不能保证终结器什么时候会被调用，甚至有时候它根本不会被调用。

判定：我们不应使用终结器。在大多数情况下，你可以在终结器中执行你需要的操作，并且能实现良好的异常处理。如果你一定需要它，定义一个 close() 方法（或者是 like）并注明什么时候它应该被调用。我们可以吧 InputStream 当做一个例子来说明一下。在这种情况下，它并不是一定要在终结器中输出日志信息，但是，如果并不希望输出太多日志信息的话，在终结器中输出小段日志信息是很合适的方法。

### 完全限定引用

当你希望使用 foo 包中的 Bar 类的时候，你又两种方法实现：

1. import foo.*;
2. import foo.Bar;

第一种方式优点：可能会减少 import 语句的数量。
第二种方式优点：把要使用的类描述出来了，使得代码在维护时更具可读性。

判定：引用安卓代码的时候使用第二种形式。Java 标准库（java.util.\*， java.io.\*等）和单元测试代码（junit.framework.\*）是例外。

## Java 资源库规范

使用安卓中的 Java 库和工具会带来许多便利。有些情况下，这种便利性在一些重要方面发生了改变，因为旧的代码可能使用的是已经弃用的模式或者资源库。使用这样的代码时，直接使用已有的模式是没问题的。但是当创建新的组件的时候，不要使用弃用的资源库。

## Java 样式规范

### 使用 Javadoc 标准注释

每个文件都会在其顶部放置版权声明。然后是包的声明和引用语句，用空行把不同作用的语句块分离开来。接下来是类或者接口的声明。在 Javadoc 标准注释中，要描述类或者接口的作用。

~~~

/*
 * Copyright (C) 2013 The Android Open Source Project 
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at 
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software 
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and 
 * limitations under the License.
 */

package com.android.internal.foo;

import android.os.Blah;
import android.view.Yada;

import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * Does X and Y and provides an abstraction for Z.
 */

public class Foo {
    ...
}

~~~

对于每个类和有价值的公共方法，你都应该为之书写一个 Javadoc 注释，该注释至少包含一句关于类或方法作用的描述。并且这个描述应该以第三人称描述性谓词开头。

示例：

~~~

/** Returns the correctly rounded positive square root of a double value. */
static double sqrt(double a) {
    ...
}

~~~

或者：

~~~

/**
 * Constructs a new String by converting the specified array of 
 * bytes using the platform's default character encoding.
 */
public String(byte[] bytes) 

~~~

你并不需要为没价值的 get 或者 set 方法书写 Javadoc，比如，对于 setFoo() 方法，你在 Javadoc 中只能说设置了 Foo 的值。如果该方法所做的事情更加复杂（比如强制约束或者有一个重要的副作用），那你应该写一个 Javadoc。假设之前的例子中，Foo 这个属性的意思并不明确，那就该添加一个 Javadoc。

你所写的每一个方法，不论是公有的还是其他的，都将受益于 Javadoc。而公共函数是API的一部分，所以需要 Javadoc。

安卓目前并不强制开发者使用特定的 Javadoc 书写风格，但是你应该遵循 [How to Write Doc Comments for the Javadoc Tool](http://www.oracle.com/technetwork/java/javase/documentation/index-137868.html) 中的规范。

### 编写短小的方法（Short Method）

在可行的范围内，方法应该尽量向小型化和集中化靠拢。但是，也要认识到，有些情况下较长的方法更加合适，所以对于方法的长度并不做硬性要求。如果某个方法的代码超出了40行，那么应该考虑一下是否能将它分解为小方法并且不影响到程序的结构。

### 在标准的地方定义字段

字段应该在文件顶部定义，或者在即将使用它们的方法前定义。

### 限制变量的范围

局部变量的存在范围应该尽可能地低。这样做将会增加代码的可读性和可维护性，并且还能降低出错的可能性。在嵌套的代码块中，每个变量应该在能够包含所有使用该变量的语句的最小的代码块中声明。

局部变量应该在第一次被使用时声明。几乎每个局部变量的声明都应该包括合理的初始化，如果你没有没办法在此处获得足够的信息去初始化这个变量，那么你应该将声明推迟到你获得足够信息的地方。

这个规则的一个例外就是 try-catch 语句。如果一个变量初始化的时候，其初始值是一个抛出已检查异常的方法的返回值，它就必须在 try 语句内初始化。如果这个变量必须在 try 块之外被使用，那么它就要在 try 块之外声明，尽管在那里它并没有被合理地初始化：

~~~

// 实例化类 cl，表示某种集合
Set s = null;
try {
    s = (Set) cl.newInstance();
} catch(IllegalAccessException e) {
    throw new IllegalArgumentException(cl + " not accessible");
} catch(InstantiationException e) {
    throw new IllegalArgumentException(cl + " not instantiable");
}

// 运用这个集合
s.addAll(Arrays.asList(args));

~~~

但即使是这种情况也能通过把 try-catch 语句封装在一个方法中来避免：

~~~

Set createSet(Class cl) {
    // 实例化的类cl，表示某种集合
    try {
        return (Set) cl.newInstance();
    } catch(IllegalAccessException e) {
        throw new IllegalArgumentException(cl + " not accessible");
    } catch(InstantiationException e) {
        throw new IllegalArgumentException(cl + " not instantiable");
    }
}

...

// 运用这个集合
Set s = createSet(cl);
s.addAll(Arrays.asList(args));

~~~

对于循环变量，除非有令人信服的理由，否则应该在 for 语句中声明：

~~~

for (int i = 0; i < n; i++) {
    doSomething(i);
}

~~~

其他类型的循环变量：

~~~

for (Iterator i = c.iterator(); i.hasNext(); ) {
    doSomethingElse(i.next());
}

~~~

### 有序的引用语句

引用语句的顺序如下：

1. 安卓的引用
2. 第三方的引用
3. java 和 javax 的引用

为了更好地符合IDE的设置，这些引用语句应该是这样的：

- 每个分组按字母排序，其中大写字母放在小写字母前。（比如，Z 在 a 前面）
- 每个主要的组之间应该用空行隔开（android, com, junit, net, org, java, javax）

最初在顺序的风格上并没有要求。也就是说 IDE 总是在改变这些顺序，或者说使用 IDE 的开发人员不得不禁用自动引用的特性然后手动维护这些引用语句，这是相当不好的。当问及 Java 风格的时候，开发者们喜欢的风格是五花八门的。这和我们对于“尽量有序并且一致”的需求完全不同，所以我们选择了一种风格，更新了风格指南，并且让 IDE 来遵循它。我们希望，随着用户不断使用 IDE 来编码，最终所有的引用语句都能符合这种风格。

这种风格是这样的：

- 人们希望并且往往放在顶部的引用语句（android）
- 人们希望并且往往放在底部的引用语句（java）
- 人们要使用这种风格很简单
- IDE 能够遵循这种风格

静态引用的使用和位置一直都存在争议。有的人喜欢把静态引用分散在已有的引用语句中，有些人则希望把它们放在其他所有引用语句的上方或者下方。此外，我们目前也没有得出一种能够让所有 IDE 都遵循统一顺序的方法。

由于多数人认为这件事并不重要，所以你需要自行判断，并尽可能保持一致。

### 使用空格来缩进

我们使用4个空格来缩进，而不是使用 tab 键。对此感到困惑的话，请和你周围的代码保持一致。

语句换行的时候我们使用8个空格来缩进，包括方法的调用和赋值语句。例如，这样是正确的：

~~~

Instrument i =
        someLongExpression(that, wouldNotFit, on, one, line);

~~~

下面这样则是不正确的：

~~~

Instrument i =
    someLongExpression(that, wouldNotFit, on, one, line);

~~~


### 遵循字段命名惯例

- 非公有且非静态的字段以 m 开头
- 静态字段以 s 开头
- 其他字段以小写字母开头
- 公有的静态字段全部大写并用下划线连接

例子如下：

~~~

public class MyClass {
    public static final int SOME_CONSTANT = 42;
    public int publicField;
    private static MyClass sSingleton;
    int mPackagePrivate;
    private int mPrivate;
    protected int mProtected;
}

~~~

### 使用标准的花括号风格

花括号不单独占一行，应该放在和前面代码同一行的位置。就像下面这样：

~~~

class MyClass {
    int func() {
        if (something) {
            // ...
        } else if (somethingElse) {
            // ...
        } else {
            // ...
        }
    }
}

~~~

我们需要在一个条件语句中使用括号。只有整个条件语句中只有一行代码，你才可以把他放在一行而不使用括号。就如下面这样是合法的：

~~~

if (condition) {
    body(); 
}

~~~

而这样也是可行的：

~~~

if (condition) body();

~~~

不过，虾米那这样也是可以的：

~~~

if (condition)
    body();  // 不推荐这样的风格

~~~

### 控制行的长度

你代码的每一行的长度应该限制在100个字符以内。

为此我们做了很多讨论，最终决定，一行的长度应该限制在100个字符以内。

例外1：如果一个注释语句含有一个超过100个字符的简单命令或者是一个URL，那么为了方便剪切和粘贴，你可以不在意行的长度。

例外2：引用语句可以不管这个限制，因为人们很少在意他们。这样也能简化工具的编写。

### 使用标准 Java 标注（Annotations）

标注应该先于同一语言元素中的其他修饰符。简单的标注（如 @Override）可以和其他语言元素在同一行中列出。如果有多个标注，或者参数化的标注，他们应该各自单独占一行并按字母顺序排列。

Java 预定义的三种标注在安卓中的标准用法如下：

- @Deprecated. 当被标注的元素不鼓励使用的时候你必须使用 @Deprecated 标注，你还需要使用 Javadoc 添加 @deprecated 标记并且该标记应该用可选的名字实现。此外，请记住，@Deprecated 方法依旧是可以执行的。（对于以前的代码，如果有一个 Javadoc 的 @deprecated 标记，请为代码加入 @Deprecated 标注。
- @Override. 当一个方法重写了其父类中的声明或者实现的时候，必须用 @Override 来注明。比如，如果你有一个方法有 Javadoc 的标记 @inheritdocs，并且是从一个类（不是接口）中派生而来的，你就必须再为方法添加 @Override 说明该方法重写了父类中的相关方法。
- SuppressWarnings：@SuppressWarnings 标注应该只在无法消除警告的情况下使用。如果一个警告通过了“无法消除”的测试，那么 @SuppressWarnings 标注会被使用，以确保所有的警告都反映了代码中的实际问题。

当必须使用 @SuppressWarnings 标注的时候，必须在其之前写一个 TODO 注释来说明“无法消除”的情况。这通常会标记出一个使用了一个比较尴尬的接口的违规类。例子如下：

~~~

// TODO: The third-party class com.third.useful.Utility.rotate() needs generics 
@SuppressWarnings("generic-cast")
List<String> blix = Utility.rotate(blax);

~~~

当需要 @SuppressWarnings 标注的时候，应该重构代码以隔离标注中使用的软件元素。

### 熟悉首字母缩略词

在为变量、方法以及类命名的时候，最好使用首字母缩略词。这些名称将会更具可读性：

| Good | Bad |
| ---- | --- |
| XmlHttpRequest | XMLHTTPRequest |
| getCustomerId | getCustomerID |
| class Html | class Html |
| String url | String URL |
| long id | long ID |

不论 JDK 还是 安卓的代码库都使用了首字母缩略词，因而他们是相当一致的，因此，想使自己的代码与他们一致是几乎不可能的。请咬紧牙关，尽量熟悉首字母缩略词。

### 使用 TODO 注释

为代码使用 TODO 注释是暂时，短期的解决方案，或者说是够用的但是并不完美。

TODO 注释必须用大写的 TODO 开头，后面跟一个冒号：

~~~

// TODO: Remove this code after the UrlTable2 has been checked in.

~~~

或者：

~~~

// TODO: Change this to use a flag instead of a constant.

~~~

如果你的 TODO 语句是“在未来的某一时期做什么”的格式，请确保你包含了一个求具体的日期（2005年11月份）或者一个特定的事件（在所有产品开发者了解v7协议后删除此代码）。

### 谨慎使用 Log

当日志记录是必须的时候，它对性能有显著的负面影响并且如果它不保持一定程度的简洁性的话，就会迅速失去其有效性。测试系统提供五种不同级别的日志记录：

- Error：这个等级的日志信息应该只在出现致命错误的时候使用，比如，有些事件会导致用户可见的结果并且如果不显示地删除数据，卸载应用，清理数据分区或者重新刷掉整个手机（可能更糟）的时候就无法修正该结果，这个级别的信息会一直记录下来。在向统计数据的服务器提交报告的时候，说明 ERROR 级别的日志中的情况通常是一种好的选择。
- Warning：这个等级的日志信息应该在事情比较严重或者出现预期之外的结果时使用，比如，有些事件会导致用户可见的结果，但是通过一些显示的动作，就像是等待或重启应用程序的应用甚至于下载新版的应用程序或者重新启动设备，可以在不丢失数据的情况下恢复。这个级别的信息会一直记录下来。在向统计信息的服务器报告的时候，说明 WARNING 级别的日志信息也是会考虑的。
- INFORMATIVE：这一级别的日志应当记录的是大多数人会感兴趣的信息。比如，当发现某种情况会造成广泛的影响时，尽管它不一定是错误，也应该记录。这种情况只应该由一个被认为在该域中是最具权威性（避免由非权威的组件做重复记录）的模块记录。始终记录该级别的日志信息。
- DEBUG：这一级别的日志信息应该用于进一步说明在设备上发生了什么和调查及调试异常行为相关的事件。你应该只有在需要收集关于组件的足够信息的时候进行记录。如果你的调试日志是主要的日志信息，那你可能需要使用 verbose 日志记录。这一级别的日志应当记录，即时是在发行版中也应当用 if(LOCAL_LOG) 或者 if(LOCAL_LOGD) 代码块中来包含该日志。而在LOCAL_LOG[D] 中则定义你的类或者子组件，故而要禁用所有这类记录是有可能的。在 if(LOCAL_LOG) 块中不应当含有执行的逻辑，所有为日志而构建的字符串也应该放在 if(LOCAL_LOG) 块中。如果日志调用会导致字符串的构建发生在 if(LOCAL_LOG) 块之外，那么不应该把日志调用分散到函数调用中。还有些代码仍然使用 if(localLOGV)，这也是可接受的，尽管这个名称并不规范。
- VERBOSE：这个级别的日志用于记录其他所有的信息，只会在调试版本中记录，并且应当包含在 if(LOCAL_LOGV)块中。故而它可以在默认情况下编译，任何因此构建的字符串在应当在 if(LOCAL_LOGV) 块中出现，并且在发行版中将会被剥离出来。

注意：

- 在给定的模块中，出了 VERBOSE 级别以外，一个错误应该尽可能只报告一次。：在模块内的函数调用链，只有最内层的函数应当返回错误信息，并且同一模块内的调用者只应添加一些有助于分离问题的日志信息。
- 在一连串模块中，除了 VERBOSE 级别以外，当底层模块检测到的无效数据来自较高模块，并且只有在日志提供了调用者无法使用的信息的时候，低级别模块将记录这种情况到 DEBUG 日志。具体来说，没有必要在抛出异常的，或者被记录的信息被包含在错误信息中的时候，记录当时的情况（异常中应该会包括所有相关的信息）。这在框架和应用的交互间尤为重要，并且由第三方应用程序造成的情况能够由框架合理处理的时候，不应该触发高于 DEBUG 级别的记录。唯一能触发 INFORMATIVE 或者更高级别记录的情况应该是当模块或者应用在其本身的自别检测到错误，或者错误来自较低级别的时候。
- 当有些日志记录可能会发生多次的时候，实施一些限制速率的机制来防止重复输出很多与副本相同（或者很相似）的日志信息是一个好方法。
- 失去网络连接被一致认为且期望是不应该无理由记录的。在 app 内失去网络连接会出现的后果应当在 DEBUG 或者 VERBOSE 级别记录（根据后果是否足够严重并且超出预期的情况足够被记录在发行版中）。
- 在一个文件系统中，一个可以使用或者代表第三方的完整文件系统不应该输出高于 INFORMAL 的日志。
- 来自不信任源的无效数据（包括任何共享设备上的文件，或者通过网络连接取得的数据）被检测出无效的时候，我们认为且应当不触发任何高于 DEBUG 级别的日志记录。（甚至应该尽可能地限制日志记录）
- 在心中记住，当对 String 对象使用+连接符的时候，会隐式地创建一个默认大小（16个字符）的 StringBuilder 对象，而且可能会有其他的临时 String 对象。比如，显示创建StringBuilder 的花销并不使用默认的‘+’操作符要大（而且事实上更加高效）。也请记住，即时没有读日志信息，调用 Log.v() 的代码，包括字符串的构建，是会被编译并且运行在发行版本中的。
- 任何用于给其他人看并且在发行版中也可用的日志，应当尽可能地简洁，并且应当是合理易懂的。这包括到 DEBUG 为止的全部日志。
- 如果可能的话，有意义的的日志应该保持在单行内。单行的长度在80到100个字符内是完全可以被接受的，但是长度在130到160各字符也是可以接受的（包括标签的长度），只是应当尽量避免。
- 报告成功的日志绝对不能使用高于 VERBOSE 级别的日志。
- 临时记录用于诊断一个很难重现的问题，而且应该保持在 DEBUG 或者 VERBOSE 级别，并且应当封闭在 if 块之中以允许其在编译期间能够被禁用。
- 对于在日志中泄露的安全信息要小心。私有信息应该尽量避免。受保护的内容是必须避免的。在编写框架代码的时候这是非常重要的，因为它并不会预先知道什么是私有信息或者受保护的信息。
- 不要使用 System.out.println()（或者 printf() 原生代码）。System.out 和 System.err 可以重定向到 /dev/null，所以你的打印语句不会有可见的影响。然而，所有银这些调用而构建的字符串仍然会被输出。
- 关于日志的最好的规则就是，你的日志不会把其他日志推到缓冲区中，正如其他日志不会这样对你的日志一样。

### 保持一致

我们的部分想法：保持一致。如果你在编辑代码，请花一点时间去观察一下周围人的代码以确定你的风格。如果他们在 if 语句的周围使用空格，你也应该这样。如果他们的注释是包含在用星号构成的方块里，你也要把注释放在星号构成的方块里。

需要风格指南的关键是有一个共同的代码词汇表，于是人们可以专注于你所说的内容，而不是你说话的方式。我们提出了整体的风格规范，所以人们知道这个专业词汇表。但是局部的风格也是很重要的，如果你在文件中加入的代码看起来和周围代码明显不同，那么读者读到这里的时候，这些代码会使他们的节奏被打断，这应当尽量避免。

## Javatests 风格规范。

### 遵循测试方法的命名便捷性

当为测试方法命名的时候，你可以使用下划线从特定的测试情况中分离出要测试的东西。这种风格可以更好地看出是在什么情况下进行测试。

例如：

~~~

testMethod_specificCase1 testMethod_specificCase2

void testIsDistinguishable_protanopia() {
    ColorMatcher colorMatcher = new ColorMatcher(PROTANOPIA)
    assertFalse(colorMatcher.isDistinguishable(Color.RED, Color.BLACK))
    assertTrue(colorMatcher.isDistinguishable(Color.X, Color.Y))
}

~~~