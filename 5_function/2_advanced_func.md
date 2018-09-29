## 鲜为人知的函数

> 变者，法之至也。——魏禧

**部分施用**和**柯里化**都有能力操纵函数或方法的参数数量。先看部分施用，函数cook可做*西红柿炒蛋*这道菜，根据菜谱，cook接受两个参数：西红柿数量和蛋的数量。
```scala
scala> case class Food(material: String)
defined class Food

scala> def cook(egg: Int, tomato: Int): Food = Food("Scrambled " + egg  + " eggs with " + tomato + " tomatoes")
cook: (tomato: Int, egg: Int)Food
```
细化这道菜的烹饪过程，第一道工序是加蛋，第二道工序是加西红柿。为了保证不同厨师做出来的口味一致，我们把这一过程标准化，第一道工序统一加三个蛋，那么第二道工序可以表示为：
```scala
val addTomato = cook(3, _: Int)
addTomato: Int => Food = $$Lambda$1446/448233725@6b8683b8

```
像这样，把函数多个参数中的部分参数固定的过程，称为**部分施用**。部分施用的结果仍是函数，其参数是原函数剩下的未固定的参数。固定了蛋的个数后，加多少个西红柿就让各个大厨自由发挥了：
```scala
scala> addTomato(4)
res0: Food = Food(Scrambled 3 eggs with 4 tomatoes)

scala> addTomato(3)
res1: Food = Food(Scrambled 3 eggs with 3 tomatoes)
```
可见，有了部分施用后，我们可以先抽象出一般性的函数(cook)，在此基础上再写出具体的函数(addTomato)。部分施用，使得在不复制代码的情况下，甚至不用额外定义辅助函数`def addTomato(num: Int) = cook(3, num)`
，就得到了两个同时可用的函数。当然，没有任何理由限制哪几个参数可以被固定，所以固定西红柿数量，鸡蛋数量自由发挥也是可以的：`val addEgg = cook(_: Int , 2)`。

cook可以视为对一种厨艺的描述——把若干数量鸡蛋和西红柿变成一道菜的能力：`(Int, Int) => Food`。部分施用的过程，好比是加工半成品：`Int => (Int => Food)`，这个半成品蕴含另一个能力——把若干数量的西红柿直接变成一道菜。

由此，可以认为`(Int, Int) => Food`和`Int => (Int => Food)` 
在综合能力上是等效的。但是，通过把参数固定的方式实现这一转化，消弱了函数的通用性和灵活性。有没有一般化的方式，把多个参数的函数变换成单一参数（最初函数的第一个参数）的函数，并且返回一个接受余下参数的函数？有，它就是**柯里化**。

```scala
scala> val curriedCook = (cook _).curried
curriedCook: Int => (Int => Food) = scala.Function2$$Lambda$1455/2131088063@3779b701
```

<div class="alert alert-info">
柯里化概念由美国数学家、逻辑学家Haskell Brooks Curry提出。著名的函数式语言Haskell便是以他的名字命名的。值得一提的是，名字中另两个单词Brook和Curry
也是两门编程语言。他对计算机科学的贡献之大，可见一斑。
</div>

有了curriedCook，我们不必事先固定鸡蛋个数，拥有更多灵活性；反过来用它来定义部分施用函数，也非常方便。
```scala
scala> curriedCook(3)(4)
res2: Food = Food(Scrambled 3 eggs with 4 tomatoes)

scala> curriedCook(4)(3)
res3: Food = Food(Scrambled 4 eggs with 3 tomatoes)

scala> val addTomato = curriedCook(3)
addTomato: Int => Food = scala.Function2$$Lambda$1457/1287107889@2c49eefd
```
curriedCook，即柯里化后的cook，与众不同的一点是，它有两个参数列表，每个列表一个参数。

既然`(Int, Int) => Food`和`Int => (Int => Food)`是等效的，括号并不影响最终效果，何不去掉括号？所以二者在形式上可以统一为`Int => Int => 
Food`。即，curriedCook可以声明为`Int => Int => Food`：
```scala
scala> val curriedCook: Int => Int => Food = (cook _).curried
curriedCook: Int => (Int => Food) = scala.Function2$$Lambda$1455/2131088063@6afe08bc
```
理解柯里化，有种比较繁琐但很通俗的方式，借助闭包将其展开成以下形式：
```scala
val curriedCook: Int => Int => Food = {
  a: Int => {
      b: Int => cook(a, b)
  }
}
```
行文至此，不禁要问，柯里化有什么意义呢？柯里化很好地体现了一种解决问题的方式：把未知转化为已知。对于不同参数的函数，讨论和优化的方式不尽相同，利用柯里化，把函数完全变成“接受一个参数；返回一个值”的固定形式，攻克这一种形式，就能推广到做任意参数数量的情形，可谓四两拨千斤。

编程上的实用意义在于，作为第一等公民的函数，有了更灵活的调用方式。考虑一个对列表做过滤的函数，方式一：
```scala
scala> def filter[A](l: List[A], f: A => Boolean) = l.filter(f)
filter: [A](l: List[A], f: A => Boolean)List[A]

scala> filter(List(1, 2, 3, 4), {a: Int => a < 3})
res4: List[Int] = List(1, 2)
```
方式二：
```scala
scala> def curriedFilter[A](l: List[A])(f: A => Boolean) = l.filter(f)
curriedFilter: [A](l: List[A])(f: A => Boolean)List[A]

scala> curriedFilter(List(1, 2, 3)){_ < 3}
res5: List[Int] = List(1, 2)
```
显然，方式二更加清晰明了。所以当函数的最后一个参数是函数时，常常写成柯里化形式。
另外，当引入隐含参数时，就不得不使用柯里化形式了：
```scala
// 合法：
def plus(a: Int)(implicit b: Int): Int = a + b
// 非法：
def plus(a: Int, implicit b: Int): Int = a + b
```

<div class="alert alert-info">
我们已经两次碰到了下划线"_"，在Scala里，下划线有很多含义和用途。<br/>
<ul>
<li>cook(3, _: Int)和{_ < 
3}两个例子中的下划线都表示函数参数占位符，是最常见的用途。如果有多个参数，则按顺序依次对应，如List(1, 2, 3).foldLeft(0){_ + _}。<br/>
</li>
<li>
(cook _)中的下划线表示Eta expansion，注意，无论cook有多少个参数，都只是一个下划线。cook是一个“方法”，方法不像Int、String
或其它类型的值那样独立存在，它必须依附于所在的结构，如class
、object 或trait。而作为一等公民的函数可以像其它类型的值一样传递、返回或保存在List之类的集合中。所以当需要引用某方法时，应先将其扩展为函数，一般情况下，编译器会自动完成这种拓展，其它时候（如本文的情况）就要手动拓展。这种拓展好比再包裹一层调用，所以(cook _)也可以写成cook(_, _)。
</li>
</ul>
 </div>
