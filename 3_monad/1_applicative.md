# 容器里的函数

> 这人总想把自己包在壳子里，仿佛要为自己制造一个套子，好隔绝人世，不受外界影响。 ——《装在套子里的人》

介绍Functor时说到，类型构造器F可以理解为容器，发挥下想象力，这个容器既然可以装类型，能不能装函数呢？显然可以，比如下面的ff：
```scala
scala> val ff: Option[Int=>String] = Some(x=>x.toString + " stars")
ff: Option[Int => String] = Some($$Lambda$1801/1106811850@256dcf9c)
```
或者，顺着之前Functor的思路：之前定义的map接受的函数f一直是一个入参，一个返回值。如果是多个入参，比如下面的f2，执行map后得到的也是装在容器里的函数：

```scala
scala> val f2: (Int, Int) => Int = (x, y) => x+y
scala> Some(3).map(i => (j:Int) => f2(i, j))
res0: Option[Int => Int] = Some($$Lambda$1702/547090744@27816139)
```

为了能执行容器里的函数，需要对Functor做改造，为了区分，改名为Apply：
```scala
scala> trait Apply[F[_]] {
           def apply[A, B](fa: F[A])(f: F[A => B]): F[B]
       }
defined trait Apply
```
以option容器为例，入参为一个容器里的值和一个相同容器里的函数，返回值仍然装在容器里。
```scala
scala> val optionApply = new Apply[Option] {
           def apply[A, B](fa: Option[A])(f: Option[A=>B]): Option[B] = (fa, f) match {
              case (Some(fa), Some(f)) => Some(f(fa))
              case (_, _) => None
          }
      }
optionApply: Apply[Option] = $anon$1@38e0a6e

scala> optionApply.apply(Some(3))(ff)
res1: Option[String] = Some(3 stars)
```
接下来的问题是：
* 输入的值必须包在容器里，能否同时支持在和不在容器两种情况呢？
* 容器里的函数是一个入参，能否支持多个呢？

这两个限制如果突破了，灵活性提高不小，使用范围也将扩大。第一个问题很容易解决，加一个能把值装入容器的方法，为了区分，再次改名为Applicative：
```scala
scala> trait Applicative[F[_]] {
           def apply[A, B](fa: F[A])(f: F[A => B]): F[B]
           def pure[A](a: A): F[A]
       }
defined trait Applicative
```
这样一来，如果碰到"裸值"，先执行pure装进容器里，就能适配apply了。

针对问题二，如果我们能把两个参数的函数转变为一个参数的，就有理由相信可以支持任意多个参数。利用函数的克里化，这种转变并不复杂，下面两个参数的apply2就可以用一个参数的apply来表示：
```scala
scala> trait Applicative[F[_]] {
    def apply[A, B](fa: F[A])(f: F[A => B]): F[B]
    def pure[A](a: A): F[A]
    def apply2[A, B, C](fa: F[A], fb: F[B])(f: (A, B) => C): F[C] =  {
        //返回A => B型的函数，或写成val fabc = pure(f.curried)
        val fabc = pure({a: A => {b: B => f(a, b)}})
        val fbc = apply(fa)(fabc)
        apply(fb)(fbc)
    }
}
defined trait Applicative
```

以List为例，看一组实际应用：
```scala
scala> object listApp extends Applicative[List] {
  def pure[A](a: A): List[A] = List(a)

  override def apply[A, B](fa: List[A])(f: List[A => B]): List[B] = f match {
    case head:: tail => fa.map(head) ++ apply(fa)(tail)
    case Nil => Nil
  }
}
defined object listApp

scala> listApp.apply2(List(1, 2), List(3, 4))((_, _))
res2: List[(Int, Int)] = List((1,3), (1,4), (2,3), (2,4))

scala> listApp.apply2(List(1, 2), List(3, 4))(_ + _)
res3: List[Int] = List(4, 5, 5, 6)
```
注意，apply2计算结果的元素数量等于两个入参元素数量的乘积。其原因可以从apply2和apply的实现中找到，apply2执行第一次apply时得到的fb是部分施用的函数列表，长度和fa，即例子中的`List(1, 2)`一致，第二次apply时，则对fb中的每个元素执行该列表的每一个函数。

上述过程中，apply2通过apply推导而来，凭直觉，apply应该也得能apply2表示，事实也确实如此：
```scala
scala> trait Applicative[F[_]] {
           def apply2[X, Y, Z](fx: F[X], fy: F[Y])(g: (X, Y)=> Z): F[Z]
           def pure[A](a: A): F[A]
           def apply[A, B](fa: F[A])(fab: F[A => B]): F[B] = apply2(fab, fa)((f, x) => f(x))
       }
defined trait Applicative
```
这里用到的字母比较多，apply2用X, Y, Z，而不是A, B, C，只是为了便于解释说明apply方法。
X是一个A => B类型的函数； Y是A类型的入参；g通过对A执行A=>B得到B，所以Z的类型就是B，说明apply2和apply的返回类型是一致的。

综合起来，可以发现：
* Applicative是在Apply基础上添加pure。Applicative和Apply的关系类似于Monoid和Semigroup的关系。
* apply和apply2是等价的，二者可以相互表示，所以实际使用时，定义一个即可。
* 更有意思的是，Functor的标志性方法map也可以用apply推导而来：
```scala
def map[A, B](fa: F[A])(fab: A => B): F[B] = apply(fa)(pure(fab))
```
所以，Applicative天然是个Functor，可以从Functor拓展而来。

稍做总结，有两个角度理解Applicative比常规Functor更厉害的地方。首先，Applicative知道如何在容器环境中执行装在同样容器里的函数。比如，给定一个Option[Int]容器，它知道如何对它执行`Option[Int => Int]`，而常规Functor里的map只具备执行`Int => Int`的能力。另一个角度是，Applicative能执行多参数的函数，而常规Functor只能执行单参数函数。
