## 代数数据类型

对于函数式编程，再怎么强调"代数推导"都不为过。回忆下编程和代数的映射关系，函数映射为创建对象的操作，集合内的对象映射为类型。函数是重点，函数的组合，我们在第一章做了完整介绍。函数并不是全部，经过前面的论证，我们现在完全有理由猜测，类型和与之对应的对象也有很多共同点。

且不看环、群、域等抽象化的结构，就看代数研究的最直观的对象：数字。我们知道，两个数字在加法或乘法的作用下，产生一个新的数字。那么，类型在加法或乘法——如果支持的话，的作用下，也应产生新的类型。就让我们看看这两种最简单，却又最常见的类型生类型的方法吧。

##### 用加法组合类型

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
 
##### 用乘法组合类型

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
//求面积
def area(s: Shape): Double = s match {
  case Triangle(a, b, c) =>
    val p = (a + b + c) / 2
    sqrt(p * (p - a) * (p - b) * (p - c))
  case Circle(r) => Pi * pow(r, 2)
}
```

##### 其它方法组合类型

代数里的运算可不只加法和乘法，所以难免会问除法、指数等运算是否也可以用于“制造”类型呢？答案是肯定的，比如，很有意思的是，我们讨论了这么久的函数，其实就是指数类型。函数`def func(b: 
B): A`是类型B到类型A的映射关系，假设`B`是一个包含3个元素的集合，`A`是2个元素的集合，则对于`B`内的每个元素都有`A`内的两个潜在映射对象，这种映射是独立互不干扰的，因此共有`2
^3`共8种可能，表达的正是指数关系。啊哈，我们又多了一视角理解函数。

根据[柯里-霍华德同构](https://zh.wikipedia.org/wiki/%E6%9F%AF%E9%87%8C-%E9%9C%8D%E5%8D%8E%E5%BE%B7%E5%90 
%8C%E6%9E%84)，在程序的类型中，有集合论中不少概念的影子。只是它们并不像加法、乘法类型那么常用。

##### 类型的运算

借着对代数的熟悉，我们挖掘了不少对程序的新鲜认识。受此鼓舞，让我们继续大胆探索，看看诸如“任何数和0相加都不变”、“交换率”、“结合率”等概念对类型是否适用。

首先，回忆上节的内容，我们知道，`Nothing`类型表示什么都没有，对应了“0”的概念；`unit`有且仅有一个值：`()`，对应了“1”的概念（`null`也是唯一对象，但指代的是“引用”，有别于“值”）。

简洁起见，用波浪号`~`表示“等价”关系，一种表示效果上相同但定义又不相同的关系。比如`(a, unit) ~ a`，`Tuple`是乘法类型，表示同时拥有，而`unit`仅有一个固定值，当你说"同时拥有一个`a`和一个固定的`() `时，和说“拥有一个`a`”传达的信息量是一样的。用熵来理解就是，我们并不需要额外的空间来存储不变的信息。

有了这个铺垫后，让我们来看一些常见运算的对比：

* `a * 1 = a`，`(A, unit) ~ A`。
* `a + 0 = a`，`Either[A, Nothing] ~ A`，无法创建`Nothing`的实例，所以虽然声称可以，但你实际上永远不可能选择它，只能是`A`。
* `a * 0 = 0`，`(A, Nothing) ~ Nothing`，同时拥有`A`和一个不存在的事物，结果只能是一场空。
* `a * (b + c)`，`(A, Either[B, C]) ~ Either[(A, B), (A, C)]`，`A`是必须的，`B`和`C`任选一个，那么要么是`A-B`组合，要么是`A-C`组合。
* `1 + 1 = 2`，`Either[True, False] ~ Boolean`，注意是并查集，我们把左边的`Unit`叫作`True`，右边的叫作`False`。
* `1 + a`，`Either[Unit, A]`。
* `a ^ 0 = 1`，`Nothing => A ~ ()`，这是个“荒唐”的函数，因为没有人可以调用它，因此无论怎么实现它，都是等效的实现。
* `1 ^ a = 1`，`A => Unit ~ ()`，无论`A`有多少种取值可能，它们都只有一个归宿：映射到唯一的值`()`。
* `a ^ 1 = a`，`Unit => A ~ A`，源头`b`就一个值`()`，有多少映射可能取决于终点`A`。
* `b ^ (a + c) = b ^ a * b ^ c`，`Either[A, C] => B ~ (A => B, C => B)`。
* `(a ^ b) ^ c = a ^ (b ^ c)`，`C => (B => A) ~ (C => B) => A`， 咦，这不就是**柯里化**吗？
* `(a * b)^c = a^c * b^c`，`C => (A, B) ~ (C => A, C => B)`。


这就是代数数据类型，知道这些又有什么用呢？似乎没有直接的帮助，它提供的是新的观察视角和建模方式，当正面进攻受挫时，借道代数直取后方，不失为妙计一个。