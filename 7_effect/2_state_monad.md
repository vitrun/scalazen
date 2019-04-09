## State Monad

>

#### 状态管理

状态管理在程序中极其常见，有简单的二元状态，如开关的开启和关闭也有多元的状态，如人的情绪有快乐、沮丧、愤怒、伤心等，也有无限的状态，如计算器的计算结果。状态的切换过程可以描述为：

 \[前状态] + \[事件/增量] = \[后状态]

为了得到新状态，必须知道原状态和作用于它身上的事件。这似乎是很简单的事，下面就以加法计算器为例，说明如何管理状态的切换，过程有什么潜在的问题，以及如何规避。

用`state: Int`的计算结果表示计算器状态。加法运算为：
```scala
// 为了便于区分：
type TOTAL = Int
type DELTA = Int
def add(prevState: TOTAL, digit: DELTA): TOTAL = {
     prevState + digit
}
```
那么，对多个数求和的过程可表示为：
```scala
object AddCalculator extends App {
  val state0 = 0
  val state1 = add(state0, 2)
  val state2 = add(state1, 9)
}
```
这段代码的问题显而易见，每次`add`都要带上之前的`state`，代码重复性高，写起来繁琐，又容易出错。我们需要一种更聪明的方式来组合上述语句。如果每次`add`只传入增量数，`state`能自动携带上，该有多好，就像这样：
```scala
for {
  _ <- add(0)
  _ <- add(2)
  total <- add(9)
} yield total
```
接下来我们将一步步构造出这个形式。首先，改造`add`方法，让它不仅返回新的运算结果，还返回改变了多少：
```scala
// 返回新的运算结果和当前改变的量，并使用两个参数列表
def add(delta: DELTA)(total: TOTAL): (TOTAL, DELTA) = (total + delta, delta)

// 克里化
def add_curried(delta: DELTA): TOTAL => (TOTAL, DELTA) = t => (delta + t, delta)
```
克里化之后的语句，看上去有点臃肿，为了简化，定义类型`TRN`。
```scala
// 简化，用函数携带状态量TOTAL
type TRN = TOTAL => (TOTAL, DELTA)

def plus(delta: DELTA): TRN = t => (delta + t, delta)
def minus(delta: DELTA): TRN = plus(-delta)
```
`TRN`其实从另一个视角表达了状态的切换过程：
\[后状态] = \[前状态] + \[增量]


#### State Monad
这简单到似乎没什么意义，把`1 + 2 = 3`表达成`3 = 1 + 2`而已。但关键点正在于这个简单的转换。之前我们说过，函数式编程可以称为面向表达式编程，执行的过程即是规约化表达式得到最终值的过程。举个例子：初始值为0，依次进行`+1`, `-2`, `+3`，可以把初始值先放一边， 规约化中间的运算，即把`1 - 2 + 3`规约化为`2`，然后再作用于初始值，即：`0 + 2 = 2`。

按此思路，当把若干个`add`和`minus`组合在一起的时候，其实是在组合以`delta`为输入以`TRN`为输出的函数，此时`total`可以视为`TRN`函数内部的一个常量。接下来的问题是，如何组合（规化）它们。我们需要把一个`DELTA => TRN`和一个已有的`TRN`组合起来，得到新的`TRN`，这让人联想起`flatMap`，让我们试着构造一个：
```scala
def flatMap(transit: TRN)(f: DELTA => TRN): TRN = total => {
  // transit是关于total的函数，返回(TOTAL, DELTA)
  val (newTOTAL, delta) = transit(total)
  // f(delta)得到TRN，再同上执行
  f(delta)(newTOTAL)
}
```
再来看`type TRN = TOTAL => (TOTAL, DELTA)`，除了是关于`TOTAL`的函数，是不是也是可以盛装`DELTA`的容器？没错，这是一个夹带了`TOTAL`的容器，而这正是我们想要的。现在，让我们重新编写开头`state0`，`state1`，`state2`三步计算过程：
```scala
val x: TRN = plus(0)
val y: TRN = flatMap(x)(_ => plus(2))
val z: TRN = flatMap(y)(_ => plus(9))

println(z(0))
```
`x, y, z`只是在组合、规化中间计算过程，得到函数`z`，`println`时再代入初始状态`0`。我们没有把中间状态一路反复携带，但书写依然非常繁琐。好在这个问题非常容易解决，我们有语法糖，把这串代码改写成面向对象的风格：
```scala
// 用case class表述state monad
case class State(run: TRN) {
  def flatMap(f: DELTA => State): State = State {
    total =>
      val (newTotal, delta) = run(total)
      f(delta).run(newTotal)
  }

  def map(f: DELTA => DELTA): State = State {
    total =>
      val (newTotal, delta) = run(total)
      (newTotal, f(delta))
  }
}
```
用`State`这个`Monad`封装之后，就可以用`for`语句了：
```scala
def newPlus(num: Int): State = State {
  t => (t + num, num)
}

def newMinus(num: Int): State = newPlus(-num)
val res: State = for {
  _ <- newPlus(0)
  _ <- newPlus(2)
  _ <- newMinus(1)
  _ <- newMinus(-1)
  a <- newPlus(9)
  } yield a

println(res.run(0))
```
做个简单的总结。我们转变对状态切换的理解角度，通过定义`TOTAL => (TOTAL, DELTA)`，或者，更一般性地写成`S => (S, A)`，实现了不同函数共享状态的机制（无论组合多少个`newPlus`或`newMinus`，期间传递的`total`不变）。然后，定义`State Monad`，方便地组合这些函数，避免了容易出错的手动传递方式。当然，从定义上很容易看出，`State`本身并不保存状，状态`s`和值`a`只是作为函数的参数在调用链中传递。另一个注意点是，对于`State`（以及其它`Monad`），在最后触发调用前，比如执行`res.run(0)`之前，都只是在描述程序将起到的效果，实际并没有改变什么。整个过程可以看成是在写一个完整的函数，最终执行时，这个函数把一个初始状态变成最终状态。这和指令式编程中碰到一行即可认为执行了这行，有非常大的区别。

