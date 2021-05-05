# ocaml快餐教程(3) - 基本结构

## 分支结构

ocaml中支持用if...then...else表达式。
例：

```ocaml
# let pass x = if x>=60 then "pass" else "fail" ;;
val pass : int -> string = <fun>
# pass 60 ;;
- : string = "pass"
# pass 59 ;;
- : string = "fail"
```

如果要做多个分支的话，可以进行嵌套：

```ocaml
# let a x = if x>0 then 1 else (if x=0 then 0 else -1) ;;
val a : int -> int = <fun>
# a 0 ;;
- : int = 0
# a 1 ;;
- : int = 1
```

再次强调，ocaml是强类型语言，所以if表达式的then分支和else分支的类型必须是相同的。

比如下面的，then返回int而else返回float是不行的：

```ocaml
# let f x = if x > 0 then 0 else 0. ;;
Error: This expression has type float but an expression was expected of type
         int
```

如果想要switch case一样的功能，我们还是通过模式匹配的方式来做，这次我们用match x with的方式来写。每一个match用"|"开头，条件与表达式用"->"来分隔。"_"用来匹配default项。
匹配可以用字符串，我们来看个例子：
```ml
# let arg_check arg = match arg with
  | "-h" -> "Help"
  | _ -> "undefined" ;;
val arg_check : string -> string = <fun>
# arg_check "Hello" ;;
- : string = "undefined"
```

## 递归

下面我们来看循环结构，作为函数式语言，我们的解决方案是用函数递归来实现。

于是我们就开写：
```ml
let fact4 x = if x=0 then 1 else x * (fact4 x-1) ;;
```
结果报错了，说fact4没有被定义。

这里我们需要给递归使用的函数名加一个rec关键字：
```ml
# let rec fact3 x = if x=0 then 1 else x * (fact3 (x-1)) ;;
val fact3 : int -> int = <fun>
# fact3 0 ;;
- : int = 1
# fact3 1 ;;
- : int = 1
# fact3 2;;
- : int = 2
# fact3 3;;
- : int = 6
# fact3 4;;
- : int = 24
# fact3 10;;
- : int = 3628800
# fact3 20;;
- : int = 2432902008176640000
```

上面这个不支持负数，我们可以再修改一下：
```ml
let rec fact5 x = if x<1 then 1 else x * (fact5 (x-1)) ;;
val fact5 : int -> int = <fun>
# fact5 0;;
- : int = 1
# fact5 (-1) ;;
- : int = 1
# fact5 10;;
- : int = 3628800
```

## 通吃多种类型 - 联合类型

虽然ocaml是强类型的，比起其它语言限制要多一些，但是它也有一般静态类型语言少有的特性，即联合类型。

联合类型的最简单用法是用作枚举类型。

我们看个例子：
```ml
# type lang = SML | Ocaml | Haskell;;
type lang = SML | Ocaml | Haskell
```
从此以后，SML, Ocaml和Haskell都可以当成常量来使用了：
```ml
# let a1 = SML ;;
val a1 : lang = SML
```

我们发现，刚才的几种语言全是函数式语言，我们给它们定义一个函数式语言的类型：
```ml
type func_lang = SML | Ocaml | Haskell;;
```

然后我们将lang扩充为func_lang和C, java的组合：
```ml
type lang = C | Java | Func of func_lang ;;
```

注意，这里Func不是一个新枚举了，而是一个func_lang类型对象的生成函数。
现在如果lang类型的变量是func_lang类型的话，需要调用Func函数。我们来看个例子：
```ml
# let l1 = Func(Ocaml) ;;
val l1 : lang = Func Ocaml
```

好，小试牛刀之后，我们来开始实现不同整数之间跨种族的四则运算，我们以加法为例。

简单起见，我们只支持两种类型int32和int64:
```ml
type int32s = I32 of int32 | I64 of int64;;
```

然后我们就写新的加法吧，4种情况都覆盖到：
```ml
let add_int32s = function (I32 x, I32 y) -> I32 (Int32.add x y)
  | (I32 x, I64 y) -> I64(Int64.add (Int64.of_int32(x)) y)
  | (I64 x, I32 y) -> I64(Int64.add x (Int64.of_int32(y)))
  | (I64 x, I64 y) -> I64(Int64.add x y);;
```

现在我们只要加上构造函数，就可以int32和int64之间不用转换直接运算了，结果还是int32s类型。

不过别死心眼啊，不是说输入是int32s，输出就还得是int32s啊。正常来说，计算完了转成int更有助于进一步计算，我们修改一下：
```ml
let add_int32_v2 = function (I32 x, I32 y) -> Int32.to_int(Int32.add x y)
  | (I32 x, I64 y) -> Int64.to_int(Int64.add (Int64.of_int32(x)) y)
  | (I64 x, I32 y) -> Int64.to_int(Int64.add x (Int64.of_int32(y)))
  | (I64 x, I64 y) -> Int64.to_int(Int64.add x y);;
```

赋给别的变量，就变成普通int了：
```ml
# let a2 = add_int32_v2(I32 64l, I64 128L);;
val a2 : int = 192
```

## 自动生成构造函数 - 多态变体

别看ocaml是强静态类型语言，对于提升效率也是不遗余力的。

比如ocaml可以自动帮我们推导构造函数，我们只要给个类型让系统自己去生成即可，方法是在构造函数名字前加个“`”，例：
```ml
# `Ints 1 ;;
- : [> `Ints of int ] = `Ints 1
# `Ints 1n ;;
- : [> `Ints of nativeint ] = `Ints 1n
# `Ints 1l;;
- : [> `Ints of int32 ] = `Ints 1l
# `Ints 1L;;
- : [> `Ints of int64 ] = `Ints 1L
```

也可以将其绑定到变量上：
```ml
# let a3 = `Ints 3 ;;
val a3 : [> `Ints of int ] = `Ints 3
# let a4 = `Ints 4l;;
val a4 : [> `Ints of int32 ] = `Ints 4l
```

有个`Ints的变量，我们可以通过模式匹配的方式还原成原始类型：
```ml
# let f1 x = match x with
  | `Ints x -> x;;
val f1 : [< `Ints of 'a ] -> 'a = <fun>
```
找两个例子试试：
```ml
# f1 a3 ;;
- : int = 3
# f1 a4 ;;
- : int32 = 4l
```

## 可选值

联合类型更邪的是还可以玩成可选值，也就是可以是空值。在定义类型的时候将None值加上就好了。

我们看个小例子：
```ml
# type int32_v3 = None | I2 of int ;;
type int32_v3 = None | I2 of int
```

构造可选值的时候，我们使用Some加在普通的定义之上：
```ml
# let a11 = Some(I2 1);;
val a11 : int32_v3 option = Some (I2 1)
```

我们看到，在int32_v3类型后面，加上了option，说明是可为空的。

## 小结

这一节我们介绍了if表达式和递归函数定义方法，以及在别的语言中较少见的联合类型或者叫做变体，变体可以自动生成构造函数，联合类型也可以是可选的。
