# Scala with cats 中文翻译



### 前言

本书的主要目的有两个：

- 介绍monads, functors和其他函数式编程模式以及它们的设计结构；
- 以上这些概念在Cats中是如何实现的；

Monads及相关概念对于函数式编程来说，就相当于面向对象编程中的设计模式，主要用于重用设计。但相比面向对象设计模式来说主要有两个不同：

- 它是有严格定义的
- 它是非常普遍，通用的

这两个点似乎非常难以理解，而且也非常抽象，然而，类似于Monads的概念却被应用在各种各样的场景中。

在这本书中我们通过多种方式来阐述这些概念，帮助你理解这些模型，它们是怎么使用的以及什么场景下是合适的。

我们使用各种案例，图片展示，许多的小例子，当然还有数学定义，希望通过它们，你能从中得到对你有价值的东西。

Ok，让我们开始吧！

### 版本

这本书是基于Scala 2.13.3和Cats 1.0.0，以下是build.sbt的一部分，包含相关的依赖和设置：

```scala
scalaVersion := "2.12.3"

libraryDependencies +=
  "org.typelevel" %% "cats-core" % "1.0.0"

scalacOptions ++= Seq(
  "-Xfatal-warnings",
  "-Ypartial-unification"
)
```

### **模版工程**

为了方便，我们创建了一个Giter8 template工程，你可以直接使用，通过以下方式clone到本地：
```shell
$ sbt new underscoreio/cats-seed.g8
```

 它会生成一个项目包含Cats的相关依赖，有关如何运行示例代码或者在Scala控制台进行交互，请参考README.md。

cats-seed是一个最简版的template，如果你更喜欢ba􏰁eries-included star􏰀ng point，可以切换使用Typelevel的sbt-catalysts template:

```shell
$ sbt new typelevel/sbt-catalysts.g8
```

 它会生成一个包含一组依赖和编译插件的工程，以及单元测试和[tut-enabled](https://github.com/tpolecat/tut) 文档（这个项目已经废弃）相关的模版，更多信息请看项目的主页[catalysts](https://github.com/typelevel/catalysts)和[sbt-catalysts](https://github.com/typelevel/sbt-catalysts)。

以上操作需使用SBT 0.13.13及以上版本。

### 本书排版

本书包含了很多的技术信息和程序代码，我们使用很多排版惯例来减少歧义并突出重要的概念。

#### 排版惯例

。。。

#### 源代码

。。。

#### 标注框

我们使用两种类型的标注框来突出特别的内容：

。。。

### 致谢

。。。

### 支持者

。。。





##                                              **Part I**

​                                                                                 **Theory**

**Chapter 1**

**介绍**





