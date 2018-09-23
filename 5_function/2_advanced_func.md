# 鲜为人知的函数

> 变者，法之至也。——魏禧

**部分施用**和**柯里化**都有能力操纵函数或方法的参数数量。先看部分施用，函数cook可做*西红柿炒蛋*这道菜，根据菜谱，cook接受两个参数西红柿数量和蛋的数量。
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
像这样，把函数多个参数中的部分参数固定的过程，称为**部分施用**。部分施用的结果仍是函数，参数是原函数剩下的未固定的参数。固定了蛋的个数后，加多少个西红柿就让各个大厨自由发挥了：
```scala
scala> addTomato(4)
res0: Food = Food(Scrambled 3 eggs with 4 tomatoes)

scala> addTomato(3)
res1: Food = Food(Scrambled 3 eggs with 3 tomatoes)
```
可见，有了部分施用后，我们可以先抽象出一般性的函数(cook)，在此基础上再写出具体的函数(addTomato)。部分施用，使得在不复制代码的情况下，甚至不用额外定义def 
addTomato(num: Int) = cook(3, num)
，就得到了两个同时可用的函数。当然，没有任何理由限制哪几个参数可以被固定，所以固定西红柿数量，鸡蛋数量自由发挥也是可以的：val addEgg = cook(_: Int , 2)。

cook可以视为对一种厨艺的描述——把若干数量鸡蛋和西红柿变成一道菜的能力：(Int, Int) => 
Food。部分施用的过程，好比是加工半成品：Int => (Int => Food)，这个半成品蕴含另一个能力——把若干数量的西红柿直接变成一道菜。

由此，可以认为(Int, Int) => Food和Int => (Int => Food) 
在综合能力上是等效的。但是，通过把参数固定的方式实现这一转化，消弱了函数的通用性和灵活性。有没有一般化的方式，把多个参数的函数变换成单一参数（最初函数的第一个参数）的函数，并且返回一个接受余下参数的函数？有，它就是**柯里化**。
```scala
scala> val curriedCook = (cook _).curried
curriedCook: Int => (Int => Food) = scala.Function2$$Lambda$1455/2131088063@3779b701
```
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

既然(Int, Int) => Food和Int => (Int => Food)是等效的，括号并不影响最终效果，何不去掉括号？所以二者在形式上可以统一为Int => Int => 
Food。即，curriedCook可以声明为Int => Int => Food：
```scala
scala> val curriedCook: Int => Int => Food = (cook _).curried
curriedCook: Int => (Int => Food) = scala.Function2$$Lambda$1455/2131088063@6afe08bc
```

<div class="alert alert-success">
柯里化概念由美国数学家、逻辑学家Haskell Brooks Curry提出。著名的函数式语言Haskell便是以他的名字命名的。值得一提的是，名字中另两个单词Brook和Curry
也是两门编程语言。他对编程的贡献之大，可见一斑。
</div>
