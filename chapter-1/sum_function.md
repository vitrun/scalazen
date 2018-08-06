# 从求和函数说起
#### 归纳出Monoid
让我们从一个非常简单的求和函数说起。下面函数对整数列表求和：
```scala
scala> def sum(xs: List[Int]): Int = xs.foldLeft(0){_ + _}
sum: (xs: List[Int])Int

scala> sum(List(1, 2, 3, 4))
res0: Int = 10
```

再用前面定义的IntMonoid重写sum函数：
```scala
scala> def sum(xs: List[Int]): Int = xs.foldLeft(IntMonoid.mzero)(IntMonoid.mappend)
sum: (xs: List[Int])Int

scala> sum(List(1, 2, 3, 4))
res1: Int = 10
```

当然，到目前为止，只是简单的替换，并没什么特别的。显然，求和函数不只限定于Int，其它类型如Double, Long也可以做。但为不同类型提供不同的函数显然不是上策，最好能定义与类型无关的函数。

```scala
scala> trait Monoid[A] {
         def mappend(a1: A, a2: A): A
         def mzero: A
       }
defined trait Monoid
```
于是，sum可以写成：
```scala
scala> def sum[A](xs: List[A], m: Monoid[A]): A = xs.foldLeft(m.mzero)(m.mappend)
sum: [A](xs: List[A], m: Monoid[A])A
```
但是，不同类型（集合）对应的mappend和mzero并不相同，必须明确定义这两方法，才能支持sum函数。Int类型的上面已经定义过了，改成继承自一般性的Monoid：
```scala
scala> object IntMonoid extends Monoid[Int] {
         def mappend(a: Int, b: Int): Int = a + b
         def mzero: Int = 0
       }
defined module IntMonoid

scala> sum(List(1, 2, 3), IntMonoid)
res2: Int = 6
```
String类型的定义如下：
```scala
scala> object StringMonoid extends Monoid[String] {
        def mappend(a: String, b: String): String = a + b
        def mzero: String = ""
      }
defined object StringMonoid

scala> sum(List("a", "b", "c"), StringMonoid)
res3: String = abc
```
不难发现，只要定义了类型（集合）的mappend（运算）和mzero（单位元），该类型便自动具备了进行sum运算的能力。
#### 使用隐含参数
比较麻烦的是，每次都要手动传入Monoid的具体类型。使用scala的implicit parameter特性，可以省去此麻烦。
```scala
scala> def sum[A](xs: List[A])(implicit m:Monoid[A]):A = xs.foldLeft(m.mzero)(m.mappend)
sum: [A](xs: List[A])(implicit m: Monoid[A])A

scala> implicit val intMonoid = IntMonoid
intMonoid: IntMonoid.type = IntMonoid$@410c1f44

scala> sum(List(1, 2, 3))
res4: Int = 6
```
另一种方式是使用context bound：
```scala
scala> def sum[A: Monoid](xs: List[A]): A = {
     |   val m = implicitly[Monoid[A]]
     |   xs.foldLeft(m.mzero)(m.mappend)
     | }
sum: [A](xs: List[A])(implicit evidence$1: Monoid[A])A

scala> sum(List(1, 2, 3))
res5: Int = 6
```

scala寻找隐含参数的范围还包括companion对象，所以还可以把隐含值定义在companion对象内，为了方便阅读，把完整代码贴出来：
```scala
scala> :paste
// Entering paste mode (ctrl-D to finish)

trait Monoid[A] {
    def mappend(a: A, b: A): A
    def mzero: A
}

object Monoid {
    implicit val IntMonoid: Monoid[Int] = new Monoid[Int] {
        def mappend(a: Int, b: Int): Int = a + b
        def mzero = 0
    }
}

def sum[A: Monoid](xs: List[A]): A =  {
    val m = implicitly[Monoid[A]]
    xs.foldLeft(m.mzero)(m.mappend)
}


// Exiting paste mode, now interpreting.

defined trait Monoid
defined object Monoid
sum: [A](xs: List[A])(implicit evidence$1: Monoid[A])A

scala> sum(List(1, 2, 3))
res0: Int = 6

```
至此，我们从最具体的算数运算一步步归纳得到抽象的Monoid，也正因为抽象，它的应用非常广泛，函数式编程的很多概念都会有它的影子。而基于代数原理的抽象，也是函数式编程区别于面向对象编程的重要特点，这一点，在后面会更进一步感受。

