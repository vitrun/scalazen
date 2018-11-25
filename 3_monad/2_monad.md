## 触发链式反应

> 链反应（或连锁反应、链式反应，Chain reaction）是指反应的产物或副产物又可作为其他反应的原料，从而使反应反复发生。——维基百科


#### 熟悉又陌生的Monad
对于map方法可接受的函数，我们已经接触了Functor中的常规函数`A => B`，以及Applicative中装在容器里的函数：`F[A => B]`。所以，当出现`A => F[B]`时，大概你不会感到奇怪。再次，为了区别，把接受`A => F[B]`的map称为flatMap，对应的特殊Functor称为Monad：
```scala
scala> trait Monad[F[_]] {
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B] = flatten(map(fa)(f))
  def flatten[A](ffa: F[F[A]]): F[A]
  def pure[A](a: A): F[A]
  def map[A, B](fa: F[A])(f: A => B): F[B]
}
defined trait Monad
```
经过map后得到F[F[B]]，套了两层容器，经过*flatten*压扁之后，得到只剩下一层容器的F[B]。
```text
                       用A => F[B]做map                   flatten
            F[A]  ------------------------->  F[F[B]]  -----------> F[B]
```
Option就是一个Monad：
```scala
scala> val f = (i: Int) => Some((i + 1) + " stars")
scala> Some(3).flatMap(f)
res0: Option[String] = Some(4 stars)
```
flatten方法构成了Monad区别于一般Functor的关键因素，如果没有它，我们只能不断地嵌套容器，无法消除重复的容器。正是"压扁容器"的能力，让flatMap有了比map更大的威力，它包含了map所能实现的功能。所以，比起上面的Monad定义，更常用、理简洁的定义是：
```scala
scala> trait Monad[F[_]] {
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
  def pure[A](a: A): F[A]
}
defined trait Monad
```
map可以用flatMap表示：
```scala
def map[A, B](fa: F[A])(f: A => B): F[B] = flatMap(fa){x => pure(f(x))}
```
反过来，如果没有flatten，则flatMap无法用map表示。甚至，flatten也可以用flatMap表示：
```scala
def flatten[A](ffa: F[F[A]]): F[A] = flatMap(ffa)(x => x)
```
scala官方库中并没有一个Monad类，但它已然随处可见。List、Set、Option等，我们再熟悉不过了，它们都是Monad。判断是否Monad并不是看有没有方法叫flatMap和pure，甚至并不看有没有签名一致的方法，而是看行为表现是否符合Monad概念。没错，Monad是一个概念，一个熟悉又陌生的概念。

假设一个基本类型的值x，一个容器实例m（装了某个值）和`Int => M[Int]`类型的函数f、g，如果以下法则得到满足，m就是Monad：
* 左单位元法则： `pure(x).flatMap(f) == f(x)`
* 右单位元法则： `m.flatMap(pure) == m`
* 结合性法则： `m.flatMap(f).flatMap(g) == m.flatMap(x => f(x).flatMap(g))`

Applicative是比Functor更强的概念，有更多约束，Functor的map可以用Applicative的apply表示；相应地，Monad是比Applicative还要强的概念，其约束进一步增加，Applicative的apply可以用Monad的flatMap表示：
```scala
def apply[A, B](fab: F[A => B])(fa: F[A]): F[B] = flatMap(fab)(f => map(fa)(f))
```
这个推导过程可能不是那么显而易见，当把`F[A => B]`传给F[C]时（注意，为了区分，我们把flatMap的A换成C），说明C代表的是个函数，还缺的`C => F[B]`可通过map来构造，map克里化后可写为`F[A] => C=> F[B]`，现成的fa满足了F[A]参数，部分施用之后得到的正是缺少的`C => F[B]`。

用一句伪代码表示三者的递进关系：Monad extends Applicative extends Functor。

#### 触发链式反应
Functor可以无限组合，多个map串联起来形成调用链，但因为Functor的map接受的是A => A函数，前后类型必须保持一致，致使应用场景受到很大限制。Monad打破了这一限制，能够触发真正的链式反应。

web应用中，根据用户输入做一系列操作是很常见的功能。比如，博客应用中，用户通过邮箱登录，打开最近发布的文章，查看文章的评论。这一过程中，我们需要调用以下方法：
* `getUser: String => Option[User]` // 从db中加载用户
* `getBlog: User => Option[Blog]` // 查询文章
* `getComment: Blog => Option[Comment]` // 查询评论
注意，每个方法的返回都是装在Option容器里的值，表示可能存在，也可以不存在要查询的内容。

```scala
scala>
trait User {}
trait Blog {}
trait Comment {}

def getUser(name: String): Option[User] = Some(new User {})
def getBlog(user: User): Option[Blog] = Some(new Blog {})
def getComment(blog: Blog): Option[Comment] = Some(new Comment {})

val comment = getUser("hello@world.com")
  .flatMap(getBlog)
  .flatMap(getComment)
res0: Option[Comment] = Some($anon$1@78ffe38)
```
Monad优雅地解决了中间结果的不确定性问题。操作的结果可能是合法值，也可能是非法值或抛出异常，当要把多个这样的操作连在一起调用时，必须判断每个函数的结果是否合法，只有合法的情况下才能继续调用下一个操作，程序员只需要关注正确的路径，发生异常时自动返回到最外层。想象一下，如果不用Monad，难免要写一堆if-else判断，检查option容器是否为空。固然也是可行的，但牺牲了可阅读性和可维护性。

Monad把getUser、getBlog之类高度可复用的代码单元，直观地组合起来，构成优雅的序列化计算。Monad可构建由多个步骤组成的数据处理管道，每一步骤都由对应的Monad提供额外的计算逻辑，并把计算结果放置到同一容器内，保持前后步骤间的适配。

这种把代码单元串联起来的功能，很像很多指令式编程语言，如C，Java里的";"，既是前后代码的分界点，也是前后运算的粘合剂。

Scala里的for语句有很丰富的语义，其中一个便是flapMap的一个语法糖，上面的例子也可以for表达式来写：
```scala
scala> val result: Option[Comment] = for (
    user <- getUser("hello@world.com");
    blog <- getBlog(user);
    comment <- getComment(blog)
  ) yield(comment)
result: Option[Comment] = Some($anon$1@6faeb7dc)
```

一般而言，
```scala
for (p <- m; q <- n...) yield (m, n)
```
相当于：
```scala
m.flatMap(p => n.map(q => (m, n)))
```
