# 升级传递依赖的版本

## 直接依赖与依赖约束

一个组件可以有两种不同的依赖关系：

- 直接依赖是组件直接需要的。直接依赖也被称为一级依赖。例如，如果你的项目源代码需要Guava，那么Guava应该被声明为直接依赖。


- 传递依赖是你的组件需要的依赖，但只是因为另一个依赖需要它们。


通常依赖管理的问题是关于传递依赖的。开发者经常通过添加直接依赖关系来错误地修复传递依赖问题。为了避免这种情况，Gradle提供了依赖约束的概念。

## 增加对传递依赖的约束

依赖约束允许您在构建脚本中定义声明依赖关系和传递性依赖关系的版本或版本范围。这种约束方式呈现了一个应该被应用于所有依赖关系配置的首选方法。当Gradle试图将一个依赖关系解析为一个模块的版本时，[所有带有版本的依赖关系声明](..\02-declaring-dependency-versions\rich_versions.md)、所有传递性依赖关系和该模块的所有依赖关系约束都会被考虑在内。匹配所有条件的最高版本被选中。如果没有找到这样的版本，Gradle就会出错，显示冲突的声明。如果发生这种情况，你可以调整你的依赖关系或依赖约束声明，或者在需要的情况下对传递性依赖关系进行其他调整。与依赖性声明类似，依赖性约束声明也是[由配置来限定范围](..\01-core-dependency-management\declaring_dependencies.md)的，因此可以有选择地对构建的部分进行定义。如果依赖约束影响了解析结果，那么之后仍然可以应用任何类型的[依赖解析规则](..\03-controlling-transitive-dependencies\resolution_rules.md)。

***build.gradle***

```groovy
dependencies {
    implementation 'org.apache.httpcomponents:httpclient'
    constraints {
        implementation('org.apache.httpcomponents:httpclient:4.5.3') {
            because 'previous versions have a bug impacting this application'
        }
        implementation('commons-codec:commons-codec:1.11') {
            because 'version 1.9 pulled from httpclient has bugs affecting this application'
        }
    }
}
```

在这个例子的依赖声明中，所有的版本号都被省略了。相反，版本是在约束块中定义的。`commons-codec:1.11` 的版本定义只有在 `commons-codec` 被引入作为转义依赖关系时才会被用到，因为 `commons-codec` 在项目中没有被定义为依赖关系。这样的约束就没有效果。依赖约束也可以定义一个[丰富的版本约束](..\02-declaring-dependency-versions\rich_versions.md)，并支持严格的版本以强制使用某一个版本，即使它与传递性依赖定义的版本相矛盾（例如，如果版本需要降级）。

> 依赖约束只有在使用[Gradle模块元数据](..\06-publishing\publishing_gradle_module_metadata.md)时才会被发布。这意味着目前只有当Gradle用于发布和使用时，它们才会被完全支持（即当使用Maven或Ivy消费模块时，它们会 "丢失"）。
>

依赖约束本身也可以被传递性地添加。