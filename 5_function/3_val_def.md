## 是函数，也是值
> 色不异空，空不异色；色即是空，空即是色。——《般若波罗蜜多心经》

提到函数时，有时候称之为"方法"，有时候又称之为"函数"；有时候用def定义（如cook），有时候又用val定义（如curriedCook）。一般情况下，这两种形式可以混用，并不需要刻意区分。但需要不需要区分和能不能区分是两个层次，当我们以认真的表情来审视二者的区别，收获的不光是这个问题的答案，而是关于函数的另一番结论。

首先，严格来说，"用def定义函数"的说法并不准确，def定义的是"方法"而不是"函数"，所以它具有以下特点：
* 必须定义在类或对象内。
* 可以访问所在类的的成员。
* 隐式地引用了所在实例，如List的map方法，用个隐含的指向List对象的引用。

事实上，def定义的方法会翻译成Java类的类似方法。有兴趣的同学可以把这个DefFunc类编译成class文件，再反编译成java代码，便很容易看出形如`public int echo(int);`的java版本。
```scala
class DefFunc {
  def echo(a: Int): Int = a
}
```
val定义的函数，其实是对函数接口的实现，根据参数数量的不同，Scala提供了Function0、Function1……一直到Function22。
```scala
class ValFunc {
  val echo = (a: Int) => a
}
```
注意，val定义函数时的语法，转换符`=>`前面是输入参数，后面是函数体，缺省的返回值类型由编译器自动推导得到。

对ValFunc执行类似DefFunc的编译和反编译过程，便会发现echo是一个实现了Function1接口的匿名类的实例，和普通的val用法 ，如`val anInt = 3`，并无二致。Function系统接口定义的apply方法，是真正的逻辑实体，`echo(3)`不过是`echo.apply(3)`的语法糖。既然echo是Function1接口的实例，那么以下繁琐的写法是等效的：
```scala
val echo = new Function1[Int, Int] {
  def apply(a: Int) = a
}
```
可见，相比于Int、String、以及自定义类型的值，函数并没什么特别。作为一等公民，它与其它对象一样，有内置方法，如compose、andThen，内置以及，如compose、andThen，内置以及，如compose、andThen，内置以及前面提到过的curried等，还可以作为参数传给其它函数。这点是方法所不具备的，下面写法之所以成立，是编译器自动对方法做了Eta转换：`reEcho(2, echo)`相当于`reEcho(2, echo _)`。
```scala
scala> def reEcho(a: Int, f: Int => Int): Int = f(a)
reEcho: (a: Int, f: Int => Int)Int

scala> reEcho(2, echo)
res0: Int = 2
```
与方法pk，函数也存在力有不逮的时候，方法可以直接使用泛型：
```scala
def len[A, B](a: A, b: B): Int = {
  a.toString.length + b.toString.length
}
```
编译器却会报怨函数使用泛型，这种写法无法通过：`val len[A, B] = (a: A, b: B) => a.toString.length + b.toString.length`。绕过去的方式是先定义一个实现了Function2接口的类，再创建该类的实例，和前面用Function1接口定义echo函数类似，繁琐又不常用，按下不表。



