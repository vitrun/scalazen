# 从求和函数说起

让我们从一个非常简单的求和函数说起。下面函数对整数列表求和：
```scala
scala> def sum(xs: List[Int]): Int = xs.foldLeft(0){_ + _}
sum: (xs: List[Int])Int

scala> sum(List(1, 2, 3, 4))
res0: Int = 10
```

