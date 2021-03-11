# Maven 学习

* Maven 译为「专家」、「内行」，是一个 **软件项目管理** 及 **自动构建** 工具，由 Apache 软件基金会所提供。
* 基于项目对象模型（POM）概念，Maven 利用一个中央信息片断来管理一个项目的构建、报告和文档等步骤。
* Maven 也可被用于构建和管理各种项目，例如 C#，Ruby，Scala 和其他语言编写的项目。

该教程主要针对：**Maven 相关特点** 与 **运用** 的学习

目录：

- [Maven 学习](#maven-学习)
  - [项目管理](#项目管理)
    - [约定配置](#约定配置)
    - [Maven POM](#maven-pom)
  - [自动构建](#自动构建)
    - [Maven 构建生命周期](#maven-构建生命周期)
  - [Maven 仓库](#maven-仓库)
  - [Maven 插件](#maven-插件)
  - [Maven 引入外部依赖](#maven-引入外部依赖)
  - [参考链接](#参考链接)

## 项目管理

### 约定配置

Maven 提倡使用一个共同的标准目录结构，Maven 使用约定优于配置 TODO （解释约定大于配置）的原则，大家尽可能的遵守这样的目录结构。如下所示：

目录 | 目的
---- | ----
${basedir} | 存放pom.xml和所有的子目录
${basedir}/src/main/java | 项目的java源代码
${basedir}/src/main/resources | 项目的资源，比如说property文件，springmvc.xml
${basedir}/src/test/java | 项目的测试类，比如说Junit代码
${basedir}/src/test/resources | 测试用的资源
${basedir}/src/main/webapp/WEB-INF | web应用文件目录，web项目的信息，比如存放web.xml、本地图片、jsp视图页面
${basedir}/target | 打包输出目录
${basedir}/target/classes | 编译输出目录
${basedir}/target/test-classes | 测试编译输出目录
Test.java | Maven只会自动运行符合该命名规则的测试类
~/.m2/repository | Maven默认的本地仓库目录位置

### Maven POM

## 自动构建

POM( Project Object Model，项目对象模型 ) 是 Maven 工程的基本工作单元，是一个XML文件，包含了项目的基本信息，用于描述项目如何构建，声明项目依赖，等等。

执行任务或目标时，Maven 会在当前目录中查找 POM。它读取 POM，获取所需的配置信息，然后执行目标。

POM 中可以指定以下配置：

项目依赖
插件
执行目标
项目构建 profile
项目版本
项目开发者列表
相关邮件列表信息

### Maven 构建生命周期

## Maven 仓库

## Maven 插件

## Maven 引入外部依赖

## 参考链接

* [Maven 教程](https://www.runoob.com/maven/maven-tutorial.html)
