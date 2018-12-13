Gradle

## Gradle 기초

### What is Gradle?
Gradle is an open-source build automation tool focused on flexibility and performance. Gradle build scripts are written using a Groovy or Kotlin DSL.

**Highly customizable** — Gradle is modeled in a way that customizable and extensible in the most fundamental ways.

**Fast** — Gradle completes tasks quickly by reusing outputs from previous executions, processing only inputs that changed, and executing tasks in parallel.

**Powerful** — Gradle is the official build tool for Android, and comes with support for many popular languages and technologies.

+ 사실 최근에는 Gradle이 느리다는 얘기가 꽤 있어서 2번 항목에 대해서는.. 아이엠그루트..

### [Gradle Build Lifecycle](https://docs.gradle.org/current/userguide/build_lifecycle.html)
We said earlier that the core of Gradle is a language for dependency based programming. In Gradle terms this means that you can define tasks and dependencies between tasks. Gradle guarantees that these tasks are executed in the order of their dependencies, and that each task is executed only once. These tasks form a Directed Acyclic Graph. There are build tools that build up such a dependency graph as they execute their tasks. Gradle builds the complete dependency graph before any task is executed. This lies at the heart of Gradle and makes many things possible which would not be possible otherwise.

Your build scripts configure this dependency graph. Therefore they are strictly speaking build configuration scripts.


#### Build phases
A Gradle build has three distinct phases.

##### Initialization
Gradle supports single and multi-project builds. During the initialization phase, Gradle determines which projects are going to take part in the build, and creates a Project instance for each of these projects.

##### Configuration
During this phase the project objects are configured. The build scripts of all projects which are part of the build are executed.

##### Execution
Gradle determines the subset of the tasks, created and configured during the configuration phase, to be executed. The subset is determined by the task name arguments passed to the gradle command and the current directory. Gradle then executes each of the selected tasks.




### Android Build Process

***Gradle과 Android 플러그인은 Android Studio와 독립적으로 실행***


![android build process](https://developer.android.com/images/tools/studio/build-process_2x.png?hl=ko)

일반적인 Android 앱 모듈의 빌드 프로세스를 나타내며 다음과 같은 일반적인 단계를 따릅니다.

컴파일러는 소스 코드를 DEX(Dalvik Executable) 파일로 변환하고 그 외 모든 것을 컴파일된 리소스로 변환합니다. 이 DEX 파일에는 Android 기기에서 실행되는 바이트코드가 포함됩니다.
APK Packager는 DEX 파일과 컴파일된 리소스를 단일 APK에 결합합니다. 그러나 앱을 Android 기기에 설치하고 배포할 수 있으려면 APK를 서명해야 합니다.

APK Packager는 디버그 또는 릴리스 키스토어를 사용하여 APK를 서명합니다.
디버그 버전의 앱(즉, 테스트 및 프로파일링 전용의 앱)을 빌드 중인 경우에는, 패키저가 디버그 키스토어로 앱에 서명합니다. Android Studio는 디버그 키스토어로 새 프로젝트를 자동으로 구성합니다.

릴리스 버전의 앱(즉, 외부에 릴리스할 앱)을 빌드 중인 경우에는, 패키저가 릴리스 키스토어로 앱에 서명합니다. 릴리스 키스토어를 생성하려면, Android Studio에서 앱 서명을 참조하세요.
최종 APK를 생성하기 전에, 패키저는 기기에서 실행될 때 더 적은 메모리를 사용하도록 앱을 최적화하기 위해 zipalign 도구를 사용합니다.
빌드 프로세스가 끝나면 배포, 테스트 또는 외부 사용자에게 릴리스할 수 있는 디버그 APK 또는 릴리스 APK가 생성됩니다.

[**Differences between .class and .dex files in Java & Android**](https://xsolve.software/blog/differences-between-class-and-dex-files-in-java-android/)


### Groovy syntax


#### task

#### closure


#### [Closure](https://trickyandroid.com/gradle-tip-2-understanding-syntax/)

Closure is a key concept which we need to grasp to better understand Gradle. Closure is a standalone block of code which can take argument, return values and be assigned to a variable.

It is some sort of a mix between Callable interface, Future, function pointer, you name it...

Essentially this is a block of code which is executed when you call it, not when you create it.

```Groovy
def myClosure = { println = 'Hello world!' }

//execute closure
myClosure()

//output
Hello world!
```

It can accepts a parameter

```
def myClosure = { String str -> println str }

//execute closure
myClosure('Hello')

//output
Hello
```

Or if closure accepts only 1 parameter, it can be ref as `it`
```
def myClosure = { println it }

//execute closure
myClosure('Hello')

//output
Hello
```

Argument types are optional, so example above can be simplified to
```
def myClosure = {str, num -> println "$str : $num" }

//execute our closure
myClosure('my string', 21)

//output
my string : 21
```

Closure can reference variables from the current context (read class). By default, current context - is the class within this closure was created

```
def myVar = 'Hello World!'
def myClosure = {println myVar}

//execute closure
myClosure()

//output
Hello world!
```

Another cool feature is that current context for the closure can be changed by calling Closure#setDelegate(). This feature will become very important later:

```
def MyClosure = { println myVar }
MyClass m = new MyClass()
myClosure.setDelegate(m)
myClosure()


class myClass {
	def myVar = 'Hello world from class'
}

//output
Hello world from class
```


**The real benefit of having closures - is an ability to pass closure to different methods which helps us to decouple execution logic.**


```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}
```

### Project
>All top level statements within build script are delegated to `Project` instance

This meaens that Project is the starting point for all my searches.

This being - let's try to find buildscript {} script block.

>Script block is a method call which takes a closure as a parameter


when we call `buildscript { ... }` - we execute method `buildscript` which accepts Closure.

>Delegates to ScriptHandler from buildscript. It means that execution scope for the closure we pass as an input parameter will be changed to ScriptHandler. In our case we passed closure which executes repositories(closure) and dependencies(closure) methods. Since closure is delegated to ScriptHandler, let's try to search for dependencies method within ScriptHandler class.

`void dependencies(Closure configurationClosure)`
which according to documentation, configures dependencies for the script. Here we are seeing another terminology: Executes the given closure against the DependencyHandler. Which means exactly the same as "delegates to \[somethig\]" - this closure will be executed in scope of another class (in our case - DependencyHandler)

delegates to \[something\] and configures \[something\]
statements which mean exactly the same - closure will be execute against specified class.

Gradle extensively uses this delegation strategy, so it is really important to understand terminology here



Script blocks

By default, there is a set of pre-defined script blocks within `Project`, but Gradle plugins are allowed to add new script blocks.

It means that if you are seeing something like `something { ... }` at the top level of your build script and you couldn't find neither script block or method which accepts closure in the documentation(most likely some plugin which you applied added this script block)



### build.gradle analysis
```
apply plugin: 'com.android.application'
...

android {
    compileSdkVersion rootProject.ext.compileSdkVersion

    defaultConfig {
        applicationId "com.navercorp.apollo.mercury"
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        multiDexEnabled true
        ...
    }

    flavorDimensions "default"

    productFlavors {
        ...
    }
    
    ...
}

dependencies {
   ...
}

configurations.all {
    resolutionStrategy {
		...
    }
    ...
}

```

실제로 여기서 사용되는 언어들은 JSON 같이 생겼지만
JSON / XML도 실제 java code도 아니다.

But it's not a language that's developed by scratch.
It sits on top of a generic scripting language called Groovy.

Groovy has a lot of syntactic sugar in other features to allow us to write build scripts that look a lot more like natural language when compared to using something like java.

Groovy intergrates perfectly with java, which is the language that Gradle platform is wirtten in.

The main thing you need to learn when using Gradle is the Gradle build language which is where the keywords like Android and task are coming from.

But it also helps a lot to understand how this build language sits on top of Groovy and Java.

The entire build script has what we call a delegate object, which exposes the Gradle build language to Groovy scripting language within the build script.

If you wirte a Gradle plugin, you could write it in any language and use the same delegate object.

The Gradle build language is also called the Gradle DSL, or Domain Specific Language.

a domain specific langauge is a language that is finely tailored for a specific task.

In this case the domain we're talking about is Android build.

Note. there's a big difference betweene describing your build and providng explict instructions on how to actually make the build happen.

The Gradle DSL declared in so you're only responsible for describing your build and Gradle itself knows how to make it happen.

That means your buld scripts can be much shorter and much easier to understand.

However, within the build script you have a full blown powerful programming language at your disposal.

We do recommend that you keep build scripts declarative, and try not to pollute it with low-level logic.

That's what Gradle plugins are for.

You can write them in any JVM lagnage like Java, Groovy or Scala.


Groovy exists to fill a hole for Java developers who need a scripting language.

It's terse, expressive and it operates extremely well with java and has some special features that make it ideally suited to creating domain specific languages.


### Gradle DSL Reference
As with any other software API or Framework, we will want to become very comfortable with finding your way around the Gradle documentation. Gradle provides a few differenct forms of documentation, but we'll focus specifically on the DSL reference, which is a good first place to look if you want to know what all configuration options are available, in any given part of your build scrpt. The Gradle DSL reference covers all the different build script components. Many of which we'll talk about later in this course. Since we're currently concerned with tasks, let's take a look at the task types that are available as part of the Gradle distribution. As you can see there are a number of built in taks types, varying from followup operations to compilation, to source code analysis. Since on of the most common build actions involves copying files. Let's take a look at the copy task API. For most built in task types examples are provided for common configuration use cases. If we look further, we can see the various task configuration properties as well as methods availavle. Many of the method description reference other greater API types, Since many API methods atake a groovy closure as an argument. it's helpful to know what arguments will be passed into the closure. the eachFile method, for example, takes a closure as an argument. The closure that we pass in as an argument, is then given an instance of FileCopyDetails as its argument. We can click this link, and then get additional detials about the options that are available on this class. The Gradle DSL reference is your best friend.

## Android Gradle

Android Plugin DSL Reference
http://google.github.io/android-gradle-dsl/current/index.html

## Improve build speed

### Use Latest Gradle & JVM version
내용 없음

### Parallel execution
Most builds consist of more than one project and some of those projects are usually independent of one another. Yet Gradle will only run one task at a time by default, regardless of the project structure (this will be improved soon).

The extent of those improvements depends on your project structure and how many dependencies you have between them. A build whose execution time is dominated by a single project won’t benefit much at all, for example. Or one that has lots of inter-project dependencies resulting in few tasks that can be executed in parallel. But most multi-project builds should see a worthwhile boost to build times.

gradle.properties
```
org.gradle.parallel=true
```

### Optimize Configuration
Gradle build goes through 3 phases: initialization, configuration, and execution. The important thing to understand here is that configuration code always executes regardless of which tasks will run. That means any expensive work performed during configuration will slow every invocation down, even simple ones like gradle help and gradle tasks.ㄹ



### Instant Run
- Hot / warm / cold swap

### Incremental build
![](/⁨Macintosh HD⁩ ▸ ⁨사용자⁩ ▸ ⁨inkyu.park⁩ ▸ ⁨데스크탑⁩/스크린샷 2018-12-13 오후 4.16.18)

As our builds become increasingly more complex, we want to ensure that we don't redo any work that has already been done the last time we executed our build. This is especially important during development, when we run our builds often many times a day with just some minor changes. It would be a huge impediment to the development process if our build had to start from scratch every time. We call the idea of only doing the minimum amount of work necessary incremental builds. For example, let's consider an Android application. Building our app requires compiling our code, generating source files, and packaging static resources into the final APK. If we were to say, change one of our layout files, we don't want to have to compile our code again. That would be unnecessary. Gradle accomplishes this by tracking each task's inputs and outputs. Before each task is run, Gradle saves a snapshot of the inputs used by the task. If that particular task doesn't have any snap shots of its input yet, or if the inputs have changed, then Gradle will run the task again. Gradle additionally saves a snapshot of the outputs created by this task. The next time Gradle goes to run the same task, it compares the inputs to the snap shot it saved earlier. If the inputs match, Gradle then also checks the outputs. If the outputs haven't been messed with since the last time the task ran, then the task can be skipped. If the outputs have changed or are missing, then the task must run again. When Gradle determines that no work needs to be done and the task can be skipped, the task is said to be up-to-date.




### [Deamon](https://docs.gradle.org/current/userguide/gradle_daemon.html)
>A daemon is a computer program that runs as a background process, rather than being under the direct control of an interactive user.

Gradle runs on the Java Virtual Machine (JVM) and uses several supporting libraries that require a non-trivial initialization time. As a result, it can sometimes seem a little slow to start.


The solution to this problem is the Gradle Daemon: a long-lived background process that executes your builds much more quickly than would otherwise be the case. We accomplish this by avoiding the expensive bootstrapping process as well as leveraging caching, by keeping data about your project in memory. Running Gradle builds with the Daemon is no different than without. Simply configure whether you want to use it or not - everything else is handled transparently by Gradle.


**Why the Gradle Daemon is important for performance**
The Daemon is a long-lived process, so not only are we able to avoid the cost of JVM startup for every build, but we are able to cache information about project structure, files, tasks, and more in memory.

The reasoning is simple: improve build speed by reusing computations from previous builds. However, the benefits are dramatic: we typically measure build times reduced by 15-75% on subsequent builds. We recommend profiling your build by using --profile to get a sense of how much impact the Gradle Daemon can have for you.


**How does the Gradle Daemon make builds faster?**
The Gradle Daemon is a long lived build process. In between builds it waits idly for the next build. This has the obvious benefit of only requiring Gradle to be loaded into memory once for multiple builds, as opposed to once for each build. This in itself is a significant performance optimization, but that’s not where it stops.

A significant part of the story for modern JVM performance is runtime code optimization. For example, HotSpot (the JVM implementation provided by Oracle and used as the basis of OpenJDK) applies optimization to code while it is running. The optimization is progressive and not instantaneous. That is, the code is progressively optimized during execution which means that subsequent builds can be faster purely due to this optimization process. Experiments with HotSpot have shown that it takes somewhere between 5 and 10 builds for optimization to stabilize. The difference in perceived build time between the first build and the 10th for a Daemon can be quite dramatic.

The Daemon also allows more effective in memory caching across builds. For example, the classes needed by the build (e.g. plugins, build scripts) can be held in memory between builds. Similarly, Gradle can maintain in-memory caches of build data such as the hashes of task inputs and outputs, used for incremental building.



```sh
$ gradle --status
```

sample output:
```
    PID VERSION                 STATUS
  28411 3.0                     IDLE
  34247 3.0                     BUSY
```

얻덯게?

```groove
gradle.properties

org.gradle.daemon=true
```


공식 레퍼런스
1. 빌드 변형(build flavor) 생성
	- 불필요한 리소스 컴파일 회피
	- 불필요한 빌드 프로세스 활성화 -> 증분 및 클린 빌드 속도 감소 (filter 사용)

2. 디버그 빌드 시 Crashlytics 비활성화

```
android {
  ...
  buildTypes {
    debug {
      ext.enableCrashlytics = false
    }
}
```

또한 디버그 빌드 시 런타임에 Crashlytics 키트를 비활성화해야 하며, 이를 위해 앱에서 Fabric 지원을 시작하는 방법을 변경해야 합니다(아래 참조).

```
// Initializes Fabric for builds that don't use the debug build type.
Crashlytics crashlyticsKit = new Crashlytics.Builder()
    .core(new CrashlyticsCore.Builder().disabled(BuildConfig.DEBUG).build())
    .build();

Fabric.with(this, crashlyticsKit);
```


4. Debug 빌드 시 정적 빌드 구성 값 사용



### lint

### proguard

### DexOptions

[Dex in process](https://medium.com/google-developers/faster-android-studio-builds-with-dex-in-process-5988ed8aa37e)
Since Android Studio 2.1, enables Dex in process to improve the time taken for full builds, as well as improving instant run performance


![dex in process](https://cdn-images-1.medium.com/max/1200/1*kjTJ5WZzupoOfvvPEzhrFw.png)

***Dex in process allows multiple dex processes to run within a single VM*** that's also shared with gradle, a featured that significantly improves build times, including initial and clean builds that means you need to increase the maximum heap size of the Gradle daemon to incorporate the memory requirements of the Dex processes.



>The default Gradle Daemon VM memory allocation is 1 gigabyte — which is insufficient to support dexInProcess, so to take advantage you’ll need to set it to at least 2 gigabytes.

```
android {
	...
    dexOptions {
        // Sets the maximum number of DEX process 
        // that can be stared concurrently
    	maxProcessCount 8
        
        // Sets the maximum memory allocation pool size
        // for the dex operation
       	javaMaxHeapSize "2g"
        
        // Enables Gradle to pre-dex library dependencies.
        // Whether to pre-dex libraries. This can improve incremental builds, but clean builds may be slower.
        preDexLibraries true
	}
}
```

더 자세한 내용은 [DexOptions DSL Document](http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.DexOptions.html#com.android.build.gradle.internal.dsl.DexOptions)을 참고하세요.



### dependencies
1. confilct
2. api / implementation

### tips
gradle build profiling
dependency check(map, tree)
confilct resolve

### references

Gradle reference
https://guides.gradle.org/performance/

Android, Optimize your build speed
https://developer.android.com/studio/build/optimize-your-build?hl=ko

TL;TR

```gradle
// Add this in your global gradle.properties file 
// at ~/.gradle/gradle.properties
// Enable Gradle Daemon
org.gradle.daemon=true
// Enable Configure on demand
org.gradle.configureondemand=true
//Enable parallel builds
org.gradle.parallel=true
// Enable Build Cache
android.enableBuildCache=true
//Enable simple gradle caching
org.gradle.caching=true
// Increase memory allotted to JVM
org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m - 
XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
```
