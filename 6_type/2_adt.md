## 代数数据类型

对于函数式编程，再怎么强调"代数推导"都不为过。回忆下编程和代数的映射关系，函数映射为创建对象的操作，集合内的对象映射为类型。函数是重点，函数的组合，我们在第一章做了完整介绍。函数并不是全部，经过前面的论证，我们现在完全有理由猜测，类型和与之对应的对象也有很多共同点。

且不看环、群、域等抽象化的结构，就看代数研究的最直观的对象：数字。我们知道，两个数字在加法或乘法的作用下，产生一个新的数字。那么，类型在加法或乘法——如果支持的话，的作用下，也应产生新的类型。就让我们看看这两种最简单，却又最常见的类型生类型的方法吧。

##### 加法类型

关于类型，什么样的组合操作能具有加法运算的效果？如果存在某种类型，要么是已知的值A，要么是已知的值B，那么它就有两种可能性，这不正是`1 + 1 = 2`吗？所以，我们把这种只能取其中一个值的类型称为**加法类型**。

Java的枚举类型表达的正是此意：
`enum QUALITY {GOOD, BAD}`。如果在此基础上再加1，则是`2 + 1 = 3`：`enum QUALITY {GOOD, MEDIUM, BAD}`。

Scala没有enum，但可以用`case`类实现相同效果。你可能无意识地使用了加法类型，比如，下面的`ByteOrBool`类型，由`Byte`和`Boolean`两种类型组合而成，`ByteOrBool = Byte + Boolean`，**新类型可能的取值总数是两个组成类型的取值总数之和**，共有258种（256 + 2）种取值可能。
```scala
sealed trait ByteOrBool
case class ImByte(v: Byte) extends ByteOrBool
case class ImBool(v: Boolean) extends ByteOrBool
```

`Boolean`本身就是加法类型，要么是`true`要么是`false`。常用于封装返回值的`Either[A, B]`表示要么是A类型，要么是B类型，也是加法类型。值得注意的是，从集合角度看，加法类型对应的是"并查集"（Disjoint Union)，即使两个集合的元素完全相同，也视为不同的元素。如：
```scala
sealed trait BoolOrBool
case class Bool1(v: Boolean) extends BoolOrBool
case class Bool2(v: Boolean) extends BoolOrBool
```
我们知道`Boolean + Boolean = Boolean`，但因为是并查集，所以这是两个不同的`Boolean`，`BoolOrBool`的取值空间应为`{true1, false1, true2, false2}`，共4个可能的取值。

观察到，通过加法得到的复合类型，表达的含义是“XXX是一种YYY”。`ByteOrBool`的值，要么是`ImByte`，要么是`ImBool`；`BoolOrBool`要么是`Bool1`，要么是`Bool2`。两个例子均从`sealed trait`继承，因为要对YYY的可能性做“穷举检验”，必须在这个文件内完整定义所有子类，任何其它地方不能再做扩展。因此，在使用模式匹配（Pattern Matching）处理加法类型弄，编译器可校验是否针对所有情况做了判断，防止编码时有所遗漏。
 
模式匹配是“解构”加法类型的有力武器。
```scala
sealed trait Shape
case class Circle(radius: Double) extends Shape
case class Triangle(a: Double, b: Double, c: Double) extends Shape

def isRound(s: Shape): Boolean = s match {
  case Circle(_) => true
  case _ => false
}
```
 
##### 乘法类型

若**新类型可能的取值总数是两个组成类型的取值总数之积**，则为**乘法类型**。乘法类型也是常见的类型，如：`case class Contact(addr: String, phone: 
Int)`。 `Contact`类型要求`String`和`Int`同时存在，而不是加法类型那样只选择其中一个。 所以，它的取值空间是各自的乘积，`Contact = String * 
Int`，有无限多个。乘法类型，表达的是“XXX包含YYY”，`Contact`必须有一个`String`和一个`Int`。又如三角形必须包含三条边的长度，因此用乘法类型建模：`case class 
Triangle(a: Double, b: Double, c: Double)`。

最简单的乘法类型，当属Scala的元组（Tuple）。实际的复合类型取决于元素的类型，比如`(99, "Scala")`是`Tuple2[Int, String]`。类型的顺序是不能随意调换的，虽然`Tuple2[Int, String]`和`Tuple2[String, Int]`承载的信息完全相同，但二者并不是相同的类型。这很自然，符合我们日常的习惯，无法把`Tuple2[Int, String]`类型的值作为函数`def func(v: Tuple2[String, Int])`的参数。

`Tuple`和`case class`表达的乘法类型在信息上并无区别，只不过前者简洁明了，合适简单直观的场景，后者用有业务语义的名字命名组成部分和其自身，合适语义比较晦涩的复杂场景。

回头看`Shape`，会发现它是一个既有加法类型又有乘法类型的复合类型。`Shape`要么是一个`Circle`，要么是一个`Triangle`，二选一，是一个加法关系；而`Triangle `本身同时包含三个`Double`，是一个乘法类型。

结构模式匹配，可以用简洁的代码灵活地解构这种复合情况：
```scala
import math.{sqrt, pow, Pi}
def area(s: Shape): Double = s match {
  case Triangle(a, b, c) =>
    val p = (a + b + c) / 2
    sqrt(p * (p - a) * (p - b) * (p - c))
  case Circle(r) => Pi * pow(r, 2)
}
```

##### 类型的运算
