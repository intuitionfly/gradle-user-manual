# 入门

每个人都有开始的地方，如果你是第一次接触Gradle，这是开始的地方。

## 开始之前

为了有效地使用Gradle，你需要知道它是什么，并了解它的一些基本概念。所以在你开始认真使用Gradle之前，我们强烈建议你阅读[什么是Gradle](what_is_gradle.md)

即使你有使用Gradle的经验，我们也建议你阅读[关于Gradle的5件事](what_is_gradle.md)，因为它清除了一些常见的误解。

## 安装

如果你想做的是运行一个现有的Gradle构建，那么你不需要安装Gradle，如果该构建有一个[Gradle Wrapper](reference/gradle_wrapper.md)，可以通过构建根目录下的gradlew和/或gradlew.bat文件识别。你只需要确保你的系统满足[Gradle的先决条件](installation.md)。

Android Studio中已经安装了Gradle，所以在这种情况下你不需要单独安装Gradle。

为了创建一个新的build或添加一个Wrapper到现有的build中，你将需要根据[这些说明](installation.md)安装Gradle。请注意，除了该页面上描述的那些方法外，可能还有其他方法来安装Gradle，因为几乎不可能跟踪所有的包管理器。

## 尝试Gradle

积极使用Gradle是学习Gradle的好方法，所以一旦你安装了Gradle，就可以尝试其中一个入门的实践教程。

[创建一个基本的Gradle构建](https://guides.gradle.org/creating-new-gradle-builds/)

[构建Android应用程序](https://guides.gradle.org/building-android-apps/)

[构建Java库](https://guides.gradle.org/building-java-libraries/)

[构建Kotlin JVM库](https://guides.gradle.org/building-kotlin-jvm-libraries/)

[构建C++库](https://guides.gradle.org/building-cpp-libraries/)

[创建构建扫描](https://guides.gradle.org/creating-build-scans/)

还有许多其他的教程和指南，你可以按类别进行筛选--例如基础知识。

## 命令行与IDE

有些人是铁杆的命令行用户，而有些人则喜欢永远不离开他们的IDE的舒适度。很多人乐此不疲地使用这两者，Gradle努力不歧视。Gradle得到了[几个主要IDE](reference/third_party_integration.md)的支持，所有可以从[命令行](reference/command_line_interface.md)完成的事情都可以通过[Tooling API](reference/third_party_integration.md)提供给IDE。

Android Studio和IntelliJ IDEA用户在编辑时，应该考虑使用[Kotlin DSL构建脚本](api/kotlin_dsl.md)，以获得卓越的IDE支持。

## 执行Gradle构建

如果你按照任何一个上面的教程，你会执行一个Gradle构建。但是，如果给你一个没有任何说明的Gradle构建，你该怎么做呢？

这里有一些有用的步骤可以遵循。

 1. 确定项目是否有Gradle包装器，如果有就用--IDE默认在包装器可用时使用它。

 2. 发现项目结构。

  用IDE导入构建，或者从命令行运行`gradle projects`。如果只列出了根项目，那就是一个单项目构建，否则就是一个[多项目构建]()

 3. 找出你可以运行的任务。

  如果你已经将构建的任务导入到IDE中，你应该可以访问一个显示所有可用任务的视图。在命令行中，运行`gradle tasks`。

 4. 通过`gradle help --task <taskname>`了解更多关于任务的信息。

  `help`任务可以显示任务的额外信息，包括哪些项目包含该任务以及该任务支持哪些选项。

 5. 运行您感兴趣的任务。
    许多基于惯例的构建都集成了Gradle的[生命周期任务]()，所以当你没有更具体的任务要做的时候，就使用这些任务。例如，大多数的构建都有 "clean"、"check"、"assemble "和 "build "任务。
    在命令行中，只要运行`gradle <taskname>`就可以执行某个任务。你可以在[对应的用户手册章节](command_line_interface)中了解更多关于命令行执行的信息。如果你使用的是IDE，请查看它的文档来了解如何运行任务。

Gradle构建通常遵循项目结构和任务的标准约定，所以如果你熟悉其他同类型的构建--如Java、Android或原生构建--那么构建的文件和目录结构应该是熟悉的，以及许多任务和项目属性。

对于更专业的构建或有重大定制的构建，你最好能获得关于如何运行构建以及可以配置哪些[构建属性](running-builds/build_environment.md)的文档。

## 编写Gradle构建

学习创建和维护Gradle构建是一个过程，而且需要一点时间。我们建议你从合适的核心插件和他们的项目约定开始，然后随着你对这个工具的了解，逐渐加入自定义。

这里有一些有用的第一步，在你掌握Gradle的旅程。

 1. 试着看一两个基本教程，看看Gradle构建的样子，特别是那些与你工作的项目类型相匹配的教程（Java、原生、Android等）。
 2. 确保你已经阅读了[关于Gradle的5件事](what_is_gradle.md)，你需要知道!
 3. 了解Gradle构建的基本元素：[项目](authoring-builds/tutorial_using_tasks.md)、[任务](authoring-builds/more_about_tasks.md)和[文件API](authoring-builds/working_with_files.md)。
 4. 如果你正在为JVM构建软件，一定要阅读[构建Java和JVM项目](jvm/building_java_projects.md)和[在Java和JVM项目中测试](jvm/java_testing.md)中关于这些类型项目的具体内容。
 5. 熟悉与Gradle打包的[核心插件](core-plugins/plugin_reference.md)，因为它们提供了很多有用的功能。
 6. 学习如何[编写可维护的构建脚本](authoring-builds/authoring_maintainable_build_scripts.md)，并[最好地组织你的Gradle项目](authoring-builds/organizing_gradle_projects.md)。

用户手册包含了很多其他有用的信息，你可以在Gradle指南中找到更多关于Gradle各种功能的教程。

## Gradle与第三方工具集成

Gradle的灵活性意味着它可以很容易地与其他工具一起工作，比如那些列在我们的[Gradle和第三方工具](reference/third_party_integration.md)页面上的工具。

有两种主要的集成模式：

 * 一个工具驱动Gradle--使用它来提取关于构建的信息并运行它--通过[Tooling API](reference/third_party_integration.md)。
 * Gradle通过第三方工具的API为工具调用或生成信息--这通常是通过插件和自定义任务类型完成的。

拥有现有的基于Java的API的工具一般都是直接集成的。你可以在Gradle的[插件门户](https://plugins.gradle.org/)上找到许多这样的集成。
