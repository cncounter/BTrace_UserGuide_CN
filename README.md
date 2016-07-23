# BTrace_UserGuide_CN
BTrace用户指南_中文翻译

中文文档GitHub地址: [https://github.com/cncounter/BTrace_UserGuide_CN](https://github.com/cncounter/BTrace_UserGuide_CN)

英文文档在线地址: [https://kenai.com/projects/btrace/pages/UserGuide](https://kenai.com/projects/btrace/pages/UserGuide)

BTrace 项目源码: [https://github.com/btraceio/btrace](https://github.com/btraceio/btrace)

起始时间: 2016年5月19日




#  BTrace User's Guide 

#####  ver. 1.2 (20101020) 

**BTrace** is a safe, dynamic tracing tool for Java. BTrace works by dynamically (bytecode) instrumenting classes of a running Java program. BTrace inserts tracing actions into the classes of a running Java program and hotswaps the traced program classes. 

##  BTrace Terminology 

**Probe Point**

"location" or "event" at which a set of tracing statements are executed. Probe point is "place" or "event" of interest where we want to execute some tracing statements. 


**Trace Actions or Actions**

Trace statements that are executed whenever a probe "fires".


**Action Methods**

BTrace trace statements that are executed when a probe fires are defined inside a static method a class. Such methods are called "action" methods. 


##  BTrace Program Structure 

A BTrace program is a plain Java class that has one or more 

```
public static void
```
methods that are annotated with [BTrace annotations](#btrace_anno). The annotations are used to specify traced program "locations" (also known as "probe points"). The tracing actions are specified inside the static method bodies. These static methods are referred as "action" methods. 

##  BTrace Restrictions 

To guarantee that the tracing actions are "read-only" (i.e., the trace actions don't change the state of the program traced) and bounded (i.e., trace actions terminate in bounded time), a BTrace program is allowed to do only a restricted set of actions. In particular, a BTrace class

*   can **not** create new objects.

*   can **not** create new arrays.

*   can **not** throw exceptions.

*   can **not** catch exceptions.

*   can **not** make arbitrary instance or static method calls - only the public static methods of com.sun.btrace.BTraceUtils class  or methods declared in the same program may be called from a BTrace program.

*   **(pre 1.2)** can **not** have instance fields and methods. Only static public void returning methods are allowed for a BTrace class. And all fields have to be static.

*   can **not** assign to static or instance fields of target program's classes and objects. But, BTrace class can assign to it's own static fields ("trace state" can be mutated).

*   can **not** have outer, inner, nested or local classes.

*   can **not** have synchronized blocks or synchronized methods.

*   can **not** have loops (for, while, do..while)

*   can **not** extend arbitrary class (super class has to be java.lang.Object)

*   can **not** implement interfaces.

*   can **not** contains assert statements.

*   can **not** use class literals. 

These restrictions could be circumvented by running BTrace in *unsafe* mode. Both the tracing script and the engine must be set up to require *unsafe* mode. The script must be annotated by *@BTrace(unsafe = true)* annotation and the engine must be started in *unsafe* mode. 

##  A simple BTrace program (1.2) 

```
// import all BTrace annotations
import com.sun.btrace.annotations.*;
// import statics from BTraceUtils class
import static com.sun.btrace.BTraceUtils.*;

// @BTrace annotation tells that this is a BTrace program
@BTrace
class HelloWorld {

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

```

##  A simple BTrace program ( Steps to run BTrace 

1.  Find the process id of the target Java process that you want to trace. You can use jps tool to find the pid.

2.  Write a BTrace program - you may want to start modifying one of the samples.

3.  Run btrace tool by the following command line: 
```
    btrace <pid> <btrace-script>
    ```

##  BTrace Command Line 

BTrace is run using the command line tool btrace as shown below:

```
btrace [-I <include-path>] [-p <port>] [-cp <classpath>] <pid> <btrace-script> [<args>]
```

where

*   _include-path_ is a set of include directories that are searched for header files. BTrace includes a simple preprocess with support for #define, #include and conditional compilation. It is **not** like a complete C/C++ preprocessor - but a useful subset. See the sample "ThreadBean.java". If -I is not specified, BTrace skips the preprocessor invocation step.

*   _port_ is the port in which BTrace agent listens. This is optional argument.

*   _classpath_ is set of directories, jar files where BTrace searches for classes during compilation. Default is ".".

*   _pid_ is the process id of the traced Java program

*   _btrace-script_ is the trace program. If it is a ".java", then it is compiled before submission. Or else, it is assumed to be pre-compiled [i.e., it has to be a .class] and submitted. 

###  optional 

*   _port_ is the server socket port at which BTrace agent listens for clients. Default is 2020.

*   _path_ is the classpath used for compiling BTrace program. Default is ".".

*   _args_ is command line arguments passed to BTrace program. BTrace program can access these using the built-in functions "$" and "$length". 

##  Pre-compiling BTrace scripts 

It is possible to precompile BTrace program using btracec script. btracec is a javac-like program that takes a BTrace program and produces a .class file.

```
btracec [-I <include-path>] [-cp <classpath>] [-d <directory>] <one-or-more-BTrace-.java-files> 

```

where

*   _include-path_ is a set of include directories that are searched for header files. BTrace includes a simple preprocess with support for #define, #include and conditional compilation. It is **not** like a complete C/C++ preprocessor - but a useful subset. See the sample "ThreadBean.java". If -I is not specified, BTrace skips the preprocessor invocation step.

*   _classpath_ is the classpath used for compiling BTrace program(s). Default is "."

*   _directory_ is the output directory where compiled .class files are stored. Default is ".". 

This script uses BTrace compiler class - rather than regular javac and therefore will validate your BTrace program at compile time [so that you can avoid BTrace verify error at runtime]. 

##  Starting an application with BTrace agent 

So far, we saw how to trace a running Java program. It is also possible to start an application with BTrace agent in it. If you want to start tracing the application from the very "beginning", you may want to start the app with BTrace agent and specify trace scripts along with it [i.e., BTrace agent is attach-on-demand loadable as well as pre-loadable agent] You can use the following command to start an app and specify BTrace script files. But, you need to [precompile your BTrace scripts](#precompile) for this kind of usage.

```
java -javaagent:btrace-agent.jar=script=<pre-compiled-btrace-script1>[,<pre-compiled-btrace-script1>]* <MainClass> <AppArguments> 

```

When starting the application this way, the trace output goes to a file named .btrace in the current directory. Also, you can avoid starting server for other remote BTrace clients by specifying noServer=true as an argument to the BTrace agent.
There is a convenient script called **btracer** to do the above:

```
btracer <pre-compiled-btrace.class> <application-main-class> <application-args> 

```

###  Supported Arguments 

*   **bootClassPath** - boot classpath to be used

*   **systemClassPath** - system classpath to be used

*   **debug** - turns on verbose debug messages (true/false)

*   **unsafe** - do not check for btrace restrictions violations (true/false)

*   **dumpClasses** - dump the transformed bytecode to files (true/false)

*   **dumpDir** - specifies the folder where the transformed classes will be dumped to

*   **stdout** - redirect the btrace output to stdout instead of writing it to an arbitrary file (true/false)

*   **probeDescPath** - the path to search for probe descriptor XMLs

*   **script** - the path to a script to be run at the agent startup

*   **scriptdir** - the path to a directory containing scripts to be run at the agent startup

*   **scriptOutputFile** - the path to a file the btrace agent will store its output

###  Important system properties 

*   **btrace.agentname** - use to distinguish the outputs of various btrace agents running on the same machine

##  BTrace Annotations 

###  Method Annotations 

*   [@com.sun.btrace.annotations.OnMethod](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnMethod.html) annotation can be used to specify target class(es), target method(s) and "location(s)" within the method(s). An action method annotated by this annotation is called when the matching method(s) reaches specified the location. In OnMethod annotation, traced class name is specified by "clazz" property and traced method is specified by "method" property. "clazz" may be a fully qualified class name (like **java.awt.Component** or a regular expression specified within two forward slashes. Refer to the samples [NewComponent.java](https://kenai.com/projects/btrace/sources/hg/content/samples/NewComponent.java) and [Classload.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Classload.java?rev=317). The regular expression can match zero or more classes in which case all matching classes are instrumented. For example **/java\\.awt\\..+/** matches all classes in **java.awt package**. Also, method name can be a regular expression as well - matching zero or more methods. Refer to the sample [MultiClass.java](https://kenai.com/projects/btrace/sources/hg/content/samples/MultiClass.java). There is another way to abstractly specify traced class(es) and method(s). Traced classes and methods may be specified by annotation. For example, if the "clazz" attribute is specified as **@javax.jws.WebService** BTrace will instrument all classes that are annotated by the WebService annotation. Similarly, method level annotations may be used to specify methods abstractly. Refer to the sample [WebServiceTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/WebServiceTracker.java). It is also possible to combine regular expressions with annotations - like **@/com\\.acme\\..+/** matches any class that is annotated by any annotation that matches the given regular expression. It is possible to match multiple classes by specifying super type. i.e., match all classes that are subtypes of a given super type. **+java.lang.Runnable** matches all classes implementing **java.lang.Runnable** interface. Refer to the sample [SubtypeTracer.java](https://kenai.com/projects/btrace/sources/hg/content/samples/SubtypeTracer.java).

*   [@com.sun.btrace.annotations.OnTimer](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnTimer.html) annotation can be used to specify tracing actions that have to run periodically once every N milliseconds. Time period is specified as long "value" property of this annotation. Refer to the sample [Histogram.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Histogram.java)
*   [@com.sun.btrace.annotations.OnError](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnError.html) annotation can be used to specify actions that are run whenever any exception is thrown by tracing actions of some other probe. BTrace method annotated by this annotation is called when any exception is thrown by any of the other BTrace action methods in the same BTrace class.

*   [@com.sun.btrace.annotations.OnExit](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnExit.html) annotation can be used to specify actions that are run when BTrace code calls "exit(int)" built-in function to finish the tracing "session". Refer to the sample [ProbeExit.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ProbeExit.java).

*   [@com.sun.btrace.annotations.OnEvent](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnEvent.html) annotation is used to associate tracing methods with "external" events send by BTrace client. BTrace methods annotated by this annotation are called when BTrace client sends an "event". Client may send an event based on some form of user request to send (like pressing Ctrl-C or a GUI menu). String value may used as the name of the event. This way certain tracing actions can be executed whenever an external event "fires". As of now, the command line BTrace client sends "events" whenever use presses Ctrl-C (SIGINT). On SIGINT, a console menu is shown to send an event or exit the client [which is the default for SIGINT]. Refer to the sample [HistoOnEvent.java](https://kenai.com/projects/btrace/sources/hg/content/samples/HistoOnEvent.java)
*   [@com.sun.btrace.annotations.OnLowMemory](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnLowMemory.html) annotation can be used to trace memory threshold exceed event. See sample [MemAlerter.java](https://kenai.com/projects/btrace/sources/hg/content/samples/MemAlerter.java)
*   [@com.sun.btrace.annotations.OnProbe](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/OnProbe.html) annotation can be used to specify to avoid using implementation internal classes in BTrace scripts. @OnProbe probe specifications are mapped to one or more @OnMethod specifications by the BTrace VM agent. Currently, this mapping is done using a XML probe descriptor file [accessed by the BTrace agent]. Refer to the sample [SocketTracker1.java](https://kenai.com/projects/btrace/sources/hg/content/samples/SocketTracer1.java) and associated probe descriptor file java.net.socket.xml. When running this sample, this xml file needs to be copied in the directory where the target JVM runs (or fix probeDescPath option in btracer.bat to point to whereever the .xml file is). 

###  Argument Annotations 

*   [@com.sun.btrace.annotations.Self](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/Self.html) annotation can be used to mark an argument to hold **this** (or **self**) value. Refer to the samples [AWTEventTracer.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AWTEventTracer.java) or [AllCalls1.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AllCalls1.java?rev=404)
*   [@com.sun.btrace.annotations.Return](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/Return.html) annotation can be used to mark an argument to hold the return value. Refer to the sample [Classload.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Classload.java)
*   [@com.sun.btrace.annotations.ProbeClassName (since 1.1)](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/ProbeClassName.html) annotation can be used to mark an argument to hold the probed class name value. Refer to the sample [AllMethods.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AllMethods.java?rev=404)
*   [@com.sun.btrace.annotations.ProbeMethodName (since 1.1)](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/ProbeMethodName.html) annotation can be used to mark an argument to hold the probed method name. Refer to the sample [WebServiceTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/WebServiceTracker.java)
*   since 1.2 it accepts boolean parameter **fqn** to get a fully qualified probed method name

*   [@com.sun.btrace.annotations.Duration](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/Duration.html) annotation can be used to mark an argument to hold the duration of the method call in nanoseconds. The argument must be a long.  Use with **Kind.RETURN** or **Kind.ERROR** locations.

*   [@com.sun.btrace.annotations.TargetInstance (since 1.1)](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/TargetInstance.htm) annotation can be used to mark an argument to hold the called instance value. Refer to the sample [AllCalls2.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AllCalls2.java?rev=404)
*   [@com.sun.btrace.annotations.TargetMethodOrField (since 1.1)](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/TargetMethodOrField.html) can be used to mark an argument to hold the called method name. Refer to the samples [AllCalls1.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AllCalls1.java?rev=404) or [AllCalls2.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AllCalls2.java?rev=404)
*   since 1.2 it accepts boolean parameter **fqn** to get a fully qualified target method name

####  Unannotated arguments 

The unannotated BTrace probe method arguments are used for the signature matching and therefore they must appear in the order they are defined in the traced method. However, they can be interleaved with any number of annotated arguments. If an argument of type *AnyType[]* is used it will "eat" all the rest of the arguments in they order.
The exact meaning of the unannotated arguments depends on the **Location** used:

*   **Kind.ENTRY, Kind.RETURN**- the probed method arguments

*   **Kind.THROW** - the thrown exception

*   **Kind.ARRAY_SET, Kind.ARRAY_GET** - the array index

*   **Kind.CATCH** - the caught exception

*   **Kind.FIELD_SET** - the field value

*   **Kind.LINE** - the line number

*   **Kind.NEW** - the class name

*   **Kind.ERROR** - the thrown exception 

###  Field Annotations 

*   [@com.sun.btrace.annotations.Export](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/Export.htm) annotation can be used with BTrace fields (static fields) to specify that the field has to be mapped to a jvmstat counter. Using this, a BTrace program can expose tracing counters to external jvmstat clients (such as jstat). Refer to the sample [ThreadCounter.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadCounter.java)
*   [@com.sun.btrace.annotations.Property](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/Property.html) annotation can be used to flag a specific (static) field as a MBean attribute. If a BTrace class has atleast one static field with @Property attribute, then a MBean is created and registered with platform MBean server. JMX clients such as VisualVM, jconsole can be used to view such BTrace MBeans. After attaching BTrace to the target program, you can attach VisualVM or jconsole to the same program and view the newly created BTrace MBean. With VisualVM and jconsole, you can use MBeans tab to view the BTrace domain and check out it's attributes. Refer to the samples [ThreadCounterBean.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadCountBean.java) and [HistogramBean.java](https://kenai.com/projects/btrace/sources/hg/content/samples/HistogramBean.java).

*   [@com.sun.btrace.annotations.TLS](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/TLS.html) annotation can be used with BTrace fields (static fields) to specify that the field is a thread local field. Please, be aware that you can not access such marked fields in other than **@OnMethod** handlers. Each Java thread gets a separate "copy" of the field. In order for this to work correctly the field type must be either  **immutable** (eg. primitives) or **cloneable** (implements  Cloneable interface and overrides clone() method).These thread local fields may be used by BTrace programs to identify whether we reached multiple probe actions from the same thread or not. Refer to the samples [OnThrow.java](https://kenai.com/projects/btrace/sources/hg/content/samples/OnThrow.java) and [WebServiceTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/WebServiceTracker.java)

###  Class Annotations 

*   [@com.sun.btrace.annotations.DTrace](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/DTrace.html) annotation can be used to associate a simple one-liner D-script (inlined in BTrace Java class) with the BTrace program. Refer to the sample [DTraceInline.java](https://kenai.com/projects/btrace/sources/hg/content/samples/DTraceInline.java?rev=317).

*   [@com.sun.btrace.annotations.DTraceRef](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/DTraceRef.html) annotation can be used to associate a D-script (stored in a separate file) with the BTrace program. Refer to the sample [DTraceRefDemo.java](https://kenai.com/projects/btrace/sources/hg/content/samples/DTraceRefDemo.java?rev=317).

*   [@com.sun.btrace.annotations.BTrace](http://btrace.kenai.com/javadoc/1.2/com/sun/btrace/annotations/BTrace.html) annotation must be used to designate a given Java class as a BTrace program. This annotation is enforced by the BTrace compiler as well as by the BTrace agent. 

##  DTrace Integration 

Solaris DTrace is a dynamic, safe tracing system for Solaris programs - both kernel and user land programs. Because of the obvious parallels b/w DTrace and BTrace, it is natural to expect integration b/w BTrace and DTrace. There are two ways in which BTrace is integrated with DTrace.

*   BTrace program can raise a DTrace probe - by calling dtraceProbe -- see BTraceUtils javadoc referred above. For this feature to work, you need to be running on **Solaris 10 or beyond**. For other platforms (Solaris 9 or below or any other OS), dtraceProbe() will be a no-op.

*   BTrace program can associate a D-script with it-- by @DTrace annotation (if the D-script is a simple one liner) or by @DTraceRef if the D-script is longer and hence stored outside of the BTrace program. See DTrace integration samples in the BTrace samples section below. This feature works using the . For this DTrace feature to work (o.e., being able to run associated D-script), you need to be running on **Solaris 11 build 35 or beyond**. You may want to check whether you have **/usr/share/lib/java/dtrace.jar** on your machine or not. @DTrace and @DTraceRef annotations are ignored on other platforms (Solaris 10 or below or any other OS). 

##  BTrace Samples 

[BTrace samples](https://kenai.com/projects/btrace/sources/hg/content/samples)

#####  One liners about samples: 

*   [AWTEventTracer.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AWTEventTracer.java?rev=267) - demonstrates tracing of AWT events by instrumenting EventQueue.dispatchEvent() method. Can filter events by instanceof check. This sample filters and prints only focus events.

*   [AllLines.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AllLines.java?rev=404) - demonstrates line number based BTrace probes. It is possible to probe into any class (or classes) and line number(s). When the specified line number(s) of specified class(es) is reached, the BTrace probe fires and the corresponding action is executed.

*   [AllSync.java](https://kenai.com/projects/btrace/sources/hg/content/samples/AllSync.java?rev=404) - demonstrates tracing of a synchronized block entry/exit.

*   [ArgArray.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ArgArray.java?rev=267) - prints all input arguments in every readXXX method of every class in java.io package. Demonstrates argument array access and multiple class/method matching in probe specifications.

*   [Classload.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Classload.java?rev=267) - prints stack trace on every successful classload (defineClass returns) by any userdefined class loader.

*   [CommandArg.java](https://kenai.com/projects/btrace/sources/hg/content/samples/CommandArg.java?rev=404) - demonstrates BTrace command line argument access.

*   [Deadlock.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Deadlock.java?rev=317) - demonstrates @OnTimer probe and deadlock() built-in function.

*   [DTraceInline.java](https://kenai.com/projects/btrace/sources/hg/content/samples/DTraceInline.java?rev=267) - demonstrates @DTrace annotation to associate a simple one-line D-script with the BTrace program.

*   [DTraceDemoRef.java](https://kenai.com/projects/btrace/sources/hg/content/samples/DTraceDemoRef.java?rev=267) - demonstrates @DTraceRef annotation to associate a D-script file with the BTrace program. This sample associates classload.dwith itself.

*   [FileTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/FileTracker.java?rev=404) - prints file open for read/write by probing into File{Input/Output}Stream constructor entry points.

*   [FinalizeTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/FinalizeTracker.java?rev=404) - demonstrates that we can print all fields of an object and access (private) fields (read-only) as well. This is useful in debugging / troubleshooting scenarios. This sample prints info on close() /finalize() methods of java.io.FileInputStream class.

*   [Histogram.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Histogram.java?rev=404) - demonstrates usage of BTrace maps for collecting histogram (of javax.swing.JComponent objects created by an app - histogram by subclass name and count).

*   [HistogramBean.java](https://kenai.com/projects/btrace/sources/hg/content/samples/HistogramBean.java?rev=404) - demonstrates JMX integration. This sample exposes a Map as MBean attribute using @Property annotation.

*   [HistoOnEvent.java](https://kenai.com/projects/btrace/sources/hg/content/samples/HistoOnEvent.java?rev=404) - demonstrates getting trace output based on client side event. After starting the script by BTrace client, press Ctrl-C to get menu to send event or exit. On sending event, histogram is printed. This way client can "pull" trace output whenever needed rather than BTrace agent pushing the trace output always or based on timer.

*   [JdbcQueries.java](https://kenai.com/projects/btrace/sources/hg/content/samples/JdbcQueries.java?rev=408) - demonstrates BTrace aggregation facility. This facility is similar to DTrace aggregation facility.

*   [JInfo.java](https://kenai.com/projects/btrace/sources/hg/content/samples/JInfo.java?rev=404) - demonstrates printVmArguments(), printProperties() and printEnv() built-in functions.

*   [JMap.java](https://kenai.com/projects/btrace/sources/hg/content/samples/JMap.java?rev=404) - demonstrates dumpHeap() built-in function to dump (hprof binary format) heap dump of the target application.

*   [JStack.java](https://kenai.com/projects/btrace/sources/hg/content/samples/JStack.java?rev=317) - demonstrates jstackAll() built-in function to print stack traces of all the threads.

*   [LogTracer.java](https://kenai.com/projects/btrace/sources/hg/content/samples/LogTracer.java?rev=317) - demonstrates trapping into an instance method (Logger.log) and printing private field value (of LogRecord object) by field() and objectValue() built-in functions.

*   [MemAlerter.java](https://kenai.com/projects/btrace/sources/hg/content/samples/MemAlerter.java?rev=267) - demonstrates tracing low memory event by @OnLowMememory annotation.

*   [Memory.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Memory.java?rev=404) - prints memory stat once every 4 seconds by a timer probe. Demonstrates memory stat built-in functions.

*   [MultiClass.java](https://kenai.com/projects/btrace/sources/hg/content/samples/MultiClass.java?rev=317) - demonstrates inserting trace code into multiple methods of multiple classes using regular expressions for clazz and method fields of @OnMethod annotation.

*   [NewComponent.java](https://kenai.com/projects/btrace/sources/hg/content/samples/NewComponent.java?rev=317) - tracks every java.awt.Component creation and increments a counter and prints the counter based on a timer.

*   [OnThrow.java](https://kenai.com/projects/btrace/sources/hg/content/samples/OnThrow.java?rev=317) - prints exception stack trace every time any exception instance is created. In most scenarios, exceptions are thrown immediately after creation. So, it we get stack trace of throw points.

*   [ProbeExit.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ProbeExit.java?rev=317) - demonstrates @OnExit probe and exit(int) built-in function.

*   [Profiling.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Profiling.java?rev=404) - demonstrates the usage of the profiling support

*   [Sizeof.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Sizeof.java?rev=404) - demonstrates "sizeof" built-in function that can be used to get (approx.) size of a given Java object. It is possible to get size-wise histogram etc. using this built-in.

*   [SocketTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/SocketTracker.java?rev=404) - prints every server socker creation/bind and client socket accepts.

*   [SocketTracker1.java](https://kenai.com/projects/btrace/sources/hg/content/samples/SocketTracker1.java?rev=404) - similar to SocketTracker.java sample, except that this sample uses @OnProbe probes to avoid using Sun specific socket channel implementation classes in BTrace program. Instead @OnProbe probes are mapped to @OnMethod probes by BTrace agent.

*   [SysProp.java](https://kenai.com/projects/btrace/sources/hg/content/samples/SysProp.java?rev=317) - demonstrates that it is okay to probe into System classes (like java.lang.System) and call BTrace built-in functions in the action method.

*   [SubtypeTracer.java](https://kenai.com/projects/btrace/sources/hg/content/samples/SubtypeTracer.java?rev=267) - demonstrates that it is possible to match all subtypes of a given supertype.

*   [ThreadCounter.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadCounter.java?rev=317) - demonstrates use of jvmstat counters from BTrace programs.

*   [ThreadCounterBean.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadCounterBean.java?rev=404) - demonstrates exposing the BTrace program as a JMX bean with one attribute (using @Property annotation).

*   [ThreadBean.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadBean.java?rev=267) - demonstrates the use of preprocessor of BTrace [and JMX integratio].

*   [ThreadStart.java](https://kenai.com/projects/btrace/sources/hg/content/samples/ThreadStart.java?rev=317) - demonstrates raising DTrace probes from BTrace programs. See also jthread.d - the associated D-script. This sample raises a DTrace USDT probe whenever the traced program enters java.lang.Thread.start() method. The BTrace program passes JavaThread's name to DTrace.

*   [Timers.java](https://kenai.com/projects/btrace/sources/hg/content/samples/Timers.java?rev=404) - demonstrates multiple timer probes (@OnTimer) in a BTrace program.

*   [URLTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/URLTracker.java?rev=404) - prints URL every time URL.openConnection returns successfully. This program uses jurls.d D-script as well (to show histogram of URLs opened via DTrace).

*   [WebServiceTracker.java](https://kenai.com/projects/btrace/sources/hg/content/samples/WebServiceTracker.java?rev=404) - demonstrates tracing classes and methods by specifying class and method level annotations rather than class and method names.