## 尺有所短寸有所长

> 夫尺有所短，寸有所长，物有所不足，智有所不明，数有所不逮，神有所不通。——《楚辞·卜居》

我们已经见识到了Monad作为升级版的Applicative，能用上一步骤的结果触发下一步运算，进而形成链式反应。这种形式之常见，似乎就此奠定了Monad的重要地位。然而尺有所短，寸有所长，它们各有自己的用武之地。在进入下个议题之前，有必要将二者放在一起审视一番。

Monad是严厉版的Applicative，它可接受的函数`A => F[B]`，必须在A类型的值被计算出来后才能执行，所以后续步骤必须基本前一步骤的结果；Applicative是宽松版的Monad，它接受函数`F[A => B]`，和前一步的具体结果没有关联，所以不对前一步做任何假设，也不关心其结果。

这个区别使得Monad适合串行计算，Applicative适合并行计算。

前面查询评论的例子中，getBlog必须在getUser有结果之后才能执行，像这种业务逻辑本身有前后依赖关系的场景，天然合适使用Monad。需要指出的是，一旦使用了Monad模式，即使业务上没有严格的前后关系，也会以串行方式依次执行，如下面代码中，虽然相互之间没有依赖，依然是按alice，bob，eve的顺序执行的。
```scala
scala> val group = for {
  alice <- getUser("Alice")
  bob <- getUser("Bob")
  eve <- getUser("Eve")
} yield (alice, bob, eve)
group: Option[(User, User, User)] = Some(($anon$1@4c541b00,$anon$1@1f26bd62,$anon$1@61f18997))
```

但其实这三个用户的查询可以同时进行，所以更适合用Applicative表达，遗憾的是scala语言本身不自带对Applicative的融合操作，我们无法写出像for语句这般清晰简洁地代码。好在有不少库封装了这种操作，比如scalaz库的|@|操作符，提供了很大便利。

简单起见，我们从[Maven](https://search.maven.org/remotecontent?filepath=org/scalaz/scalaz-core_sjs1.0.0-M5_2.12/7.2.26/scalaz-core_sjs1.0.0-M5_2.12-7.2.26.jar)仓库下载scalaz核心包，在终端里执行以下命令直接引入依赖。
```bash
scala -classpath scalaz-core_sjs1.0.0-M5_2.12-7.2.26.jar
```
以下两句代码将引入scalaz中常用类型、操作符等，为了减少篇幅，后面代码引用将省略这两句。
```scala
import scalaz._
import Scalaz._
```
终于可以非常方便地组合Applicative了，甚至比Monad的flatMap或for语句还更简明一些。
```
scala> val group = (getUser("Alice") |@| getUser("Bob") |@| getUser("Eve")) {(_, _, _)}
group: Option[(User, User, User)] = Some(($anon$1@5e5283a4,$anon$1@24922f21,$anon$1@57f5be22))
```
|@|操作符要借助于scalaz的一个private类：ApplicativeBuilder，用若干个Applicative为参数生成Applicative构造器， 再应用提供的函数得到最后结果，显然，结果的容器还是Option。
```scala
val builder = ApplicativeBuilder[Option, User, User, User] = getUser("Alice") |@| getUser("Bob") |@| getUser("Eve")
val group = builder.apply((_, _, _))
```

眼光锐利的读者可能注意到了，既然Monad从Applicative强化而来，Applicative也**可以**（不是一定）有Monad的能力，即串行执行的能力。其实，Applicative的核心方法apply可以从flatMap推导出来，已经表明了这一点。
```scala
def apply[A, B](fab: F[A => B])(fa: F[A]): F[B] = flatMap(fab)(map(fa))
```
重写的apply方法是flatMap和map的组合，因为flatMap方法保证了必须按先后顺序挨个执行，导致了apply方法也必须按顺序执行。当然，这也没什么奇怪的，一般性而言，Applicative不对前置步骤的结果做假设，也就意味着可以串行，也可以并行。具体是哪种，取决于实际计算发生前，整个计算结构是否确定，如果各步骤的计算逻辑是明确的，则可以并行；如果必须在运行时通过前置步骤获得信息才能明确，则只能串行。

实际使用时该如何选择呢，特别是在scala语言本身对Applicative的操作不太方便的情况下？一个原则是，在满足要求的情况下，使用最少假设的抽象模型。假设的前提条件越少，留给以后优化的空间就越大，代码被复用的可能性也越大。Applicative没有Monad那样的前后依赖的假设，少了限制，那么编译器或者基于此做进一步操作的代码库，有更大的空间采取针对性的优化措施，这样往往能达到更好的执行性能。
