## 大丈夫能屈能伸
> 将质子从十一维展开二维，在上面蚀刻电路，改造为智能计算机，可以自由收缩和展开维度，最多可以收缩到十一维，从而使得该质子获得人工智能，并可通过终端控制。——《三体》

在[再谈求和函数](/2_functor/2_sum_func_again.md)中，把定义了foldLeft方法的结构抽象成Foldable类。Foldable这个名字并不是随意起的，顾名思义，它是一种"可折叠"集合。

鉴于Foldable之常用，以及它对Traversable的铺垫意义，有必要花点篇幅来专门讲讲它。

有多种方式定义Foldable，一个极简的组成为foldMap和foldRight，基于它可以推导出众多常用的方法。
```scala
trait Foldable[F[_]] {
  def foldLeft[A, B](fa: F[A], z: B)(op: (B, A) => B): B

  def foldRight[A, B](fa: F[A], z: B)(op: (A, B) => B): B

  def foldMap[A, B](fa: F[A])(f: A => B)(implicit F: Monoid[B]): B

  def fold[M: Monoid](t: F[M]): M = foldMap[M, M](t)(x => x)
}
```
先看foldLeft和foldRight方法，前者从左向右（从头到尾）遍历；后者从右向左（从尾到头）遍历。注意二元函数**op**的参数顺序恰恰相反，前者累积值在前，后者累积值在后，这和遍历时的走向是一致的。当二元函数满足交换率时，foldLeft和foldRight得到的结果相同，如对List[Int]做加法运算，无论从哪个方向开始遍历，最终值是一样的。但如果是减法，因减数和被减数不能交换，其结果不同。我们以scala自带的foldLef和foldRight做演示：
```scala
scala> List(1, 2, 3).foldLeft(0)(_ - _)
res0: Int = -6

scala> List(1, 2, 3).foldRight(0)(_ - _)
res1: Int = 2
```

常见的可遍历容器，如List、Set、Seq、Vector和Stream都是可折叠的。

Foldable让Monoid有了更多发挥的空间。Foldable需要一个积累值（Accumulator）和一个二元函数将其和集合内的值按顺序结合起来。而Monoid是关于如何结合数值的，天然定义了这个二元函数和初始的累积值，所以在Monoid上定义Foldable是件非常轻松的事：
```scala
def foldLeft[A](xs: List[A])(m: Monoid[A]): A = xs.foldLeft(m.zero)(m.combine)
```

二者结合，产生很多有用的函数，如foldMap：游览可折叠结构，过程中先对元素进行map操作，再用Monoid的combine结合map的结果。如果这么解释还是有点抽象的话，foldMap有个热门的同义词mapReduce，想必你已经很熟悉了。给定一个类型A的集合，和`A => B`的函数，在已知如何合并B（比如，整数的加法）的情况下，可以算出一个汇总值，比如经典例子统计字符数。由于不限定运算的顺序，所以可以优化为并行计算。

```scala
val listFoldable = new Foldable[List] {
    override def foldLeft[A, B](fa: List[A], z: B)(op: (B, A) => B): B = {
      fa.foldLeft(z) { (accum, item) => op(accum, item) }
    }

    override def foldRight[A, B](fa: List[A], z: B)(op: (A, B) => B): B = {
      fa.foldRight(z) { (item, accum) => op(item, accum) }
    }

    override def foldMap[A, B](fa: List[A])(f: A => B)(implicit F: Monoid[B]): B = {
      fa.foldLeft(F.zero) { (accum, item) => F.combine(accum, f(item)) }
    }
}
val stars = listFoldable.foldMap(List(1, 2, 3))(_ + " stars ")
```

"折叠"这个字眼，直观上给人把整个列表压缩成一个值的印象，如对包含多个整数的列表做加法的折叠操作，得到的是一个整数。但折叠也可以操作结构不变，原来有多少元素，折叠后依然有多少元素。甚至可以用折叠操作推导出保持结构的函数map、filter和flatMap等，如：
```scala
def map[A, B](l: List[A])(f: A => B): List[B] = l.foldRight(List[B]()) {
        (item, acc) => f(item) :: acc
    }
```
你能用foldLeft实现列表的逆序操作吗？

还记得吗，Semigroup是弱化版的Monoid，相比之下少了zero，即不支持空的情况。相应地，如果基于Semigroup做折叠，也将要求是非空的数据结构。为了区分，称之为Foldable1（没错，这不是个好名字，姑且忍受下吧），其它方面和Foldable相同，就不再重复了。
