# 链式反应

对于map方法可接受的函数，我们已经接触了Functor中的常规函数A => A，以及Applicative中装在容器里的函数：F[A => A]。所以，当出现A => F[A]时，大概你不会感到奇怪。再次，为了区别，把接受A => F[A]的map称为flatMap，对应的特殊Functor称为Monad：
```scala
scala> trait Monad[F[_]] {
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B] = flatten(map(fa)(f))

  def flatten[A](ffa: F[F[A]]): F[A]

  def unit[A](a: A): F[A]

  def map[A, B](fa: F[A])(f: A => B): F[B]
}
```
经过map后得到F[F[B]]，套了两层容器，经过*flatten*压扁之后，得到只剩下一层容器的F[B]。
```text
                       用A => F[B]做map                   flatten
            F[A]  ------------------------->  F[F[B]]  -----------> F[B]
```
注意flatMap并不要求A => F[A]，而是更灵活的A => F[B]，Int => List[String]、Int => Optional[OwnClass]等等，如：
```scala
scala> val f = (i: Int) => Some((i + 1) + " stars")
scala> Some(3).flatMap(f)
res0: Option[String] = Some(4 stars)
```
