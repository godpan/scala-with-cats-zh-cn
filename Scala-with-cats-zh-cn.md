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

本书包含了很多的技术信息和程序代码，我们使用很多排版约定来减少歧义并突出重要的概念。

#### 排版约定

新的术语和短语以斜体显示。 在基础的介绍之后，将使用普通的罗马字体。

程序代码，文件名以及文件内容中的术语将用等宽字体书写。 请注意，我们不会区分单数形式和复数形式。 例如，我们可能使用String或Strings来代表java.lang.String。

对外部资源的引用将以超链接的形式出现， 另外使用超链接和等宽字体组合来引用API文档，例如：[scala.Option](https://www.scala-lang.org/api/current/scala/Option.html)。

#### 源代码

源代码块将会按照下面这个例子书写，适当的高亮语法显示：

```scala
object MyApp extends App {
	println("Hello world!") // Print a fine message to the user!
}
```

大多数代码通过tut来进行编译。 tut背后使用了Scala控制台，因此我们将一些控制台样式的输出显示为注释：

```scala
 "Hello Cats!".toUpperCase
// res0: String = HELLO CATS!
```

#### 标注框

我们使用两种类型的标注框来突出特别的内容：

。。。

### 致谢

。。。

### 支持者

。。。



## <center>**Part I**</center>



## <center>**Theory**</center>



![Theory](/Users/panguansen/github/scala-with-cats-zh-cn/Theory.png)




### **Chapter 1**

## **介绍**

Cats包含各种各样的函数式编程工具，并允许开发者自己选择想要使用的内容。 这些工具多数以type class的形式提供，我们可以将其应用于现有的Scala类型。

Type class是一种编程范式源自于Haskell，它允许我们不通过传统方式的继承以及修改代码的方式便可以给原有的代码加上新功能。

在本章中我们将会刷新你之前在[Essen􏰂al Scala]()这本书中理解的type  class 概念，首先我们来看一下Cats的代码库。我们将会使用两个type class的例子，Show 和 Eq，利用它们为这本书做一个铺垫。

我们将type class应用在抽象数据类型，模式匹配，value classes，和类型别名，presenti􏰂ng a structured approach to functi􏰂onal programming in Scala.

（上述中的“class”这个词并不直接等价于scala或者java中的类）



## 1.1 剖析Type class

Type class模式由三个主要模块组成，

