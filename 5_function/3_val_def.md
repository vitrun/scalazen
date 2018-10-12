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
注意，val定义函数时的语法，转换符`=>`前面是输入参数，后面是函数体，缺省的返回值类型由编译器自动推导得到。val定义值的常用形式是`variableName: type = value`，定义函数也可以用这种形式，函数值和其它类型值并无差异：
```scala
scala> val echo: Int => Int = {a => a}
echo: Int => Int = $$Lambda$1395/1361950369@5d38011e

scala> val f: (Any) => Boolean = {
  case i: Int => i % 2 == 0
}
f: Any => Boolean = $$Lambda$1527/266267045@3eb04c79

scala> List(3, 4).filter(f)
res0: List[Int] = List(4)
```

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
res1: Int = 2
```
与方法pk，函数也存在力有不逮的时候，方法可以直接使用泛型：
```scala
def len[A, B](a: A, b: B): Int = {
  a.toString.length + b.toString.length
}
```
编译器却会报怨定义函数时使用泛型：`val len[A, B] = (a: A, b: B) => a.toString.length + b.toString.length`。绕过去的方式是先定义一个实现了Function2接口的类，再创建该类的实例，和前面用Function1接口定义echo函数类似，繁琐又不常用，按下不表。

以上，val用于定义函数，def用于定义方法，二者分工明确。可能会让人迷惑的是，def也可以直接用于定义函数，`def echo: Int => Int = {a => a}`也是合法的。姑且把两种方式定义的函数称为"val函数"和"def函数"。二者的差异比较微妙，val函数在定义时就会求值；而def函数在调用时才触发求值，REPL的反应证明了这点：
```scala
scala> val even: Int => Boolean = ???
scala.NotImplementedError: an implementation is missing

scala> def even: Int => Boolean = ???
even: Int => Boolean

scala> even
scala.NotImplementedError: an implementation is missing
```
当然，这和人们对val、def一贯的印象是一致的，除非加了lazy修饰，否则val是立刻被求值的，而def
定义方法并不意味着方法立刻执行，在实际调用时才会执行。基于此，便不难得到两种求值策略：val
函数后续调用时不再重新求值，总是返回定义时得到的值，而def函数每次调用都重新执行，得到的值可能不一样（视函数逻辑而定）。
请试试把test前头的val改为def，看看两次调用test的结果有什么不同。
```scala
val test: () => Int = {
  val r = util.Random.nextInt
  () => r
}

test()
test()
```

函数的参数形式也会影响求值策略，试看以下两个函数，注意参数x的不同定义方式：
```scala
def callByValue(x: Int) = {
  println("x1=" + x)
  println("x2=" + x)
}

def callByName(x: => Int) = {
  println("x1=" + x)
  println("x2=" + x)
}
```
前者是传值（by-value），后者是传名（by-name）。 在终端内分别执行`callByValue(something)`和`callByName(something)`，对比结果即可发现二者的区别。
```scala
def something() = {
  println("calling something")
  1
}
something: ()Int
```
可以简单地认为：
* 传值参数类似于val定义，当绑定到函数时，参数体一次性完成求值。
* 传名参数类似于def定义，当在函数内部每次使用到时，都对参数体进行一次求值。

下面这个例子，特别能说明传名参数的意义，如果把testCondition改为传值，whilst将陷入死循环（这里为了演示不得不改变i，函数式编程杜绝var这种直接修改的做法）。
```scala
def whilst(testCondition: => Boolean)(codeBlock: => Unit) { 
  while (testCondition) {
    codeBlock
  }
}

var i = 1
whilst(i < 3) {
  println(i)
  i += 1
}
```
但刻意提“传值“、“传名”的概念有故弄玄虚之嫌。换个视角解读这两种参数形式，会发现它们在求值策略上的区别是再自然不过的。`Int`就是个平常的值类型，绑定函数时求出该值，这是一次性操作；而`=>Int
`可以理解为不需要参数同时返回值为Int的函数，所以此时绑定的是作为一等公民的函数，每次使用到时就执行一次，正是我们对函数的一贯预期。
