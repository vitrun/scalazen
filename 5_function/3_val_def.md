## 是函数，也是值
> 色不异空，空不异色；色即是空，空即是色。——《般若波罗蜜多心经》

提到函数时，有时候称之为"方法"，有时候又称之为"函数"；有时候用def定义（如cook），有时候又用val定义（如curriedCook）。一般情况下，这两种形式可以混用，并不需要刻意区分。但需要不需要区分和能不能区分是两个层次，当我们以认真的表情来审视二者的区别，收获的不光是这个问题的答案，而是关于函数的另一番结论。

首先，严格来说，"用def定义函数"的说法并不准确，def定义的是"方法"而不是"函数"，所以它具有以下特点：
* 必须定义在类或对象内。
* 可以访问所在类的的成员。
* 隐式地引用了所在实例，如List的map方法，用个隐含的指向List对象的引用。

事实上，def定义的方法会翻译成Java类的类似方法。执行`scalac -Xprint:all DefFunc.scala`编译这个DefFunc类：
```scala
class DefFunc {
  def echo(a: Int): Int = a
}
```
再执行`javap DefFunc`，得到：
```java
Compiled from "DefFunc.scala"
public class DefFunc {
  public int echo(int);
  public DefFunc();
}
```
很明显，bytecode中的方法echo就是DefFunc中用def定义的方法。
