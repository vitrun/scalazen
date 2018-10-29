## 再谈求和函数

泛型这种一阶级别的抽象已经不容易掌握了，更高阶的抽象需要我们付出更多的心力，难免要问一问，它能达成什么效果，值得吗？

抽象是为了覆盖更多场景，取得更高的代码复用度。依然以求和函数为例，看一看使用高阶类型的威力。

前面我们定义了sum函数，能对任何具备Monoid特征的类型求和：
```scala
scala> def sum[A: Monoid](l: List[A]): A = {
        val m = implicitly[Monoid[A]]
        l.foldLeft(m.zero)(m.combine)
      }
sum: [A](l: List[A])(implicit evidence$1: Monoid[A])A

scala> sum(List("a", "b", "c"))
res0: String = abc

scala> sum(List(1, 2, 3))
res1: Int = 6
```

如果我们不满足只对求和的元素做抽象，更进一步，保存这些元素的"容器"能否不限定于List呢，Array或者Set行不行呢？答案自然是可行的，问题在于，能不能避免为每种容器都定义一个sum呢，毕竟代码逻辑都一样的，重复写框架代码不仅不酷，还容易出错。

分析发现sum函数只是使用到了List的foldLeft方法，也就是说，sum应该能支持任意具有foldLeft方法的容器，我们用Foldable（可折叠）来描述这种特质的容器，显然，Foldable要求定义foldLeft方法：
```scala
trait Foldable[F[A]] {
    def foldLeft[A](xs: F[A])(m: Monoid[A]): A
}
```
于是，sum的定义可重写为：
```scala
def sum[F[_]: Foldable, A: Monoid](xs: F[A]): A = {
    val f = implicitly[Foldable[F]]
    val m = implicitly[Monoid[A]]
    f.foldLeft(xs)(m)
}
```
这样，如果要获得对某容器的支持，只要声明该容器如何转成Foldable即可。比如，要让sum支持List和Set，只要在companion对象中做如下声明：
```scala
object Foldable {
    implicit val ListFoldable: Foldable[List] = new Foldable[List] {
        def foldLeft[A](xs: List[A])(m: Monoid[A]) = xs.foldLeft(m.zero)(m.combine)
    }
    implicit val SetFoldable: Foldable[Set] = new Foldable[Set] {
        def foldLeft[A](xs: Set[A])(m: Monoid[A]) = xs.foldLeft(m.zero)(m.combine)
    }
}
```
从此，sum就支持了Set和List容器，容器内可以是Int，也可以是String。
```scala
scala> sum(Set(1, 2, 3))
res7: Int = 6

scala> sum(List(1, 2, 3))
res8: Int = 6

scala> sum(List("a", "b"))
res9: String = ab

scala> sum(Set("a", "b"))
res10: String = ab
```

从代码量看，和分别对不同容器定义各自的sum方法比起来，似乎没有减少，反而可能更多了。这是因为sum方法本身非常简单，而实际业务代码量一般远不止这些，逻辑越复杂，上述方式的优势就越明显。而且，它把业务逻辑和对不同容器的支持分离开，代码更易维护。
