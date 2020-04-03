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

### 模版工程

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



## <center>Part I</center>



## <center>Theory</center>



![Theory](/Users/panguansen/github/scala-with-cats-zh-cn/Theory.png)




### Chapter 1

## 介绍

Cats包含各种各样的函数式编程工具，并允许开发者自己选择想要使用的内容。 这些工具多数以type class的形式提供，我们可以将其应用于现有的Scala类型。

Type class是一种编程范式源自于Haskell，它允许我们不通过传统方式的继承以及修改代码的方式便可以给原有的代码加上新功能。

在本章中我们将会刷新你之前在[Essen􏰂al Scala]()这本书中理解的type  class 概念，首先我们来看一下Cats的代码库。我们将会使用两个type class的例子，Show 和 Eq，利用它们为这本书做一个铺垫。

我们将type class应用在抽象数据类型，模式匹配，value classes，和类型别名，presenti􏰂ng a structured approach to functi􏰂onal programming in Scala.

（上述中的“class”这个词并不直接等价于scala或者java中的类）



### 1.1 剖析Type class

Type class模式主要由3个模块组成：

- Type class self
- Type class Instances
- Type class interface

#### 1.1.1 The Type Class

Type class可以看成一个接口或者API，用于定义我们想要实现功能。在Cats中，Type class相当于至少带有一个类型参数的trait。举个例子，我们可以通过以下的代码描述一个基本的功能：“序列化成JSON”。

```scala
// Define a very simple JSON AST 声明一些简单的JSON AST
sealed trait Json
final case class JsObject(get: Map[String, Json]) extends Json 
final case class JsString(get: String) extends Json
final case class JsNumber(get: Double) extends Json
case object JsNull extends Json

// The "serialize to JSON" behaviour is encoded in this trait 序列话JSON方法定义在这个Trait里
trait JsonWriter[A] {
  def write(value: A): Json
}
```

 这个例子中JsonWriter就是我们定义的一个type class，上述代码中还包含Json类型相关的代码。

#### 1.1.2 Type Class Instances

Type Class 实例就是对特定类型的Type Class的实现，包括Scala的基本类型以及我们自己定义的类型。

在Scala中，我们通过创建一个Type Class的实现来进行Type Class 实例的声明，并用**implicit**这个关键词进行标记：

```scala
final case class Person(name: String, email: String)
object JsonWriterInstances {
  implicit val stringWriter: JsonWriter[String] =
    new JsonWriter[String] {
      def write(value: String): Json =
        JsString(value)
    }
  implicit val personWriter: JsonWriter[Person] =
    new JsonWriter[Person] {
      def write(value: Person): Json =
        JsObject(Map(
          "name" -> JsString(value.name),
          "email" -> JsString(value.email)
        ))
}
// etc...
}
```

#### 1.1.3 Type Class Interfaces

Type Class  Interface包含对我们想要对外部暴露的功能。interfaces是指接受 type class instance 作为 `implicit` 参数的泛型方法。

通常有两种方式去创建interface：

- Interface Objects
- Interface Syntax

##### Interface Objects

创建interface最简单的方式就是将方法放在一个单例object中：

```scala
object Json {
  def toJson[A](value: A)(implicit w: JsonWriter[A]): Json = w.write(value)
}
```

在使用之前，我们需要导入我们所需的type class instances，然后就可以调用相关的方法：

```scala
import JsonWriterInstances._
Json.toJson(Person("Dave", "dave@example.com"))
// res4: Json = JsObject(Map(name -> JsString(Dave), email -> JsString
     (dave@example.com)))
```

这里我们并没有指定对应的implicit parameters，但是编译器会帮我们在导入的type class instances中寻找一个跟相应类型匹配的type class instance，并插入对应的位置：

```scala
Json.toJson(Person("Dave", "dave@example.com"))(personWriter)
```

##### Interface Syntax

我们也可以使用扩展方法使已存在的类型拥有interface methods，在Cats中将此称为“*syntax*”：

```scala
object JsonSyntax {
  implicit class JsonWriterOps[A](value: A) {
    def toJson(implicit w: JsonWriter[A]): Json =
      w.write(value)
  } 
}
```

使用interface syntax之前，我们除了导入它本身以外，还需导入我们所需的type class instance：

```scala
import JsonWriterInstances._
import JsonSyntax._
Person("Dave", "dave@example.com").toJson
// res6: Json = JsObject(Map(name -> JsString(Dave), email -> JsString(dave@example.com)))
```

 同样，编译器会自动帮我们寻找所需implicit parameters并插入对应的位置：

```scala
Person("Dave", "dave@example.com").toJson(personWriter)
```

##### The implicitly Method

Scala标准库提供了一个泛型的type class interface叫做implicitly，它的声明非常简单：

```scala
 def implicitly[A](implicit value: A): A = value
```

它接收一个implicit参数并返回该参数，我们可以使用implicitly调用implicit scope中的任意值，只需要指定对应的类型无需其他操作，便能得到对应的instance对象。

```scala
import JsonWriterInstances._
// import JsonWriterInstances._
implicitly[JsonWriter[String]]
// res8: JsonWriter[String] = JsonWriterInstances$$anon$1@38563298
```

在Cats中，大多数type class都提供了其他方式去调用对应的instance。但是在代码调试过程中，implicitly有着很大的用处。我们可以在代码中插入implicitly相关代码，来确保编译器能找到对应的type class instance（若无对应的type class instance则编译的时候会抱错）以及不会出现歧义性（比如implicit scope存在两个相同的type class instance）。

### 1.2 Working with Implicits

对于Scala来说，使用type class就得跟 implicit values 和implicit parameters打交道，为了更好的使用它，我们需要了解以下几个点。

#### 1.2.1 Packaging Implicits

奇怪的是,在Scala中任何标记为implicit的定义都必须放在object或trait中，而不是放在顶层。在上一小节的例子中，我们将所有的type class instances打包放在JsonWriterInstances中。同样我们也可以把它放在JsonWriter的伴生对象中，这种方式在Scala中有特殊的含义，因为这些instances会直接在*implicit scope*里面，无需单独导入。

#### 1.2.2 Implicit Scope

正如我们看到的一样，编译器会自动寻找对应类型的type class instances，举个例子，下面这个例子就会编译器就会自动寻找**JsonWriter[String]**对应的instance：

```scala
Json.toJson("A string!")
```

编译器会从以下几个*implicit scope*中寻找适合的instance：

- 自身及继承范围内的instance
- 导入范围内的instance
- 对应type class以及参数类型的伴生对象中

只有用implicit关键词标注的instance才会在*implicit scope*，而且如果编译器在引入的implicit scope中发现重复的instance声明，则会编译抱错：

```scala
implicit val writer1: JsonWriter[String] =
  JsonWriterInstances.stringWriter
implicit val writer2: JsonWriter[String] =
  JsonWriterInstances.stringWriter
Json.toJson("A string")

// <console>:23: error: ambiguous implicit values:
// both value stringWriter in object JsonWriterInstances of type => JsonWriter[String]
//  and value writer1 of type => JsonWriter[String]
// match expected type JsonWriter[String] 
// Json.toJson("A string")
//
```

但Scala中的*implicit*规则远比这复杂的多，但这些不在本书的讨论范围之内（如果你想对*implicit*有更深入的了解，可以参考这些内容：[this Stack Overflow post on implicit scope](https://stackoverflow.com/questions/5598085/where-does-scala-look-for-implicits)和[this blog post on implicit priority](http://eed3si9n.com/revisiting-implicits-without-import-tax)）。对于我们来说，通常把type class instances放在以下四个地方：

1. 一个单独的object中，比如上面提到的JsonWriterInstances；
2. 一个单独的trait中；
3. type class的伴生对象中；
4. 我们所使用类型的伴生对象中，比如JsonWriter[A]，即A的伴生对象中；

如果是第一种方式的，我们在使用之前通过import导入，第二种方式的话通过继承trait引入，另外两种方式的，无需单独导入，它们默认就在对应类型的implicit scope中。

#### 1.2.3 Recursive Implicit Resolu􏰀on

编译器除了能直接寻找对应类型type class instance，还拥有组合type class instance的能力。

之前我们都是通过 implicit val来声明type class instances ，这非常简单，实际上我们有两种方式去声明instances：

1. 通过 implicit val来声明具体类型的type class instances；
2. 利用 implicit methods通过其他类型的type class instances来生成新的instances；

我们为什么要通过其他类型的type class instances来生成新的instances呢？一个很明显的例子，我们如何让Option类型可以应用JsonWriter这个type class。对于系统中的任意类型的Option[A]，都得需要有对应的type class instance，我们可能会尝试通过声明所有instance：

```scala
implicit val optionIntWriter: JsonWriter[Option[Int]] = ???
implicit val optionPersonWriter: JsonWriter[Option[Person]] = ???
// and so on...
```

显然，这种方式是不易扩展的，对于系统中的任意类型A，我们都必须去声明两个instance，一个作用于A，一个作用于Option[A]。

幸运的是，我们可以基于A的instance来构造Option[A]的instance，而且这是一个通用逻辑：

- 假如option是Some(a: A)，则使用A的instance；
- 假如option是None，则返回JsNull；

我们通过implicit def来实现：

```scala
implicit def optionWriter[A](implicit writer: JsonWriter[A]): JsonWriter[Option[A]] =
  new JsonWriter[Option[A]] {
    def write(option: Option[A]): Json =
      option match {
        case Some(aValue) => writer.write(aValue)
        case None         => JsNull
		} 
 }
```

这个方法包含一个implicit参数writer，并通过它来构造一个Option[A]的JsonWriter instance。我们来看一个表达式：

```scala
Json.toJson(Option("A string"))
```

编译器首先会去寻找对应的type class instance，这里是optionWriter[String]，所以为表达式加上对应的implicit参数：

```scala
Json.toJson(Option("A string"))(optionWriter[String])
```

因为这里optionWriter是用implicit def声明的，而且需要一个implicit writer: JsonWriter[A]参数，所以编译器会继续寻找，这里的对应instance是stringWriter，最终完整的表达式：

```scala
Json.toJson(Option("A string"))(optionWriter(stringWriter))
```

通过这种方式，编译器会在引入的implicit scope中竟可能的寻找符合的instance，最终组合成所需要类型的type class instance。

> *Implicit Conversions*
>
> 在我们使用implicit def构建type class instance的时候，我们使用implicit参数，如果我们不使用implicit声明参数，编译器则不会自动去寻找填充参数。
>
> 使用implicit方法但是不使用implicit parameters在Scala中是另一种模式，叫做*implicit conversion*。跟之前内容中提到的Interface Syntax也是不同的，它是一个implicit class并使用扩展方法。implicit conversion是一种古老的编程模式，目前Scala已经不赞成使用了。而且当你使用该语法时，编译器会提出警告，如果你确定要使用，则需手动引入scala.language.implicitConversions：
>
> ```scala
> implicit def optionWriter[A]
> (writer: JsonWriter[A]): JsonWriter[Option[A]] =
> ???
> // <console>:18: warning: implicit conversion method optionWriter should be enabled
> // by making the implicit value scala.language.implicitConversions visible.
> // This can be achieved by adding the import clause 'import scala.language.implicitConversions'
> // or by setting the compiler option -language: implicitConversions.
> // See the Scaladoc for value scala.language.implicitConversions for a discussion
> // why the feature should be explicitly enabled.
> //
> //     implicit def optionWriter[A]
>                     ^
> // error: No warnings can be incurred under -Xfatal-warnings.
> ```
>
> 

### 1.3 Exercise: Printable Library

Scala可以通过toString方法将一个任意一个值转换成String。但是这种方式有一些缺陷：

- 它对Scala中的每个类型都进行了实现，但是使用有很大限制；
- 不能对特定类型进行特定的实现；

让我们声明一个Printable type class去解决这些问题吧：

1. 声明一个type class Printable[A]包含一个方法format，该方法接受一个类型为A的参数并返回String。
2. 创建一个名为PrintableInstances的object，包含Printable[String]和Printable[Int]的instance声明。
3. 创建一个名为Printable的object，包含两个泛型方法：
   1. format方法：接受一个类型为A的参数和相关类型的Printable，使用Printable将参数转换为String。
   2. print方法：与format方法参数一致，但返回值时Unit，它执行的操作是通过println将类型为A的参数输出到控制台。

代码见[示例]()

##### Using the Library

我们可以把Printable这个功能封装成类库，然后在使用的地方引入，我们先来定义一个case class：

```scala
final case class Cat(name: String, age: Int, color: String)
```

接下来我们实现一个Printable[Cat]类型的instance，对应format的返回结果应为：

```scala
NAME is a AGE year-old COLOR cat.
```

 最后我们对功能进行了实现（代码见[示例]()）

##### Bett􏰁er Syntax

我们将使用前面介绍的**Interface Syntax**的语法，让Printable相关的功能更容易使用：

1. 创建一个PrintableSyntax的object。
2. 在PrintableSyntax中声明一个implicit class PrintableOps[A]对A类型的值进行包装。
3. 在PrintableOps[A]声明两个方法：
   - format接受一个implicit Printable[A]的参数，返回String；
   - print接受一个implicit Printable[A]的参数，返回Unit；
4. 使用扩展方法对上一个例子进行不一样实现；

（代码见[示例]()）

### 1.4 Meet Cats

在先前的章节我们学习了如何在Scala中去实现一个type class，在本节中我们学习Cats中实现的type class。

Cats是的设计是模块化，你可以自由选择自己想要的type class、instance、interface methods。让我们来看第一个例子：[cats.Show](http://typelevel.org/cats/api/cats/Show.html)。

Show的功能跟我们在上节实现的Printable基本一致。它的主要功能就是帮助我们将数据以更友好的方式输出的控制台，而不是通过toString方法，下面是它的一个简要声明：

```scala
package cats
trait Show[A] {
  def show(value: A): String
}
```

#### Impor􏰀ng Type Classes

Show这个type class声明在[cats](http://typelevel.org/cats/api/cats/)这个包里，我们可以直接进行import：

```scala
import cats.Show
```

在Cats中，每个type class的伴生对象中都有一个apply方法，用于查找我们指定类型对应的instance：

```scala
val showInt = Show.apply[Int]
// <console>:13: error: could not find implicit value for parameter
//  instance: cats.Show[Int]
// val showInt = Show.apply[Int]
```

糟糕，竟然报错了，因为apply方法是通过implicit来查找对应的instance，所以我们需要导入相应的instance到implicit scope。

#### Impor􏰀ng Default Instances

 [cats.instances](https://typelevel.org/cats/api/cats/instances/)这个包提供了很多默认实现的instances，我们可以通过一下方式来引入它们，每种类型的包都包含了该类型对于Cats中所有type class的instance实现：

- [cats.instances.int](https://typelevel.org/cats/api/cats/instances/package$$int$)提供所有Int的instances
- [cats.instances.string](https://typelevel.org/cats/api/cats/instances/package$$string$)提供所有Stirng的instances
- [cats.instances.list](https://typelevel.org/cats/api/cats/instances/package$$list$)提供所有List的instances
- [cats.instances.option](https://typelevel.org/cats/api/cats/instances/package$$option$)提供所有Option的instances
- [cats.instances.all](https://typelevel.org/cats/api/cats/instances/package$$all$)提供Cats中的所有instances

有关可用导入的详细信息，请参见[cats.instances](https://typelevel.org/cats/api/cats/instances/)包。

让我们来引入Int和String对应Show的instances：

```scala
import cats.instances.int._    // for Show
import cats.instances.string._ // for Show

val showInt: Show[Int] = Show.apply[Int] 
val showString: Show[String] = Show.apply[String]
```

很好，我们引入了Int和String对应Show的instances，现在可以使用它们来打印Int和String的数据：

```scala
val intAsString: String = showInt.show(123)
// intAsString: String = 123

val stringAsString: String = showString.show("abc")
// stringAsString: String = abc
```

#### Impori􏰀ng Interface Syntax

我们可以使用*interface syntax*让Show变的更容易使用，首先我们需要先导入[cats.syntax.show](https://typelevel.org/cats/api/cats/syntax/package$$show$)，它会为任意类型添加一个show的扩展方法，前提是implicit scope已经有了对应类型的instance：

```scala
import cats.syntax.show._ // for show

val shownInt = 123.show
// shownInt: String = 123

val shownString = "abc".show
// shownString: String = abc
```

Cats为每一个type class都提供了syntax，我们可以按需使用，在后面的章节，我们会继续它们。

#### **Impori􏰀ng All The Things!**

在这本书中，我们对于每个示例都是按需导入，只导入需要的instance和syntax。然而，有些时候这也是相当费时的，你可以通过以下方式简化导入：

- import cats._  导入Cats中所有的type class
- import cats.instances.all._ 导入Cats中所有的instances
- import cats.syntax.all._ 导入Cats中所有的syntax
- import cats.implicits._  导入Cats中所有的instances和syntax

大多数时候我们只需要全部导入即可：

```scala
import cats._
import cats.implicits._
```

但当遇到命名冲突或者implicit冲突的时候，我们就需要更具体导入。

#### Defining Custom Instances

下面我们来自定义一个关于Show的instance：

```scala
import java.util.Date

implicit val dateShow: Show[Date] =
  new Show[Date] {
    def show(date: Date): String =
      s"${date.getTime}ms since the epoch."
}
```

但是，Cats提供了一些更简洁的方法去声明instance。对于Show来说，在其伴生对象中有两个方法帮助我们创建自定义类型的instance：

```scala
object Show {
  
  // Convert a function to a `Show` instance:
  def show[A](f: A => String): Show[A] = ???
  
  // Create a `Show` instance from a `toString` method:
  def fromToString[A]: Show[A] = ???
}
```

使用这些方法会比传统创建instance的方式更加快速：

```scala
implicit val dateShow: Show[Date] = Show.show(date => s"${date.getTime}ms since the epoch.")
```

我们可以看到，确实简洁了不少，Cats为很多type class都提供了类似的辅助方法来创建instance，可以从头直接创建instance，也可以基于其他类型的instance创建新的instance，比如：基于Int类型的instance创建Option[Int]类型的instance。

#### 1.4.6 Exercise: Cat Show

使用Show type class重写上面章节Printable的例子，代码见[示例]()

### 1.5 Example: Eq

本章节我们继续来学习一个非常实用的type class：[cats.Eq](https://typelevel.org/cats/api/cats/kernel/Eq.html)。Eq主要是为了类型安全的判等设计的，因为Scala内置的 ==操作符有时会给我们带来困扰。

大多数Scala程序员都应该写过类似下面的代码：

```scala
List(1, 2, 3).map(Option(_)).filter(item => item == 1)
// res0: List[Option[Int]] = List()
```

可能很多人都不会犯这么简单的错误，但是这是可能存在的，filter里面的判断逻辑会一直返回false，因为Int和Option[Int]是不可能相等的。

这是开发者的错，我们应该用Some(1)去比较而不是1。然而这在技术上来说并不能说它是错的，因为==可以作用于任意的两个对象，不用关心具体的类型。Eq的设计，解决了这个问题，因为它是类型安全的。

#### **1.5.1 Equality, Liberty, and Fraternity**

我们可以使用Eq对任意给定类型的对象进行类型安全的判等：

```scala
package cats

trait Eq[A] {
  def eqv(a: A, b: A): Boolean
  // other concrete methods based on eqv...
}
```

与Show类似，关于Eq的interface syntax，声明在[cats.syntax.eq](https://typelevel.org/cats/api/cats/syntax/package$$eq$)这个包中，它提供了两个执行判等的方法，你可以直接使用，当然前提是在implicit scope中有对于的instance：

- === 比较两个对象相等
- =!= 比较两个对象不相等

#### **1.5.2 Comparing Ints**

让我们来看些例子，首先我们需要先导入对应的type class:

```scala
import cats.Eq
```

接着，我们来获取一个Int的instance：

```scala
import cats.instances.int._ // for Eq

val eqInt = Eq[Int]
```

我们可以直接使用eqInt来进行判等：

```scala
eqInt.eqv(123, 123)
// res2: Boolean = true

eqInt.eqv(123, 234)
// res3: Boolean = false
```

 不同于Scala的==操作符，假如你试图用eqv去比较两个不同类型的对象时，编译将会报错：

```scala
eqInt.eqv(123, "234")
// <console>:18: error: type mismatch; // found : String("234")
// required: Int
// eqInt.eqv(123, "234")
// ^
```

我们同样可以使用interface syntax语法，需要先导入[cats.syntax.eq](https://typelevel.org/cats/api/cats/syntax/package$$eq$)，然后我们就可以直接使用 === 和 =!=方法：

```scala
import cats.syntax.eq._ // for === and =!=

123 === 123
// res5: Boolean = true

123 =!= 234
// res6: Boolean = true
```

同样，我们尝试去比较两个不同类型的对象时也会编译报错：

```scala
123 === "123"
// <console>:20: error: type mismatch;
//  found   : String("123")
//  required: Int
//        123 === "123"
//         
```

#### **1.5.3 Comparing Op􏰀ons**

接下来我们来看一个更有趣的例子—Option[Int]。如果要比较Option[Int]类型的值，我们需要先导入Option以及Int对应的instances：

```scala
import cats.instances.int._    // for Eq
import cats.instances.option._ // for Eq
```

现在我们来尝试进行一些比较：

```scala
Some(1) === None
// <console>:26: error: value === is not a member of Some[Int] // Some(1) === None
// 
```

编译发现了一个错误，因为类型没匹配上，我们导入的是Int以及Option[Int]对应Eq的instances，所以Some[Int]是无法比较的。要解决这个问题我们需要将参数的类型指定为Option[Int]：

```scala
 (Some(1) : Option[Int]) === (None : Option[Int])
// res9: Boolean = false
```

更友好的方式是利用标准库中的Option.apply和Option.empty方法：

```scala
 Option(1) === Option.empty[Int]
// res10: Boolean = false
```

或者使用[cats.syntax.option](https://typelevel.org/cats/api/cats/syntax/package$$option$)中特殊的语法：

```scala
import cats.syntax.option._ // for some and none

1.some === none[Int]
// res11: Boolean = false
1.some =!= none[Int]
// res12: Boolean = true
```

#### **1.5.4 Comparing Custom Types**

我们可以为自定义的类型创建一个关于Eq的instance，它接收一个(A, A) => Boolean 的方法返回一个Eq[A]：

```scala
import java.util.Date
import cats.instances.long._ // for Eq

implicit val dateEq: Eq[Date] =
  Eq.instance[Date] { (date1, date2) =>
    date1.getTime === date2.getTime
  }
val x = new Date() // now
val y = new Date() // a bit later than now

x === x
// res13: Boolean = true
x === y
// res14: Boolean = false
```

#### **1.5.5 Exercise: Equality, Liberty, and Felinity**

实现一个Cat类型关于Eq的instance：

```scala
final case class Cat(name: String, age: Int, color: String)
```

并对下面这些对象进行判等操作：

```scala
val cat1 = Cat("Garfield",   38, "orange and black")
val cat2 = Cat("Heathcliff", 33, "orange and black")

val optionCat1 = Option(cat1)
val optionCat2 = Option.empty[Cat]
```

 代码见[示例]()

### 1.6 Controlling Instance Selec􏰀on

 在使用type class的时候，我们必须考虑以下两个问题，因为它们对于如何选择instance至关重要：

- 假设B是A的子类型，那么声明为A类型的instance能作用于B吗？

  举个例子，假如我们声明了一个JsonWriter[Option[Int]]的instance，那么Json.toJson(Some(1))能使用这个instance吗？（Some是Option的子类型）

- xxxx

#### **1.6.1 Variance**

当我们在声明type class时，可以使用可变的类型参数，这样可以让type class也有“变型”的能力。

在Essential scala中提到，variance跟子类型有关，假如可以在任意接收A类型值的地方用B类型值代替，那么可以说B是A的子类型。

当我们在定义类型构造器的时候，可以使用annotations来标明它是否是支持协变或者逆变的。举个例子，我么可以使用**”+“**符号表示它是协变的：

```scala
trait F[+A] // the "+" means "covariant"
```

##### Convariance

convariance代表着假如B是A的子类型，那么F[B]也是F[A]的子类型。这对很多类型的建模都很有用，比如List和Option：

```scala
trait List[+A]
trait Option[+A]
```

在Scala中支持协变的集合允许我们使用子类型的集合代替父类型的集合。比如我们可以在任何接收List[Shape]的地方使用List[Circle]代替，因为Circle是Shape的子类型：

```scala
sealed trait Shape
case class Circle(radius: Double) extends Shape

val circles: List[Circle] = ???
val shapes: List[Shape] = circles
```

那么什么是逆变呢？我们可以使用**”-“符号表示它是逆变的：

```scala
trait F[-A]
```

##### Contravariance

不同的是，contravariance代表着假如B是A的子类型，那么F[A]也是F[B]的子类型。逆变在对”**processes（处理）**“建模的时候非常有用，比如我们定义一个JsonWriter：

```scala
trait JsonWriter[-A] {
  def write(value: A): Json
}
// defined trait JsonWriter
```

进一步看其中的原理，我们要知道variance其实就是用一个值替换另一个值的能力。考虑一个场景，我们有两个类型值，Shape和Circle类型，以及两个JsonWriters，同样分别是Shape和Circle类型的：

```scala
val shape: Shape = ???
val circle: Circle = ???

val shapeWriter: JsonWriter[Shape] = ???
val circleWriter: JsonWriter[Circle] = ???

def format[A](value: A, writer: JsonWriter[A]): Json = writer.write(value)
```

现在你可以问自己一个问题：”format方法支持哪些参数组合呢？“。假如value为circle，那么writer可以为任一一个，因为所有Circle都是Shape。但反过来，shape不能与circleWriter组合，因为不是所有的Shape都是Circle。

这种情况下，我们就会使用逆变参数来进行建模。JsonWriter[Shape]是JsonWriter[Circle]子类型，因为Circle是Shape的子类型，这意味着任何接收JsonWriter[Circle]类型值的地方，都可以用shapeWriter代替。

##### **Invariance**

不变相对来说更容易理解，在类型构造的时候不需要指定”**+**“或者”**-**“的符号：

```scala
trait F[A]
```

这意味着不管A和B是什么关系，F[A]和F[B]都不再是对方的子类型，这也是Scala的默认方式。

当编译器在寻找implicit值的时候，除了自身类型implicit值以外，子类型的implicit值也在匹配范围之内，因此我们在一定程度上可以使用可变类型来控制type class instance的选择。

但这同样会导致一些问题，假设我们有以下代数数据类型：

```scala
sealed trait A
final case object B extends A
final case object C extends A
```

思考以下两个问题：

1. 子类型的值是否可以使用父类型的Instance？举个例子，我们声明了一个A类型的instance，那么它是否可以作用于B或者C？
2. 同时存在子类型以及父类型的instance的时候，优先选择哪个？比如现在我们分别声明A和B的instance，这时有一个B类型的值，它是否会优先选择B instance吗？

事实上，我们无法同时拥有两者，我们可以对以下三种情况进行归纳：

| Type Class Variance  | 不可变 | 协变 | 逆变 |
| -------------------- | ------ | ---- | ---- |
| 是否可以使用父类型？ | No     | No   | Yes  |
| 优先选择子类型？     | No     | Yes  | No   |

很明显，没有完美的类型系统。Cats更倾向于使用不变的类型，这意味着我们需要指定更具体的instance，举个例子一个Some[Int]的值直接使用Option[Int]类型的instance，如果需要的话，可以将Some[Int]类型的值声明为Option[Int]，比如 Some(1) : Option[Int]，或者使用一些更便捷的方法，比如我们在之前1.5.3章节中看到的Option.apply, Option.empty, some, none等方法。

#### **1.7 Summary**

在本章中，我们首先学习了什么是type class，然后实现了一个我们自己定义的type class：Printable，紧接着我们学习了Cats中的两个type class：Show和Eq。

现在，我们来看Cats的基本结构：

- type class都声明在[cats](http://typelevel.org/cats/api/cats/)这个包中。
- 每一个type class都有一个伴生对象，里面包含着一个apply方法，用于指定对应的instance，以及一个或者多个用于创建instance的构造方法，以及其他一系列相关的辅助方法。
- 默认的instance都放在[cats.instance](http://typelevel.org/cats/api/cats/instances/)这个包中，组织结构是以类型参数而不是type class，举个例子所有Int类型的instance会放在一起，而不是关于Show对应所有类型的instance放在一起。
- 很多type class都提供interface syntax，放置在[cats.syntax](http://typelevel.org/cats/api/cats/syntax/)中。

在接下去的章节中，我们将会学习一些应用广泛且强大的type class，比如Semigroup, Monoid, Functor, Monad, Semigroupal, Applicative, Traverse等，在每一个示例下，我们都会介绍这个type class所拥有的能力，遵循的法则，以及在Cats中是如何实现的。这些Type class相对Show以及Eq来说，要更抽象许多，虽然这会让我们更难学习，但在实际上它们却更有用。



### Chapter 2

###  **Monoids and Semigroups**

在本章中，我们将探索两个type class：**monoid**和**semigroup**。他们的主要功能是对值进行相加或者组合，并且很多类型都有对应这个两个type class的instance，比如Int，String，List，Option等，接下来让我们先了解一些简单的类型和操作，尝试理解其背后原理。

##### **Integer additi􏰀on**

Int的加法是满足**封闭性**的，也就是说两个Int相加会生一个新的Int：

```scala
 2+1
// res0: Int = 3
```

同时还存在一个“**幺元0**”满足a + 0 == 0 + a == a：

```scala
2+0
// res1: Int = 2
0+2
// res2: Int = 2
```

另外它也满足**结合律**：

```scala
(1 + 2) + 3
// res3: Int = 6
1 + (2 + 3)
// res4: Int = 6
```

**Integer mul􏰀plica􏰀tion**

Int的乘法同样满足上面我们描述加法的三个特性，只不过它的幺元由0变为了1：

```scala
1*3
// res5: Int = 3
3*1
// res6: Int = 3
```

也满足**结合律**：

```scala
(1 * 2) * 3
// res7: Int = 6

1 * (2 * 3)
// res8: Int = 6
```

##### String and sequence concatena􏰁tion

String类型同样也可以相加，可以使用字符串拼接作为一个操作符：

```scala
 "One" ++ "two"
// res9: String = Onetwo
```

幺元为一个空字符串:

```scala
"" ++ "Hello"
// res10: String = Hello

"Hello" ++ ""
// res11: String = Hello
```

同样，拼接也满足结合律：

```scala
("One" ++ "Two") ++ "Three"
// res12: String = OneTwoThree

"One" ++ ("Two" ++ "Three")
// res13: String = OneTwoThree
```

注意，这里我们是使用++来进行序列拼接，而不是+，除了字符串（可以看出Char类型的sequence）以外，我们还可以去其他类型的sequence执行相同的操作，拼接作为操作符，空sequence为单位元。

#### 2.1 Defini􏰁tion of a Monoid

现在我们已经看过好几个“additi􏰁on”的例子，它们都有一个添加的操作符和一个单位元，其实这就是Monoid，对于类型A来说：

- 有一个操作满足(A,A)=>A
- A类型有一个empty的值

这个定义用Scala表示非常容易，我们来看一下在Cats中的一个简单定义：

```scala
trait Monoid[A] {
  def combine(x: A, y: A): A
  def empty: A
}
```

monoid除了拥有这两个最基本的定义外，还需要满足一些法则，就是我们前面提到的“**结合律**”和“**幺元**”，对于A类型中的值，比如x，y，z必须满足以下两个函数：

```scala
def associativeLaw[A](x: A, y: A, z: A)
      (implicit m: Monoid[A]): Boolean = {
  m.combine(x, m.combine(y, z)) ==
    m.combine(m.combine(x, y), z)
}

def identityLaw[A](x: A)
      (implicit m: Monoid[A]): Boolean = {
  (m.combine(x, m.empty) == x) &&
    (m.combine(m.empty, x) == x)
}
```

