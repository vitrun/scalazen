## 参数化类型

Scala强大的类型系统拥有丰富的表现力。表现力体现在用更少的代码表达更多的含义，或者说，消灭重复代码。有很多操作是和类型无关的，比如，`def append(l: List[Int], e: 
Int): List[Int]`、`def size(l: List[Int]): 
Int`，不论`List`里放的是`Int`、`String`还是其它类型，`append`和`size`的操作都是一样的。那么，我们希望有与元素类型无关的通用的代码统一表达。类似于Java
的的泛型，Scala的可以在`class`或`trait`定义中使用参数化类型，如：
```scala
case class Pair[A, B](fst: A, snd: B)
```
显然，相比于具体类型，参数化后函数对类型知道得更少了，这似乎对编码有点不利，但反直觉的是，我们反而从这种更加抽象的类型上推导出更多关于函数的信息。当然，这有个前提，就是我们讨论的是具有确定性、不带副作用的纯函数。

事实上，在认证类型可用于逻辑推导时，我们已经说明了这点，举例代码用到的`State[S, 
A]`正是参数化类型。这里再举一个更简单的例子，看下面的函数，名字“神秘”得让人猜不透，又没有文档、没有实现代码，这是故意的，就是让你只从参数判断函数语义。
```scala
def mysteryFunc[A, B](p: Pair[A, B]): A = ???
```
你也许会说这无从判断。但事实上是可以的，如果——再次强调，是纯函数的话。试分析“神秘”函数，它接受一个`Pair`类型，返回返回类型与`Pair
`的左边类型相同。首先，它不可能返回右值，因为类型不符合；也不可能无中生有，生生造一个`A
`类型的值并返回，正是因为这是个参数化的类型，函数无从知晓类型的细节，所以无法构建其实例；也无法从别处获取，如果从磁盘、数据库读取，那“神秘”函数必然要接受一个文件句柄或数据库连接，但它没有。所以，这么排除下来，“神秘”函数的逻辑就只能是返回`Pair`的左值。

与之对比的是，如果知道了`A`、`B`具体是什么类型，比如`Pair[Int, 
Byte]`，我们反而无法得到这个结论。因为除了返回左值，还可能是直接忽略之，返回一个`Int`常量，甚至是和右值做一定计算后得到的值。

少而抽象的信息，得到了明确、具体的结论；多而具体的信息，得到的却是模糊的结论，这正是反直觉的地方。原因在于，类型参数化完全是基于类型系统的逻辑，而后者在具象了之后，给予了程序员更大的发挥空间，还依赖了人的“理智”。借用区块链的时髦语言，前者是“去信任”的，不存在信任不信任人的问题，是数学奠定了信任的基石。可见，类型参数化是一强大的抽象工具，消灭脚手架代码，消灭人的不理智。

需要指出的是，并不是越抽象越好，以更少量的代码提供更高的灵活性，看起来能更好的解决问题，但往往更难阅读和理解。Scala推崇“Least Power”原则：如果存在多个解决方案，选择刚好能解决问题的那个，等到有明确需求再重构、泛化不迟。

考虑到Scala的同时支持"OO"和"FP"两种范式，类型系统需要同时应对类型的层次性和多态性，带来边界和型变两个话题。

##### 边界

Scala用边界（`>:`和`<:`）来规定参数化类型的变化范围，进而限制多态性，如`trait Box[F <: 
Fruit]`，`Box`可以存放所有`Fruit`的子类型。还记得吗，所有类型的上限都是`Any`，下限都是`Nothing`，引用类型的上限是`AnyRef`，下限是`Null
`。比如，值不可为`null`的类型可以写成`trait Box[F >: Null]`。

##### 型变

当参数化遇到继承时，引发的问题是：如果`B extends A`，那么`Box[B]`和`Box[A]`有什么继承关系吗？这涉及了复杂类型与其参数类型在继承关系上的影响，称之为“型变”，有三种可能：协变、逆变和不变。以水果和装水里的盒子为例，
```scala
class Fruit
class Apple extends Fruit
```
三种型变关系可以表达为：
![variance](/imgs/variance.png)
* **协变（Covariance）**，参数化类型的继承方向与类型参数一致，用`+`表示，对于`Box[+F]`，这是合法的：`val r: Box[Fruit] = new Box[Apple]{}`。
* **逆变（Contravariance）**，参数化类型的继承方向与类型参数相反，用`-`表示，对于`Box[-F]`，这是合法的：`val r: Box[Apple] = new Box[Fruit]{}`。
* **不变（Invariance）**，参数化类型的继承方向与类型参数无关，即`Box[F]`，此情形下，上面两个赋值都是非法的。

为什么要提出型变，它有什么作用呢？根本性的原因是我们想“少写代码多做事”，追求更高的语言表现力。继承关系也能提高表现力，但代价是增加类之间的耦合程度。为了缓解这个问题，使用继承时应遵循“里氏替换原则”：所有引用基类的地方必须能透明地使用其子类的对象。这要求当重载或实现父类的方法时，
* 方法的前置条件（即方法的入参）和父类的一样或更宽松。
* 方法的后置条件（即方法的返回值）和父类的一样或更严格。

当我们对扩展`Box`时，兼顾`Box`、`Fruit`的继承关系和里氏原则，当`Fruit`位于返回值位置时，应该使用“协变”或“不变”。下面的例子可以通过类型检查，因为子类`AppleBox`的`get`返回类型`Apple`比父类的`Fruit`更严格。
```scala
class Box[+F] {
  def get: F = ???
}

class FruitBox(fruit: Fruit) extends Box[Fruit]

class AppleBox(orange: Apple) extends Box[Apple] {
  override def get: Apple = orange
}
```
当`Fruit`作为方法入参时，应使用“逆变”或“不变”：
```scala
class Box[-F <: Fruit] {
  def contains(something: F): Boolean = ???
}

class FruitBox(fruit: Fruit) extends Box[Fruit]

class AppleBox(orange: Apple) extends Box[Apple] {
  override def contains(something: Apple): Boolean = ???
}
```
理解上比较反直觉的地方在于，`Box`的`contains`方法接受的已经是最宽松的输入`F`了，不存在属于`Fruit`但又比`F`更宽松的类型。所以得“逆向”思考，把`AppleBox`视为`FruitBox`的父类。这样，子类的`contains`方法接受比父类更宽松的入参，就符合里氏原则了。

问题来了，如果同时存在`get`和`contains`方法，即既要真协变，又要逆变，怎么办？下面例子的`Box`就是这种情况，却是类型安全的，没有引发编译器的报怨：
```scala
class Box[+F <: Fruit] {
  def get: F = ???
  def contains[T >: F](something: T): Boolean = ???
}

class FruitBox(fruit: Fruit) extends Box[Fruit]

class AppleBox(apple: Apple) extends Box[Apple] {
  override def get: Apple = apple
  override def contains[Apple](something: Apple): Boolean = ???
}
```
我们依然把`Box`声明为“协变”，同时在`contains`方法中引入新的类型标识`T >: F`，使用前文所述的边界，硬性要求`T`比`F`更宽泛，再把`T`作为入参，使得“逆变”要求也得到满足。

