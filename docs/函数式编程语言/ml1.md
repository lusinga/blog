# Standard ML快餐教程(1) - 初识

好久没写快餐教程了，下面开始一个新的系列，关于函数式编程语言的系列。打算写三种语言：Standard ML，ocaml和Haskell。
这几门语言都不是新贵了，其中Standard ML的知名度可能最低。因为ML系列的影响力，其实我们已经从新的语言如rust等中学到过这些老语言的很多知识了。现在我们可能只要将它们还原回去就好。

## 运行环境

Standard ML，顾名思义，是ML语言的一种标准规范，主要的版本有SML 90和SML 97。
标准的实现就有很多种方式。比如以交互式编译为主要特点的SML/NJ，还有以优化编译为主要目标的mlton。另外一个重要的Standard ML编译器是剑桥大学的David Matthews开发的poly/ML，因为著名的推理证明工具Isabelle是使用polyml开发的。

SML/NJ是由贝尔实验室的David MacQueen和普林斯顿大学的Andrew Appel于1986年发起的。为了满足1997年SML的修订，1998年1月推出了110版本。时至2020年，主版本号还是110，比如我现在用的版本是v110.97。

SML/NJ的项目主页在：[http://www.smlnj.org/](http://www.smlnj.org/)。其源代码是使用svn管理的。mlton和polyml的源代码都在github上管理。mlton的地址为：[https://github.com/MLton/mlton](https://github.com/MLton/mlton)，ployml的地址为：[https://github.com/polyml/polyml/](https://github.com/polyml/polyml/)。

安装的话，因为以上三种语言工具都属于比较通用成熟的工具，在mac上的homebrew和Ubuntu之类的主流Linux发布版本中都有支持。
比如在macOS上，可以使用homebrew进行安装这三种环境：
```
brew install mlton
brew install polyml
brew cask install smlnj
```

在Ubuntu下可以使用下面的命令安装：
```
apt install smlnj
apt install mlton
apt install polyml
```

安装好之后，运行sml命令，就可以进入sml/nj的交互式环境，就可以开始学习编写Standard ML代码了，"-"是smlnj的提示符：
```
Standard ML of New Jersey v110.79 [built: Sat Oct 26 12:27:04 2019]
-
```

我们可以先写个HelloWorld程序练练手,比如叫hello.sml:
```sml
val a = 1 :int ;
print "Hello,World\n";
```

smlnj可以通过sml hello.sml来调用。
也可以通过mlton hello.sml来编译，然后会生成hello，运行hello就可以看到运行结果。

使用polyc进行编译的时候，会报下面的错误：
```
poly: : error: Value or constructor (main) has not been declared
Found near PolyML.export (List.nth (CommandLine.arguments (), 3), main)
Static Errors
```
这是因为polyml需要一个main函数。
我们写个main函数吧：
```ml
fun main() = print "Hello,World\n";
```
然后调用polyc去编译：
```
polyc hello2.sml -o hello2
```
最后运行hello2就可以看到运行结果。

polyml也可以通过poly命令交互式运行。
```
Poly/ML 5.8.1 Release
```
提示符为">"。

## 数据类型

我们可以通过交互式界面来了解下基本数据类型的用法。

### 数字类型

比如我们输入一个整数1：
```sml
> 1;
val it = 1: int
```
我们可以看到，这样一个整数被赋给一个叫it的默认变量，其类型为int. 

-1在Standard ML中表示为"~1"，我们来看下:
```sml
> val a = 0 - 1;
val a = ~1: int
```

符点数是real类型：
```sml
> val b = 1e8;
val b = 100000000.0: real
```

### 字符串

Standard ML支持字符串类型：
```sml
> val d = "Hello";
val d = "Hello": string
```

### 布尔值

Standard ML使用true和false两个关键字来表示bool类型：
```sml
> true;
val it = true: bool
> false;
val it = false: bool
> 1>0;
val it = true: bool
```

### 空值

Standard ML的空类型为unit，用来表示一个空表():
```sml
> ();
val it = (): unit
> val g = ();
val g = (): unit
```

同样，空的记录也是unit:
```sml
> val x3 = {};
val x3 = (): unit
```

### 字符类型

通过“#”，也可以描述char类型：
```sml
> val e = #"c";
val e = #"c": char
```

ord函数用来将字符转成整数，chr相反将ASCII值转成字符。

```sml
> val x4 = ord(#"a");
val x4 = 97: int
> val x5 = chr(65);
val x5 = #"A": char
```

## 数据结构

### 元组

元组是使用括号描述的列表。类型将成为几个类型之乘积。
比如(3.0, true)的类型是real * bool. 

```sml
> val x6 = (3.0, true);
val x6 = (3.0, true): real * bool
```

### 列表

不同于元组，列表的元素必须是相同的，用方括号表示。
空列表不是unit，也是个列表类型。

```sml
- val a1 = [];
val a1 = [] : 'a list
```

只要类型相同就可以，元组也没问题：
```sml
- val a2 = [1,2];
val a2 = [1,2] : int list
- val a3 = ["h","e"];
val a3 = ["h","e"] : string list
- val a4 = [(3,4),(2,5)];
val a4 = [(3,4),(2,5)] : (int * int) list
```

### 记录

类似于哈希表的一种结构，指定key与value:
```sml
- val a5 = {name="xulun",age=20};
val a5 = {age=20,name="xulun"} : {age:int, name:string}
```

如果以数字1,2,3等为key，则建立的记录就是元组。我们看个例子：
```sml
- val a7 = {1=true, 2=3.0};
val a7 = (true,3.0) : bool * real
```

### 多赋值

Standard ML支持模式匹配多赋值，比如我们使用变量元组和值元组，举个例子看下就清楚了：
```sml
> val (x1,x2) = (1.0, true);
val x1 = 1.0: real
val x2 = true: bool
```

## 语句

### 函数

可以通过fun来定义函数，我们直接看个求平方的例子：
```sml
> fun s2 (x:int):int = x*x;
val s2 = fn: int -> int
> s2(4);
val it = 16: int
```

可以不指定类型，由系统进行推断：
```sml
- fun add1 x y = x + y;
val add1 = fn : int -> int -> int
```

如果想用浮点类型，可以指定一个，其它由系统推断：
```sml
- fun add2 (x:real) y = x + y;
val add2 = fn : real -> real -> real
```

### 注释

Standard ML使用(**)来写注释。

### 条件语句

Standard ML支持if then else语句，如果if为真则返回then后面的表达式，否则返回else分支的结果。
```sml
- fun neg1 (x:int) : bool = if x < 0 then true else false;
val neg1 = fn : int -> bool
```
还记得-1在sml表示为~1吧，我们测试下上面的if判断：
```sml
- neg1(1);
val it = false : bool
- neg1(~1);
val it = true : bool
```

### case语句

通过case语句，可以处理多分支的条件，每个条件进行自己的匹配。格式为case x of x1 => something1 | x2 => something2 | ... 

我们来看个例子：
```sml
- fun grade1(x:int) = case x of 0 => false | 1 => true | x => false;
val grade1 = fn : int -> bool
- grade1(1);
val it = true : bool
- grade1(0);
val it = false : bool
- grade1(100);
val it = false : bool
```

## 标准库

做为一种标准语言，Standard ML也支持可移植性的标准库。我们可以调用这些库像其它语言一样愉快地编程了。sml的标准库叫做BASIS。
第一次调用的时候，basis会被加载进来：

```sml
- Char.isUpper(#"a");
[autoloading]
[library $SMLNJ-BASIS/basis.cm is stable]
[library $SMLNJ-BASIS/(basis.cm):basis-common.cm is stable]
[autoloading done]
val it = false : bool
```

再来几个例子，比如计算实数的绝对值：
```sml
- Real.abs(~1.5);
val it = 1.5 : real
```

整数转实数：
```sml
- val a10 = Real.fromInt(1);
val a10 = 1.0 : real
```

求平方根：
```sml
- val a11 = Math.sqrt(2.0);
[autoloading]
[autoloading done]
val a11 = 1.41421356237 : real
```

字符串连接：
```sml
- val a11 = String.concat(["Hello","World"]);
val a11 = "HelloWorld" : string
```

一起看起来都不错。类型，分支，函数等都有了。还有强大的标准库，恭喜你，已经在Standard ML的世界里活下来了 ：）
