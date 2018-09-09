# 穿越火线

> If you know something inside out, you know it very well.

可以简明地概括之前几个容器的关键点：

* Functor：定义如何对装在容器内的元素执行函数。
* Applicative：定义如何对装在容器内的元素执行一个装在容器内的函数。
* Monad：定义如何对装在容器内的元素执行一个返回装在容器内的值的函数。

可见，我们一直在和容器，或者说上下文，打交道，对函数本身、入参或返回值在容器内还是容器外做讨论。大概你也猜到了，接下来要说的还是涉及容器的特性。没错，这回我们聊容器的嵌套，F[G[\_]]如何变化成G[F[\_]]。抽象地看，似乎不易理解这种变换有什么实际意义，还是拿案例说明吧。

从数据库内根据id列表批量查询对应记录是很常见的操作。
```scala
  case class User(uid: Int)

  //模拟耗时的查询
  def getUser(uid: Int): Future[User] = Future.successful(User(uid))

  val userIds = List(343, 8481, 8914)
  val allUsers: List[Future[User]] = userIds.map(getUser)
```

List[Future[\_]]让Future的等待变得很麻烦，需要逐一操作，如果能转换成Future[List[\_]]就方便多了，直接整体等待即可。利用Foldable，达成这点并不困难，下面代码的users即是Future[List[User]]，可直接用这个最终的Future做阻塞，等待异步结果。
```scala
import scala.concurrent.{Await, Future}
import scala.concurrent.duration._
import scala.concurrent.ExecutionContext.Implicits.global

scala> val users = userIds.foldLeft(Future.successful(List[User]()))((accum, uid) => {
    val user: Future[User] = getUser(uid)
    for {
      accum <- accum
      user <- user
    } yield accum :+ user
})
users: scala.concurrent.Future[List[User]] = Future(<not completed>)

scala> Await.result(users, 1.second)
res0: List[User] = List(User(343), User(8481), User(8914))
```

Foldable固然可以，但有不少脚手架代码，为了说明，我们再举一例：把指定数值转换成星星数，数值小0时认为是非法数据。类似地，对于Vector[Int]，和函数Int => Option[String]，本来在map后将得到Vector[Some[String]]，但我们希望的是Some[Vector[String]]。
```scala
def getStar(num: Int): Option[String] = if (num < 0) None else Some(num + " stars")

val res3 = Vector(1, 2, 3).foldLeft(Some(Vector()): Option[Vector[String]])((accum, item) => {
  val star = getStar(item)
  for {
    acc <- accum
    star <- star
  } yield acc :+ star
})
```
对比两个例子，不难发现，当要把F[A]转换为G[F[B]]时，大部分都是模板化的代码，不同点仅是：
* 累积值不同，但形式一致，都是一个空的数据结构G[F[\_]]。
* 合并函数不同，但签名一致，op都是A => G[B]。

op函数应作为参数传入，因为它本来就是个性化的业务逻辑。因此，问题仅剩下如何为不同的G提供相同的实现。如何成功解决这点，我们可以再把上述foldLeft模板代码再抽象成新的方法。幸运的是，Applicative具备这个能力。回忆下前面对它的介绍：
> Applicative是在Apply基础上添加pure。Applicative和Apply的关系类似于Monoid和Semigroup的关系。

借用pure把"裸值"装进容器里的能力。不论F具体是什么类型，我们都能把一个空的F装到G里作为折叠操作的初始累积值。如此一来，合并函数则变成要合并两个装在G内的值，Applicative的apply2恰好又能派上用场了。这样，二次抽象终于可以进行了，当然，以我们一贯的做法，又要换个新的名字：Traverse。

```scala
trait Traverse[F[_]] {
  def traverse[G[_], A, B](fa: F[A])(f: A => G[B]): G[F[B]]

  def sequence[G[_], B](fgb: F[G[B]]): G[F[B]] = traverse(fgb)(identity)
}
```
其中，sequence依赖于traverse，作用很明显，完成一次"穿越"，保持元素不变的同时，把里面的F和外面的G换个位置。

剩下的F我们暂时还无法提供统一的实现，但可以作个分类，相同性质的F归为一类，提供相同的Traverse实现，这里先按下不表。还是简单地以List为例，看看具体的Traverse实现：
```scala
scala> val listTraverse = new Traverse[List] {
  override def traverse[G[_] : Applicative, A, B](fa: List[A])(f: A => G[B]) = {
    val m = implicitly[Applicative[G]]
    fa.foldLeft(m.pure(List.empty[B])) { (accum, item) => m.apply2(accum, f(item))(_ :+ _) }
  }
}
listTraverse: Traverse[List] = $anon$1@5261d921

scala> listTraverse.traverse(List(1, 2, 3))(x => Some(x): Option[Int])
res1: Option[List[Int]] = Some(List(1, 2, 3))
```
隐式值m是为G创建Applicative的辅助工具，m的pure和apply2成为整体实现的核心。正如Monoid之于Foldable，Applicative之于Traversable（Traverse也称为Traversable）也有着重要意义。


由此反观Monoid和Applicative，你是不是对它们有了更深的认识呢？它们都擅长于合并容器内的对象，但从append和apply2的定义可以看出，Monoid本身就知道合并的逻辑，而Applicative需要外部定义。

也许你也发现了，随着讨论的深入，越来越多概念在不经意之间，发生了令人意外的关联。这是函数式思维学习过程的一大乐趣，概念之间并不是孤立的，而是连贯的、呼应的，成体系的。

读者不防思考，我们从Foldable抽象出的Traverse，能否反过来表达Foldable呢？它和Functor是否也存在某种关系呢？
