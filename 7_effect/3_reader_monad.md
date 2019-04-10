## Reader Monad

>

以下场景想必读者不陌生：涉及到数据库，很多方法都需要用到数据库连接；很多方法都依赖从同一个文件中读取的配置；甚至，在一串函数调用链中，末尾函数需要的参数其它函数都没有用到，但参数的值却只能在链的开关给出，为了传递该参数，调用链上的所有函数都得添加该参数。比如，下面典型的web服务流程中，用户输入的内容在`Controller`收集，在`Service 3`使用，中间的`Service 1`和`Service 2`不得以也要加上该内容作为入参。

Controller -> Service 1-> Service 2 -> Service 3

这当然很丑陋，我们要把它变美，用`Reader Monad`。理解了`State Monad`的思路后，我们知道，可以把依赖对象参数化——作为函数的参数，在最后触发执行中才传入该参数，进而实现将依赖延迟的目的。`Reader Monad`的指导思想也是类似的，简单起见，我们以计算圆柱体的体积为例，介绍它的结构和使用。

从最直观的定义开始：
```scala
  // 面积，r为半径
  def area(pi: Double, r: Double): Double = pi * r * r
  // 体积，h为高
  def volume(pi: Double, r: Double, h: Double): Double = area(pi, r) * h
```  
`volume`本身并不依赖`pi`，只是简单地把它传递给`area`。所以，我们要把对`pi`的依赖推迟到真正需要用到它的那一刻，即把`pi`作为最后的触发函数`run`的参数：`volumeR(r, h).run(pi)`，`volumeR`是中间函数组合后的结果，具体定义接下来会推导出。

`run`以`pi`为输入，输出体积值，所以其签名是`Double => Double`，这就要求`volumeR`输出一个包含`run`方法的对象，我们用`case class`来定义该对象的类型：
```scala
case class Reader[E, A](run : E => A) {

}
```
那么，`area`和`volume`都应返回`Reader`，相应改造如下：
```scala
  def areaR(r: Double): Reader[Double, Double] = Reader { pi => pi * r * r}

  def volumeR(r: Double, h: Double): Reader[Double, Double] = areaR(r).map(a => a * h)
```
好了，`pi`从`areaR`和`volumeR`的入参中消失，变成`run`的入参，只有真正调用`run`时才会用到。但这个代码还不能运行，因为缺少`map`的定义。`map`把底面积的计算和乘以高的计算组合起来，你大概猜到了，`Reader`是一个`Monad`，没错，到此，可以给出其完整定义了：

```scala
case class Reader[E, A](run: E => A) {
  // Reader接受匿名函数，函数入参为类型E
  def flatMap[B](f: A => Reader[E, B]): Reader[E, B] = Reader { e => 
    val a = run(e)
    // f(a)得到Reader[E, B]，再执行其run，得到B
    f(a).run(e)
  }
  
  def map[B](f: A => B): Reader[E, B] = Reader { e => 
    f(run(e))
  }
}
```
改造完毕，我们来计算下底面半径为`3`，高为`2`的圆柱体的体积，`pi`就取`3.14`：
```
scala> val res = volumeR(3, 2).run(3.14)
res: Double = 56.519999999999996
```
