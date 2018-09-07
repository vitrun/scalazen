# 开枝散叶

有时候，我们并不满足于使用现成的Monad，需要为自定义的数据结构添加Monad特性。下面以常见的数据结构**树**为例，说明这一过程。
```scala
sealed trait Tree[+A]

final case class Leaf[A](value: A) extends Tree[A]

final case class Branch[A](left: Tree[A], right: Tree[A]) extends Tree[A]
```
简单起见，依然使用我们自定义的极简Mond，并提供隐式单例treeMonad，进行隐式函数调用的MonadMapper。
```scala
import scala.language.higherKinds

trait Monad[F[_]] {
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]

  def unit[A](a: A): F[A]

  def map[A, B](fa: F[A])(f: A => B): F[B] = flatMap(fa) { x => unit(f(x)) }
}

implicit object treeMonad extends Monad[Tree] {
  override def unit[A](value: A): Tree[A] = Leaf[A](value)

  override def flatMap[A, B](tree: Tree[A])(func: A => Tree[B]): Tree[B] = tree match {
    case Leaf(value) => func(value)
    case Branch(left, right) => Branch(flatMap(left)(func), flatMap(right)(func))
  }
}

implicit class MonadMapper[M[_]: Monad, A](v: M[A]) {
  private def m = implicitly[Monad[M]]
  def flatMap[B](f: A => M[B]): M[B] = m.flatMap(v)(f)
  def map[B](f: A => B): M[B] = m.map(v)(f)
}

```
通过隐式调用，编译器自动写了有用但无趣的脚手架代码，让手动写的代码简练了不少：
```scala
scala> val leaf: Tree[Int] = Leaf(3)
leaf: Tree[Int] = Leaf(3)

scala> leaf.flatMap(x => Leaf(x + 1))
res0: Tree[Int] = Leaf(4)
```

Monad本质上是一种**序列化计算结构的抽象**。Option抽象了可能成功也可能失败，但没有返回失败原因的计算；Either类似，但带了失败原因；List抽象了可能有多个结果的计算；Future抽象了可能未来某个时间才结束的计算。到底要为手头的数据结构赋予什么计算结构，是我们首先要思考的问题。回到数据结构树上来，看看实际执行的效果：
```scala
scala> val branch: Tree[Int] = Branch(Leaf(10), Leaf(-10))
branch: Tree[Int] = Branch(Leaf(10),Leaf(-10))

scala> branch.flatMap(x => Branch(Leaf(x - 1), Leaf(x + 1))).flatMap(x => Branch(Leaf(x * 2), Leaf(x * 3)))
res1: Tree[Int] = Branch(Branch(Branch(Leaf(18),Leaf(27)),Branch(Leaf(22),Leaf(33))),Branch(Branch(Leaf(-22),Leaf(-33)),Branch(Leaf(-18),Leaf(-27))))
```

显然，这里所抽象的逻辑是按给定的函数f对根节点进行分裂。

具备了Monad特性后，现在Tree就免费支持了很多操作，只是由于我们定义的Monad实在太精简了，一下子看不出都有哪些额外的操作。像常用的第三方库scalaz或cats，就提供了基于Monad的诸多操作，后面我们会接触到。纵然是这样，scala语言本身就支持的for操作是我们保底可以获得的奖励：
```scala
def branch[A](left: Tree[A], right: Tree[A]): Tree[A] = Branch(left, right)

def leaf[A](a: A): Tree[A] = Leaf(a)

for {
  l1 <- branch(leaf(10), leaf(-10))
  l2 <- branch(leaf(l1 - 1), leaf(l1 + 1))
  l3 <- branch(leaf(l2 * 2), leaf(l2 * 3))
} yield l3
```

for语句有很丰富的语义，形如下面的表达式中，是monad的flatMap操作的语法糖。
```scala
for (p <- m; q <- n...) yield (m, n)
```
相当于：
```scala
m.flatMap(p => n.map(q => (m, n)))
```

以上，我们通过为树这一数据结构添加Monad，让它有了开枝散叶的功能。Monad概念的强大，不仅体现在它在现有库中的地位，还体现在它本身可以开枝散叶，移植到具体业务场景中。
