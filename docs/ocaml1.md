# ocaml快餐教程(1) - 强类型语言

Keep说：自律给我自由。
在汽车、航天、铁路等高可靠性要求的代码中，经常要求使用MISRA C/C++标准，该标准对于C语言中不同整数类型之间的赋值有比较严格的要求。有些同学对此叫苦不迭，觉得在没有意义的修复上浪费了太多的时间。

如果他们学习了ocaml语言的话，就会觉得这样的要求是天经地义的。在ocaml中，浮点数进行加减乘除运算做用的运算符跟整数是不一样的。

比如我们在ocaml中计算1+2，这个OK:
```ocaml
# 1 + 2 ;;
- : int = 3
```

我们想计算1.0 + 2.0，对不起，不能这么玩：
```ml
# 1.0 + 2.0 ;;
Error: This expression has type float but an expression was expected of type
         int
```

好吧，算你狠。那我计算两个长整数，用+可以吧？不行：
```ml
# 1L + 2L;;
Error: This expression has type int64 but an expression was expected of type
         int
```

64位不行，我用32位整数总可以了吧？不行：
```ml
# 1l + 2l;;
Error: This expression has type int32 but an expression was expected of type
         int
```

这个int，既不是int32，也不是int64，是像C语言的int一样的跟机器字长相关的整数。ocaml中，nativeint也是跟机器字长相关的，我们将其当成int使用还不行吗？没错，不行：
```ml
# let a1 = 1n ;;
val a1 : nativeint = 1n
# let a2 = 2n;;
val a2 : nativeint = 2n
# a1 + a2 ;;
Error: This expression has type nativeint
       but an expression was expected of type int
```

那么，我们想要给2个浮点数做加减乘除，该如何写呢？+-/*后面也加上一个小数点：
```ml
# 1. +. 2. ;;
- : float = 3.
```

如果是其它那些整数类型想要进行运算，对不起，没有运算符了，请直接调用相关类的方法吧：
```ml
# Int32.add 1l 4l;;
- : int32 = 5l
# Int64.add 100L 2000L;;
- : int64 = 2100L
# Nativeint.add 50n 40n;;
- : nativeint = 90n
```

## 几句话迅速安装

在Ubuntu上，可以用apt install ocaml进行安装。
在MacOS上，可以使用Homebrew进行安装。

安装好之后，运行ocmal命令就可以进入交互模式。

## ocaml的数值类型及其转换

我们将上面数值类型整理一下：
![ocaml numbers](https://upload-images.jianshu.io/upload_images/1638145-63c4eda40be2d544.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

知道了有什么，我们再列举下没有什么：
- 没有8位整数
- 没有16位整数
- 没有无符号类型
- 没有32位浮点数

### 整数的最大值和最小值

很多同学对于数值类型有很多经验了，但是我还是要说，不要假设，在ocaml这种类型比较复杂的环境中，我们还是得看下真实的最大值和最小值是多少：
```ml
# Int.max_int;;
- : int = 4611686018427387903
# Int32.max_int;;
- : int32 = 2147483647l
# Int64.max_int;;
- : int64 = 9223372036854775807L
# Nativeint.max_int;;
- : nativeint = 9223372036854775807n
# Int.min_int;;
- : int = -4611686018427387904
# Int32.min_int;;
- : int32 = -2147483648l
# Int64.min_int;;
- : int64 = -9223372036854775808L
# Nativeint.min_int;;
- : nativeint = -9223372036854775808n
```

请注意，在64位系统上，nativeint与int64范围是一致的，但是int比它们少一位，所以差了两倍。

我们可以通过Sys.int_size来查看int类型的大小：
```ml
# Sys.int_size ;;
- : int = 63
```

需要注意的一点是，int类型是一直都有的，但是Int模块是4.08版本才引入进来的。这是相当晚近的事情，从ocaml 4.08才加入，这是2019年6月14日发布的。

在早于4.08版的ocaml上，Int是不存在的：
```ml
# Int.max_int;;
Error: Unbound module Int
```

比如Ubuntu 20.04现在使用的ocaml是4.08.1，将将及格。Ubuntu 18.04的ocaml还是4.05.0，还没满足这个要求。

### 无符号操作

虽然不支持无符号类型，但是ocaml为int32, int64和nativeint三种类型提供了几种无符号操作。这也是4.08版才引入进来的。

这其中包括：
- unsigned_div：除法
- unsigned_rem：求余
- unsigned_to_int：转化为整数
- unsigned_compare：比较

例：
```ml
# Int32.unsigned_div 2147483648l 2l;;
- : int32 = 1073741824l
```

### 类型转换

其他整数类型，通过转换成int型来进行互转。
转成int的函数是to_int，把int转换成本类型是of_int。

我们看几个例子：
```ml
# Int32.to_int(30l);;
- : int = 30
# Int32.of_int(40);;
- : int32 = 40l
```

主要整数类型都提供了one和zero来表示1和0：
```ml
# Int64.to_int(Int64.one);;
- : int = 1
# Int64.of_int(Int.one);;
- : int64 = 1L
```
另外还有minus_one表示-1：
```ml
# Nativeint.to_int(Nativeint.minus_one);;
- : int = -1
```

### 浮点数

浮点数用Float类型来表示，这个模块是4.07版本引入的。

浮点数的边界有三个值：最大值，最小值，最小间距值：
```ml
# Float.max_float;;
- : float = 1.79769313486231571e+308
# Float.min_float;;
- : float = 2.22507385850720138e-308
# Float.epsilon;;
- : float = 2.22044604925031308e-16
```

4.08版开始，提供了is_integer函数来判断是不是一个小数点后为0的数：
```ml
# Float.is_integer(1e8);;
- : bool = true
```

那么在4.07版之前，用什么方式来做浮点数和整数的转换呢？我们有float_of_int和int_of_float两个函数来实现：
```ml
# float_of_int(100);;
- : float = 100.
# int_of_float(1e8);;
- : int = 100000000
```

前面讲过了，浮点数进行计算的时候，用的运算符是在整数运算符后加点的运算符：
```ml
# 1e8 +. 1e6;;
- : float = 101000000.
# 1e8 *. 1e-8;;
- : float = 1.
```

可以用科学计数法来表示浮点数，如果e后面是负数的话直接写负号，写括号是不能被识别的。

另外，Float提供了常用数学函数。如平方根，乘方，三角函数等基本数学函数都是由Float类提供的，如果整数需要进行此类计算，可以转化成Float再计算：
```ml
# Float.to_int( Float.sqrt( Int.to_float(144)));;
- : int = 12
```

## 比较运算符只有一套

ocaml提供了bool类型，只有两个值true和false。
bool值可以通过not运算符取反，&&进行与运算，|| 进行或运算。

ocaml也同样提供了=,<,>,>=,<=,<>几种比较运算符。
坏消息是，比较运算符要求前后的类型还是一致的。
好消息是，这次没有'<.'这样的判断浮点值的运算符了，int32, int64等类型的值也可以用这些运算符来进行比较了。

我们来看几个例子：
```ml
# 1 >= 2;;
- : bool = false
# 1l <= 5l ;;
- : bool = true
# 20L <> 10L ;;
- : bool = true
# 0.1 < 0.5 ;;
- : bool = true
# (20L > 10L) && (5l < 3l) ;;
- : bool = false
```

## 字符与字符串

本节的最后，我们看下字符与字符串。
字符和C语言一样，是个8位的数字，使用上是单引号加字符。但是，在其上没有定义算术运算。可以通过Char.code函数将字符转换成ASCII码：
```ml
# Char.code('A');;
- : int = 65
```
使用Char.chr函数将0~255的值转换成字符，例：
```ml
# Char.chr(32);;
- : char = ' '
# Char.chr(255);;
- : char = '\255'
```

ocaml的字符串不是字符数组，是一串用双引号括起来的字符。
字符串可以使用中文：
```ml
# let a = "中文";;
val a : string = "中文"
```
计算字符串长度通过String.length函数。
如果是中文的话需要注意，String.length获取的是实际占用的字节数，与编码方式相关：
```ml
# let a = "中文";;
val a : string = "中文"
# String.length a
  ;;
- : int = 6
```

string的最大长度可以通过Sys.max_string_length来获取：
```ml
# Sys.max_string_length;;
- : int = 144115188075855863
```

虽然不是字符数组，但是可以通过String.get函数来获取第n个字符，n从0开始计数：
```ml
# String.get "Hello" 2;;
- : char = 'l'
# String.get "Hello" 0;;
- : char = 'H'
```
