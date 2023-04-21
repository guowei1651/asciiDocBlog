[DevOps实践](https://www.jianshu.com/c/f8fa98feb686)系列文章，请参见连接。

## 概述
Gradle是CI过程工具，而不是系统。持续集成过程中的构建、自动化测试、打包、发布都可以使用Gradle来完成。而持续进程过程为我们降低各方面成本，提高产品信心，提高产品质量有着非常重要的作用（不要问我为啥）。而我们很多研发人员非常讨厌CI过程，这一点的问题原因是研发人员没有从CI过程中获取任何利益，而且还增加了维护成本。关于这一点等有机会的时候和大家讨论一下《怎么统一研发，质量，测试，管理之间的利益》（统一各方面的利益之后众志成城，万众一心，我们的产品会更上一层楼)。

## 使用总结：
在使用Gradle过程中发现在Gradle中的很多特性，很多原理都很特别。这里说明几个使用实践。（这里不与makefile，maven，ant进行对比）

* #### 关于Gradle的执行的过程：
很多介绍Gradle的地方，都说Gradle脚本是一种配置脚本。他们说这句话的原因是因为在Gradle中提供了很多插件，启用插件后直接调用方法闭包进行配置就可以完全完成项目需要的功能。所以，Gradle很多时候都是以配置文件的形式存在的。

但，Gradle的脚本不简单是配置文件。从Gradle执行的三个步骤可以看出：
1> Initialization：Gradle 支持单项目或者多项目构建，在该阶段，Gradle认哪些项目会参与构建，然后为每一个项目创建 Project 对象
2> Configuration：这个阶段就是配置 Initialization 阶段创建的 Project 对象，所有的配置脚本都会被执行
3> Execution：这个阶段 Gradle 会确认哪些在 Configuration 阶段创建和配置的 Task 会被执行，哪些 Task会被执行取决于gradle命令的参数以及当前的目录，确认之后便会执行

大家可以从Gradle执行过程得出，Gradle的使用过程不止配置脚本那么简单。具体可以看：
1> 在初始化阶段，Gradle没有指定默认的初始化脚本，必须使用Gradle的命令行参数进行指定。这个过程可以使用Gradle的Hook来完成相应的回调注册，以控制整个Gradle执行过程。

参见：[https://docs.gradle.org/current/userguide/init_scripts.html](https://docs.gradle.org/current/userguide/init_scripts.html)
[https://docs.gradle.org/current/dsl/org.gradle.api.invocation.Gradle.html#org.gradle.api.invocation.Gradle:addListener(java.lang.Object)](https://docs.gradle.org/current/dsl/org.gradle.api.invocation.Gradle.html#org.gradle.api.invocation.Gradle:addListener(java.lang.Object))

2> 在配置阶段，执行的是相关的配置。比如说Project中的依赖管理闭包块。属性设置等都会直接执行。但是这个阶段不会执行task内部的action。所以，这个阶段才叫做配置阶段。

3> 在执行阶段，会执行task的相关操作。在相对复杂的项目中很多时候需要在执行阶段对项目构建进行动态配置，在这个阶段可以对项目中各种属性在进行配置。但是，动态配置过程必须在使用这些配置之前完成，要不也没有任何用处。

所以，从上面可以得出，Gradle的执行过程不像我们之前接触过的CI工具那样要不是纯配置，要不是纯动态。这一点是Gradle很大的特点。这种方式为我们很好的融合了配置 与 动态构建过程，平衡了两方的优缺点。

* #### Gradle配置过程
在脚本执行的时候，Gradle会配置一些特定类型的对象，这些对象就被称为脚本的 delegate 对象，也是构建领域的领域对象，下文简称 DO。
DO 的意义就在于脚本可以使用被代理的对象的方法——这一点非常重要，是脚本中可调用方法的重要来源。
参见：[https://www.muzileecoding.com/gradlestudy/gradle-advaced-do.html](https://www.muzileecoding.com/gradlestudy/gradle-advaced-do.html)

只要是被委托对象有的方法，都是可以直接调用的。而且还有一个比较复杂的方法查找过程，一层层的查找方法。所以，在查找在某一部分可以使用的方法是一个比较麻烦的过程。而在Gradle中的文档又写的很晦涩会导致学习成本提升，效率降低等问题。不过一般情况下直接查找DSL相关描述即可。

* #### Gradle脚本模块化
模块化过程分两个大类：配置模块化 和 过程模块化。
配置模块化是将Gradle的脚本分模块的写道不同的脚本下，来提高Gradle脚本的可读性与可维护性。具体方法：

<pre>apply from: 'other.gradle'</pre>

使用apply来引入其他的脚本，但是，在真正使用过程中发现被引入的脚本的委托对象好像不像一般脚本那样。不能直接使用Project一些特定的属性或者task，只能在主脚本中通过不同的方式进行调用。

过程模块化是在被引入脚本中实现过程，方法，类等等，但是需要在主脚本里进行调用。这种方法可以使用Gradle直接调用groovy脚本中的实现来完成，但是，没有找到方法来使Gradle脚本调用groovy脚本。所以，这里的过程模块化还是使用配置模块化来进行引入。不过使用特殊的方式进行方法，类的导出。具体如下：

```
// Define methods as usual
def commonMethod1(param){ return true }
def commonMethod2(param){ return true } // Export methods by turning them into closures
ext{
commonMethod1 = this.&commonMethod1
otherNameForMethod2 = this.&commonMethod2
}
```

在被引入脚本中使用ext进行导出，在主脚本中就可以正常使用了。

参见：[http://stackoverflow.com/questions/18715137/extract-common-methods-from-gradle-build-script](http://stackoverflow.com/questions/18715137/extract-common-methods-from-gradle-build-script)
[https://docs.gradle.org/current/userguide/organizing_build_logic.html#sec:configuring_using_external_script](https://docs.gradle.org/current/userguide/organizing_build_logic.html#sec:configuring_using_external_script)

* #### 关于Groovy
在Gradle脚本中可以使用任意合法的Groovy语句。从Groovy的文档可以了解到Groovy是一种可以在运行期和编译器进行元编程的强大语言。元编程又是DSL的强大支撑。所以，Gradle才实现了这么强大的DSL，让我们更容易理解Gradle。
参见：[http://www.groovy-lang.org/metaprogramming.html](http://www.groovy-lang.org/metaprogramming.html)

## 总结：
总结了一些Gradle的实践，但是，这里最主要的核心内容还是怎么提高程序的可理解性，可维护性，可测试性的内容。提高这些更是为了降低成本。
