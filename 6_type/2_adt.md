## 代数数据类型

对于函数式编程，再怎么强调"代数推导"都不为过。回忆下编程和代数的映射关系，函数映射为创建对象的操作，集合内的对象映射为类型。函数是重点，函数的组合，我们在第一章做了完整介绍。函数并不是全部，经过前面的论证，我们现在完全有理由猜测，类型和与之对应的对象也有很多共同点。

且不看环、群、域等抽象化的结构，就看代数研究的最直观的对象：数字。我们知道，两个数字在加法或乘法的作用下，产生一个新的数字。那么，类型在加法或乘法——如果支持的话，的作用下，也应产生新的类型。就让我们看看这两种最简单，却又最常见的类型生类型的方法吧。

##### 加法类型

关于类型，什么样的组合操作能具有加法运算的效果？如果存在某种类型，要么是已知的值A，要么是已知的值B，那么它就有两种可能性，这不正是`1 + 1 = 2`吗？所以，我们把这种只能取其中一个值的类型称为**加法类型**。

Java的枚举类型表达的正是此意：
`enum QUALITY {GOOD, BAD}`。如果在此基础上再加1，则是`2 + 1 = 3`：`enum QUALITY {GOOD, MEDIUM, BAD}`。

Scala没有enum，但有`case`类实现相同效果。准确地说，是`sealed`修饰的`case`类，因为这有这样取值的可能性才是确定的。比如，下面的`ByteOrBool`类型，`ByteOrBool = Byte + Boolean`，总共有258种（256 + 2）种取值可能。
```scala
sealed trait ByteOrBool
case class ByteOfByteOrBool(v: Byte) extends ByteOrBool
case class BoolOfByteOrBool(v: Boolean) extends ByteOrBool
```

`Boolean`类型本身就是加法类型，要么是`true`要么是`false`。常用于封装返回值的`Either[A, B]`表示要么是A类型，要么是B类型，也是加法类型。值得注意的是，从集合角度看，加法类型对应的是"并查集"（Disjoint Union)，即使两个集合的元素完全相同，也视为不同的元素。如：
```scala
sealed trait BoolOrBool
case class Bool1(v: Boolean) extends BoolOrBool
case class Bool2(v: Boolean) extends BoolOrBool
```
我们知道`Boolean + Boolean = Boolean`，但因为是并查集，所以这是两个不同的`Boolean`，`BoolOrBool`的取值空间应为`{true1, false1, true2, false2}`，共4个可能的取值。

##### 乘法运算



##### 类型的运算
