# 自己变成自己

函子和函数很像，它们有很多相同的性质。我们可以通过函数的性质来了解函子的性质。

Endofunction（自函数）把一个类型映射到自身类型，比如Int=>Int, String=>String等。 而Functor描述两个范畴的映射关系，相应地，Endofunctor（自函子）便是把一个范畴映射到自身范畴的函子。

![functor](../imgs/endofunctor.png)

从上图不难看出，最简单的Endofunctor莫过于Identity函子了，A还是A，B还是B，函数f：A=>B也保持不变。和endofunction类似，它不改变内部元素和关系，原样保持输入的范畴。


