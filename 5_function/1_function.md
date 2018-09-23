# 人尽皆知的函数

> 凡式中含天，为天之函数。——李善兰

中国古代“函”字与“含”字通用，都有着“包含”的意思。函数，指一个量随着另一个量的变化而变化。这是运动式的理解，还可以从集合、映射的角度定义，函数为两集合间的一种对应关系：输入值集合中的每项元素皆能对应唯一一项输出值集合中的元素。如下图的所示：

![functor](../imgs/domain_range.png)

f是一个将所有定义域X（红色区块）中的点x∈X映射到点f(x)∈Y的函数。点f(x)的集合（黄色区块）为函数f的值域，Y（蓝色区块）为f的到达域。在scala中，定义域和值域都是"类型"，关于类型，后面我们再讨论，如函数：
```scala
val square: Int => Int = x => x * x
```
定义域（domain）和到达域（codomain）都是Int。需要指出的是，值域（range或image）并不一定要包含整个对应域。如，3属于square函数的到达域，但并不在值域中，因为3的平方根不是整数。

函数本身也可以作为函数的定义域或到达域，这种输入或输出函数的函数，称为高阶函数，如：

```scala
trait List[A] {
  def filter(f: A => Boolean): List[A]
}
```

*List[A].filter* 就是一个高阶函数，接受函数A => Boolean，返回值*List[A]*。

定义域和到达域都是类型，函数可以作为定义域和到达域，可见函数本身也是一种类型。
```scala
type Conv[A] = A => String
def string(num: Int): Conv[Int] = x => (x + num).toString
```
用Conv代替A => String更简洁明了。

继续。如果一个函数的定义域（输入）可以接受不同类型，我们称之为*多态*函数。Scala不支持多态函数，但可以通过apply方法模拟出多态。
```scala
scala> case object identity {
  def apply[A](value: A): A = value
}
defined object identity

scala> identity(3)
res0: Int = 3

scala> identity("4")
res1: String = 4
```
这个例子中，identity
可以接受任意类型，并返回原始值。也许你已经发现，这里的多态和面对对象世界里的多态有所不同。我们没有用父子类、接口等继承关系，而是对参数做了抽象，因此，也称为*参数多态*（parametric polymorphism）。

<div class="alert alert-success">
Scala的apply方法是一种语法糖，用于描述类或对象的最主要使用场景。identity(3)是下面方法调用的简化写法。
<p>
scala> identity.apply(3)<br/>
res2: Int = 3<br/>
</p>
Scala既是函数式语言，也是面向对象语言，函数本身也是对象。相比于编程员“<b>调用</b>函数f，传入参数x”，数学家更习惯于“<b>应用</b>函数f到参数x”。<br/>
当你定义以下函数时，其实是定义了Function1[Int,Int]类型的对象，f是对该对象的引用。
<p>
val f =  (x: Int) => x + 1
</p>
除了f(3)这种函数式的写法，还可以用繁琐的面向对象式写法f.apply(3)。
</div>

