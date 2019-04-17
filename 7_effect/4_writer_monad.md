## Writer Monad

>

#### 

看过携带`S => (S, A)`的`State Monad`和携带`E => A`的`Reader Monad`之后，我们对各种`Monad`的"套路"已有所了解，理解其它形式的`Monad`应不在话下。下面是`Writer Monad`的定义：
```scala
case class Writer[T, M](a: T, m: M)(implicit mo: Monoid[M]) {

  def flatMap[B](f: T => Writer[B, M]): Writer[B, M] = f(a) match {
    case Writer(b, m2) => Writer(b, mo.combine(m, m2))
  }

  def map[B](f: T => B): Writer[B, M] = Writer(f(a), m)
}
```
`M`是个`Monoid`，还记得其基本特性是可以按自定义的操作进行求和：
```scala
trait Monoid[T] {

  def combine(m: T, m2: T): T

  def zero: T
}
```
`Writer`的`flatMap`方法便是利用它累加了`m`和`m2`。我们以`String`的累加为例，看看`Writer`的使用：
```scala
implicit val strMonoid: Monoid[String] = new Monoid[String] {
override def combine(m: String, m2: String): String = m + "; " + m2

override def zero: String = ""
}

val res = for {
  i <- Writer(1, "set to 1").flatMap(i => Writer(i + 2, "added 2"))
  j <- Writer(i + 3, " added 3")
} yield j

println(res.a, res.m)
```
将打印出："(6, inited to 1; added 2;  added 3)
"。哪怕上面的代码解读起来有点抽象，看到这个输出后，就不难理解了。`Writer`的功能是忠实地按顺序记录了每一步做的操作。其意义在于，对于每一步计算，只需要记录本身所做的计算即可，如`1, "set
 to 1"`、`i => Writer(i + 2, "added 2")`，不用关心如何把新产生的记录累加到已有的日志中。在Java等指令式编程语言中，以下写日志的方式很常见：
```java
import java.util.logging.Logger;

class Foo {
  private Integer sum = 0;
  private static final Logger logger = Logger.getLogger("fooLogger");

  private Integer add(Integer val) {
    sum += val;
    logger.info("added " + val);
    return sum;
  }
}
```
但这里的`add`是有副作用的，日志被打印到外设中了，如果把日志也当成返回值的一部分，则需要程序员自己处理，既麻烦又易出错。`Writer
`完美解决了此问题，保持纯函数的同时，程序员无需操心日志的维护。

当然，日志只是个例子，`Writer`可累加的不并限于`String`类型。只要是个`Monoid`，即可以从0开始一个元素一个元素地累加的类型，都行。这么看，`Writer Monad`其实是一个函数式的构造器模式。
