# 重温函数和类型

#### 从一阶到高阶
根据维基百科的定义，
> 函数为两集合间的一种对应关系：输入值集合中的每项元素皆能对应唯一一项输出值集合中的元素。

函数的概念并不局限于数之间的映射关系，例如若定义函数Capital(country)为每个国家当前的首都，那么给予输入值西班牙就会输出唯一值马德里：Capital(Spain)=Madrid。

数学上的集合可以简单理解为程序中的类型，因此，程序中的函数描述的是特定类型的具体值之间的映射关系，如：
```scala
scala> def toStr(x: Int): String = x.toString
toStr: (x: Int)String

scala> toStr(3)
res0: String = 3

scala> def toSet[Int](x: List[Int]): Set[Int] = x.toSet
toSet: [Int](x: List[Int])Set[Int]

scala> toSet(List(1, 2, 3))
res1: Set[Int] = Set(1, 2, 3)
```

给特定类型的一个值，返回另一类型（或相同类型）的值。从这个意义看，函数可以理解为**值构造器**。

我们在[重温代数运算](/monoid/1_revisit_algebra.html)中用的手段——抽象，又可以派上用场了。值抽象的结果为类型，如-1, 3, 99等抽象为Int，"scala", "java"等抽象为String。构造器描述的映射关系保持不变，那么，值构造器相应变成了**类型构造器**：

```scala
scala> class List[T] {}
defined class List

//Int => List[Int]
scala> List[Int]()
res2: List[Int] = List()

//String => List[String]
scala> List[String]()
res3: List[String] = List()
```

熟悉Java的同学可能一眼就看出来了，类型构造器和Java中的泛型是一回事。

数学上，函数有一阶导数、二阶或更高阶导数的概念。零阶导数就是函数本身，相应地，值构造器可以视为零阶类型，只不过没有人这么称呼它而已。**一阶类型**也就是类型构造器，或者说泛型，比较常见。继续推演，很容易得到二阶、三阶……N阶类型，只不过我们不太用得到，也没有必要分得这么清楚，就把二阶或以上的统一称为**高阶类型**了，如：
```scala
scala> import scala.language.higherKinds
import scala.language.higherKinds

scala> class List2[C[T]] {}
defined class List2
```

简单总结类型在抽象层次上的递进关系，如下：
* **特定类型** 比如Int, String, List[Int]。
* **泛型类型** 用于构造特定类型的类型, 也称类型构造器，比如List。
* **高阶类型** 以泛型作为参数的泛型，比如List2，和后面即将介绍的Functor。

#### 高阶类型的使用
泛型这种一阶级别的抽象已经不容易掌握了，更高阶的抽象需要我们付出更多的心力，难免要问一问，它能达成什么效果，值得吗？

抽象是为了覆盖更多场景，取得更高的代码复用度。依然以求和函数为例，看一看使用高阶类型的威力。

前面我们定义了sum函数，能对任何具备Monoid特征的类型求和：
```scala
scala> def sum[A: Monoid](l: List[A]): A = {
     |   val m = implicitly[Monoid[A]]
     |   l.foldLeft(m.mzero)(m.mappend)
     | }
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
        def foldLeft[A](xs: List[A])(m: Monoid[A]) = xs.foldLeft(m.mzero)(m.mappend)
    }
    implicit val SetFoldable: Foldable[Set] = new Foldable[Set] {
        def foldLeft[A](xs: Set[A])(m: Monoid[A]) = xs.foldLeft(m.mzero)(m.mappend)
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

从代码量看，和分别对不同容器定义各自的sum方法比起来，似乎没有减少，反而可能更多了。这是因为sum方法本身非常简单，而实际业务代码量一般远不止这些，逻辑越复杂，上述方式的优势就越明显。而且，它把业务逻辑和对不同容器的支持分离开了，使得代码更易维护。
