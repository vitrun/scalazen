## State Monad

>

#### 状态管理

状态管理在程序中极其常见，有简单的二元状态，如开关的开启和关闭也有多元的状态，如的人情绪有快乐、沮丧、愤怒、伤心等，也有无限的状态，如计算器的计算结果。状态的切换过程可以描述为：

 \[前状态]---<事件>--->\[后状态]

为了得到新状态，必须知道原状态和作用于它身上的事件。这似乎是很简单的事，下面就以加法计算器为例，说明如何管理状态的切换，过程有什么潜在的坑，以及如何规避。

用`case class CalculatorState(sum: Int)`表示计算器状态。加法运算为：
```scala
def add(prevState: CalculatorState, digit: Int): CalculatorState = {
    CalculatorState(prevState.sum + digit)
}
```
那么，对多个数求和的过程可表示为：
```scala
object AddCalculator extends App {
  def add(prevState: CalculatorState, digit: Int): CalculatorState = {
    CalculatorState(prevState.sum + digit)
  }

  val state0 = CalculatorState(0)
  val state1 = add(state0, 2)
  val state2 = add(state1, 9)
}
```
这段代码的问题显而易见，每次`add`都要带上之前的`state`，代码重复性高，又容易写错。如果能自动带上`state`，每次`add`只传入增量数，该有多好，就像这样：
```scala
for {
  _ <- add(1)
  _ <- add(2)
  total <- add(9)
} yield total
```
那就让我们从这段代码出发，逆向推导出实现这个写法所需的条件。
