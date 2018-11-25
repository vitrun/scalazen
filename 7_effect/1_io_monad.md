## IO Monad

> 三体世界制造的647号小宇宙的门出现，程心与关一帆进入了647号小宇宙，在那里试图到达宇宙大坍缩后产生的新宇宙。——《三体》

##### IO Monad的使用
纯函数的世界固然美满，我们不能忘记现实世界的骨感：哪怕只是在屏幕打印一个像素，从磁盘或网络读取一个字节，都是不纯粹的操作，即都是有副作用的。

不和外界打交道的“纯软件”是没有意义的。因此，我们要承认并接纳这种非纯粹性，并清楚地标识它，从而有效地把程序的纯粹部分和非纯粹部分隔离开。IO Monad就是这道门，穿过它，便将混乱无序拋诸脑后，置身纯净整洁的新世界。

从定义上看，和普通Monad相比，IO Monad除了名字之外并没什么特别的。
```scala
class IO[A](codeBlock: => A) {
  def run: A = codeBlock

  def flatMap[B](fab: A => IO[B]): IO[B] = {
    val result1: IO[B] = fab(codeBlock)
    val result2: B = result1.run
    IO(result2)
  }

  def map[B](f: A => B): IO[B] = flatMap(a => IO(f(a)))
}

object IO {

  def pure[A](a: A): IO[A] = IO(a)

  def apply[A](a: => A): IO[A] = new IO(a)
}
```
顾名思义，IO Monad是盛有输入输出作用的容器。如，从屏幕上读取字符串的结果是`String`，打印字符串的结果是`Unit`。为了清晰地表明这是有副作用的指令，我们用`IO[String]`和`IO[Unit]`额外包裹了作用。

```scala
def readLine(): IO[String] = IO(scala.io.StdIn.readLine())
def writeLine(s: String): IO[Unit] = IO(println(s))
```

IO Monad在使用形式上就这么简单，返回值类型再套上`IO
`而已。它提醒调用者注意该函数在和外界交互，是带有副作用的。这样，纯函数式世界的一个好处得以保全：只看签名就能判断函数的意图，而不用翻阅实现代码。

**一切都和组合有关**。因为定义了`flatMap`和`map`，便有了另一个好处：可以用`for`语法糖把多个IO Monad串起来，形成一系列有依赖关系的，更复杂的逻辑，如：
```scala
object IoApp extends App {

  def loop: IO[Unit] = for {
    _ <- writeLine("Command me.")
    input <- readLine()
    _ <- writeLine("Your command is: " + input)
    _ <- if(input == "quit") IO(Unit) else loop
  } yield ()

  loop.run
}
```

##### 深入IO Monad
需要指明的是，正如把IO Monad引入Haskell的Wadler所说，Monad只是表明了一种副作用的存在，并没有消除它，也就是说，函数并不会因此变得纯粹。`readLine
`的执行结果仍然是不确定的，当输入"hello"时得到`IO
.pure("hello")`，输入"world"时得到`IO.pure("world")`，显然这是不同的结果。我们无法把非纯粹的事物变得纯粹。我们要做的是尊重并规范它们，防止它们污染纯函数式世界。

对比以下两行代码，会发现，直接调用的IO操作会立刻执行，而`IO`包裹的则不会。
```scala
val result1 = println("hello world")
val result2 = IO{println("hello world")}
```
本质上，后者是对IO作用的一种**描述**，实际操作会被延迟到触发时才执行。描述了如何打印字符串`hello 
world`的`result2`变成一等公民，可以像其它类型的值一样到处传递。继函数之后，又一个普通视角里的次等公民被统一对待了。

我们知道，函数式编程试图借鉴代数做法，用表达式描述问题，对表达式**求值**来得到问题的答案，而不是通过**执行**一系列预设**指令**得到答案。这一思想在纯函数的世界里得以完美贯彻，但涉及到I/O时便显得力有不逮。本质上是因为要对现实世界产生作用——函数式意义上的副作用，有赖于操作系统，最终则是硬件的原生功能。这些功能不是函数式语言及其背后的代数可以承接的。在屏幕上打印字符这种操作无法通过求值完成，却是软件运行系统可轻松完成的。这是纯函数世界和现实世界的边界，是“魔法”发生的地方。所以，这些延迟的IO作用总是在这个边界上触发，如`IoApp`中的`loop.run`。

借用Learn You a Haskell for Great Good一书的说法：“可以把I/O操作想象成长着小脚的箱子，小脚伸进现实世界，如在墙上涂鸦，还可能带回了数据。取出数据的唯一办法是打开箱子，而且只能在进行另一个I/O操作时取出。这是如此，Haskell把代码的纯粹和非纯粹部分干净地分离开了。”

只能在另一个I/O操作里取出当前I/O的数据，是因为I/O操作是非纯粹的，如果在纯函数内调用了I/O操作，就破坏了函数的纯粹性。而非纯粹函数不用遵循什么规则，可以做任意需要的操作，包括调用其它非纯函数，和调用纯函数。下图绿色背景表示“纯粹世界”，黄色背景表示“非纯粹世界”，只允许非纯粹世界穿越调用纯粹世界，而不允许反过来。
![io_call](/imgs/io_call.png)

**IO Monad是引用透明的（Referential 
Transparent
）**。乍一看，这非常让人费解，既然前面已经说它是不纯粹的，怎么又是引用透明的呢？引用透明的定义是，如果变量与其代表的表达式任意替换都不会改变语义，则就是引用透明。我们用代码做个试验：

```scala
val input = IO(Random.nextInt(10))

val x = for {
  a <- input
  b <- input
} yield (a, b)

val y = for {
  a <- IO(Random.nextInt(10))
  b <- IO(Random.nextInt(10))
} yield (a, b)
```
虽然执行`x.run`和`y.run`后得到的结果很可能不一样，但`x`和`y`本身表达语义完全一样。用变量`input`的表达式`IO(Random.nextInt(10))`代替`input`并没有改变程序的语义，说明`IO`是引用透明的。

引用透明和纯粹性并不是一回事。纯粹性是说程序在**执行**时不会产生副作用，是对运行时特性的描述；而引用透明是指替换变量和表达式不会改变语义，是对逻辑含义的描述。

