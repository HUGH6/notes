# 复合数据类型

主要有四种：数组、slice、map和结构体。

数组和结构体都是有固定内存大小的数据结构，而slice和map是动态的数据结构，它们将根据需要动态增长。

## 数组

数组的长度是固定的，因此在golang中很少直接使用数组。和数组对应的类型是slice，它是可以增长和收缩的动态序列，功能也更灵活。

**数组下标访问**

数组的每个元素可以使用下标访问，下标从0开始。

**获取数组元素个数**

内置函数`len`可以获取数组中元素个数。

```go
var a [10]int
fmt.Println(a[0])
fmt.Println(len(a))

for index, ele := range a {
	fmt.Printf("%d %d\n", index, ele)
}

for _, ele := range a {
	fmt.Printf("%d\n", ele)
}
```

**数组初始化（零值/字面值）**

默认情况下，数组的每个元素被初始化为对应类型的零值。

也可以使用字面值语法用一组值来初始化数组。

```go
var q [3]int = [3]int{1,2,3}
var r [3]int = [3]int{5,2,1}
fmt.Println(r[2], q[2])
```

在使用字面值初始化数组时，可以在`[]`中使用更`...`符号，这样，数组长度会根据初始化的字面值的个数来计算，因此可以简化为如下写法。

```go
q2 := [...]int{9, 8, 7}
fmt.Println(len(q2))
```

**不同长度的数组是不同的数组类型**

数组的长度是数组类型的一部分，因此[3]int和[4]int是不同的数组类型。

```go
var q [3]int = [3]int{1,2,3}
q = [4]int{1,2,3,4}	// compile error
```

**数组长度需是常量**

数组的长度必须是常量表达式，因为数组的长度需要在编译阶段确定。

**直接指定字面值索引初始化数组**

可以直接指定一个索引和对应值列表的方式初始化数组，这种形式的初始化中，初始化索引的顺序是无关紧要的，而且没用到的索引可以省略，这些未指定的索引将用零值初始化。

```go
// 下例中定义了一个100长度的数组，除了索引99处初始化为2，其他索引处初始化为零值0
arr2 := [...]int{99: 2}
```

**数组间的比较**

如果一个数组的元素类型是可以相互比较的，那么这个数组类型也是可以相互比较的。

可以直接通过`==`运算符来比较两个数组。只有当两个数组的所有元素都相等的时候数组才是相等的。

```go
arr3 := [3]int{1,2,3}
arr4 := [3]int{1,2,3}
arr5 := [3]int{4,2,3}
fmt.Println(arr3 == arr4, arr3 == arr5, arr4 == arr5)	// true false false
arr6 := [2]int{1,2}
fmt.Println(arr5 == arr6) // compile error: cannot compare [2]int == [3]int
```

**函数传递数组参数**

当调用函数时，函数的每个调用参数会被赋值给函数内部的参数变量，所以函数参数变量接受的是一个赋值的副本，并不是原始调用的变量。

这种方式在传递数组时效率较低，某些编程语言会隐式地将数组作为引用或指针对象传入被调用的函数。golang对待带数组的方式与这些语言不同。

但可以显式地传递数组指针到函数中，这样，对数组的任何修改都可以反馈到调用者。

```go
func zero(ptr * [3]int) {
	for i := range ptr {
		ptr[i] = 0
	}
}
```

**数组是僵化的类型**

虽然可以通过指针来向函数传递数组参数，但由于数组类型包含来僵化的长度信息，因此，上例中的zero函数可以接受[3]int类型的数组指针，但不能接受[4]int类型的数组指针。

## slice

slice代表变长的序列，序列中每个元素都有相同的类型。一个slice一般写作[]T，slice的语法和数组很像，只是没有固定的长度。

**slice内部实现**

slice和数组之间有着紧密的关系。slice是一个轻量级的数据结构，提供来访问数组子序列元素的功能，而且==slice底层确实引用了一个数组==。

一个slice由三个部分构成：

* 指针：指针指向第一个slice元素对应的底层数组元素的地址（slice的第一个元素并不一定就是数组的第一个元素）。
* 长度：对应slice中元素的数目，长度不能超过容量。
* 容量：一般是从slice的开始位置到底层数组的结尾位置。

![2021-05-07_19-54](/home/huzihan/笔记/notes/golang/image/2021-05-07_19-54.png)

内置的len和cap函数分别返回slice的长度和容量。

```go
arr := [...]int{1,2,3,4,5,6,7,8,9}
s := arr[2:5]
r := arr[4:7]
fmt.Println(len(s))
fmt.Println(cap(s))
```

多个slice之间可以共享底层的数据，并且引用的数组部分区间可能重叠。

![2021-05-07_19-53](/home/huzihan/笔记/notes/golang/image/2021-05-07_19-53.png)

**创建slice**

例如有如下数组：

```go
s := [...]int{1,2,3,4,5}
```

操作s[i:j]用于创建一个新的slice，引用s的第i个元素到第j-1个元素的子序列。新的slice将只有j-i个元素。

如果i位置的索引被省略的话将使用0代替：`s[:j]`。

如果j位置的索引被省略的话将使用`len(s)`代替：`s[i:]`。

**扩展slice**

如果切片操作超出`cap(s)`的上限将导致一个panic异常，但是超出`len(s)`则意味着扩展了slice，因为新slice的长度会变大。

```go
arr := [...]int{1,2,3,4,5,6,7,8,9}
s := arr[2:4]

fmt.Println(len(s)) // 2
fmt.Println(cap(s)) // 7
fmt.Println(s) // [3 4]

fmt.Println(s[:20])	// panic: out of range

endlessArr := s[0: len(s) + 1] // 扩展了slice
fmt.Println(endlessArr) // [3 4 5]
```

**修改slice影响底层数组**

因为slice底层引用来数组，当对slice中的元素进行修改时，底层数组对应元素也会被修改。同时，其他应用这同一个底层数组的其他silce的对应元素也会修改。

```go
arr := [...]int{1,2,3,4,5,6,7,8,9}
slice1 := arr[2:4]                   // slice1: [3 4]
slice2 := slice1[0: len(slice1) + 1] // slice2: [3 4 5]

slice1[0] = -1        // 修改slice1
fmt.Println(slice1)   // slice1改变：[-1 4]
fmt.Println(arr)      // 底层数组改变：[1 2 -1 4 5 6 7 8 9]
fmt.Println(slice2)   // slice2改变：[-1 4 5]
```

**字符串切片/[]byte切片**

字符串的切片操作和[]byte字节类型切片的切片操作是类似的。都写作x[m:n]，并且都是返回一个元素字节序列的子序列，底层都是共享之前的底层数组。

x[m:n]切片操作对于字符串则生成一个新字符串，对[]byte则是生成一个新的[]byte。

**向函数传递slice**

向函数传递slice，既可以实现对底层数组的修改，也可以避免参数复制时复制整个数组的低效。

因为slice包含了指向底层数组第一个slice元素的指针，因此向函数传入slice，会复制slice，而这只是对底层数组创建了一个新的slice别名，因此这将允许函数内部修改底层数组的元素，且避免传参时直接复制整个数组。

```go
// 例子：向函数传递slice，通过修改slice来实现反转slice中的元素
// slice被反转，其底层数组也被反转
func rev(s []int) {
	for i, j := 0, len(s) - 1; i < j; i, j = i + 1, j - 1 {
		s[i], s[j] = s[j], s[i]
	}
}

arr := [5]int{1,2,3,4,5}
s := arr[:]

rev(s)
fmt.Println(s)	// [5 4 3 2 1]
fmt.Println(arr)// [5 4 3 2 1]
```

**slice与数组初始化语法的差异**

slice和数组的字面值初始化语法很类似，都是用花括号包含一系列的初始化元素，但slice并没有指明序列的长度，这会隐式地创建一个合适大小的数组，然后slice的指针指向底层数组。

```go
arr   := [3]int{1,2,3}	// 数组字面值初始化
slice := []int{1,2,3}	// slice字面值初始化
```

**slice的字面值可按顺序指定初始化序列或通过索引和元素值指定**

就像数组字面值一样，slice的字面值也可以按顺序指定初始化值序列，或者是通过索引和元素值指定，或者的两种风格的混合语法初始化。

```go
slice := []int{2:1,3:2,4:3}
fmt.Println(slice)	// [0 0 1 2 3]
```

**slice不能比较**

和数组不同的是，slice之间不能比较，因此不能使用`==`操作符判断两个slice是否含有全部相等的元素。

标准库提供来高度优化的`bytes.Equal`函数判断两个字节型slice是否相等，但对于其他类型的slice，我们必须自己展开每个元素进行比较。

```go
func equal(x, y []int) bool {
	if len(x) != len(y) {
		return false
	}

	for i := range x {
		if x[i] != y[i] {
			return false
		}
	}

	return true
}
```

slice唯一合法的比较操作是和nil比较：

```go
if slice == nil {/* ... */}
```

一个零值的slice等于nil。

一个nil值的slice并没有底层数组，其长度和容量都是0。但是也有非nil值的slice的长度和容量也是0的，比附[]int{}或make([]int,3)[3:]。与任意类型的nil值一样，我们可以用`[]int(nil)`类型转换表达式来生成一个对于类型slice的nil值。

如果需要判断一个slice是否为空，需要使用`len(s) == 0`来判断，而不应该用`s == nil`判断。

除了和nil相等外，一个nil值的slice的行为和其他任意0长度的slice一样。

**make函数创建slice**

内置的make函数创建一个指定元素类型、长度和容量 的slice。容量可以省略，这种情况下，容量等于长度。

```go
make([]T, len)
make([]T, len, cap)
```

在底层，make创建来一个匿名的数组变量，然后返回一个slice，只有通过返回的slice才能引用底层匿名的数组变量。

在第一种语句中，slice是整个数组的view。在第二种语句中，slice只引用来底层数组的前len个元素，但是容量将包含整个的数组，额外的元素是留给未来的增长用的。

### append函数

内置的append函数用于向slice追加元素。

```go
var sl []int
var ar [3]int = [3]int{1,2,3}

for _, el := range ar {
	sl = append(sl, el)
}
fmt.Println(sl)	// [1 2 3]
```

使用自己实现的appendInt函数演示append的内部大致过程：

```go
func appendInt(x []int, y int) []int {
	var z []int
	zlen := len(x) + 1
	if zlen <= cap(x) {
		z = x[:zlen]
	} else {
		zcap := zlen
		if zcap < 2*len(x) {
			zcap = 2 * len(x)
		}
		z = make([]int, zlen, zcap)
		copy(z, x)
	}
	z[len(x)] = y
	return z
}
```

每次调用appendInt函数，必须先检测slice底层数组是否有足够的容量来保存新添加的元素。如果有足够空间的话,直接扩展slice(依然在原有的底层数组之上)，将新添加的y元素复制到新扩展的空间，并返回slice。因此，输入的x和输出的z共享相同的底层数组。

如果没有足够的增长空间的话，appendInt函数则会先分配一个足够大的slice用于保存新的结果，先将输入的x复制到新的空间，然后添加y元素。结果z和输入的x引用的将是不同的底层数组。

内置的append函数可能使用比appendInt更复杂的内存扩展策略。因此，通常我们并不知道append调用是否导致了内存的重新分配，因此我们也不能确认新的slice和原始的slice是否引用的是相同的底层数组空间。同样，我们不能确认在原先的slice上的操作是否会影响到新的slice。因此，通常是将append返回的结果直接赋值给输入的slice变量:

```go
slice = append(slice, element)
```

更新slice变量不仅对调用append函数是必要的,实际上对应任何可能导致长度、容量或底层数组变化的操作都是必要的。要正确地使用slice,需要记住尽管底层数组的元素是间接访问的,但是slice对应结构体本身的指针、长度和容量部分是直接访问的。要更新这些信息需要像上面例子那样一个显式的赋值操作。从这个角度看,slice并不是一个纯粹的引用类型,它实际上是一个类似下面结构体的聚合类型:

```go
type IntSlice struct {
    ptr *int
    len, cap int
}
```

内置的append函数则可以追加多个元素,甚至追加一个slice。

```
var x []int
x = append(x, 1)
x = append(x, 2, 3)
x = append(x, 4, 5, 6)
x = append(x, x...)
fmt.Println(x)
```

通过对appendInt稍作修改，我们也可以实现类似特性：

```go
func appendInt(x []int, y ...int) []int {
	var z []int
	zlen := len(x) + len(y)
	// ... expand z to at least zlen...
	copy(z[len(x):], y)
	return z
}
```

## map

map是一个无序的key/value对的集合，其中所有的key都是不同的。通过给定的key，可以在常数时间复杂度内检索、更新或删除对应的value。

golang中，一个map就是一个哈希表的引用，map类型可以写为map[K]V，其中K和V分别对应key和value。

map中所有的key都是相同的类型，所有的value也是相同类型，但是key和value可以是不同类型。

map中的key必须是支持`==`比较运算符的数据类型，map可以通过测试key是否相等来判断是否已经存在。

**创建map**

内置的make函数可以创建一个map：

```go
m := make(map[string]int)
```

还可以用map字面值的方式创建map，这种方式可以指定一些初始的key/value：

```go
m := map[string]int{
    "a": 1,
    "b": 2,
}
```

相当于：

```go
m := make(map[string]int)
m["a"] = 1
m["b"] = 2
```

如果不指定初始的key/value，那么创建的是空map：

```go
m := map[string]int
```

**访问元素**

map中的元素可以通过key对应的下标语法来访问：

```go
m["a"] = 3
fmt.Println(m["a"])
```

**删除元素**

内置函数delete可以删除元素：

```go
delete(m, "a")
```

即使对应元素不在map中也没关系，这些操作是安全的。

**访问不存在元素返回零值**

如果一个查找失败将返回value类型对应的零值。

而且`x += y`和 `x++`等简短赋值语法也可以用在map上。

但有时可能需要判断一个元素是否真的存在map中，可以通过如下方式：

```go
key, ok := m["a"]
if !ok {/*... */}
```

经常可以看到这两个被结合起来使用：

```go
if key, ok := m["a"]; !ok {
	/*...*/
}
```

**禁止对元素取地址**

但map中的元素并不是一个变量，因此不能对它们进行取地址操作：

```go
_ = &m["a"] // compile error
```

禁止取址的原有是map可能随着元素数量的增长而重新分配更大的空间，从而导致原先的地址无效。

**遍历map中的key/value**

可以使用range风格的for循环遍历map中的key/value：

```go
for key, value := range m {
	fmt.Printf("%s\t%d\n", key, value)
}
```

map的迭代顺序是不确定的，并且不同的哈希函数可能导致不同的遍历顺序。在实践中，遍历的顺序是随机的，每次遍历的顺序都不相同。

如果想要按顺序遍历key/value，需要我们手动对key进行排序：

```go
import "sort"

var keys []string
for key := range m {
    keys = append(keys, key)
}
sort.Strings(keys)
for _, key := range keys {
    fmt.Printf("%s\t%d\n", key, m[key])
}
```

**map的零值是nil**

map类型的零值是nil，也即没有引用任何哈希表。

```go
var m map[string]int
fmt.Println(m == nil) // true
fmt.Println(len(m) == 0) // true
```

map上的大部分操作，包括查找、删除、len和range循环都可以安全地工作在nil值的map上，它们的行为和空的map一致。

但是向一个nil的map存入元素将导致panic异常。在向map存入数据前必须先创建一个map。

**map不能判等**

和slice一样，map不能判等，唯一的例外是和nil进行比较。

要判断两个map好似否包含同样的key和value，必须通过一个循环实现：

```go
func equal(x, y map[string]int) bool {
    if len(x) != len(y) {
        return false
    }
    for k, xv := range x {
        if yv, ok := y[k]; !ok || yv != xv {
            return false
        }
    }
    return true
}
```

**用map时下set**

golang中并没有set类型，但map中的key也是不相同的，可以用map来实现set的功能。

**map的key必须是可比较类型**

map的key必须是可比较类型，但有时我们需要一个map或者set的key是slice这类不能比较的类型，一般这种情况可以通过两个步骤绕过此项限制：

* 定义一个辅助函数k将slice转为string类型的key，确保只有x和y相等时，k(x)==k(y)才成立。
* 创建一个key为string类型的map，每次对map操作时先用辅助函数k将slice进行转换。

**map的value类型也可以是聚合类型**

map的value也可以是聚合类型，比如一个map或slice等。

```go
// 用来表示一个图
var gragh = make(map[string]map[string]bool)
```

## 结构体

结构体是一种聚合的数据类型，由零个或多个任意类型的值聚合成的实体。

```go
type Employee struct {
    ID		int
    Name	string
    Address	string
}
```

**访问成员变量**

结构体的成员遍历可以通过点操作符访问，并且由于结构体变量的成员也是变量，因此可以直接对每个成员赋值。

```go
var alice Employee
alice.Name = "alice"
alice.Address = "....."
```

也可以对结构体成员变量取地址，然后通过指针访问。

```go
address := &alice.Address
*address = "......"
```

点操作符也可以和指向结构体的指针一起工作。

```go
var p *Employee = &alice
p.Address = "......"

// 相当于
(*p).Address = "....."
```

**结构体成员的输入顺序**

结构体成员的顺序不同，就表明定义来不同的结构体类型。

**结构体成员的导出**

如果结构体成员的名字是大写的，那么该成员就是导出的。

**结构体的聚合**

一个命名S的结构体类型不能再包含S类型的成员，一个聚合的值不能包含它自身。

但是S类型的结构体可以包含*S指针类型的成员。

```go
type tree struct {
	value	int
	left, right	*tree
}
```

**结构体的零值**

结构体的零值是每个成员的零值。

**空结构体**

如果结构体没有任何成员的话就是空结构体，写作struct{}，大小为0，也不包含任何信息。

### 结构体字面值

结构体值也可以通过结构体字面值表示，结构体字面值可以指定每个成员的值。

```go
type Point struct {
	X, Y int
}

p := Point{1, 2}
```

有两种字面值语法，一种写法如上，要求以结构体成员定义的顺序为每个成员指定一个字面值。

```go
p := Point{1, 2}
```

更常用的是第二种语法，以成员名字和相应的值来初始化，可以包含部分或全部的成员。

这种形式下，如果成员被忽略将默认用零值。

```GO
p := Point{
	X: 1,
	Y: 2
}
```

两种形式的写法不能混用。

而且不能用第一种写法在外部包中来初始化结构体中为导出的成员。

**结构体可以作为函数的参数和返回值**

在golang中，所有函数参数都是值拷贝传入的，函数参数将不再是函数调用时的原始变量。

比较大的结构体通常会用指针的方式传入和返回，效率会更高。

如果要在函数内部修改结构体成员，则必须以指针传入。

### 结构体比较

如果结构体的全部成员都是可以比较的，那么结构体也是可以比较的，那么这两个结构体就可以使用==或!=运算符比较。

相等运算符将比较两个结构体的每个成员。

```go
type Point struct {X, Y int}

p := Point{1, 2}
q := Point{1, 3}

fmt.Println(p == q)
```

可比较的结构体和其他可比较的类型一样，可以用于map的key类型。

```go
type Point struct {X, Y int}

m := make(map[Point]int)
```

### 结构体嵌入和匿名成员

golang中一个命名的结构体可以包含另一个结构体类型的匿名成员，这样就也通过简单的点运算符x.f访问匿名成员链中嵌套的x.d.e.f成员。

普通的嵌套结构体如下：

```go
type Point struct {
	X, Y int
}

type Circle struct {
	Center Point
	Radius int
}
```

这种嵌套的方式使得结构体类型层次变得清晰，但是在访问每个成员时，也会比较繁琐：

```go
var c Circle
c.Point.X = 1
c.Point.Y = 2
```

golang有匿名嵌入的特性，可以使得我们只声明一个成员对应的数据类型而不指定成员的名字，这类成员就叫匿名成员。

匿名成员的数据类型必须是命名的类型或指向一个命名类型的指针。

```go
type Point struct {
    X, Y int
}

type Circle struct {
    Point
    Radius int
}
```

得益于匿名嵌入特性，我们可以直接访问叶子属性而不需要给出完整的路径：

```go
var c Circle
c.X = 1		// c.Point.X = 1
c.Y = 2		// c.Point.Y = 2
```

显示形式访问叶子成员的语法依然有效，因此匿名成员并不是真的无法访问了。

此时，匿名成员都有自己的名字，就是命名的类型名字，只不过这些名字在点操作符中是可选的。

不过，结构体字面值并没有简短表示匿名成员的语法，结构体字面值必须遵循类型声明时的结构（以下两种语法等价）：

```go
c := Circle{Point{1, 2}, 3}

c := Circle {
	Point: Point {
		X: 1,
		y: 2,
	},
	Radius: 3
}
```

由于匿名成员也有一个隐式的名字，因此不能同时包含两个相同类型的匿名成员，这会导致名字冲突。

同是，因为成员的名字是由其类型隐式地决定的，所有匿名成员也有可见性的规则约束。如果在包外部，匿名成员不是导出的，则简短的匿名成员访问语法也是禁止的。

匿名成员特性并不要求是结构体类型，任何命名的类型都可以作为结构体的匿名成员。为什么要嵌套一个没有任何子成员类型的匿名会曾有类型呢？答案是匿名类型的方法集。简短的点运算符语法可以用于选择匿名成员嵌套的成员，也可以用于访问它们的方法。实际上，外层的结构体不仅是获得了匿名成员类型的所有成员，而且也获得了该类型导出的全部方法。这个机制可以用于将一个有简单行为的对象组合成有复杂行为的对象。

组合是golang中面向对象编程的核心。