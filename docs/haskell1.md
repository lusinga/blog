# Haskell快餐教程(1) - 初见

Haskell是一门由委员会发明的纯函数式语言。最早的标准制定于1990年，后来在1998年有较大的修订。最新的标准是2010年推出的，具体内容可以在这里看到：[https://www.haskell.org/onlinereport/haskell2010/](https://www.haskell.org/onlinereport/haskell2010/)

编译器我们可以使用ghc: [https://www.haskell.org/ghc/](https://www.haskell.org/ghc/)

ghc本身是个编译器，但是也提供了一个交互式的环境ghci. 我们后面的操作都以ghci为例。

## 数据类型

Haskell是门静态类型的语言，数据类型是我们基本上比较熟悉的。
为了查看数据类型，我们先在ghci中打开数据类型的开关：

```haskell
:set +t
```

### 布尔值

Haskell使用==表示相等，使用/=表示不等。

```haskell
Prelude> 1 == 1
True
it :: Bool
Prelude> 1 /= 1
False
it :: Bool
```

与C语言一样，&&表示逻辑与，||表示逻辑或。

例：

```haskell
Prelude> let a = 1
a :: Num p => p
Prelude> a > 0 && a < 10
True
it :: Bool
```

### 字符与字符串

Haskell的字符串也是字符数组。字符用单引号，字符串用双引号：
```
Prelude> let c1 = 'C'
c1 :: Char
Prelude> c1
'C'
it :: Char
Prelude> let s2 = "C"
s2 :: [Char]
Prelude> s2
"C"
it :: [Char]
```

我们用字符数组的方式来表示，结果获取的一样是个字符串，我们看个例子：
```haskell
Prelude> let s3 = ['H','e']
s3 :: [Char]
Prelude> s3
"He"
it :: [Char]
```

字符串采用++进行连接：
```haskell
Prelude> let s1 = "Hello" ++ "World"
s1 :: [Char]
Prelude> s1
"HelloWorld"
it :: [Char]
```

### 数值

Haskell的数值系统比较有趣。
首先，底层没啥神秘的，像整数Int，单精度浮点数Floating，双精度浮点数Double之类的都如你所想：
```haskell
Prelude> let a2 = 1 :: Double
a2 :: Double
Prelude> a2
1.0
it :: Double
Prelude> let b1 = 1 :: Int
b1 :: Int
Prelude> b1
1
it :: Int
```

在Standard ML中我们用":"来指示类型，而在Haskell中使用两个冒号。

指定一个类型之后，剩下的类型由系统自动推断：
```haskell
Prelude> let b2 = b1 * 2
b2 :: Int
Prelude> b2
2
it :: Int
```

如果一个整型与浮点数进行操作会报错：
```haskell
Prelude> let b3 = b1 + 0.1

<interactive>:99:15: error:
    • No instance for (Fractional Int) arising from the literal ‘0.1’
    • In the second argument of ‘(+)’, namely ‘0.1’
      In the expression: b1 + 0.1
      In an equation for ‘b3’: b3 = b1 + 0.1
```

但是，如果不指定类型，一个整型字面值与浮点型字面值是可以计算的，因为系统默认是用Fractional类来处理它们的：
```haskell
Prelude> let b4 = 1 + 0.1
b4 :: Fractional a => a
Prelude> b4
1.1
it :: Fractional a => a
```

Haskell也支持像log, sin, sqrt之类的数学函数，它们的返回值是Floating类型的：
```hs
Prelude> log(2)
0.6931471805599453
it :: Floating a => a
Prelude> sin(pi)
1.2246467991473532e-16
it :: Floating a => a
Prelude> sqrt(2)
1.4142135623730951
it :: Floating a => a
```

### 元组

与其它语言类似，Haskell使用"()"来表示元组。元组中的类型任意。
例：
```hs
*Test2> let t2 = (2:: Int, 3::Double)
t2 :: (Int, Double)
*Test2> t2
(2,3.0)
it :: (Int, Double)
```

### 列表

作为一种函数式语言，Haskell当然也支持列表，也是用"[]"来表示，每个列表中的元素都是相同的。

例：
```hs
*Test2> let l1 = [1::Int,2,3,4]
l1 :: [Int]
*Test2> l1
[1,2,3,4]
it :: [Int]
```

不出意外的，Haskell也支持".."运算符生成序列：
```hs
*Test2> let l2 = [(1::Int)..10]
l2 :: [Int]
*Test2> l2
[1,2,3,4,5,6,7,8,9,10]
it :: [Int]
```

## hs和lhs文件

下面我们开始在文件中写haskell源代码。
haskell支持两种格式的代码：hs格式和lhs格式。
其中hs格式就是普通的源代码了，举个例子：
```hs
module Test where
    double x = (+) x x
```

这里面顺便刚好介绍下运算符的前序用法，除了像x+x这样的中序用法，也可以将运算符放在前面，此时需要用括号括起来。

在hs文件中，用"--"来表示单行注释。"{- -}"来表示多行注释。

lhs则是带有文档的格式，代码需要用">"加上空格开头。
例：
```lhs
> module Test2 where
> double2 x = x * 2
```
对于lhs来说，因为是代码被标明，其余的全都是注释，也就不用专门的注释符号了。

head和tail用于取表头元素和除去表头剩下的部分：
```hs
*Test2> head l2
1
it :: Int
*Test2> tail l2
[2,3,4,5,6,7,8,9,10]
it :: [Int]
```

另外，还可以采用模式匹配的方法来实现取表头尾：
```hs
*Test2> let(h:t)=l2
h :: Int
t :: [Int]
*Test2> h
1
it :: Int
*Test2> t
[2,3,4,5,6,7,8,9,10]
it :: [Int]
```

取最后一个元素的话，使用last函数：
```
*Test2> l3
[1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0,10.0]
it :: [Double]
*Test2> last l3
10.0
it :: Double
```

可以通过length函数来求列表长度：
```hs
*Test2> length l2
10
it :: Int
```

## 函数

像上面的double和double2都是函数定义的方法。不需要fun和fn之类的关键字。

定义函数可以像上面一样直接定义。也可以采用模式匹配和哨兵表达式。

### 模式匹配

模式匹配就是列举出函数的各种情况，然后从前到后去挨个匹配。
我们举个例子计算斐波那契数列：
```lhs
> fib1 0 = 1
> fib1 1 = 1
> fib1 x = (+) (fib1(x-1)) (fib1(x-2))
```

我们还可以给函数加个类型声明：
```lhs
> fib1 :: Integer -> Integer
> fib1 0 = 1
> fib1 1 = 1
> fib1 x = (+) (fib1(x-1)) (fib1(x-2))
```

### 哨兵表达式

哨兵表达式是用布尔表达式来替代要匹配的内容。

```lhs
> fib2 x
>  | x<0 = 0
>  | x<2 && x>=0 = 1
>  | otherwise = (+) (fib2(x-1)) (fib2(x-2))
```

### lambda表达式

lambda表达式也就是匿名函数，在Haskell中使用"\\参数列表 -> 函数体"的方式表示。

我们来一个取列表中每个元素的倒数的例子：
```hs
*Test2> let l3 = [1..10::Double]
l3 :: [Double]
*Test2> l3
[1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0,10.0]
it :: [Double]
*Test2> map(\x -> 1/x)l3
[1.0,0.5,0.3333333333333333,0.25,0.2,0.16666666666666666,0.14285714285714285,0.125,0.1111111111111111,0.1]
it :: [Double]
```

匿名函数也可以用于filter函数，用于过滤列表中的内容：
```hs
*Test2> filter(\x -> x > 5)l3
[6.0,7.0,8.0,9.0,10.0]
it :: [Double]
```

光有map不行，还得有能reduce的函数啊。比如对列表求和，可以用sum函数：
```hs
*Test2> sum(l3)
55.0
it :: Double
```
也有求乘积的product函数：
```hs
*Test2> l3
[1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0,10.0]
it :: [Double]
*Test2> last l3
10.0
it :: Double
```

我们可以用fold系函数来完成这样的操作, foldl是从左往右计算。我们尝试用foldl来实现下product的功能：
```hs
*Test2> foldl (\x prod -> (*) prod x) 1 l3
3628800.0
it :: Double
```
其实根据交换律，从右往左算也没啥区别：
```hs
*Test2> foldr (\x prod -> (*) prod x) 1 l3
3628800.0
it :: Double
```

对于运算符，还不用这么复杂，有个foldl1:
```
*Test2> foldl1 (*) l3
3628800.0
it :: Double
```

## 生成可编译运行的代码

刚才我们都一直在ghci中工作，下面我们尝试可以通过ghc编译过的代码。
可编译的代码有两个特殊的要求：一个是要有Main模块作为入口；另一个是要有一个叫做main的IO Monad。
一听IO Monad很吓人啊，我们先看个例子：
```hs
module Main where
    double x = (+) x x
    main = putStrLn "Hello,World"
```
这样在不了解细节的情况下，看起来跟命令式语言好像也区别不太大哈。
我们调用ghc去编译，运行编译后的代码，就可以打印Hello,World了。

我们先来个lhs的例子：
```lhs
> module Main where
> double2 x = x * 2
> fact22 0 = 1
> fact22 x = (*) (x) (fact22(x-1))
> fact32 x 
>  | x > 1 = (*) x (fact32(x-1))
>  | otherwise = 1
> fib1 :: Integer -> Integer
> fib1 0 = 1
> fib1 1 = 1
> fib1 x = (+) (fib1(x-1)) (fib1(x-2))
> fib2 :: Integer -> Integer
> fib2 x
>  | x<0 = 0
>  | x<2 && x>=0 = 1
>  | otherwise = (+) (fib2(x-1)) (fib2(x-2))
> main = putStrLn "Welcome to Haskell"
```

## haskell的包管理工具

haskell使用cabal做为包管理工具。

以ubuntu和debian为例，可以通过
```
sudo apt install cabal-install
```
来安装cabal。
macOS上可以通过```brew install cabal-install```来安装。

然后就可以像其它工具一样使用cabal了，比如cabal update更新包列表，cabal install来安装包。

例：
```
cabal install yesod
```

要想查找包列表或者是搜索包，可以到[http://hackage.haskell.org/](http://hackage.haskell.org/)上查看。

如果想像python的virtualenv一样管理haskell，可以使用haskell-stack: [https://docs.haskellstack.org/](https://docs.haskellstack.org/). 

## 小结

作为一门委员会定义的纯函数语言，Haskell语言的初衷是可以成为函数式语言的通用平台。
我们这一节介绍了Haskell的基本特性，这些特性因为广泛地被近些年的新语言学习，所以其实我们并不陌生。关于函数式编程的方法，柯里化，类，monad等较深入的话题，我们在后面会慢慢展开。


