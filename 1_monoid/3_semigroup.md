# Monoid的副产品

事实上，Monoid是从Semigroup派生而来的。**Semigroup**（半群）相对于Monoid少了单位元，它只要求满足结合律，从代码上看则只定义了combine运算。
```scala
scala> trait Semigroup[A] {
        def combine(a: A, b: A): A
     }
defined trait Semigroup
```
于是，Monoid可重新定义为：
```scala
scala> trait Monoid[A] extends Semigroup[A] {
        def zero(a: A):A
     }
defined trait Monoid
```
可见，当我们为类型A定义了Monoid，将同时获赠一个Semigroup，Semigroup是Monoid的副产品。对于输入类型为Semigroup的函数，我们可以直接传入Monoid。

很多Semigroup都是Monoid，当但有些数据类型却无法定义zero方法。我们知道整数集合对于加法运算是个Monoid，但如果把集合限定为正整数，便无法定义zero方法，它就退化成了Semigroup。

回到结合律和单位元上，试着用代码表述它们，分别如下：
```scala
def associativeLaw[A](x: A, y: A, z: A)(implicit m: Monoid[A]): Boolean  = m.combine(m.combine(x, y), z) == m.combine(x, m.combine(y, z))

def identityLaw[A](x: A)(implicit m: Monoid[A]): Boolean = (m.combine(x, m.zero) == x) && (m.combine(m.zero, x) == x)
```

眼尖的读者会问，只是定义了combine和zero而已，怎么就说分别对应了结合律和单位元呢？ 相当不错的问题，答案是不保证，或者说，这是你的职责，如果你声称某个类为Monoid时，你必须保证它满足上述法则。也只有它真的是个Monoid时，你才能享受到数学上严谨推导带来的好处。

从实践的角度看，当我们定义Monoid时，应主动检查声明的combine是否满足associativeLaw， zero是否满足identityLaw。如下面是对Set类型定义的Monoid，请读者分析是否满足这两个法则。

```scala
scala> import scala.language.higherKinds
import scala.language.higherKinds

scala> class setUnionMonoid[A] extends Monoid[Set[A]] {
     def combine(x: Set[A], y: Set[A]): Set[A] = x.union(y)
     def zero(x: Set[A]): Set[A] = Set.empty[A]
     }
defined class setUnionMonoid

```
