# BTrace_UserGuide_CN
BTrace用户指南_中文翻译

中文文档GitHub地址: [https://github.com/cncounter/BTrace_UserGuide_CN](https://github.com/cncounter/BTrace_UserGuide_CN)

英文文档在线地址: [https://kenai.com/projects/btrace/pages/UserGuide](https://kenai.com/projects/btrace/pages/UserGuide)

BTrace 项目源码: [https://github.com/btraceio/btrace](https://github.com/btraceio/btrace)

起始时间: 2016年5月19日








#  BTrace User's Guide 

#  BTrace用户指南


#####  ver. 1.2 (20101020) 

#####  


**BTrace** is a safe, dynamic tracing tool for Java. BTrace works by dynamically (bytecode) instrumenting classes of a running Java program. BTrace inserts tracing actions into the classes of a running Java program and hotswaps the traced program classes. 

**BTrace** 是一款针对Java的动态跟踪工具,安全又可靠。BTrace可以对已经启动的Java程序通过字节码插装类来动态执行. BTrace对运行程序的Java类插入跟踪操作代码并对相应的类进行热替换(hotswap)。



##  BTrace术语



**探测点(Probe Point)**


"location" or "event" at which a set of tracing statements are executed. Probe point is "place" or "event" of interest where we want to execute some tracing statements. 

是一个“位置(location)”或“事件(event)”, 在探测点可以执行相应的跟踪代码。探测点是我们想要执行某些跟踪代码的地“地方(place)”或“事件(event)”。




**跟踪动作/动作(Trace Actions or Actions)**

Trace statements that are executed whenever a probe "fires".

在某个探测点“触发(fires)”时执行的跟踪语句。



**处理方法 (Action Methods)**


BTrace trace statements that are executed when a probe fires are defined inside a static method a class. Such methods are called "action" methods. 

BTrace跟踪代码需要放在 class 的一个静态方法(static method)中, 在某个探测点“触发(fires)”时执行 。这种静态方法被称为“处理”方法 (Action Methods)。



##  BTrace程序结构


A BTrace program is a plain Java class that has one or more 

BTrace程序是一个普通的Java类, 其中有一到多个静态方法:


```
public static void
```
methods that are annotated with [BTrace annotations](#btrace_anno). The annotations are used to specify traced program "locations" (also known as "probe points"). The tracing actions are specified inside the static method bodies. These static methods are referred as "action" methods. 

静态方法的返回值类型为 `void` . 同时需要有 [BTrace 注解]()。注解用于指定跟踪程序的“位置”(也称为“探测点”)。跟踪代码就是静态方法的 body。这些静态方法称为“处理”方法。



##  BTrace的限制(Restrictions)


To guarantee that the tracing actions are "read-only" (i.e., the trace actions don't change the state of the program traced) and bounded (i.e., trace actions terminate in bounded time), a BTrace program is allowed to do only a restricted set of actions. In particular, a BTrace class

为了保证跟踪行为是 “只读” 的。(即跟踪行为不能改变被跟踪程序的状态)和边界(即.在限定的时间内,跟踪行为执行完毕), BTrace程序只允许执行一系列严格限制的操作。BTrace类具有如下限制:




*   can **not** create new objects.

*   不能创建新对象。


*   can **not** create new arrays.

*   不能创建新数组。


*   can **not** throw exceptions.

*   不能抛出异常。


*   can **not** catch exceptions.
*   不能捕获异常


*   can **not** make arbitrary instance or static method calls - only the public static methods of com.sun.btrace.BTraceUtils class  or methods declared in the same program may be called from a BTrace program.

*   不能调用任何实例方法或静态方法,只有 `com.sun.btrace.BTraceUtils` 类的 ` public static` 方法和BTrace 程序定义的方法可以调用。


*   **(pre 1.2)** can **not** have instance fields and methods. Only static public void returning methods are allowed for a BTrace class. And all fields have to be static.

*   **(1.2版本之前)** 不能含有实例域和实例方法。BTrace类中只允许静态的 public void 返回方法。而且所有的字段都必须是静态的。


*   can **not** assign to static or instance fields of target program's classes and objects. But, BTrace class can assign to it's own static fields ("trace state" can be mutated).

*   不能赋值给目标程序的静态域或者实例域(instance field)。但是, BTrace 类可以赋值给自身的静态域(static fields, “跟踪状态”是可以修改的)。


*   can **not** have outer, inner, nested or local classes.

*   不能含有 外部类,内部类,嵌套类或局部类(outer, inner, nested or local classes)。


*   can **not** have synchronized blocks or synchronized methods.

*   不能含有同步块或同步方法(synchronized blocks or synchronized methods)。


*   can **not** have loops (for, while, do..while)

*   不能使用循环(for, while, do..while)


*   can **not** extend arbitrary class (super class has to be java.lang.Object)

*   不能继承任何一个类(父类必须是 `java.lang.Object` )


*   can **not** implement interfaces.

*   不能实现接口。


*   can **not** contains assert statements.

*   不能使用断言语句(assert)。


*   can **not** use class literals. 
*   不能使用类名称字面常量(class literals)。


These restrictions could be circumvented by running BTrace in *unsafe* mode. Both the tracing script and the engine must be set up to require *unsafe* mode. The script must be annotated by *@BTrace(unsafe = true)* annotation and the engine must be started in *unsafe* mode. 

这些限制可以规避在*不* BTrace模式下运行。跟踪脚本和引擎的设置必须要*不*模式.脚本必须由* @BTrace注释(不安全= true)*注释和引擎必须启动*不*模式。


##  A simple BTrace program (1.2) 

##  一个简单的BTrace程序(1.2)


```
// import all BTrace annotations
import com.sun.btrace.annotations.*;
// import statics from BTraceUtils class
import static com.sun.btrace.BTraceUtils.*;

' ' '
/ /导入所有BTrace注释
进口com.sun.btrace.annotations。*;
/ /从BTraceUtils进口静力学类
进口静态com.sun.btrace.BTraceUtils。*;


// @BTrace annotation tells that this is a BTrace program
@BTrace
class HelloWorld {

/ / @BTrace注释告诉这是一个BTrace程序
@BTrace
HelloWorld类{


    // @OnMethod annotation tells where to probe.
    // In this example, we are interested in entry 
    // into the Thread.start() method. 
    @OnMethod(
        clazz="java.lang.Thread",
        method="start"
    )
    void func() {
        sharedMethod(msg);
    }




   void sharedMethod(String msg) {
        // println is defined in BTraceUtils
        println(msg);
   }
}

空白sharedMethod(字符串味精){
/ /定义println BTraceUtils
println(味精);
}
}


```

' ' '


##  A simple BTrace program ( Steps to run BTrace 

##  在简单的BTrace program to run BTrace(步骤


1.  Find the process id of the target Java process that you want to trace. You can use jps tool to find the pid.

2.  找到目标Java进程的进程id,您想要跟踪。您可以使用jps工具找到pid。



1.  Write a BTrace program - you may want to start modifying one of the samples.

2.  写一个BTrace程序,你可能想要开始修改的一个样本。



1.  Run btrace tool by the following command line: 
```
    btrace <pid> <btrace-script>
```

3所示。通过下面的命令行运行btrace工具:
' ' '
btrace < pid > < btrace-script >
' ' '


##  BTrace Command Line 

##  BTrace命令行


BTrace is run using the command line tool btrace as shown below:

BTrace运行使用命令行工具BTrace如下所示:


```
btrace [-I <include-path>] [-p <port>] [-cp <classpath>] <pid> <btrace-script> [<args>]
```

! ! ! ! ! ! !
btrace[㈠][-p include-path > < <港口>][-cp < < classpath >][btrace-script >室内滞留喷洒> < < args >]
! ! ! ! ! ! !


where

在哪里


*   _include-path_ is a set of include directories that are searched for header files. BTrace includes a simple preprocess with support for #define, #include and conditional compilation. It is **not** like a complete C/C++ preprocessor - but a useful subset. See the sample "ThreadBean.java". If -I is not specified, BTrace skips the preprocessor invocation step.

*   _include-path_是一组包括寻找头文件的目录。BTrace包含一个简单的预处理,支持# define,# include和条件编译.这是* * * *不像一个完整的C / c++预处理器,但是一个有用的子集。看过样品“ThreadBean.java”。如果没有指定- i,BTrace跳过预处理程序调用步骤。


*   _port_ is the port in which BTrace agent listens. This is optional argument.

*   _port_ BTrace代理侦听的端口。这是可选的参数。


*   _classpath_ is set of directories, jar files where BTrace searches for classes during compilation. Default is ".".

*   _classpath_设置的目录,在编译jar文件在BTrace搜索类。默认是“。”。


*   _pid_ is the process id of the traced Java program

*   _pid_跟踪Java程序的进程id


*   _btrace-script_ is the trace program. If it is a ".java", then it is compiled before submission. Or else, it is assumed to be pre-compiled [i.e., it has to be a .class] and submitted. 

*   _btrace-script_跟踪程序。如果它是一个”。java”,那么编译之前提交。否则,它被认为是预编译(即。,它必须是一个。类)和提交。


###  optional 

###  可选


*   _port_ is the server socket port at which BTrace agent listens for clients. Default is 2020.

*   _port_ BTrace代理的服务器套接字端口侦听客户端。默认是2020。


*   _path_ is the classpath used for compiling BTrace program. Default is ".".

*   _path_用于编译BTrace程序的类路径中。默认是“。”。


*   _args_ is command line arguments passed to BTrace program. BTrace program can access these using the built-in functions "$" and "$length". 

*   _args_命令行参数传递给BTrace程序。BTrace程序可以使用内置函数访问这些“美元”和“长度”美元。


##  Pre-compiling BTrace scripts 

##  预编译BTrace脚本


It is possible to precompile BTrace program using btracec script. btracec is a javac-like program that takes a BTrace program and produces a .class file.

可以预编译BTrace程序使用btracec脚本。btracec javac-like程序,BTrace程序并生成。类文件。


```
btracec [-I <include-path>] [-cp <classpath>] [-d <directory>] <one-or-more-BTrace-.java-files> 

' ' '
btracec[我<包括路径>][- cp <路径>][- d <目录>]< one-or-more-BTrace . java文件>


```

' ' '


where

在哪里


*   _include-path_ is a set of include directories that are searched for header files. BTrace includes a simple preprocess with support for #define, #include and conditional compilation. It is **not** like a complete C/C++ preprocessor - but a useful subset. See the sample "ThreadBean.java". If -I is not specified, BTrace skips the preprocessor invocation step.

*   _include-path_是一组包括寻找头文件的目录。BTrace包含一个简单的预处理,支持# define,# include和条件编译.这是* * * *不像一个完整的C / c++预处理器,但是一个有用的子集。看过样品“ThreadBean.java”。如果没有指定- i,BTrace跳过预处理程序调用步骤。


*   _classpath_ is the classpath used for compiling BTrace program(s). Default is "."

*   _classpath_ classpath用于编译BTrace项目(s)。默认是“。”


*   _directory_ is the output directory where compiled .class files are stored. Default is ".". 

*   _directory_就是编译输出目录。类文件存储。默认是“。”。


This script uses BTrace compiler class - rather than regular javac and therefore will validate your BTrace program at compile time [so that you can avoid BTrace verify error at runtime]. 

这个脚本使用BTrace——而不是常规的javac编译器类,因此将验证您的BTrace程序在编译时(这样可以避免BTrace验证错误在运行时)。


##  Starting an application with BTrace agent 

##  启动应用程序与BTrace代理


So far, we saw how to trace a running Java program. It is also possible to start an application with BTrace agent in it. If you want to start tracing the application from the very "beginning", you may want to start the app with BTrace agent and specify trace scripts along with it [i.e., BTrace agent is attach-on-demand loadable as well as pre-loadable agent] You can use the following command to start an app and specify BTrace script files. But, you need to [precompile your BTrace scripts](#precompile) for this kind of usage.

到目前为止,我们看到了如何跟踪正在运行的Java程序。也可以启动一个应用程序与BTrace代理.如果你想开始跟踪应用程序从一开始,你可能想要开始与BTrace代理和应用程序指定跟踪脚本(即随之.,BTrace代理attach-on-demand可加载以及pre-loadable代理),您可以使用以下命令启动应用程序并指定BTrace脚本文件.但是,你需要(预编译BTrace脚本)(#预编译)对这种用法。


```
java -javaagent:btrace-agent.jar=script=<pre-compiled-btrace-script1>[,<pre-compiled-btrace-script1>]* <MainClass> <AppArguments> 

' ' '
java - javaagent:btrace-agent。jar =脚本= < pre-compiled-btrace-script1 >,< pre-compiled-btrace-script1 >]* < MainClass > < AppArguments >


```

' ' '


When starting the application this way, the trace output goes to a file named .btrace in the current directory. Also, you can avoid starting server for other remote BTrace clients by specifying noServer=true as an argument to the BTrace agent.
There is a convenient script called **btracer** to do the above:

在启动应用程序时,跟踪输出到一个文件命名。btrace在当前目录.也可以避免其他远程启动服务器BTrace客户通过指定noServer BTrace剂= true作为参数。
有一个方便的脚本叫做* * btrace * *做上面的:


```
btracer <pre-compiled-btrace.class> <application-main-class> <application-args> 

! ! ! ! ! ! !
btracer < pre-compiled-btrace。班application-main-class > < application-args > > <


```

' ' '


###  Supported Arguments 

###  支持的论点


*   **bootClassPath** - boot classpath to be used

*   * * * * bootClassPath——引导类路径


*   **systemClassPath** - system classpath to be used

*   * * * * systemClassPath——使用系统类路径


*   **debug** - turns on verbose debug messages (true/false)

*   * * * * turns调试版本verbose据电文(true /贵方调试版本)


*   **unsafe** - do not check for btrace restrictions violations (true/false)

*   * *不* *,不检查btrace限制违反(真/假)


*   **dumpClasses** - dump the transformed bytecode to files (true/false)

*   * * dumpClasses * *——把转换后的字节码文件(真/假)


*   **dumpDir** - specifies the folder where the transformed classes will be dumped to

*   * * * * dumpDir——指定的文件夹将被转换类


*   **stdout** - redirect the btrace output to stdout instead of writing it to an arbitrary file (true/false)

*   * * * * stdout,btrace的输出重定向到标准输出而不是写一个任意文件(真/假)


*   **probeDescPath** - the path to search for probe descriptor XMLs

*   * * * * probeDescPath——路径搜索调查描述符中使用xml


*   **script** - the path to a script to be run at the agent startup

*   * - * * *脚本路径代理启动时要运行的脚本


*   **scriptdir** - the path to a directory containing scripts to be run at the agent startup

*   * * * * scriptdir——路径的目录包含在代理启动脚本运行


*   **scriptOutputFile** - the path to a file the btrace agent will store its output

*   * * * * scriptOutputFile——一个文件的路径btrace代理将存储输出


###  Important system properties 

###  重要的系统属性


*   **btrace.agentname** - use to distinguish the outputs of various btrace agents running on the same machine

*   * * btrace。agentname * *——用来区分各种btrace的输出代理运行在同一台机器上


##  BTrace Annotations 

##  BTrace注释


###  Method Annotations 

###  方法的注释


*   [@com.sun.btrace.annotations.OnMethod](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnMethod.html) annotation can be used to specify target class(es), target method(s) and "location(s)" within the method(s). An action method annotated by this annotation is called when the matching method(s) reaches specified the location. In OnMethod annotation, traced class name is specified by "clazz" property and traced method is specified by "method" property. "clazz" may be a fully qualified class name (like **java.awt.Component** or a regular expression specified within two forward slashes. Refer to the samples [NewComponent.java](https://kenai.com/projects/btrace/sources/hg/content/samples/NewComponent.java) and [Classload.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Classload.java?rev=317). The regular expression can match zero or more classes in which case all matching classes are instrumented. For example **/java\\.awt\\..+/** matches all classes in **java.awt package**. Also, method name can be a regular expression as well - matching zero or more methods. Refer to the sample [MultiClass.java](https://kenai.com/projects/btrace/sources/hg/content/samples/MultiClass.java). There is another way to abstractly specify traced class(es) and method(s). Traced classes and methods may be specified by annotation. For example, if the "clazz" attribute is specified as **@javax.jws.WebService** BTrace will instrument all classes that are annotated by the WebService annotation. Similarly, method level annotations may be used to specify methods abstractly. Refer to the sample [WebServiceTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/WebServiceTracker.java). It is also possible to combine regular expressions with annotations - like **@/com\\.acme\\..+/** matches any class that is annotated by any annotation that matches the given regular expression. It is possible to match multiple classes by specifying super type. i.e., match all classes that are subtypes of a given super type. **+java.lang.Runnable** matches all classes implementing **java.lang.Runnable** interface. Refer to the sample [SubtypeTracer.java](https://kenai.com/projects/btrace/sources/hg/content/samples/SubtypeTracer.java).

[@com.sun.btrace.annotations.OnMethod](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnMethod *.html)注释可以被用来指定目标类(es),目标方法(s)和“位置(s)“在方法(年代).一个动作方法注释的注释叫做当匹配方法(s)到达指定的位置.OnMethod注释,跟踪指定类名由“clazz”属性和“方法”属性指定的跟踪方法。“clazz”可能是一个全限定类名(如* * java.awt.组件* *或正则表达式中指定两个斜杠。参考样本(NewComponent.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/NewComponent.爪哇)和[Classload.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Classload.java?rev=317).的正则表达式可以匹配零个或多个类在这种情况下,所有匹配的类仪器。例如* * / java \ \ .awt \ \ . .+ / * *匹配所有在* *的java类。awt包* *.Also, the method name can be a regular expression as well - matching zero or more. The methods Refer to the sample/MultiClass. Java (https://kenai.com/projects/btrace/sources/hg/content/samples/MultiClass.java)。有另一种抽象地指定跟踪类(es)和方法(s)。追踪可能由注释指定的类和方法.例如,如果“clazz”属性指定为* * @javax.jws。网络服务* * BTrace将仪器所有类WebService注释的注释.同样,方法级注释可用于指定抽象方法。参考示例(WebServiceTracker.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/WebServiceTracker.java)。还可以结合正则表达式注释——就像* * @ / com \ \ .acme \ \。.+ / * *匹配任何注释,注释的类匹配给定的正则表达式。可以通过指定超级类型匹配多个类。即.,匹配所有类给定的超类型的子类型。* * + . lang。可运行* *匹配所有类实现* * . lang。可运行* *接口。参考示例(SubtypeTracer.爪哇](https://kenai.com/projects/btrace/sources/hg/content/samples/SubtypeTracer.java)。


*   [@com.sun.btrace.annotations.OnTimer](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnTimer.html) annotation can be used to specify tracing actions that have to run periodically once every N milliseconds. Time period is specified as long "value" property of this annotation. Refer to the sample [Histogram.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Histogram.java)
*   [@com.sun.btrace.annotations.OnError](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnError.html) annotation can be used to specify actions that are run whenever any exception is thrown by tracing actions of some other probe. BTrace method annotated by this annotation is called when any exception is thrown by any of the other BTrace action methods in the same BTrace class.

[@com.sun.btrace.annotations.OnTimer](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnTimer *.html)注释可以被用来指定跟踪行动必须定期运行每N个毫秒。长时间被指定为“价值”属性的注释.参考样本(Histogram.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/Histogram.java)
[@com.sun.btrace.annotations.OnError](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnError *.html)可以使用注释来指定动作运行任何异常跟踪行动的其他调查.BTrace方法注释的注释时调用任何异常的任何其他类在同一个BTrace BTrace行动方法。


*   [@com.sun.btrace.annotations.OnExit](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnExit.html) annotation can be used to specify actions that are run when BTrace code calls "exit(int)" built-in function to finish the tracing "session". Refer to the sample [ProbeExit.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ProbeExit.java).

[@com.sun.btrace.annotations.OnExit](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnExit *.html)可以使用注释来指定动作运行当BTrace代码调用“退出(int)内置函数完成跟踪“会话”。参考示例(ProbeExit.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/ProbeExit.java)。


*   [@com.sun.btrace.annotations.OnEvent](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnEvent.html) annotation is used to associate tracing methods with "external" events send by BTrace client. BTrace methods annotated by this annotation are called when BTrace client sends an "event". Client may send an event based on some form of user request to send (like pressing Ctrl-C or a GUI menu). String value may used as the name of the event. This way certain tracing actions can be executed whenever an external event "fires". As of now, the command line BTrace client sends "events" whenever use presses Ctrl-C (SIGINT). On SIGINT, a console menu is shown to send an event or exit the client [which is the default for SIGINT]. Refer to the sample [HistoOnEvent.java](https://kenai.com/projects/btrace/sources/hg/content/samples/HistoOnEvent.java)
*   [@com.sun.btrace.annotations.OnLowMemory](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnLowMemory.html) annotation can be used to trace memory threshold exceed event. See sample [MemAlerter.java](https://kenai.com/projects/btrace/sources/hg/content/samples/MemAlerter.java)
*   [@com.sun.btrace.annotations.OnProbe](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnProbe.html) annotation can be used to specify to avoid using implementation internal classes in BTrace scripts. @OnProbe probe specifications are mapped to one or more @OnMethod specifications by the BTrace VM agent. Currently, this mapping is done using a XML probe descriptor file [accessed by the BTrace agent]. Refer to the sample [SocketTracker1.java](https://kenai.com/projects/btrace/sources/hg/content/samples/SocketTracer1.java) and associated probe descriptor file java.net.socket.xml. When running this sample, this xml file needs to be copied in the directory where the target JVM runs (or fix probeDescPath option in btracer.bat to point to whereever the .xml file is). 

[@com.sun.btrace.annotations.OnEvent](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnEvent *.html)注释用于将跟踪方法与“外部”事件由BTrace端发送。BTrace这个注释注释的方法被称为当BTrace客户端发送一个“事件”.客户端可以发送一个事件基于某种形式的用户请求发送(如按ctrl - c或GUI菜单)。字符串值可以作为事件的名称.这种方式确定跟踪行动时可以执行外部事件“火灾”。到目前为止,命令行BTrace客户端发送“事件”时使用按ctrl - c(SIGINT).信号情报,控制台菜单显示发送一个事件或退出客户端(SIGINT)的默认值。参考示例(HistoOnEvent.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/HistoOnEvent.java)
*(@com.sun.btrace.annotations.OnLowMemory)(http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnLowMemory.html)注释可用于跟踪内存阈值超过事件.看到样品(MemAlerter.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/MemAlerter.java)
[@com.sun.btrace.annotations.OnProbe](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnProbe *.html)可以使用注释来指定,以避免在BTrace脚本使用内部类实现.@OnProbe探针规格被映射到一个或多个@OnMethod BTrace VM规范的代理.目前,这种映射是使用XML描述符文件调查(由BTrace代理访问)。参考示例(SocketTracker1.java)(https://kenai.java.net.socket.xml com/projects/btrace/sources/hg/content/samples/SocketTracer1.java)和相关调查描述符文件.当运行这个示例,这个xml文件需要被复制在目标JVM运行的目录(或修复在btrace probeDescPath选项。蝙蝠指无论。xml文件)。


###  Argument Annotations 

###  参数注释


*   [@com.sun.btrace.annotations.Self](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/Self.html) annotation can be used to mark an argument to hold **this** (or **self**) value. Refer to the samples [AWTEventTracer.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AWTEventTracer.java) or [AllCalls1.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AllCalls1.java?rev=404)
*   [@com.sun.btrace.annotations.Return](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/Return.html) annotation can be used to mark an argument to hold the return value. Refer to the sample [Classload.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Classload.java)
*   [@com.sun.btrace.annotations.ProbeClassName (since 1.1)](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/ProbeClassName.html) annotation can be used to mark an argument to hold the probed class name value. Refer to the sample [AllMethods.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AllMethods.java?rev=404)
*   [@com.sun.btrace.annotations.ProbeMethodName (since 1.1)](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/ProbeMethodName.html) annotation can be used to mark an argument to hold the probed method name. Refer to the sample [WebServiceTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/WebServiceTracker.java)
*   since 1.2 it accepts boolean parameter **fqn** to get a fully qualified probed method name

*(@com.sun.btrace.annotations.Self)(http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/Self.html)注释可以用来标记一个论点持有* *本* *(或自我* * * *)的价值.参考样本(AWTEventTracer.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/AWTEventTracer.java)或(AllCalls1.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/AllCalls1.java?rev=404)
*(@com.sun.btrace.annotations.Return)(http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/Return.html)注释可以用来标记一个参数返回值.参考样本(Classload.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/Classload.java)
*[@com.sun.btrace.annotations。ProbeClassName(自http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/ProbeClassName](1.1)..java ?穿= 404)
*[@com.sun.btrace.annotations。ProbeMethodName(自http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/ProbeMethodName](1.1).html)加注can,mark一年论点依据probed d1078。这位[列WebServiceTracker.java](《https://kenai.com/projects/btrace/sources/hg/content/samples/WebServiceTracker.java)
* 1.2年以来它接受布尔参数* * fqn * *完全限定的探测方法名称


*   [@com.sun.btrace.annotations.Duration](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/Duration.html) annotation can be used to mark an argument to hold the duration of the method call in nanoseconds. The argument must be a long.  Use with **Kind.RETURN** or **Kind.ERROR** locations.

[@com.sun.btrace.annotations.Duration](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/Duration *.


*   [@com.sun.btrace.annotations.TargetInstance (since 1.1)](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/TargetInstance.htm) annotation can be used to mark an argument to hold the called instance value. Refer to the sample [AllCalls2.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AllCalls2.java?rev=404)
*   [@com.sun.btrace.annotations.TargetMethodOrField (since 1.1)](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/TargetMethodOrField.html) can be used to mark an argument to hold the called method name. Refer to the samples [AllCalls1.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AllCalls1.java?rev=404) or [AllCalls2.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AllCalls2.java?rev=404)
*   since 1.2 it accepts boolean parameter **fqn** to get a fully qualified target method name

*[@com.sun.btrace.annotations。TargetInstance(自http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/TargetInstance](1.1).htm)注释可以用来标记一个参数称为实例的值。参考示例(AllCalls2.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/AllCalls2.java ?穿= 404)
*(@com.sun.btrace.annotations。TargetMethodOrField(1.1)](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/TargetMethodOrField.html)可以用来标记一个论点被调用的方法名。参考样本(AllCalls1.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/AllCalls1.爪哇? rev respir = 404)或[AllCalls2.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AllCalls2.java?rev=404)
* 1.2年以来它接受布尔参数* * fqn * *完全限定目标方法名称


####  Unannotated arguments 

####  未经参数


The unannotated BTrace probe method arguments are used for the signature matching and therefore they must appear in the order they are defined in the traced method. However, they can be interleaved with any number of annotated arguments. If an argument of type *AnyType[]* is used it will "eat" all the rest of the arguments in they order.
The exact meaning of the unannotated arguments depends on the **Location** used:

的未经BTrace探针方法参数用于签名匹配,因此他们必须出现的顺序定义的跟踪方法.然而,他们可以与任意数量的交错注释参数。如果一个参数类型的* AnyType[]*使用它将“吃”所有其他参数,按照他们的顺序排列。
未经参数的具体含义取决于使用* * * *位置:


*   **Kind.ENTRY, Kind.RETURN**- the probed method arguments

*   * *。条目,善良。返回* *——探测方法参数


*   **Kind.THROW** - the thrown exception

*   * *。把* *——抛出异常


*   **Kind.ARRAY_SET, Kind.ARRAY_GET** - the array index

*   * *。ARRAY_SET,善良。ARRAY_GET * *——数组索引


*   **Kind.CATCH** - the caught exception

*   * *。抓* *——捕获异常


*   **Kind.FIELD_SET** - the field value

*   * *。FIELD_SET * *——字段值


*   **Kind.LINE** - the line number

*   * *。* *行——行号


*   **Kind.NEW** - the class name

*   * *。新* *——类名


*   **Kind.ERROR** - the thrown exception 

*   * *。错误* *——抛出异常


###  Field Annotations 

###  字段的注释


*   [@com.sun.btrace.annotations.Export](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/Export.htm) annotation can be used with BTrace fields (static fields) to specify that the field has to be mapped to a jvmstat counter. Using this, a BTrace program can expose tracing counters to external jvmstat clients (such as jstat). Refer to the sample [ThreadCounter.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadCounter.java)
*   [@com.sun.btrace.annotations.Property](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/Property.html) annotation can be used to flag a specific (static) field as a MBean attribute. If a BTrace class has atleast one static field with @Property attribute, then a MBean is created and registered with platform MBean server. JMX clients such as VisualVM, jconsole can be used to view such BTrace MBeans. After attaching BTrace to the target program, you can attach VisualVM or jconsole to the same program and view the newly created BTrace MBean. With VisualVM and jconsole, you can use MBeans tab to view the BTrace domain and check out it's attributes. Refer to the samples [ThreadCounterBean.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadCountBean.java) and [HistogramBean.java](https://kenai.com/projects/btrace/sources/hg/content/samples/HistogramBean.java).

[@com.sun.btrace.annotations.Export](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/Export *.htm)可以使用注释BTrace字段(静态字段)来指定字段必须映射到一个jvmstat计数器.使用这个,BTrace程序可以使跟踪计数器外部jvmstat客户(如stat)。参考示例(ThreadCounter.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadCounter.java)
*(@com.sun.btrace.annotations.Property)(http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/Property.html)可以使用注释标记特定的(静态)字段作为一个MBean属性.如果一个BTrace类有至少一个静态字段@ property属性,然后创建一个MBean,注册平台MBean服务器.VisualVM引领JMX客户,jconsole can,BTrace such view MBeans.attaching After BTrace to the目标方案,你可以attach VisualVM然而jconsole to the same subject方案MBean BTrace日增新兴the view.VisualVM和jconsole,您可以使用MBeans选项卡查看BTrace域和查看它的属性。参考样本(ThreadCounterBean.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadCountBean.java)和(HistogramBean.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/HistogramBean.java)。


*   [@com.sun.btrace.annotations.TLS](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/TLS.html) annotation can be used with BTrace fields (static fields) to specify that the field is a thread local field. Please, be aware that you can not access such marked fields in other than **@OnMethod** handlers. Each Java thread gets a separate "copy" of the field. In order for this to work correctly the field type must be either  **immutable** (eg. primitives) or **cloneable** (implements  Cloneable interface and overrides clone() method).These thread local fields may be used by BTrace programs to identify whether we reached multiple probe actions from the same thread or not. Refer to the samples [OnThrow.java](https://kenai.com/projects/btrace/sources/hg/content/samples/OnThrow.java) and [WebServiceTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/WebServiceTracker.java)

[@com.sun.btrace.annotations.TLS](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/TLS *.html)注释可用于BTrace字段(静态字段)来指定字段是一个线程本地字段.请注意,您不能访问这些标记字段在其他比* * @OnMethod * *处理程序。每个Java线程获得一个单独的“复制”的领域.为了让它正常工作的字段类型必须* *不可变* *(如。原语)或* *可克隆* *(实现可克隆接口和重写clone()方法).这些线程本地字段可以使用通过BTrace程序来确定我们是否达到多个探针操作相同的线程。参考样本(OnThrow.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/OnThrow.java)和[WebServiceTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/WebServiceTracker.java)


###  Class Annotations 

###  类注释


*   [@com.sun.btrace.annotations.DTrace](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/DTrace.html) annotation can be used to associate a simple one-liner D-script (inlined in BTrace Java class) with the BTrace program. Refer to the sample [DTraceInline.java](https://kenai.com/projects/btrace/sources/hg/content/samples/DTraceInline.java?rev=317).

[@com.sun.btrace.annotations.DTrace](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/DTrace *.html)注释可用于把一个简单的一行程序d脚本(在BTrace内联Java类)BTrace程序。参考示例(DTraceInline.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/DTraceInline.java?rev=317)。


*   [@com.sun.btrace.annotations.DTraceRef](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/DTraceRef.html) annotation can be used to associate a D-script (stored in a separate file) with the BTrace program. Refer to the sample [DTraceRefDemo.java](https://kenai.com/projects/btrace/sources/hg/content/samples/DTraceRefDemo.java?rev=317).

[@com.sun.btrace.annotations.DTraceRef](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/DTraceRef *.html)注释可用于关联d脚本(存储在一个单独的文件)的BTrace程序。参考示例(DTraceRefDemo.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/DTraceRefDemo.java?rev=317)。


*   [@com.sun.btrace.annotations.BTrace](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/BTrace.html) annotation must be used to designate a given Java class as a BTrace program. This annotation is enforced by the BTrace compiler as well as by the BTrace agent. 

*(@com.sun.btrace.annotations.BTrace)(http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/BTrace.html)必须使用注释指定给定的Java类作为BTrace程序.这个注释是由BTrace执行编译器以及BTrace代理。


##  DTrace Integration 

##  DTrace集成


Solaris DTrace is a dynamic, safe tracing system for Solaris programs - both kernel and user land programs. Because of the obvious parallels b/w DTrace and BTrace, it is natural to expect integration b/w BTrace and DTrace. There are two ways in which BTrace is integrated with DTrace.

Solaris DTrace动态、安全跟踪Solaris系统程序,内核和用户程序.因为明显的b / w DTrace和BTrace的相似之处,很自然的期待集成b / w BTrace和DTrace。有两种方法,用DTrace BTrace集成。


*   BTrace program can raise a DTrace probe - by calling dtraceProbe -- see BTraceUtils javadoc referred above. For this feature to work, you need to be running on **Solaris 10 or beyond**. For other platforms (Solaris 9 or below or any other OS), dtraceProbe() will be a no-op.

*   BTrace程序可以提高DTrace探测,通过调用dtraceProbe——看到BTraceUtils javadoc上面提到。对于这个功能,您需要运行在Solaris 10 * *或* *之外.为其他平台(Solaris 9或低于或任何其他操作系统),dtraceProbe()将是一个空操作。


*   BTrace program can associate a D-script with it-- by @DTrace annotation (if the D-script is a simple one liner) or by @DTraceRef if the D-script is longer and hence stored outside of the BTrace program. See DTrace integration samples in the BTrace samples section below. This feature works using the . For this DTrace feature to work (o.e., being able to run associated D-script), you need to be running on **Solaris 11 build 35 or beyond**. You may want to check whether you have **/usr/share/lib/java/dtrace.jar** on your machine or not. @DTrace and @DTraceRef annotations are ignored on other platforms (Solaris 10 or below or any other OS). 

*   BTrace程序可以与它关联d脚本,由@DTrace注释(如果d脚本是一个简单的一个班轮)或@DTraceRef如果d脚本的时间更长,因此存储BTrace之外的 程序。见下面的DTrace集成在BTrace样品部分样品。该特性使用工作。对于这个工作(o.e DTrace特性.,能够运行相关的d脚本),您需要运行在Solaris 11构建35岁或超过* * * *。您可能需要检查一下你是否已经* * / usr / share / lib / java / dtrace。jar * *在您的机器上.@DTrace和@DTraceRef注释忽略其他平台上的Solaris 10(或低于或任何其他操作系统)。


##  BTrace Samples 

##  BTrace样品


[BTrace samples](https://kenai.com/projects/btrace/sources/hg/content/samples)

[BTrace备](https://kenai.com/projects/btrace/sources/hg/content/samples)


#####  One liners about samples: 

#####  一个衬垫样品:


*   [AWTEventTracer.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AWTEventTracer.java?rev=267) - demonstrates tracing of AWT events by instrumenting EventQueue.dispatchEvent() method. Can filter events by instanceof check. This sample filters and prints only focus events.

*(AWTEventTracer.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/AWTEventTracer.java?rev=267)——演示了通过插装EventQueue AWT事件的跟踪.dispatchEvent()方法。可以通过instanceof检查过滤事件。这个示例只过滤器和打印焦点事件。


*   [AllLines.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AllLines.java?rev=404) - demonstrates line number based BTrace probes. It is possible to probe into any class (or classes) and line number(s). When the specified line number(s) of specified class(es) is reached, the BTrace probe fires and the corresponding action is executed.

*(AllLines.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/AllLines.java?rev=404)——显示行号BTrace探针.可以调查任何类(或类)和行号(s).当指定的行号(s)指定的类(es),BTrace探测火灾和相应的执行动作。


*   [AllSync.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AllSync.java?rev=404) - demonstrates tracing of a synchronized block entry/exit.

*(AllSync.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/AllSync.java?rev=404),演示了一个同步块的跟踪进入/退出。


*   [ArgArray.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ArgArray.java?rev=267) - prints all input arguments in every readXXX method of every class in java.io package. Demonstrates argument array access and multiple class/method matching in probe specifications.

*(ArgArray.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/ArgArray.java?rev=267)——打印所有输入参数在每个readXXX每个类的java方法。io包.显示参数数组访问和多类/方法在探针规格匹配。


*   [Classload.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Classload.java?rev=267) - prints stack trace on every successful classload (defineClass returns) by any userdefined class loader.

[Classload.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Classload *.java ?牧师= 267)——打印堆栈跟踪每个成功classload任何userdefined类装入器(defineClass返回)。


*   [CommandArg.java](https://kenai.com/projects/btrace/sources/hg/content/samples/CommandArg.java?rev=404) - demonstrates BTrace command line argument access.

*(CommandArg.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/CommandArg.java?rev=404)——演示BTrace命令行参数访问。


*   [Deadlock.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Deadlock.java?rev=317) - demonstrates @OnTimer probe and deadlock() built-in function.

*(Deadlock.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/Deadlock.java?rev=317)——演示@OnTimer探针和死锁()内置函数。


*   [DTraceInline.java](https://kenai.com/projects/btrace/sources/hg/content/samples/DTraceInline.java?rev=267) - demonstrates @DTrace annotation to associate a simple one-line D-script with the BTrace program.

[DTraceInline.java](https://kenai.com/projects/btrace/sources/hg/content/samples/DTraceInline *.java ?牧师= 267)演示了一个简单的一行@DTrace注释关联与BTrace d脚本程序。


*   [DTraceDemoRef.java](https://kenai.com/projects/btrace/sources/hg/content/samples/DTraceDemoRef.java?rev=267) - demonstrates @DTraceRef annotation to associate a D-script file with the BTrace program. This sample associates classload.dwith itself.

[DTraceDemoRef.java](https://kenai.com/projects/btrace/sources/hg/content/samples/DTraceDemoRef *.java ? = 267)演示了牧师@DTraceRef注释关联BTrace的d脚本文件程序。此示例将classload关联起来。dwith本身。


*   [FileTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/FileTracker.java?rev=404) - prints file open for read/write by probing into File{Input/Output}Stream constructor entry points.

[FileTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/FileTracker *.打开java ?牧师= 404)——打印文件的读/写探讨文件输入/输出} {流构造函数入口点。


*   [FinalizeTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/FinalizeTracker.java?rev=404) - demonstrates that we can print all fields of an object and access (private) fields (read-only) as well. This is useful in debugging / troubleshooting scenarios. This sample prints info on close() /finalize() methods of java.io.FileInputStream class.

[FinalizeTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/FinalizeTracker *.java ?牧师= 404),表明我们可以打印对象的所有字段和访问(私人)字段(只读)。这对调试/故障排除的情况下是很有用的.这个样例输出信息close()/ io的finalize()方法。FileInputStream类。


*   [Histogram.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Histogram.java?rev=404) - demonstrates usage of BTrace maps for collecting histogram (of javax.swing.JComponent objects created by an app - histogram by subclass name and count).

*(Histogram.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/Histogram.java?rev=404),演示了使用收集直方图(javax.swing BTrace地图.JComponent对象创建的应用程序——直方图子类名称和数量)。


*   [HistogramBean.java](https://kenai.com/projects/btrace/sources/hg/content/samples/HistogramBean.java?rev=404) - demonstrates JMX integration. This sample exposes a Map as MBean attribute using @Property annotation.

HistogramBean.java *[https://kenai.com/projects/btrace/sources/hg/content/samples/HistogramBean.java?rev=404](JMX integration demonstrates)-.这个示例公开地图MBean属性使用@ property注释。


*   [HistoOnEvent.java](https://kenai.com/projects/btrace/sources/hg/content/samples/HistoOnEvent.java?rev=404) - demonstrates getting trace output based on client side event. After starting the script by BTrace client, press Ctrl-C to get menu to send event or exit. On sending event, histogram is printed. This way client can "pull" trace output whenever needed rather than BTrace agent pushing the trace output always or based on timer.

*(HistoOnEvent.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/HistoOnEvent.java?rev=404)——显示跟踪输出基于客户端事件.BTrace脚本的客户端启动后,按ctrl - c得到菜单发送事件或退出。在发送事件,直方图是打印出来.这一“客户”can way针织而需要更有效的人权记录输出BTrace代理人pushing the记录输出黄金timer基于合作始终。


*   [JdbcQueries.java](https://kenai.com/projects/btrace/sources/hg/content/samples/JdbcQueries.java?rev=408) - demonstrates BTrace aggregation facility. This facility is similar to DTrace aggregation facility.

*(JdbcQueries.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/JdbcQueries.java?rev=408)——演示BTrace聚合设备.此设备类似于DTrace聚合设备。


*   [JInfo.java](https://kenai.com/projects/btrace/sources/hg/content/samples/JInfo.java?rev=404) - demonstrates printVmArguments(), printProperties() and printEnv() built-in functions.

*(JInfo.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/JInfo.java?rev=404)——演示printVmArguments(),printProperties()和printEnv()内置函数。


*   [JMap.java](https://kenai.com/projects/btrace/sources/hg/content/samples/JMap.java?rev=404) - demonstrates dumpHeap() built-in function to dump (hprof binary format) heap dump of the target application.

[JMap.java](https://kenai.com/projects/btrace/sources/hg/content/samples/JMap *.java ? = 404)演示了牧师dumpHeap()内置函数来转储(hprof二进制格式)目标应用程序的堆转储。


*   [JStack.java](https://kenai.com/projects/btrace/sources/hg/content/samples/JStack.java?rev=317) - demonstrates jstackAll() built-in function to print stack traces of all the threads.

*(JStack.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/JStack.java?rev=317)——演示jstackAll()内置函数打印所有线程的堆栈跟踪。


*   [LogTracer.java](https://kenai.com/projects/btrace/sources/hg/content/samples/LogTracer.java?rev=317) - demonstrates trapping into an instance method (Logger.log) and printing private field value (of LogRecord object) by field() and objectValue() built-in functions.

*(LogTracer.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/LogTracer.java?rev=317)——显示捕获到一个实例方法(记录器.日志)和印刷私有字段值(LogRecord对象)字段()和objectValue()内置函数。


*   [MemAlerter.java](https://kenai.com/projects/btrace/sources/hg/content/samples/MemAlerter.java?rev=267) - demonstrates tracing low memory event by @OnLowMememory annotation.

*(MemAlerter.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/MemAlerter.java?rev=267)——展示了事件跟踪低内存@OnLowMememory注释。


*   [Memory.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Memory.java?rev=404) - prints memory stat once every 4 seconds by a timer probe. Demonstrates memory stat built-in functions.

*(Memory.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/Memory.java?rev=404)——打印内存统计每隔4秒计时器调查.显示内存统计内置函数。


*   [MultiClass.java](https://kenai.com/projects/btrace/sources/hg/content/samples/MultiClass.java?rev=317) - demonstrates inserting trace code into multiple methods of multiple classes using regular expressions for clazz and method fields of @OnMethod annotation.

[MultiClass.java](https://kenai.com/projects/btrace/sources/hg/content/samples/MultiClass *.java ?牧师= 317)——显示跟踪代码插入多个方法的多个类使用正则表达式clazz @OnMethod注释的字段和方法。


*   [NewComponent.java](https://kenai.com/projects/btrace/sources/hg/content/samples/NewComponent.java?rev=317) - tracks every java.awt.Component creation and increments a counter and prints the counter based on a timer.

*(NewComponent.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/NewComponent.java?rev=317)每java.awt—跟踪.组件创建和增加一个计数器和柜台打印基于一个计时器。


*   [OnThrow.java](https://kenai.com/projects/btrace/sources/hg/content/samples/OnThrow.java?rev=317) - prints exception stack trace every time any exception instance is created. In most scenarios, exceptions are thrown immediately after creation. So, it we get stack trace of throw points.

*(OnThrow.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/OnThrow.java?rev=317)——打印异常堆栈跟踪每次异常实例被创建.在大多数情况下,异常被抛出后立即创建。所以我们得到的堆栈跟踪丢分。


*   [ProbeExit.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ProbeExit.java?rev=317) - demonstrates @OnExit probe and exit(int) built-in function.

*(ProbeExit.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/ProbeExit.java?rev=317)——演示@OnExit探针和退出(int)内置函数。


*   [Profiling.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Profiling.java?rev=404) - demonstrates the usage of the profiling support

*(Profiling.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/Profiling.java?rev=404),演示了使用的分析支持


*   [Sizeof.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Sizeof.java?rev=404) - demonstrates "sizeof" built-in function that can be used to get (approx.) size of a given Java object. It is possible to get size-wise histogram etc. using this built-in.

*(Sizeof.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/Sizeof.java?rev=404),演示了“运算符”内置函数,可以用来获取(约.)给定的Java对象的大小。可以得到使用这个内置的阴茎直方图等。


*   [SocketTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/SocketTracker.java?rev=404) - prints every server socker creation/bind and client socket accepts.

*(SocketTracker.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/SocketTracker.java?rev=404)——打印每个服务器猛击者创建/绑定和客户端套接字接受。


*   [SocketTracker1.java](https://kenai.com/projects/btrace/sources/hg/content/samples/SocketTracker1.java?rev=404) - similar to SocketTracker.java sample, except that this sample uses @OnProbe probes to avoid using Sun specific socket channel implementation classes in BTrace program. Instead @OnProbe probes are mapped to @OnMethod probes by BTrace agent.

*(SocketTracker1.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/SocketTracker1.java?rev=404)- SocketTracker相似.java样本,除了这个示例使用@OnProbe探针来避免使用太阳BTrace程序中的特定套接字通道实现类.相反@OnProbe探针被映射到@OnMethod探测器由BTrace代理。


*   [SysProp.java](https://kenai.com/projects/btrace/sources/hg/content/samples/SysProp.java?rev=317) - demonstrates that it is okay to probe into System classes (like java.lang.System) and call BTrace built-in functions in the action method.

*(SysProp.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/SysProp.java?rev=317),表明它是好的调查系统类(如. lang.系统),叫BTrace内置函数的操作方法。


*   [SubtypeTracer.java](https://kenai.com/projects/btrace/sources/hg/content/samples/SubtypeTracer.java?rev=267) - demonstrates that it is possible to match all subtypes of a given supertype.

*(SubtypeTracer.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/SubtypeTracer.java?rev=267)——表明,它可以匹配所有给定的超类型的子类型。


*   [ThreadCounter.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadCounter.java?rev=317) - demonstrates use of jvmstat counters from BTrace programs.

*(ThreadCounter.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadCounter.java?rev=317),演示了使用BTrace jvmstat计数器的程序。


*   [ThreadCounterBean.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadCounterBean.java?rev=404) - demonstrates exposing the BTrace program as a JMX bean with one attribute (using @Property annotation).

*(ThreadCounterBean.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadCounterBean.java ?牧师= 404),展示了公开BTrace程序作为一个JMX bean与一个属性(使用@ property注释)。


*   [ThreadBean.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadBean.java?rev=267) - demonstrates the use of preprocessor of BTrace [and JMX integratio].

*(ThreadBean.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadBean.java?rev=267)——演示了使用预处理器的BTrace[和JMX蒙特卡罗]。


*   [ThreadStart.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadStart.java?rev=317) - demonstrates raising DTrace probes from BTrace programs. See also jthread.d - the associated D-script. This sample raises a DTrace USDT probe whenever the traced program enters java.lang.Thread.start() method. The BTrace program passes JavaThread's name to DTrace.

*(ThreadStart.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadStart.java?rev=317),展示了从BTrace筹集DTrace探测项目。参见jthread.d - d脚本相关联。这个示例时提出了一个DTrace USDT探测跟踪程序进入java.lang.Thread.start()方法。BTrace程序通过JavaThread DTrace的名字。


*   [Timers.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Timers.java?rev=404) - demonstrates multiple timer probes (@OnTimer) in a BTrace program.

*(Timers.java)(https://kenai.com/projects/btrace/sources/hg/content/samples/Timers.java?rev=404)——显示多个定时器探针(@OnTimer)在BTrace程序。


*   [URLTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/URLTracker.java?rev=404) - prints URL every time URL.openConnection returns successfully. This program uses jurls.d D-script as well (to show histogram of URLs opened via DTrace).

*(URLTracker.java)URL(https://kenai.com/projects/btrace/sources/hg/content/samples/URLTracker.java?rev=404)——打印每次URL。openConnection成功返回。这个程序使用jurls.d d脚本(显示url打开通过DTrace)的柱状图。


*   [WebServiceTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/WebServiceTracker.java?rev=404) - demonstrates tracing classes and methods by specifying class and method level annotations rather than class and method names.

[WebServiceTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/WebServiceTracker *.java ?牧师= 404)——显示跟踪类和方法通过指定类和方法级别注释而不是类和方法的名称。



