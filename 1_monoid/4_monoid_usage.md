## Monoid的应用

大数据应用，如Spark和Hadoop，把数据分散到很多机器上分析，以实现容错性和横向拓展性。这意味着每台机器只返回部分数据的运算结果，我们必须把它们结合起来才能拿到最终结果。大部分场景下，这就可以看成是Monoid。

举例来说，我们要计算网站有多少访问用户，则分别计算各个部分的数字，因为整数在加法运算下构成Monoid，便可以把它们累加起来得到最终结果。

