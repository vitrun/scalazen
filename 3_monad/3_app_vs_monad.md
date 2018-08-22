# 尺有所短寸有所长

> 夫尺有所短，寸有所长，物有所不足，智有所不明，数有所不逮，神有所不通。——《楚辞·卜居》

我们已经见识到了Monad作为升级版的Applicative，能用上一步骤的结果触发下一步运算，进而形成链式反应。这种形式之常见，似乎就此奠定了Monad的重要地位。然而尺有所短，寸有所长，它们各有自己的用武之地。在进入下个议题之前，有必要将二者放在一起审视一番。

Monad是严厉版的Applicative，它可接受的函数A => F[B]，必须在A类型的值被计算出来后才能执行，所以后续步骤必须基本前一步骤的结果；Applicative是宽松版的Monad，它接受函数F[A => B]，和前一步的具体结果没有关联，所以不对前一步做任何假设，也不关心其结果。

这个区别使得Monad适合串行计算，Applicative适合并行计算。

前面查询评论的例子中，getBlog必须在getUser有结果之后才能执行，像这种业务逻辑本身有前后依赖关系的场景，天然合适使用Monad。需要指出的是，一旦使用了Monad模式，即使业务上没有严格的前后关系，也会以串行方式依次执行，如下面代码中，虽然相互之间没有依赖，依然是按alice，bob，eve的顺序执行的。
```scala
val group = for {
  alice <- getUser("Alice")
  bob <- getUser("Bob")
  eve <- getUser("Eve")
} yield (alice, bob, eve)
```

但其实这三个用户的查询可以同时进行，所以更适合用Applicative表达，遗憾的是scala语言本身不自带对Applicative的融合操作，我们无法写出像for语句这般清晰简洁地代码。好在有不少库封装了这种操作，比如scalaz库的|@|操作符，使用很方便：
```scala
val group = (getUser("Alice") |@| getUser("Bob") |@| getUser("Eve")) {(_, _, _)}
```
|@|以三个Applicative为参数生成Applicative构造器：
```scala
val builder = ApplicativeBuilder[Option, User, User, User] = getUser("Alice") |@| getUser("Bob") |@| getUser("Eve")
```
再应用提供的函数得到最后结果，显然，结果的容器还是Option。
```scala
val group = builder.apply((_, _, _))
```
