# 重温函数和类型

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

