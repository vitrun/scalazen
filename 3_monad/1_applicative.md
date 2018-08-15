# 容器里的函数

介绍Functor时说到，类型构造器F可以理解为容器，发挥下想象力，这个容器既然可以装类型，能不能装函数呢？显然可以，比如下面的ff：
```scala
scala> val f: Int => String = x => x.toString + " stars"
f: Int => String = $$Lambda$1674/1693304652@4a119605

scala> val ff: Option[Int=>String] = Some(f)
ff: Option[Int => String] = Some($$Lambda$1674/1693304652@4a119605)
```
或者，顺着之前Functor的思路：之前定义的map接受的函数f一直是一个入参，一个返回值。如果是多个入参，比如下面的f2，执行map后得到的也是装在容器里的函数：

```scala
scala> val f2: (Int, Int) => Int = (x, y) => x+y
scala> Some(3).map(i => (j:Int) => f2(i, j))
res0: Option[Int => Int] = Some($$Lambda$1702/547090744@27816139)
```


