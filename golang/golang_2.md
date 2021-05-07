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

