## Functor的意义

Functor在集合类中发挥了重要作用，因为它在对集合中的元素进行的运算是独立的，各元素之间互不干扰。这一特质构成了并行计算或分布式计算的基础，在Hadoop等map-reduce框架中，尤为重要。

另一作用来自于对Functor中的F的理解。F是类型构造器，可以视为容器，或者环境、上下文 
。这种容器常常代表了对特定“效果”的抽象，如Option的效果是抽象了“可能缺少值”这一效果，map函数只会对有值，即为Some时才生效，否则只是把None
传递下去，不管怎样，结果依然是个Option。不同容器，抽象的“效果”不同。从这个意义看，Functor
具有和“效果”协作的能力，它能对具有某种“效果”的值执行指定的函数，而执行的结果依然携带这一“效果”。

一般情况下，正如高阶导数那样，Functor作为高阶类型，本身很少被直接使用。更多的是为其它更有意思、更有用的抽象提供基础，如后面将讲到的Applicative和Monad。


