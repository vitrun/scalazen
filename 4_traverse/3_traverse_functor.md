# 再次穿越火线

> 横看成岭侧成峰，远近高低各不同。——《题西林壁》

在Id的辅助下，Functor的map可以从traverse推导得到：
```scala
type Id[X] = X

def map[A, B](fa: F[A])(f: A => B): F[B] = traverse[Id, A, B](fa)(f)
```
事实上，Traverse就是Functor，这些Functor相应地称为traversable functor。scalaz等函数式编程库的Traverse就是从Functor拓展而来。也许你还记得foldable的map也可以从foldLeft推导得到，是否可以说Foldable也是Functor呢？答案是否定的。具体可以通过验证Functor的两条法则来判断。

Functor的map一定保持容器结构不变，而Foldable既可以保持，也可以不保持。从这个意义上看，Foldable比Functor灵活，但也存在不属于Foldable的Functor，它们之间并无继承关系。而Traverse则同时是Functor和Foldable。三者的关系可以用下图表示：
![functor](../imgs/fold_func_trav.png)


大概读者朋友猜到了，Traversable从Foldable拓展而来，那么Foldable的标志性方法foldLeft和foldRight，可以用traverse推导而来。

