# 函数式，从代码到架构

- [前言](README.md)

##### Functions
- [Function](5_function/READMD.md)
    - [人尽皆知的函数](5_function/1_function.md)
    - [鲜为人知的函数](5_function/2_advanced_func.md)
    - [是函数，也是值](5_function/3_val_def.md)
    - [纯函数的特质](5_function/4_pure_fp.md)
    - [纯函数的好处](5_function/5_why_pure_fp.md)
    - [像代数推导一般](5_function/6_fp_algebra.md)
    - [优雅的递归](5_function/7_recursion.md)

##### Types
- [Type](6_type/READMD.md)
    - [为什么需要类型](6_type/1_why_type.md)
    - [代数数据类型](6_type/2_adt.md)
    - [参数化类型](6_type/3_param_type.md)
    - [类型的类型](6_type/4_higher_kinded_type.md)

##### Type classes
- [Monoid](1_monoid/README.md)
    - [重温代数运算](1_monoid/1_revisit_algebra.md)
    - [从求和说起](1_monoid/2_sum_function.md)
    - [Monoid的副产品](1_monoid/3_semigroup.md)
    - [Monoid的应用](1_monoid/4_monoid_usage.md)
- [Functor](2_functor/README.md)
    - [重温函数和类型](2_functor/1_revisit_function.md)
    - [再谈求和函数](2_functor/2_sum_func_again.md)
    - [什么是Functor](2_functor/3_what_is_functor.md)
    - [自己变成自己](2_functor/4_endofunctor.md)
    - [Functor的意义](2_functor/5_functor_point.md)
- [Monad](3_monad/README.md)
    - [容器里的函数](3_monad/1_applicative.md)
    - [触发链式反应](3_monad/2_monad.md)
    - [开枝散叶](3_monad/3_tree_monad.md)
    - [云游太虚幻境](3_monad/4_identity.md)
    - [尺有所短寸有所长](3_monad/5_app_vs_monad.md)
    - [Free Monad](3_monad/6_free_monad.md)
- [Traverse](4_traverse/README.md)
    - [大丈夫能屈能伸](4_traverse/1_foldable.md)
    - [穿越火线](4_traverse/2_traverse.md)
    - [再次穿越火线](4_traverse/3_traverse_functor.md)

##### Data
- [Optics](optics/README.md)
    - [Lenses](optics/1_lenses.md)
    - [Prism](optics/2_prism.md)
    - [ISO](optics/3_iso.md)
    - [Optional](optics/4_optional.md)
    - [Composition of Optics](optics/5_optics_composition.md)

##### Effects
- [Effect](7_effect/README.md)
    - [IO Monad](7_effect/1_io_monad.md)
    - [State Monad](7_effect/2_state_monad.md)
    - [Reader Monad](7_effect/3_reader_monad.md)
    - [Writer Monad](7_effect/4_writer_monad.md)
    - [Monad Transformer](7_effect/5_monad_transformer.md)
    - [Combining Effects](7_effect/6_combine_efects.md)

##### Application
- [From Imperative to Functional](i2f/RAEDMD.md)
    - [Replace Mutator Method](i2f/mutator_method.md)
    - [Replace Mutable Variable](i2f/mutable_var.md)
    - [Replace Loop](i2f/loop_to_fold.md)
    - [Exception Handling](i2f/exception_handle.md)

- [architecture]()
    - Functional Domain Modeling
    - Onion Architecture
