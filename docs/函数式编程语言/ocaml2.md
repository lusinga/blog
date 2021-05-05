# ocaml快餐教程(2) - 定义变量与函数

上一节我们主要介绍了ocaml的基本数据类型。在这些基本数据类型的基础上，我们可以组成更复杂的数据结构，比如我们用逗号分隔一些类型的值，就构成了元组。

比如元组(1, 10.)，就一个int * float类型的元组。

```ocaml
# (1,10.);;
- : int * float = (1, 10.)
```

再例如，由int64, int32, nativeint, int, float构成的元组：
```ml
# 1L, 1l, 1n, 1, 1. ;;
- : int64 * int32 * nativeint * int * float = (1L, 1l, 1n, 1, 1.)
```

我们再来个-1开会：
```ocaml
# Int64.minus_one, Int32.minus_one, Int.minus_one, Nativeint.minus_one ;;       
- : int64 * int32 * int * nativeint = (-1L, -1l, -1, -1n)
```

## 变量绑定与模式匹配

我们可以通过let语句，将一个表达式绑定给一个变量，如：
```ocaml
# let a = 1,1l ;; 
val a : int * int32 = (1, 1l)
```

针对于两个元素构成的元组，我们可以通过fst和snd函数来获取元素的值，例：
```ocaml
# let a = 1,1l ;; 
val a : int * int32 = (1, 1l)
# let a1 = fst a;;
val a1 : int = 1
# let a2 = snd a;;
val a2 : int32 = 1l
```

超过两个元素的元组，没法使用fst和snd函数。但是我们可以在let左边也用变量元组去绑定元组的元素，这样的操作称为模式匹配。
```ml
# let a3,a4,a5 = 3, 4., 5n ;;
val a3 : int = 3
val a4 : float = 4.
val a5 : nativeint = 5n
```

我们也可以通过and的方式进行联合赋值：
```ml
# let a6 = 6n and a7 = 7l and a8 = 8. ;;
val a6 : nativeint = 6n
val a7 : int32 = 7l
val a8 : float = 8.
```

在模式匹配中，如果有的值不需要绑定变量，我们可以赋给"_"，例：
```ml
# let a9, _ , a11 = 9e0, 10e-10, 11. ;;
val a9 : float = 9.
val a11 : float = 11.
```
10e-10的值就直接被pass掉了。

## 用let定义函数

如果绑定的值带有参数，那么就是定义了一个函数。
我们来看个例子：
```ml
# let s3 x = x * x * x ;;
val s3 : int -> int = <fun>
# s3 8 ;;
- : int = 512
```

这时候发现*和*.区分开的好处了吧，类型容易被推导出来：
```ml
# let a x = x *. x ;;
val a : float -> float = <fun>
```

这时候ocaml面向对象的特性带来加成，有些不用系统，我们都推导的出来：
```ml
# let s_2 x = Float.abs x ;;
val s_2 : float -> float = <fun>
# s_2 1e8;;
- : float = 100000000.
```

用let定义的函数，也可以支持多个参数，比如我们定义一个算加法的函数：
```ml
# let add2 x y = x +. y ;;
val add2 : float -> float -> float = <fun>
# add2 100. 200. ;;
- : float = 300.
```

我们也可以用let来定义柯里化的函数。柯里化要求每次只能有一个参数，如果有多个参数，那么处理一个之后，返回值是另一个函数。
比如上面的add2需要x和y两个参数，我们给定一个值，比如说是2，那么就变成了一个专门加2的函数了：
```ml
# let add2_2 = add2 2. ;;
val add2_2 : float -> float = <fun>
# add2_2 100. ;;
- : float = 102.
```

## function和fun

上面的定义方面没什么不好的，如果觉得跟前面讲的let绑定变量的方法不太一致，想要let f = <函数定义> 这种方式的话，可以使用function关键字来定义：
```ml
# let s2 = function x -> x *. x ;;
val s2 : float -> float = <fun>
# s2 1e-1 ;;
- : float = 0.0100000000000000019
```

function的好处是，它是柯里化的，只能支持一个参数。如果想要支持多个参数的话，我们可以嵌套。

我们来看个例子：
```ml
# let mul2 = function x -> (function y -> x *. y) ;;
val mul2 : float -> float -> float = <fun>
# mul2 1e8 1e-4 ;;
- : float = 10000.
```

另外，我们可以把参数变成元组，这样看起来可能比较像其它语言的函数：
```ml
# let div2 = function (x,y) -> x /. y ;;
val div2 : float * float -> float = <fun>
```
调用时需要给元组：
```ml
# div2 (1e8,-4.) ;;
- : float = -25000000.
```

最后，fun可以定义多参数的函数：
```ml
# let addx = fun x1 x2 x3 -> x1 + x2 +x3 ;;
val addx : int -> int -> int -> int = <fun>
# addx 1 2 3 ;;
- : int = 6
```

## 高阶函数

高阶函数，通俗地讲，就是不但值可以做为参数，函数也可以做为参数。

比如，我们想定义一个两个数的操作的函数，我们这样写：
```ml
# let higherf ops x y = ops x y ;;
val higherf : ('a -> 'b -> 'c) -> 'a -> 'b -> 'c = <fun>
```

这下我们就拥有泛型的能力了，比如针对整数加法，可以用Int.add做为ops，nativeint就用Nativeint.add，浮点数就用Float.add: 
```ml
# higherf Int. add 1 2 ;;
- : int = 3
# higherf Nativeint.add 1n 2n ;;
- : nativeint = 3n
# higherf Float.add 1. 2. ;;
- : float = 3.
```

我们再举个例子，比如三角函数计算时都要求使用弧度，我们是不是可以将角度转弧度的过程能够应用于所有的三角函数呢？
我们可以这么写：
```ml
# let deg ops x = ops ( x *. Float.pi /. 180.) ;;
val deg : (float -> 'a) -> float -> 'a = <fun>
```

这样，我们再求角度的三角函数时，就可以通过deg sin, deg cos这样的方式去调用了：
```ml
# deg sin 90. ;;
- : float = 1.
# deg sin 45. ;;
- : float = 0.707106781186547462
# deg cos 180. ;;
- : float = -1.
# deg tan 90. ;;
- : float = 16331239353195370.
# deg tan 45. ;;
- : float = 0.999999999999999889
```

## 小结

小结下，我们可以用let来绑定变量，也可以来定义函数。可以直接定义函数，也可以通过function和fun来定义。推荐使用function，这样会柯里化。
函数可以给定部分参数，变成新的函数。
函数的参数也可以是函数，这样的函数叫做高阶函数。
