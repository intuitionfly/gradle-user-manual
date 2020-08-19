# 构建Java和JVM项目

Gradle使用了一种约定优先于配置的方法来构建基于JVM的项目，它借鉴了Apache Maven的一些约定。特别是，它对源文件和资源使用相同的默认目录结构，并且它与Maven兼容的资源库一起工作。

我们将在本章中详细介绍Java项目，但大部分主题也适用于其他支持的JVM语言，如Kotlin、Groovy和Scala。如果你在使用Gradle构建基于JVM的项目方面没有太多经验，可以看看Java教程，逐步了解如何构建各种类型的基本Java项目。

> 本节中的例子使用的是Java库插件。然而，所描述的功能是所有JVM插件共享的。不同插件的具体内容可以在它们的专用文档中找到。

## 介绍

最简单的Java项目构建脚本应用了Java库插件，并可选择设置项目版本和Java兼容性版本：

***build.gradle***

```
plugins {
    id 'java-library'
}

java {
    sourceCompatibility = JavaVersion.VERSION_1_8
    targetCompatibility = JavaVersion.VERSION_1_8
}

version = '1.2.1'
```

通过应用Java库插件，你可以获得一系列的功能。



By applying the Java Library Plugin, you get a whole host of features:

 * 一个编译Java任务，它可以编译`src/main/java`下的所有Java源文件
 * 为`src/test/java`下的源文件编译TestJava任务
 * 一个测试任务，运行`src/test/java`的测试
 * 一个jar任务，它将`src/main/resources`中的主要编译类和资源打包到一个名为`<project>-<version>.jar`的单一JAR中
 * 一个为主要类生成Javadoc的`javadoc`任务

这并不足以构建任何非凡的Java项目--至少，你可能会有一些文件依赖关系。但这意味着你的构建脚本只需要特定于你的项目的信息。

> 虽然示例中的属性是可选的，但我们建议你在项目中指定它们。兼容性选项可以减轻项目在不同的Java编译器版本中构建的问题，版本字符串对于跟踪项目的进展非常重要。默认情况下，项目版本也用于存档名称中。

Java库插件还将上述任务整合到标准的Base Plugin生命周期任务中：

- jar被添加到assemble
- test被添加到check

本章其余部分解释了根据您的要求定制构建的不同途径。您还将在后面看到如何调整库、应用程序、Web应用程序和企业应用程序的构建。



## 通过源文件集声明你的源文件

Gradle的Java支持是第一个引入了构建基于源码项目的新概念：源码集。其主要思想是，源文件和资源通常按类型进行逻辑分组，如应用代码、单元测试和集成测试。每个逻辑组通常都有自己的文件依赖关系集、classpaths等。值得注意的是，构成源码集的文件不必位于同一个目录下!

源码集是一个强大的概念，它将编译的几个方面联系在一起。

- 源文件和它们的位置


- 编译classpath，包括任何所需的依赖关系（通过Gradle配置）。


- 编译后的类文件放置的地方


你可以在这张图中看到这些之间的关系：

![java-sourcesets-compilation](..\img\java-sourcesets-compilation.png)



阴影框代表源集本身的属性。除此之外，Java Library Plugin会自动为你或插件定义的每一个源集创建一个编译任务--命名为compileSourceSetJava--以及若干依赖配置。

> **Main源码集**
>
> 大多数语言插件，包括Java，都会自动创建一个名为main的源集，用于项目的生产代码。这个源集很特殊，它的名字并不包含在配置和任务的名称中，这就是为什么你只有一个编译Java任务和compileOnly和实现配置，而不是分别编译MainJava、mainCompileOnly和mainImplementation。

Java项目通常包括源文件以外的资源，如属性文件，这些资源可能需要处理--例如在文件中替换标记--并在最终的JAR中打包。Java 库插件通过为每个定义的源集自动创建一个名为 processSourceSetResources 的专用任务（或主源集的 processResources）来处理这个问题。下图显示了源集如何与该任务配合。

![java-sourcesets-process-resources](..\img\java-sourcesets-process-resources.png)


与之前一样，阴影框代表源集的属性，在这种情况下，源集包括资源文件的位置以及它们被复制到哪里。

除了主源集，Java 库插件还定义了一个测试源集，它代表项目的测试。这个源集由测试任务使用，它运行测试。你可以在Java测试章节中了解更多关于这个任务和相关主题的信息。

项目通常将这个源集用于单元测试，但如果你愿意，也可以将它用于集成、验收和其他类型的测试。另一种方法是为你的其他测试类型定义一个新的源集，这通常是出于以下一个或两个原因：

 * 想保持测试之间的分离，以便于美观和管理
 * 不同的测试类型需要不同的编译或运行时classpath或其他一些不同的设置

可以在Java测试一章中看到这种方法的一个例子，它向你展示了如何在项目中设置集成测试。

你将在以下页面中了解更多关于源集和它们提供的功能：

 * 自定义文件和目录位置
 * 配置Java集成测试

## 管理依赖

绝大多数Java项目都依赖于库，因此管理项目的依赖关系是构建Java项目的重要组成部分。依赖管理是一个很大的话题，所以我们将在这里重点介绍Java项目的基础知识。如果你想深入了解细节，请查看依赖管理的介绍。

为你的Java项目指定依赖关系只需要三个信息：

 * 你需要的依赖关系，比如名称和版本
 * 需要它做什么，比如编译或运行
 * 在哪里可以找到它

前两个在dependencies {}块中指定，第三个在repositories {}块中指定。例如，要告诉Gradle，你的项目需要3.6.7版本的Hibernate Core来编译和运行你的生产代码，并且你想从Maven中央仓库下载该库，你可以使用以下片段。

***build.gradle***

```groovy
repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.hibernate:hibernate-core:3.6.7.Final'
}
```

这三个元素的Gradle术语如下。

- 库(ex: mavenCentral()) - 在那里寻找你声明为依赖的模块


- 配置（如：实现）--命名的依赖关系集合，为了特定的目标（如编译或运行模块）而组合在一起--是Maven作用域的一种更灵活的形式。


- 模块坐标（例如：org.hibernate:hibernate-core-3.6.7.Final）--依赖关系的ID，通常采用`<group>:<module>:<version>`的形式（或Maven术语中的`<groupId>:<artifactId>:<version>`）。

你可以在这里找到更全面的依赖性管理术语表。

就配置而言，主要的配置有：

- compileOnly - 用于编译生产代码所必需的依赖，但不应该是运行时classpath的一部分。


- implementation （取代compile）--用于编译和运行时。


- runtimeOnly (取代runtime) - 只在运行时使用，不用于编译。


- testCompileOnly - 与compileOnly相同，只是它是用于测试的。


- testImplementation - 相当于实施的测试。


- testRuntimeOnly - 相当于 runtimeOnly 的测试。


你可以在插件参考章节中了解更多关于这些以及它们之间的关系。

请注意，Java Library Plugin为编译该模块和任何依赖该模块的模块所需的依赖项创建了一个额外的配置--api。

> **为什么没有compile配置？**
> Java 库插件历史上曾对编译和运行项目生产代码所需的依赖项使用过compile配置。现在它已经被废弃了，使用时会发出警告，因为它没有区分影响Java库项目的公共API的依赖和不影响的依赖。你可以在《构建Java库》中了解更多关于这种区分的重要性。

我们在这里只是做了一些表面文章，所以我们建议你在对使用Gradle构建Java项目的基础知识感到满意后，再阅读专门的依赖性管理章节。一些需要进一步阅读的常见场景包括：

 * 定义一个自定义的Maven或Ivy兼容的仓库
 * 从本地文件系统目录中使用依赖关系
 * 声明不断变化（如SNAPSHOT）和动态（范围）版本的依赖性
 * 将兄弟项目声明为依赖关系
 * 控制转义依赖和它们的版本
 * 通过复合构建测试你对第三方依赖关系的修复（比从Maven Local发布和消费更好的替代方法）

你会发现Gradle有一个丰富的API来处理依赖关系--这个API需要时间来掌握，但对于常见的场景来说，它是直接使用的。

## 编译代码

如果你遵守约定，编译你的生产和测试代码是非常简单的。

 1. 把你的生产源代码放在`src/main/java`目录下
  2. 把你的测试源码放在`src/test/java`下
  3. 在`compileOnly`或`implementation`配置中声明你的生产编译依赖关系(见前一节)
  4. 在`testCompileOnly`或`testImplementation`配置中声明你的测试编译依赖性
  5. 为生产代码运行`compileJava`任务，为测试运行`compileTestJava`

其他JVM语言插件，如Groovy的插件，也遵循同样的约定模式。我们建议你尽可能地遵循这些约定，但你不必这样做。有几个自定义的选项，你将在接下来看到。

### 自定义文件和目录的位置

想象一下，你有一个遗留项目，它的生产代码和测试代码使用一个src目录，传统的目录结构无法工作，所以你需要告诉Gradle在哪里找到源文件。你可以通过源码集配置来实现。

每个源码集定义了它的源码所在的位置，以及资源和类文件的输出目录。你可以通过使用下面的语法来覆盖约定的值：

***build.gradle***

```groovy
sourceSets {
    main {
         java {
            srcDirs = ['src']
         }
    }

    test {
        java {
            srcDirs = ['test']
        }
    }
}
```

现在Gradle只会直接在src中搜索和测试各自的源代码。如果你不想凌驾于这个约定之上，而只是想增加一个额外的源码目录，也许是一个包含了一些第三方源码的目录，想把它分开呢？语法类似：

***build.gradle***

```groovy
sourceSets {
    main {
        java {
            srcDir 'thirdParty/src/main/java'
        }
    }
}
```

最关键的是，我们在这里使用srcDir()方法来追加一个目录路径，而设置srcDirs属性会替换任何现有的值。这是Gradle中常见的约定：设置一个属性替换值，而相应的方法则添加值。

你可以在 SourceSet 和 SourceDirectorySet 的 DSL 参考中看到源集上所有可用的属性和方法。请注意，srcDirs和srcDir()都在SourceDirectorySet上。

### 更改编译器选项

大部分的编译器选项都可以通过相应的任务来访问，比如`compileJava`和`compileTestJava`。这些任务的类型是JavaCompile，所以请阅读任务参考资料，以获得最新的、全面的选项列表。

例如，如果你想为编译器使用单独的JVM进程，并防止编译失败导致构建失败，你可以使用这个配置。

***build.gradle***

```groovy
compileJava {
    options.incremental = true
    options.fork = true
    options.failOnError = false
}
```

这也是你如何可以改变编译器的冗余，在字节代码中禁用调试输出，以及配置编译器在哪里可以找到注释处理器的方法。

### 针对特定的Java版本

默认情况下，Gradle将编译Java代码到运行Gradle的JVM的语言级别。

自Java 9以来，Java编译器可以被配置为产生旧的Java版本的字节码，同时确保代码不使用任何来自较新版本的API。Gradle现在直接在CompileOptions上为Java编译支持这个发布标志。这个选项优先于下面描述的属性。

***build.gradle***

```groovy
compileJava {
    options.release = 7
}
```

Java编译器的历史选项仍然可用。

​	`sourceCompatibility`
​	定义你的源文件应被视为Java的哪个语言版本。

​	`targetCompatibility`
​	定义你的代码应该在JVM上运行的最小版本，即它决定了编译器生成的字节码的版本。

这些选项可以为每个JavaCompile任务设置，也可以在java { }扩展上为所有编译任务设置，使用相同名称的属性。

然而，这些选项并不能防止在以后的Java版本中引入的API的使用。

### 编译和测试Java 6/7

Gradle只能在Java 8或更高版本上运行。但Gradle仍然支持编译，测试，生成Javadoc和执行Java 6和Java 7的应用程序。不支持Java 5。

> 如果使用Java 9+，利用发布标志可能是一个更简单的解决方案，见上文。
>

要使用Java 6或Java 7，需要配置以下任务：

- JavaCompile任务，以fork和使用正确的JAVA_HOME。


- Javadoc任务要使用正确的javadoc可执行文件。


- 测试和JavaExec任务来使用正确的java可执行文件。


下面的示例显示了需要如何调整`build.gradle`。为了能够使构建与机器无关，应该在每个开发者机器上的用户主目录下的`GRADLE_USER_HOME/gradle.properties`中配置旧的Java主目录和目标版本的位置，如示例所示：

***gradle.properties***

```properties
# in $HOME/.gradle/gradle.properties
javaHome=/Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home
targetJavaVersion=1.7
```

***build.gradle***

```groovy
assert hasProperty('javaHome'): "Set the property 'javaHome' in your your gradle.properties pointing to a Java 6 or 7 installation"
assert hasProperty('targetJavaVersion'): "Set the property 'targetJavaVersion' in your your gradle.properties to '1.6' or '1.7'"

java {
    sourceCompatibility = JavaVersion.toVersion(targetJavaVersion)
}

def javaExecutablesPath = new File(javaHome, 'bin')
def javaExecutables = [:].withDefault { execName ->
    def executable = new File(javaExecutablesPath, execName)
    assert executable.exists(): "There is no ${execName} executable in ${javaExecutablesPath}"
    executable
}
tasks.withType(AbstractCompile) {
    options.with {
        fork = true
        forkOptions.javaHome = file(javaHome)
    }
}
tasks.withType(Javadoc) {
    executable = javaExecutables.javadoc
}
tasks.withType(Test) {
    executable = javaExecutables.java
}
tasks.withType(JavaExec) {
    executable = javaExecutables.java
}
```

### 分别编译独立源码

大多数项目至少有两套独立的源代码：生产代码和测试代码。Gradle已经将这种情况作为其Java约定的一部分，但是如果你有其他的源码集呢？最常见的情况之一是当你有单独的集成测试的某种形式或其他。在这种情况下，一个自定义的源集可能正是你所需要的。

你可以在Java测试章节中看到一个设置集成测试的完整示例。你可以用同样的方式设置其他履行不同角色的源集。那么问题就变成了：什么时候应该定义一个自定义源集？

要回答这个问题，就要考虑源集：

1. 需要用唯一的classpath进行编译。
2. 生成与主类和测试类不同的处理方式的类。
3. 形成项目的一个自然组成部分

如果你对第3点和其他两个答案中的任何一个都是肯定的，那么自定义源集可能是正确的方法。例如，集成测试通常是项目的一部分，因为它们测试`main`中的代码。此外，它们通常有自己独立于测试源集的依赖关系，或者需要用自定义测试任务来运行。

其他常见的情况则不太明确，可能有更好的解决方案。比如说：

- 分开的API和实现JAR--将它们作为单独的项目可能是有意义的，特别是当你已经有一个多项目构建的时候。


- 生成的源码--如果生成的源码应该与生产代码一起编译，请将它们的路径添加到主源码集中，并确保编译Java任务依赖于生成源码的任务。


如果你不确定是否要创建一个自定义源集，那就去做吧。它应该是直接的，如果不是，那么它可能是不合适的。

## 管理资源

许多Java项目都会使用源文件以外的资源，如图像、配置文件和本地化数据。有时，这些文件只需打包不变，有时则需要将其作为模板文件或以其他方式进行处理。无论哪种方式，Java 库插件都会为每个源集添加一个特定的 Copy 任务，进行其相关资源的处理。

该任务的名称沿用了processSourceSetResources的惯例--或主源集的processResources--它将自动把`src/[sourceSet]/resources`中的任何文件复制到一个将包含在生产JAR中的目录。这个目标目录也会被包含在测试的运行时classpath中。

由于 processResources 是 Copy 任务的一个实例，你可以进行[处理文件]()章节中描述的任何处理。

### Java属性文件和可重复构建

您可以通过WriteProperties任务轻松创建Java属性文件，它修复了Properties.store()的一个众所周知的问题，这个问题会降低增量构建的可用性。

用于编写属性文件的标准Java API每次都会产生一个唯一的文件，即使使用了相同的属性和值，因为它在注释中包含了一个时间戳，也不会重复。Gradle的WriteProperties任务会生成完全相同的输出，如果没有一个属性发生变化的话。这是通过对属性文件的生成方式进行一些调整来实现的。

- 没有时间戳注释被添加到输出中。


- 分行符与系统无关，但可以显式配置（默认为'\n'）。


- 属性按字母顺序排列

有时，在不同的机器上以字节对字节的方式重新创建档案是可取的。你要确保无论何时何地，从源代码构建一个工件都会产生相同的结果，一个字节一个字节地构建，这对于像 reproducible-builds.org 这样的项目来说是必要的。

这些调整不仅带来了更好的增量构建集成，而且还有助于实现可重复构建。从本质上讲，可重现的构建保证了无论你在什么时候或在什么系统上运行，你都能看到相同的构建执行结果--包括测试结果和产生的二进制文件。

## 运行测试

除了提供自动编译`src/test/java`中的单元测试外，Java Library Plugin还支持运行使用JUnit 3、4和5的测试（Gradle 4.6中支持JUnit 5）和TestNG。你可以得到：

- 一个Test类型的自动测试任务，使用测试源集


- 一个HTML测试报告，包括所有测试任务的运行结果


- 轻松过滤要运行的测试


- 精细控制测试的运行方式


- 创建自己的测试执行和测试报告任务的机会

你不会为你声明的每一个源集都得到一个测试任务，因为不是每一个源集都代表测试！这就是为什么你通常需要为集成测试和验收测试这样的事情创建自己的测试任务，如果它们不能包含在测试源集中。

由于涉及到测试时有很多内容，这个主题有自己的一章，我们在其中可以得知：

- 测试是如何运行的
- 如何通过过滤运行测试的子集
- Gradle如何发现测试
- 如何配置测试报告和添加自己的报告任务
- 如何使用特定的JUnit和TestNG功能

您还可以在测试的DSL参考中了解更多关于配置测试的信息

