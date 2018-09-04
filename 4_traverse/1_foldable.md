# 大丈夫能屈能伸
> 将质子从十一维展开二维，在上面蚀刻电路，改造为智能计算机，可以自由收缩和展开维度，最多可以收缩到十一维，从而使得该质子获得人工智能，并可通过终端控制。——《三体》

在[再谈求和函数](/2_functor/2_sum_func_again.md)中，把定义了foldLeft方法的结构抽象成Foldable类。Foldable这个名字并不是随意起的，顾名思义，它是一种"可折叠"集合。从当时的定义不难看出Foldable和Monoid的紧密关系。这是因为Foldable的运行需要一个积累值（Accumulator）和一个二元函数将其和集合内的值按顺序结合起来。而Monoid天然定义了这个二元函数和初始的累积值，所以在Monoid上定义Foldable是件非常轻松的事：
```scala
val ListFoldable: Foldable[List] = new Foldable[List] {
        def foldLeft[A](xs: List[A])(m: Monoid[A]) = xs.foldLeft(m.mzero)(m.mappend)
    }
```

鉴于Foldable之常用，以及它对Traversable的铺垫意义，有必要花点篇幅来专门讲讲它。

Foldable一般包含foldLeft和foldRight方法：
```scala
trait Foldable[A] {
  def foldLeft[B](z: B)(op: (B, A) => B): B
  def foldRight[B](z: B)(op: (A, B) => B): B
}
```
foldLeft从左向右（从头到尾）遍历；foldRight从右向左（从尾到头）遍历。注意二元函数**op**的参数顺序恰恰相反，前者累积值在前，后者累积值在后，这和遍历时的走向是一致的。当二元函数满足交换率时，foldLeft和foldRight得到的结果相同，如对List[Int]做加法运算，无论从哪个方向开始遍历，最终值是一样的。但如果是减法，因减数和被减数不能交换，其结果不同。
```scala
scala> List(1, 2, 3).foldLeft(0)(_ - _)
res0: Int = -6

scala> List(1, 2, 3).foldRight(0)(_ - _)
res1: Int = 2
```
常见的可遍历容器，如List、Set、Seq、Vector和Stream都是可折叠的。
