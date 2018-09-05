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

List[Future[\_]]让Future的等待变得很麻烦，需要逐一操作，如果能转换成Future[List[\_]]就方便多了，直接整体等待即可。利用Foldable，达成这点并不复杂，下面代码的users即是Future[List[User]]，可直接执行Await.result。
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
