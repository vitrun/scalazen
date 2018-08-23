# 云游太虚幻境
> 假作真时真亦假，无为有处有还无。——《红楼梦》

有不少现成的Monad可用，Option、List、Vector等等，但很多时候并不关心具体使用的容器，只要符合Monad概念即可，所以会把容器抽象掉，具体使用时再灵活选择容器，比如：
```scala
scala> def sumSquare[F[_]: Monad](a: F[Int], b: F[Int]): F[Int] =
  for {
    x <- a
    y <- b
  } yield x*x + y*y
sumSquare: [F[_]](a: F[Int], b: F[Int])(implicit evidence$1: scalaz.Monad[F])F[Int]

scala> sumSquare[Option](Some(1), Some(3))
res0: Option[Int] = Some(10)
scala> sumSquare[List](List(1), List(3))
res1: List[Int] = List(10)
scala> sumSquare(Vector(1), Vector(3))
res2: scala.collection.immutable.Vector[Int] = Vector(10)
```
但是，当试图在**特定类型**身上使用时，便会被编译器报怨：
```scala
scala> sumSquare(1, 3)
       <console>:3: error: erroneous or inaccessible type
```
如果sumSquare既能处理Monad，也能处理非Monad参数，那真是太棒了，复用程度将进一步提高。这时候，我们需要的是再进抽象一步，把Monad和非Monad也统一起来。Id就是为此而生的：
```scala
type Id[A] = A
```
通常，类型构造器返回一个装了原参数的容器，Id的神奇之处在于原样返回了，给定一个特定类型，Id类型构造器将返回该特定类型本身，并没有将其装进容器内。
```scala
scala> 123: Id[Int]
res3: scalaz.Scalaz.Id[Int] = 123

scala> "Hello": Id[String]
res4: scalaz.Scalaz.Id[String] = Hello
```
说白了，Id只是个类型别名，将特定类型伪装成套了容器的类型，如皇帝的新装般"骗过"了编译器。当然，为了欺骗成功，需要多点装模作样的动作：
```scala
scala> sumSquare(1: Id[Int], 3: Id[Int])
res5: scalaz.Scalaz.Id[Int] = 10
```
这个小伎俩有不小实际用途，sumSquare的逻辑很简单，如果非常复杂，比如接受的是依赖外部调用的Future（也可以视为Monad），就不容易进行单元测试。此时，便可以用Id替换Future做测试，因为它们都是Monad，在功能形态上等价，不影响对sumSquare本身的测试。

Id虽然是类型别名，但它的容器地位已经得到承认。像List、Option等容器一样，Id要具备Monad形态，就必须定义Monad的标志性方法，这里我们使用自己定义的Monad，而不是scalaz库的定义：
```scala
scala> trait Monad[F[_]] {
         def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
         def unit[A](a: A): F[A]
       }
defined trait Monad

val idMonad = new Monad[Id] {
  override def flatMap[A, B](a: Id[A])(func: A => Id[B]): Id[B] = func(a)
  //  如果也要重写map的话，将写成这样：
  //  override def map[A, B](a: Id[A])(func: A => B): Id[B] = func(a)
  override def unit[A](a: A): Id[A] = a
}
idMonad: Monad[scalaz.Scalaz.Id] = $anon$1@4f1ee0c6
```
不知你是否注意到，flatMap中Id[A]可以直接传给A => Id[B]，map中期待返回Id[B]，却接受A => B的返回而没有报怨。还有，flatMap和map的定义完全一致！进而，可以写出这样的代码：
```scala
scala> idMonad.flatMap(1){_ * 2}
res6: scalaz.Scalaz.Id[Int] = 2
```
在和不在容器内，貌似区别很大，却达成了和谐统一，似有似无，颇有几分神游太虚的感觉。

