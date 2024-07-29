---
title: Go的容器数据类型
date: 2022-02-03 20:33:35
tags: 
  - Go
---


## Array—数组 

### 数组的理解

- 数组是具有相同唯一类型的一组已编号且长度固定的数据项序列。
- 上述唯一类型可以说his任意的原始类型如：整型、字符串或自定义类型
- 一个数组由零个或多个元素组成。
- 数组的大小确定、类型一致
- 注意Go 中的数组是值类型而不是引用类型。
- 长度一样且每个元素的类型和值也一样的数组，才能相等。
- 数组属于值类型的，即将一个数组赋值给另外一个数组的时候，实际上就是将整个数组拷贝一份。

```go
[n]elmentType  // n:数组长度、elementType：数组元素类型
```

#### 声明

- 需要指明数组的大小和存储的数据类型，其表示形式为 `[n]T`。`n` 表示数组中元素的长度值，`T` 代表每个元素的类型。
- 数组的长度在声明的时候一定要给定，而数组一旦声明，其长度和类型都不能改变。

```go
var <数组名称> [<数组长度>]<元素类型>
```

#### 初始化

- 初始化元素通过大括号包裹起来。
- 在简略声明中，不需要将数组中所有的元素赋值。

```go
var <数组名称> = [<数组长度>]<元素类型>{元素1,元素2,...}
```

#### 省略号

数组的长度可以使用省略号 `...` 代替，这个并不是代表可以省略数组长度的声明，编译器会自动计算数组长度，它的长度值等于当前数组初始化元素的个数。

#### 索引定义

```go
[<数组长度>]<元素类型>{索引1:元素1,索引2:元素2,…}
```

#### Practice-Code

```go
package main

import "fmt"

const (
	Frist int = iota
	Second
	Third
	Fourth
)

func main() {
	var a1 [3]int
	fmt.Println(a1) // [0 0 0]
	var a2 = [3]int{1, 2, 3}
	fmt.Println(a2) // [1 2 3]
	var a3 = [3]string{"a", "b", "c"}
	fmt.Println(a3) //[a b c]
	var a4 = [5]string{"r", "t", "s"}
	fmt.Println(a4) //[r t s  ]  s后面是两个空字符串
	a5 := [5]int{1, 3, 5}
	fmt.Println(a5) //[1 3 5 0 0]
	a6 := [3]int{7}
	fmt.Println(a6) //[7 0 0]
	a7 := [...]int{11, 22, 33}
	fmt.Println(a7) //[11 22 33]  [...]表示省略数组长度，编辑器会自动计算长度
	a8 := [4]string{Frist: "1", Second: "2", Third: "3", Fourth: "4"}
	fmt.Println(a8) //[1 2 3 4]
}

```

---

### 数组的常用操作

#### 比较

- 同样类型的数组是可以相互赋值的，相同类型的数组必须是长度一样，并且每个元素的类型也一样的数组

- 数组的大小是类型的一部分。因此 `[5]int` 和 `[25]int` 是不同类型。

#### 取值

- 当数组赋值给一个新的变量时，该变量会得到一个原始数组的一个副本。
- 如果对新变量进行更改，则不会影响原始数组。

#### 修改

- 取下标并赋值

####  获取长度

- 通过将数组作为参数传递给 `len` 函数，可以得到数组的长度。

#### for遍历

- `for` 循环可用于遍历数组中的元素。

#### range遍历

- `range` 返回索引和该索引处的值， 并且可以获取数组中所有元素的总和。
- 如果你只需要值并希望忽略索引，则可以通过用 `_` 空白标识符替换索引来执行。

#### Practice-Code

```go
arr[:]      //代表所有元素
arr[:5]     //代表前五个元素，即区间的左闭右开
arr[5:]     //代表从第5个开始（不包含第5个）
len(arr)    //数组的长度
//以上操作会引发类型的变化，数组将会转化为slice
```

```go
package main

import "fmt"

func main() {
	//数组的操作---比较
	//aa := [3]int{5, 78, 8}
	//var bb [5]int
	//bb = aa //cannot use aa (type [3]int) as type [5]int in assignment

	//数组的操作---取值
	cc := [...]string{"anhui", "jiangsu", "shanghai", "zhejiang"}
	dd := cc[1]
	fmt.Println(dd) //jiangsu

	//数组的操作---修改
	cc[1] = "beijing"
	fmt.Println(cc) //[anhui beijing shanghai zhejiang]

	//数组的操作---求长度
	ee := [...]float64{33.3, 45.2, 43, 89.1}
	fmt.Println("length of ee is", len(ee)) //length of ee is 4

	//数组的操作---for遍历
	for i := 0; i < len(ee); i++ {
		fmt.Printf("%d th element of ee is %.2f\n", i, ee[i])
		//0 th element of ee is 33.30
		//1 th element of ee is 45.20
		//2 th element of ee is 43.00
		//3 th element of ee is 89.10
	}

	//数组的操作---使用range遍历
	sum := float64(0)
	for ii, v := range ee {
		fmt.Printf("%d the element of ee is %.2f\n", ii, v)
		sum += v
	}
	fmt.Println("\nsum of all elements of ee", sum) //sum of all elements of ee 210.6

	//数组的操作---使用range遍历,且忽略索引只需要获取值
	for _, v := range ee {
		fmt.Println(v)
		//33.3
		//45.2
		//43
		//89.1
	}
}
```

---

### 数组的拓展

#### 数组作为函数参数

- 当数组作为参数传递给函数时，它是按照值传递，而原始数组保持不变

```go
package main

import "fmt"

func main() {
	array := [3]int{1:2}
	modify(array)
	fmt.Println("current list",array)  //current list [0 2 0]
}

func modify(a [3]int){
	a[1] =3
	fmt.Println("modify list",a)  //modify list [0 3 0]
}
```

- 传递数组的指针，可以实现修改数组值的

```go
package main

import "fmt"

func main() {
	array := [3]int{1:2}
	modify(&array)
	fmt.Println("current list",array)
}

func modify(a *[3]int){
	a[1] =3
	fmt.Println("modify list",*a)
}
```

#### 多维数组

```go
package main

import (
    "fmt"
)

func printarray(a [3][2]string) {
    for _, v1 := range a {
        for _, v2 := range v1 {
            fmt.Printf("%s ", v2)
        }
        fmt.Printf("\n")
    }
}

func main() {
    a := [3][2]string{
        {"lion", "tiger"},
        {"cat", "dog"},
        {"pigeon", "peacock"}, // this comma is necessary. The compiler will complain if you omit this comma
    }
    printarray(a)
    var b [3][2]string
    b[0][0] = "apple"
    b[0][1] = "samsung"
    b[1][0] = "microsoft"
    b[1][1] = "google"
    b[2][0] = "AT&T"
    b[2][1] = "T-Mobile"
    fmt.Printf("\n")
    printarray(b)
  //lion tiger
	//cat dog
	//pigeon peacock

	//apple samsung
	// microsoft google
	//AT&T T-Mobile
}
```

---



## Slice—切片

### 切片的理解

- slice是对数组的一个连续片段的引用，其本身并不是数组，它指向底层的数组。
- slice是一个引用类型。
- slice默认指向一段连续的内存区域，可以是数组，也可以是slice本身。
- 如果多个slice指向相同的底层数组，其中一个的值改变了会影响全部。
- 数组是值传递，slice是引用传递
- 切片的遍历可以使用for循环，也可以使用range函数

> Note: 
>
> - 使用make()创建， 
> - 使用len()获取元素个数，
> - cap()获取容量,一般从slice的开始位置直至底层数组的结束位置

### slice的存储结构体：

```go
type IntSlice struct {
  array = unsafe.Pointer    //指向底层数组的指针
  len int                   //切片元素数量    
  cap int                   //底层数组的容量
}
```

> Note:
>
> - 切片通过内部的指针和相关属性引用数组片段，实现了变长方案，Slice并不是真正意义上的动态数组。
>
> - 合理设置存储能力，可以大幅提升性能，比如知道最多元素个数为50，那么提前设置为50，而不是先设为30，可以明显减少重新分配内存的操作。

### 切片的创建

#### 方式一:

- var 声明一个新slice,未被初始化的slice 为nil

```go
var identifier []type
//fg:
// 声明字符串切片
var strList []string
// 声明整型切片
var numList []int
// 声明一个空切片
var numListEmpty = []int{}
```

#### 方式二：

- make,动态创建切片

```go
//make( []Type, size, cap )  
init_slice := make ([]string, cap)  //cap 为预分配的元素数量，这个值设定后不影响 size，只是提前分配空间，降低多次分配空间造成性能问题
//fg:
tmp_slice1 := make([]string, 4)
tmp_slice2 := make([]int, 2, 10)
```

#### 方式三：

- array——>slice

```go
// slice [开始位置 : 结束位置]
var names = [3]string{"bobo", "jake", "tom"}
subName := names[1:2]  // subName就是一个slice

```

#### 方式四:

- new

```go
subName1 := new([]string)
```

### 切片的常用操作

#### 关于切片的内置函数

```go
len()			                      //返回切片长度
cap()			                      //返回切片底层数组容量
append()		                    //对切片追加元素
func copy(dst, src []Type) int  //将src中数据拷贝到dst中，返回拷贝的元素个数
```

```go
// 声明一个数组
var array = [10]byte{'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'}
// 声明两个slice
var aSlice, bSlice []byte

// 演示一些简便操作
aSlice = array[:3] // 等价于aSlice = array[0:3] aSlice包含元素: a,b,c
aSlice = array[5:] // 等价于aSlice = array[5:10] aSlice包含元素: f,g,h,i,j
aSlice = array[:] // 等价于aSlice = array[0:10] 这样aSlice包含了全部的元素

// 从slice中获取slice
aSlice = array[3:7] // aSlice包含元素: d,e,f,g，len=4，cap=7
bSlice = aSlice[1:3] // bSlice 包含aSlice[1], aSlice[2] 也就是含有: e,f
bSlice = aSlice[:3] // bSlice 包含 aSlice[0], aSlice[1], aSlice[2] 也就是含有: d,e,f
bSlice = aSlice[0:5] // 对slice的slice可以在cap范围内扩展，此时bSlice包含：d,e,f,g,h
bSlice = aSlice[:] // bSlice包含所有aSlice的元素: d,e,f,g
```

#### append添加元素

- 如果最终长度未超过追加到slice的容量则返回原始slice，如果超过追加到的slice的容量则将重新分配内容地址并拷贝原始数据。
- 如果空间不足以容纳足够多的元素，切片就会进行“扩容”，此时新切片的长度会发生改变。容量的扩展规律是按容量的 2 倍数进行扩充，例如 1、2、4、8、16……
- 切片开头添加元素一般都会导致内存的重新分配，而且会导致已有元素全部被复制 1 次，因此，从切片的开头添加元素的性能要比从尾部追加元素的性能差很多。
- slice支持链式操作，可以将多个append操作组合起来

```go
//开头添加
var a = []int{1,2,3}
a = append([]int{0}, a...) // 在开头添加1个元素
a = append([]int{-3,-2,-1}, a...) // 在开头添加1个切片

//尾部添加
var a []int
a = append(a, 1) // 追加1个元素
a = append(a, 1, 2, 3) // 追加多个元素, 手写解包方式
a = append(a, []int{1,2,3}...) // 追加一个切片, 切片需要解包

//链式操作
var a []int
a = append(a[:i], append([]int{x}, a[i:]...)...) // 在第i个位置插入x
a = append(a[:i], append([]int{1,2,3}, a[i:]...)...) // 在第i个位置插入切片


//自动扩容
var numbers []int
for i := 0; i < 10; i++ {
    numbers = append(numbers, i)
    fmt.Printf("len: %d  cap: %d pointer: %p\n", len(numbers), cap(numbers), numbers)
}
// log
len: 1  cap: 1 pointer: 0xc0000ae2a0
len: 2  cap: 2 pointer: 0xc0000ae2b0
len: 3  cap: 4 pointer: 0xc0000c4020
len: 4  cap: 4 pointer: 0xc0000c4020
len: 5  cap: 8 pointer: 0xc0000b00c0
len: 6  cap: 8 pointer: 0xc0000b00c0
len: 7  cap: 8 pointer: 0xc0000b00c0
len: 8  cap: 8 pointer: 0xc0000b00c0
len: 9  cap: 16 pointer: 0xc0000c6000
len: 10  cap: 16 pointer: 0xc0000c6000

```

#### 删除元素

- go语音并并没有对删除slice元素提供专用的语法或者接口

- 如果有删除的需求，需要使用slice本身的特性来删除元素

- 删除slice元素的本质是：以被删除元素为分界点，将前后两个部分的内存重新连接起来

  ![](https://raw.githubusercontent.com/ronin-sswang/upload_notes_img/main/img/202109190051698.png)

```go
//从开头删除
a := []int{1, 2, 3}
 // 删除开头1个元素
a = a[1:] // [2,3]
a = append(a[:0], a[1:]...) // 删除开头1个元素
a = append(a[:0], a[N:]...) // 删除开头N个元素
//使用copy()函数删除开头的元素
a := []int{1, 2, 3}
a = a[:copy(a, a[1:])] // 删除开头1个元素
a = a[:copy(a, a[N:])] // 删除开头N个元素

//从中间位置开始删除
a = []int{1, 2, 3, ...}
a = append(a[:i], a[i+1:]...) // 删除中间1个元素
a = append(a[:i], a[i+N:]...) // 删除中间N个元素
a = a[:i+copy(a[i:], a[i+1:])] // 删除中间1个元素
a = a[:i+copy(a[i:], a[i+N:])] // 删除中间N个元素

//从尾部删除
a = []int{1, 2, 3}
a = a[:len(a)-1] // 删除尾部1个元素
a = a[:len(a)-N] // 删除尾部N个元素
```

#### copy()拷贝

- copy()可以将一个切片复制到另外一个切片中，返回值表示实际发生复制的元素个数
- 如果加入的两个切片不一样大，就会按照其中较小的那个切片的元素个数进行复制
- 目标切片必须分配过空间且足够承载复制的元素个数，并且来源和目标的类型必须一致

```go
// 将 srcSlice 复制到 destSlice
copy( destSlice, srcSlice []T) int

//切片大小不一致
slice1 := []int{1, 2, 3, 4, 5}
slice2 := []int{5, 4, 3}
copy(slice2, slice1) // 只会复制slice1的前3个元素到slice2中
fmt.Println(slice2)
copy(slice1, slice2) // 只会复制slice2的3个元素到slice1的前3个位置
fmt.Println(slice1)
// log
[1 2 3]
[1 2 3 4 5]

// 复制对原值影响
unc main() {
    // 设置元素数量为1000
    const elementCount = 1000
    // 预分配足够多的元素切片
    srcData := make([]int, elementCount)
    // 将切片赋值
    for i := 0; i < elementCount; i++ {
        srcData[i] = i
    }
    // 引用切片数据
    refData := srcData
    // 预分配足够多的元素切片
    copyData := make([]int, elementCount)
    // 将数据复制到新的切片空间中
    copy(copyData, srcData)
    // 修改原始数据的第一个元素
    srcData[0] = 999
    // 打印引用切片的第一个元素
    fmt.Println(refData[0])
    // 打印复制切片的第一个和最后一个元素
    fmt.Println(copyData[0], copyData[elementCount-1])
    // 复制原始数据从4到6(不包含)
    copy(copyData, srcData[4:6])
    for i := 0; i < 5; i++ {
        fmt.Printf("%d ", copyData[i])
    }
}
// log


```

#### 字符串转切片

```go
str := "hello,世界"
a := []byte(str)		//字符串转换为[]byte类型切片
b := []rune(str)		//字符串转换为[]rune类型切片
```

#### 切片作为函数参数

```go
func test(s []int) {
	fmt.Printf("test---%p\n", s) // 打印与main函数相同的地址
	s = append(s, 1, 2, 3, 4, 5)
	fmt.Printf("test---%p\n", s) // 一旦append的数据超过切片长度，则会打印新地址
	fmt.Println("test---", s)    // [0 0 0 1 2 3 4 5]
}

func main() {

	s1 := make([]int, 3)
	test(s1)
	fmt.Printf("main---%p\n", s1) // 不会因为test函数内的append而改变
	fmt.Println("main---", s1)    // [ 0 0 0]
}
```



#### 多维slice

```go
//sliceName 为切片的名字
//sliceType为切片的类型，每个[]代表着一个维度，切片有几个维度就需要几个[]
var sliceName [][]...[]sliceType

// 声明一个二维整型切片并赋值
slice := [][]int{{10}, {100, 200}}
// 为第一个切片追加值为 20 的元素
slice[0] = append(slice[0], 20)
```

---



## Map—字典

> Go- map ---> python-dict

---

### map的理解

- map是一个无序键值对集合
- map的长度是不固定的，也就是和slice一样，也是一种引用类型
- 内置的len函数同样适用于map，返回map拥有的key的数量
- map和其他基本型别不同，它不是thread-safe，在多个go-routine存取时，必须使用mutex lock机制

### map的创建

- 要求所有的key的数据类型都相同，所有value的数据类型都相同
- key和value的类型可以为不同的数据类型

```go
package main

import "fmt"

func main() {
	//常规创建
	m1 := map[string]string{
		"m1": "v1",
	}
	fmt.Printf("%v\n", m1) //map[m1:v1]

	//使用make函数创建，make可以创建slice/map
	m2 := make(map[string]string)
	m2["m2"] = "v2"
	fmt.Printf("%v\n", m2) //map[m2:v2]

	//定义一个空map
	m3 := map[string]string{}
	m4 := make(map[string]string)
	fmt.Printf("%v\n", m3) //map[]
	fmt.Printf("%v\n", m4) // map[]
}
```

---

### Map 中key的类型

- map中的key 不是所有的类型都支持，该类型需要支持 == 或 != 操作，[]内的类型指任意可以进行比较的类型
- key的常用类型：
  - int
  - rune
  - string
  - 基于上述类型自定义的类型

```go
package main

import "fmt"

func main() {
	// map中key的类型
	// key为切片类型---不支持
	//a1 := []int{1, 2, 3}
	//b1 := []int{1, 2, 3}
	//if a1 == b1 {  //编译不通过
	//}

	// key 为数组类型（因为数组是值类型）--- 支持
	a1 := [3]int{1, 2, 3}
	b1 := [3]int{1, 2, 3}
	if a1 == b1 { //编译通过
		fmt.Println("true")
	}

	//interface{}类型可以作为key，但是需要加入的key的类型是可以比较的
	//var m5 map[interface{}]string
	//m6 = make(map[interface{}]string)
	//m5[[]byte("k2")] = "v2" // panic: runtime error: hash of unhashable type []uint8
	//m5[123] = "123"
	//m6[12.3] = "123"
	//fmt.Println(m5)
	//fmt.Println(m6)

	// book里面的元素都是支持== !=
	type book struct {
		name string
	}
	var m7 map[book]string
	fmt.Println(m7) //map[]
	}
}
```

---

### map的常用操作

#### 增删改查

```go
package main

import "fmt"

func main() {
	// map的增删改查
	//创建
	mm := map[string]string{
		"a": "va",
		"b": "vb",
	}
	fmt.Println(len(mm)) //2

	// 增加、修改
	// k不存在为增加，k存在为修改
	mm["c"] = ""
	mm["c"] = "11"                       // 重复增加(key相同)，使用新的值覆盖
	fmt.Printf("%#v %#v\n", mm, len(mm)) // map[string]string{"a":"va", "b":"vb", "c":"11"} 3

	// 删除
	// delete(m, k) 将k以及k对应的v从m中删掉；如果k不在m中，不执行任何操作
	delete(mm, "x")                      // 删除不存在的key,原m不影响
	delete(mm, "a")                      // 删除存在的key
	fmt.Printf("%#v %#v\n", mm, len(mm)) // map[string]string{"b":"vb", "c":"11"} 2
	delete(mm, "a")                      // 重复删除不报错,m无影响
	fmt.Printf("%#v %#v\n", mm, len(mm)) /// map[string]string{"b":"vb", "c":"11"} 2

	// 查找
	// v := m[k] // 从m中取键k对应的值给v，如果k在m中不存在，则将value类型的零值赋值给v
	// v, ok := m[k] // 从m中取键k对应的值给v，如果k存在，ok=true,如果k不存在，将value类型的零值赋值给v同时ok=false
	// 查1 - 元素不存在
	v1 := mm["x"] //
	v2, ok2 := mm["x"]
	fmt.Printf("%#v %#v %#v\n", v1, v2, ok2) // "" "" false

	// 查2 - 元素存在
	v3 := mm["a"]
	v4, ok4 := mm["a"]
	fmt.Printf("%#v %#v %#v\n", v3, v4, ok4) //"va" "va" true
}

```

---

#### 遍历

- map的遍历是无序的，固在遍历是不能通过index获取，二必须通过key获取
- 使用for range遍历时，k,v 使用同一块内存

```go
package main

import "fmt"

func main() {
	//遍历--遍历顺序随机，k,v使用的同一块内存
	ma := map[string]int{
		"a": 1,
		"b": 2,
		"c": 3,
	}
	for k, v := range ma {
		fmt.Printf("k:[%v]. v:[%v]\n", k, v)
		//k:[a]. v:[1]
		//k:[b]. v:[2]
		//k:[c]. v:[3]
	}
}
```

