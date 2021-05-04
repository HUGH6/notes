# 概述
golang是一门编译型语言，golang的工具链将源代码及其依赖转换成计算机的机器指令。

golang提供的工具都通过一个单独的go命令调用，go命令有许多子命令。

最简单的一个子命令就是run，该命令编译一个或多个以.go结尾的源文件，链接库文件，并运行最终生成的可执行文件。

```shell
go run helloword.go
```

golang原生支持unicode，可以处理全世界任何语言的文本。

build子命令可以生成可执行的二进制文件。

```
go build helloworld.go
```

golang的代码通过包（package）组织，一个包由单个目录下的一个或多个.go源文件组成，目录定义包的作用。

每个源文件都已一条package声明语句开始，表示该文件属于拿个包。紧跟着一些列导入（import）的包，之后就是存储在这个文件里的程序语句。

main包比较特殊，它定义量一个独立可执行的程序，而不是一个库。在main里的main函数也很特殊，它是整个程序秩序的入口。

import语句导入源文件依赖的包。

必须恰当导入需要的包，缺少里必要的包或者导入了不需要的包，程序都无法编译通过。这要求程序开发过程中不能引入未使用的包。

import声明必须跟在package声明之后。

函数的声明由func关键字、函数名、参数列表、返回值列表以及包含在大括号里的函数体组成。

golang不需要在语句或声明的末尾添加分号，除非一行上有多条语句。

## 命令行参数

程序的命令行参数可以从os包的Args变量获取，os包外部使用os.Args访问该变量。

os.Args是一个字符串的切片。

os.Args的第一个元素os.Args[0]是命令本身的名字，其他元素则是程序启动时传给它的参数。

## 注释

//

/* ... */

## 变量声明

var声明变量，变量会在声明时直接初始化，如果变量没有显示初始化，则被隐式赋予其类型的零值。

:=为短变量声明，只能用在函数内部，不能用于包变量。

## 运算符

golang提供里常规的数值和逻辑运算符，对string类型，+运算符链接字符串。

每次循环迭代，range产生一对值：索引以及该索引处的元素值。

空标识符（_）：可用于任何语法需要变量名但逻辑不需要的时候。

## 协程

goroutine是一种函数并发执行方式，channel是用来在goroutine之间进行参数传递的。

main函数本身也运行在一个goroutine中，而go function表示创建一个新的goroutine，并在新的goroutine中执行这个函数。

创建channel：

```go
ch := make(chan string)
```

当一个goroutine尝试在一个chennel上做send或receive操作时，这个goroutine会阻塞在调用处，直到另一个goroutine往这个channel里写入、或接受值。

# 程序结构

## 命名

golang中所有命名（函数、变量、常量、类型、语句标号、包名）都遵循一个简单的命名规则：名字必须以一个字母或下划线开头，后面可以跟任意数量的字母、数字或下划线，区分大小写。

golang的关键字不能用于自定义名字。

此外golang还有一些预定义的名字，虽然可以在定义中重新使用它们，但是应避免过度而导致语义混乱。

如果一个名字是在函数中定义，则其只在函数中有效。如果是在函数外部定义，则将在当前包的所有文件中都可以访问。

名字开头字母的大小写决定了名字在包外的可见性，如果一个包级名字开头字母大写，则在包外也可以被访问。

习惯上采用驼峰式命名。

## 声明

golang有四种声明：var、const、type和func，分别对应变量、常量、类型和函数实体对象的声明。

一个函数的声明由函数名字、参数列表、一个可选的返回值列表和包含的函数定义的函数体组成。如果函数没有返回值，那么返回值列表是省略的。

## 变量

var声明可以创建一个特定类型的变量，然后设置变量的初始值。

```golang
var 变量名 变量类型 = 表达式
```

“类型”或”表达式“两部分可以省略其中一个。

* 如果省略”类型信息“，那么将根据初始化的表达式来推导变量类型。

* 如果”表达式“被省略，那么将用零值来初始化该变量。

零值：

* 数值变量：0

* 布尔类型变量：false

* 字符串类型：""

* 接口或引用类型（包括slice、指针、map、chan和函数）变量对应的零值：nil

* 数组或结构体等聚合类型对应的零值是每个元素或字段都是对应该类型的零值。

也可以在一个声明语句中声明一组变量，或用一组初始化表达式声明并初始化一组变量。

如果省略每个变量的类型，则可以声明多个类型不同的变量变量类型由表达式推导。

```
var i, j, k int
var b, f, s = true, 2, "four"
```

一组变量也可以通过调用一个函数，由函数返回的多个返回值初始化。

```
var f, err = os.Open(name)
```

在包级声明的变量会在main入口函数执行前完成初始化，局部变量将在声明语句被执行到的时候完成初始化。

### 简短变量声明

`名字 := 表达式`的形式叫做简短变量声明语句，可用于声明和初始化局部变量，变量类型根据表达式自动推导。

简短变量声明也可以用于声明和初始化一组变量。

```
i, j := 0, 1
```

注意：不要混淆多个变量的声明和赋值

```
i, j := 1, 3 // 声明
a, b = b, a // 交换
```

简短变量声明也可以用函数返回值来声明和初始化变量。

```
i, err := os.Open(name)
```

注意：

* 简短声明变量语句左边的变量可能并不是全部刚刚声明，如果有一些变量以及在相同的词法域声明过了，那么简短声明语句对这些变量就只有赋值行为。
* 简短声明语句中至少要声明一个新的变量。

### 指针

一个变量对应来一个保存对应类型值的内存空间。一个指针的值是一个变量的地址。一个指针对应变量在内存中的存储位置。通过指针，我们可以直接读取或更新对应变量的值，而不需要知道该变量的名字。

使用`&变量名`将产生一个指向该变量的指针，指针的具体类型为`* 变量类型`。

假设变量p是一个指针，一般`*p`表达式读取指针指向的变量的值。同时，因为`*p`对应一个变量，所以该表达式也可以放在赋值语句左边，表示更新指针所指向的变量的值。

任何类型的指针的零值都是nil，如果指针变量p指向某个有效变量，那么`p != nil`测试为真。指针之间也可以进行相等测试，只有当它们指向同一个变量或全部是nil时才相等。

在golang中，返回函数中局部变量的地址也是安全的。

```
var p = f()

func f() *int {
	v := 1
	return &v
}
```

如果将指针作为函数参数，那么在函数中可以通过该指针来修改变量的值。

```go
func incr(p *int) int {
	*p++
	return *p
}

v := 1
incr(&v)
fmt.Println(incr(&v))
```

### new函数

另一个创建变量的方法是调用内建的new函数。表达式`new(T)`将创建一个类型为T的匿名变量，初始化为T类型的零值，然后返回变量的地址，返回的指针类型为`*T`。

```
p := new(int)
fmt.Println(*p)
*p = 2
fmt.Println(*p)
```

用new创建变量和普通变量声明没什么区别，除了不需要声明一个临时变量的名字外，`new(T)`还可以在表达式中使用。换而言之，new函数类似于一种语法糖。

每次调用new函数都是返回一个新的变量地址。一种特殊情况除外：

如果两个类型都是空的，也就是说类型的大小是0，例如struct{}和[0]int，有可能有相同的地址（依赖具体的语言实现）。

### 变量的生命周期

变量生命周期值程序运行期间比那里有效存在的时间间隔。

包一级的变量的生命周期和整个程序运行周期一致。

局部变量声明周期则是动态的：每次从创建一个新变量的声明语句开始，直到该变量不再被引用位置，然后变量的存储空间可能被回收。

函数的参数变量和返回值变量都是局部变量，它们在函数每次被调用时创建。

> 如何判断变量何时被回收？
>
> 可达性分析：
>
> 从每个包级变量和每个当前运行函数的每个局部变量开始，通过指针或引用的访问路径遍历，是否可以找到该变量，如果不存在这样的路径，则说明变量不再被引用，可以被回收。

因为一个变量的有效周期只取决于是否可达，因此，一个循环迭代内部的局部变量的生命周期可能超出其局部作用域，同时，局部变量可能在函数返回之后依然存在。

编译器会自动选择在栈上还是堆上分配局部变量的存储空间。

逃逸的变量需要额外分配内存，同时对性能的优化可能会产生细微的影响。

## 赋值

使用赋值语句可以更新变量的值。

* 最简单的赋值语句是`变量 = 表达式`。

* 特定的二元算术运算符和赋值语句的符合操作有一个简洁的形式：

```
a *= 1
```

* 数值变量也支持`++`和`--`语句（注意，golang中自增和自减是语句，不是表达式，因此不能被嵌套在其他表达式中）。

### 元组赋值

元组赋值是另一种形式的赋值语句，可以同时更新多个变量的值。

赋值前，赋值语句右边的所有表达式将会先进行求值，然后统一对左边的变量的值进行更新。

```
x, y = y, x
a, b = 2, 4
```

```
// 求最大公约数
func gcd(x, y int) int {
	for y != 0 {
		x, y = y, x % y
	}
	return x
}
```

有些表达式或函数会产生多个值，这类表达式或函数出现在元组赋值语句中时，必须保证等号左右两边数目一致。

和变量声明一样，可以使用`_`来丢弃不需要的值。

### 可赋值性

赋值语句是显式的赋值形式，但程序中还有很多地方会发生隐式的赋值行为。

函数调用会隐式地将调用参数的值赋值给函数的参数变量。

一个返回语句会隐式地将返回操作的值赋值给结果变量

一个复合类型的字面量也会参数赋值行为：

```
medals := []string{"gold", "silver"}
```

隐式地对slice的每个元素进行赋值操作：

```
medals[0] = "gold"
medals[1] = "bronze"
```

map和chan也有类似地隐式赋值行为。

可赋值性规则：类型必须完全匹配，nil可以赋值给任何指针或引用类型的变量。

## 类型

一个类型声明语句创建了一个新的类型名称，和现有类型具有相同的底层结构。新命名的类型提供来一个方法，用来分隔不同的概念的类型，这样即使它们底层类型相同也是不兼容的。

```
type 类型名字 底层类型
```

对于每一个类型T都有一个转型操作T(x)，用于将x转型为T。只有当两个类型底层基础类型相同时，才允许这种转型，或者两者都是指向相同底层结构的指针类型，这些转型只改变类型而不影响值本身。

数值类型之间的转型也是允许的，并且在字符串和一些特定类型的slice之间也是可以转换的。这类转型可能改变值的表现。（如将一个浮点数转换为整数将丢弃小数部分，将字符串转为[]byte类型的slice将拷贝一个字符串数据的副本）

任何情况下，运行时都不会发生转换失败的错误（错误只会发生在编译阶段）

底层数据结构决定来内部结构和表达方式，也决定来是否可以像底层类型一样对内置运算符的支持。

命名类型还可以为该类型定义新的行为。这些行为表示为一组关联到该类型的函数集合，称为类型的方法集。

```
func (c Celsius) String() string {
	return fmt.Sprintf("%g", c)
}
```

# 包和文件

golang中的包和其他语言的库或模块的概念类似。一个包的源代码保存在一个或多个.go文件中，通常一个包所在目录路径的后缀是包的导入路径。

每个包都对应一个独立的名字空间。

包还可以通过控制哪些名字是外部可见的来隐藏内部实现细节。其规则是：如果一个名字是大写字母开头，那么该名字导出。

## 导入包

在golang中，每个包都是有一个全局唯一的导入路径。

除了包的导入路径，每个包还有一个包名，包名一般是短小的名字，包名在包的声明处指定。按照惯例，一个包的名字和包的导入路径的最有一个字段相同。

如果导入来一个包但是又没有使用该包，将被当作一个编译错误处理。

## 包的初始化

包的初始化首先是解决包级变量的依赖顺序，然后按照包级变量声明出现的顺序依次初始化：

```
var a = b + c	// a 第三个初始化
var b = f()		// b 第二个初始化
var c = 1		// c 第一个初始化

func f() int {
	return c + 1
}
```

如果包中有多个.go源文件，它们将按照发送给编译器的顺序进行初始化，go语言的构建工具首先会将.go文件根据文件名排序，然后依次调用编译器编译。

对于在包级别声明的变量，如果又初始化表达式，就用表达式初始化，还有一些没有初始化表达式的，可以用一个特殊的`init()`初始化函数来简化初始化工作。每个文件都可以包含多个`init()`初始化函数。

```
func init() { /* ... */ }
```

这样的`init()`函数除了不能被调用或引用之外，其他的和普通函数类似。每个文件中的`init()`初始化函数，在程序开始执行时按照它们声明的顺序被自动调用（注意：在程序开始执行前才执行，比普通包级变量的初始化晚）。

每个包在解决一类的前提下，以导入声明的顺序初始化，每个包只会被初始化一次（如果p包导入来q包，那么在p包初始化时可以认为q包必然初始化过了）。

初始化工作是自下而上进行的，main包最后被初始化，这种方式可以保证在main函数执行前所有依赖的包都已经完成了初始化。

## 作用域

声明语句的作用域指源代码中可以有效使用这个名字的范围。

注意：不要将作用域和生命周期混为一谈。

句法块内部声明的名字是无法被外部访问的。这个块决定来内部声明的名字的作用域。

可以将块的概念推广到包括其他声明的群众，这些声明在代码中并未显式地用花括号裹起来，称之为词法块。对于全局源代码来说，存在一个整体的词法块，称为全局词法块。对于每个包，每个for、if、switch语句，也都对应词法块。每个switch和select分支也有独立的语法块。当然也包括显式书写的词法块。

# 基础数据类型

golang将数据类型分为四类：基础类型、复合类型、引用类型和接口类型。

基础类型包括：数字、字符和布尔型。

复合类型：结构体和数组。

引用类型：指针、slice、map、函数、chan

## 整形

golang数值类型包括几种不同大小的整数、浮点数、和复数。

int8、int16、int32、int64四种不同大小的有符号整数类型，分别对应8、16、32、64bit大小的有符号整数。

此外还有uint8、uint16、uint32、uint64四种无符号整数类型。

还有两种一般对应特定cpu平台机器字大小的有符号和无符号整数int和uint，这两种类型都有相同的大小，32或64bit，但我们不能对此做任何的假设，因为不同的编译器即使在相同的硬件平台上可能产生不同的大小。

unicode字符rune类型是和int32等价的类型，通常用于表示一个unicode码点。这两个名称可以互换使用。

byte也是uint8类型的等价类型，byte类型一般用于强调数值是一个原始的数据而不是一个小的整数。

还有一种无符号的整数类型uintptr，没有指定具体的bit大小但是足以容纳指针。uintptr类型只有在底层编程时才需要。

不管它们的具体大小，int、uint和uintptr是不同类型的兄弟类型。其中int和int32也是不同的类型，即使int的大小也是32bit，在需要将int当作int32类型的地方需要一个显式的类型转换操作，反之亦然。

其中，有符号整数采用2的补码形式表示，最高bit为用来表示符号位，一个n-bit的有符号数的值域是$-2^{n-1}到2^{n-1}-1 $ 。

无符号整数的所有bit位都用于表示非负数，值域是0到$2^{n}-1$。

go语言中，%取模运算法仅用于整数间的运算，%取模运算符的符号和被取模数的符号总是一致的。

`&^`运算符用于按位置零。

在`x<<n`和`x>>n`位移运算中，决定了移位操作bit数部分必须是无符号数，被操作的x数可以是有符号或无符号数。

算术上一个`x<<n`左移运算等价于乘以$2^n$，一个`x>>n`右移运算等价于除以$2^n$。左移运算用0填充空缺的bit位，无符号数的右移运算也是用0填充左边空缺的bit位，但是有符号数的右移运算会用符号位的值填充左边空缺的bit位。

## 浮点数

golang提供了两种精度的浮点数，float32和float64。

## 复数

golang提供零两种精度的复数：complex64和complex128。内置的complex函数用于构建复数，内建的real和imag函数分别返回复数的实部和虚部。

## 布尔型

布尔类型只有两种：true和false。

布尔值可以和&&和||操作符结合，并且有短路行为。

## 字符串

一个字符串十一个不可改变的字节序列。

字符串可以包含任意的数据，包括byte值0。文本字符串通常被解释位采用utf8编码的unicode码点rune序列。

内置的len函数可以返回一个字符串中的字节数目（不是rune字符数目），索引操作s[i]返回第i各字节的字节值。

如果试图访问超出字符串索引范围的字节将会导致panic异常。

字符串的值是不可变的，一个字符串包含的字节序列永远不会被改变，当然，我们可以给一个字符串变量分配一个新字符串。当使用+向字符串追加字符时，会创建新字符串值。尝试修改字符串内部数据的操作也是被禁止的。

### 字符串面值

字符串值也可以用字符串面值方式编写。

```
"hello, world"
```

一个原生的字符串面值形式是`\`...`\`，使用反引号代替双引号。在原生字符串面值中，没有转义操作，全部的内容都是字面的意思，包含退格和换行。

### unicode

unicode码点对应golang中的rune整数类型。

### utf-8

utf-8是将unicode码点编码为字节序列的变长编码。

## 常量

常量表达式的值在编译期计算，而不是在运行期。每种常量的潜在类型都是基础类型：boolean、string或数字。

常量的值不可修改。

```
const pi = 3.14
```

和变量声明一样，常量可以批量声明。

```
const (
	e = 2.7
	pi = 3.14
)
```

所有常量的计算都可以在编译期完成，这可以减少运行时的工作，也方便其他编译优化。

常量间的所有算术运算、逻辑运算和比较运算的结果也是常量，对常量的类型转换或以下函数调用都是返回常量结果：len、cap、real、imag、complex和unsafe.Sizeof。

因为它们的值是在编译期就确定的，隐藏常量可以是构成类型的一部分。

常量声明时如果没有显式指定类型，那么会从右侧表达式进行推断。

如果是批量声明的常量，除了第一个外其他的常量右边的初始化表达式都可以省略，如果省略初始化表达式则表示使用前面常量的初始化表达式写法，对应的常量也一样。

```
const (
	a = 1
	b
	c = 2
	d
)
```

### iota常量生成器

常量声明可以使用iota常量生成器初始化，它用于生成一组以相似规则初始化的常量，但是不用每行都写一遍初始化表达式。

在一个const声明语句中，在第一个声明的常量所在的行，iota将会被置为0，然后在每一个常量声明的行加一。

```
type Weekday int

const (
	Sunday Weekday = iota
	Monday
	Tuesday
	Wednesday
	Thursday
	Friday
	Saturday
)
```

### 无类型常量

大多数情况下，Go 常量在声明时并不显式指定类型，也就是说使用的是无类型常量。

有六种未明确类型的常量类型，分别是无类型的布尔型、无类型的整数、无类型的字符、无类型的浮点数、无类型的复数、无类型的字符串。

通过延迟明确常量的具体类型，无类型常量不仅可以提高更高的运算精度，而且可以直接用于更多的表达式而不需要显式的类型转换。

对于常量面值，不同的写法可能会对应不同的类型，例如0、0.0、0i和\0000虽然相同的常量值，但它们分别对应无类型的整数、无类型的浮点数、无类型的复数和无类型的字符等常量类型。同样，true和false也是物理学的布尔类型，字符串面值常量是无类型的字符串类型。

只有常量可以是无类型的，当一个无类型常量被赋值给一个变量时，无类型的常量将会被隐式转换为对应的类型。

注意：无类型整数常量转换为int，期内存大小是不确定的，但无类型浮点数和复数常量则转换为内存大小明确的float64和complex128.