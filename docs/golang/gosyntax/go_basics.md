---
title: Go的基本知识点
date: 2022-02-01 19:43:35
tags: 
  - Go
---


## 25个关键字

```go
break      default       func     interface   select
case       defer         go       map         struct
chan       else          goto     package     switch
const      fallthrough   if       range       type
continue   for           import   return      var
```



---

## 内建类型

```go
int       int8          int16         int32        int64
uint      uint8         uint16        uint32       uint64   uintptr
float32   float64       complex128    complex64
bool      byte(uint8)   rune(int32)   string       error
```



---

## 内建常量

```go
true false iota nil
```



---

## 内建函数

```go
make      len      cap    new     append    copy     close  delete
complex   real     imag
panic     recover
```



## 常用基本命令

```
go build main.go 
  编译源码文件和当前目录的代码包
  编译命令源码文件后，创建一个可执行文件
  -o: output 指定编译输出的名称，代替默认的包名。
  -i: install 安装作为目标的依赖关系的包(用于增量编译提速)。
  
go run main.go 可以直接运行 Go 程序。
  用于运行命令源码文件
  只能接受一个命令源码文件以及若干个库源码文件作为文件参数
  -a 强制编译相关代码
  -p [n] 并行编译
  -v 展示被编译的代码包名称
  -work 不删除创建的临时目录  
  
go install
  编译和安装代码包或源码文件
  安装代码包会在工作区/pkg/平台目录生成归档文件
  安装命令源码文件会在 bin目录下生成可执行文件
  
go get
  从远程代码仓库拉取代码包,拉取到 GOPATH 的 src 目录中
  -u 更新已存在的代码包
  用导入路径编译程序时，go build 命令会使用 GOPATH 的设置，在磁盘上搜索这个包。事实上， 这个导入路径代表一个 URL，指向 GitHub 上的代码库。如果路径包含 URL，可以使用 Go 工具链从 DVCS 获取包，并把包的源代码保存在 GOPATH 指向的路径里与 URL 匹配的目录里。这个获取过程 使用 go get 命令完成。go get 将获取任意指定的 URL 的包，或者一个已经导入的包所依赖的其 他包。由于go get的这种递归特性，这个命令会扫描某个包的源码树，获取能找到的所有依赖包。
```



---

## 常用转换符

- 归纳：

| 格式 |                             说明                             |
| :--: | :----------------------------------------------------------: |
|  %%  |                         一个%字面量                          |
|  %b  | 一个二进制整数值（基数为2），或是一个（高级的）用科学计数法表示的指数为2的浮点数 |
| `%c` |    字符型，可以吧输入的数字按照ASCII码相应转换为对应字符     |
| `%d` |                  一个十进制数值（基数为10）                  |
|  %e  |             以科学计数法e表示的浮点数或者复数值              |
|  %E  |             以科学计数法E表示的浮点数或者复数值              |
|  %f  |              以标准计数法表示的浮点数或者复数值              |
|  %g  |  以%e或%f表示的浮点数或复数，任何一个都以最为紧凑的方式输出  |
|  %G  |  以%E或%f表示的浮点数或复数，任何一个都以最为紧凑的方式输出  |
|  %o  |              一个以八进制表示的数字（基数为8）               |
|  %p  | 以十六进制（基数为16）表示的一个值的地址，前缀为0x，字母使用小写的a-f表示 |
|  %q  | 使用Go语法以及必须时使用转义，以双引号括起来的字符串或者字节切片 |
|  %s  | 字符串。输出字符串中的字符直至字符串中的空字符（字符串以'\0'结尾，这个'\0'即表示空字符） |
|  %t  |                 以true或者false输出的布尔值                  |
| `%T` |                   使用Go语法输出的值的类型                   |
|  %U  |    一个用Unicode表示法表示的整型码点，默认值为4个数字字符    |
| `%v` | 使用默认格式输出的内置或自定义类型的值，或者是使用其类型的String()方式输出的自定义值，如果该方法存在的话 |
|  %x  |  以十六进制表示的整型值（基数为十六），数字a-f使用小写表示   |
|  %X  |  以十六进制表示的整型值（基数为十六），数字A-F使用大写表示   |

- Pactice-Code

```go
type User struct {
    Name string
    Age int
}
user : = User{
    "overnote",
    1,
}
fmt.Printf("%%\n")                   // %
fmt.Printf("%b\n", 16)               // 10000
fmt.Printf("%c\n", 65)               // A
fmt.Printf("%c\n", 0x4f60)           // 你
fmt.Printf("%U\n", '你')              // U+4f60
fmt.Printf("%x\n", '你')              // 4f60
fmt.Printf("%X\n", '你')              // 4F60
fmt.Printf("%d\n", 'A')              // 65
fmt.Printf("%t\n", 1 > 2)            // false
fmt.Printf("%e\n", 4396.7777777)     // 4.396778e+03 默认精度6位
fmt.Printf("%20.3e\n", 4396.7777777) //            4.397e+03 设置宽度20,精度3,宽度一般用于对齐
fmt.Printf("%E\n", 4396.7777777)     // 4.396778E+03
fmt.Printf("%f\n", 4396.7777777)     // 4396.777778
fmt.Printf("%o\n", 16)               // 20
fmt.Printf("%p\n", []int{1})         // 0xc000016110
fmt.Printf("Hello %s\n", "World")    // Hello World
fmt.Printf("Hello %q\n", "World")    // Hello "World"
fmt.Printf("%T\n", 3.0)              // float64
fmt.Printf("%v\n", user)             // {overnote 1}
fmt.Printf("%+v\n", user)            // {Name:overnote Age:1}
fmt.Printf("%#v\n", user)            // main.User{Name:"overnote", Age:1}
```



---

## 关于package的理解

概念的理解

- 和python一样，把相同功能的代码放到一个目录，称之为包（package）
- 每个go程序都是由包构成，一般以包为单位进行运行
- main包（main函数）是用来生成可执行文件，并且每个程序有且只有一个main包
- 包的主要用途是提高代码的可复用性

导包的方式

- 匿名导入( _ )

```go
_ "learngo/init/lib2"
//表示导入这个包，执行导入(调用)包中的init()但是不引用该包中的元素.
//fg: 应用场景：比如数数据库驱动的注册
```

- 包别名

```go
lb2 "learngo/init/lib2"
//可直接用lb2代替lib2的包名调用包中对外的函数
```

- 嵌入式导入

```go
. "learngo/init/lib2"
//相当于lib2中的代码直接导入当前包，可直接使用lib2中的函数，无需前缀。多个嵌入式导入容易产生变量名重复错误。
```

## init函数的理解

- package以递归的形式导入，init()相当于return语句回溯到上一级调用，一般用来初始化包内参数。

![](https://raw.githubusercontent.com/ronin-sswang/upload_notes_img/main/img/202109160045202.png)

## main函数的使用

```go
//1)go语言以包作为管理单位
//2)每个文件必须先声明包
//3)程序必须有一个main包（重要）

package main

//go语言以双斜杠作为注释的标识
/*
	这是块注释
	可以多段注释
*/

import "fmt"

//入口函数，有且只有一个
func main() { //左括号必须与函数名同行，否则会报错
	//打印
	//“hello go"打印到屏幕，Println()会自动换行
	//调用函数，大部分都需要导入包
	fmt.Println("hello go") //go语言结尾没有分号
	fmt.Println("hello haha")
}
```

---

## 命名规范

- 标识符的命名要尽量采取简短且有意义。
- 为变量、函数、常量命名时采用驼峰命名法，例如 stuName、getVal。
- 如果首字母大写，则表示它可以被其它的包访问（类似于 [Java](http://c.biancheng.net/java/) 中的 public）；如果首字母小写，则表示它只能在本包中使用 (类似于 Java 中 private）。

---

## 变量声明的多样性

Var 关键字声明

- 又称变量完整声明

```go
var 变量名称 数据类型 [ = 变量初始值]             //方式一
// fg:
var name string

var 变量名1, 变量名2 <变量类型> = 初始值1,初始值2  //方式二
//fg:
var a, b = 2, 3

var (                                        //方式三
    name1 = initialvalue1,
    name2 = initialvalue2
)
```

:= 的方式声明

- 又称短变量声明

```go
name := initialvalue
//fg:
a := 1

//多变量
b, c := 2, 3
name, age := "xiang", 18 
```

- 此方式只适用于函数/方法内部，或if/for/switch/的初始化语句中
- 在函数外部无法适用（编译不通过）

_ 忽略变量

- _（下划线）是个特殊的变量名，任何赋予它的值都会被丢弃

```go
_, b := 1, 2
//将值2赋予b，并同时丢弃1
```



变量声明中的类型推断

- 如果变量有初始值，那么 Go 能够自动推断具有初始值的变量的类型
- 如果变量有初始值，就可以在变量声明中省略 `type`。
- 如果变量声明的语法是 var name = initialvalue，Go 能够根据初始值自动推断变量的类型。



变量声明注意事项

- GO声明的变量就会立即分配内存空间，也就意味着会被默认初始化值
- 若一个变量未被赋值，Go 会自动地将其初始化，赋值该变量类型的零值（Zero Value)



变量声明的practice-code

```go
package main

import "fmt"

func main() {
	//变量的定义
	//静态的语言的变量和动态语言的变量定义差异很大
	//1. 最基础的变量定义
	//var i int // int i
	//i = 20
	//fmt.Println(i)
	//定义并初始化
	//var i int = 10
	//fmt.Println(i)
	//2. 根据值自行判断变量类型（类型推断）
	//var i = 100 //并没有设置类型
	//fmt.Println(i)

	// 3. go语言既然一门新语言 省略var
	//i := 100
	//fmt.Println(i)

	//var a int = 10
	//var b = 10
	//c := 10
	//fmt.Println(a, b , c)

	//多变量定义
	//var a, b, c int
	//a, b, c = 10, 20, 30
	//fmt.Println(a, b, c)

	//var a, b, c int = 10, 20 ,30
	//fmt.Println(a, b, c)

	//// 集合类型
	//var (
	//	a int
	//	name string
	//)

	//var i int =10
	//i = 20
	//fmt.Println(i)

	//匿名变量， 变量一旦被定义 不使用
	//常量 -
	//const PI = 3.1415926
	//r := 2.0
	//运行过程中， 代码写的不好 一不小心在某个地方将PI给改掉了

	//枚举
	//const (
	//	Unknown = 0
	//	Female = 1
	//	Male = 2
	//)
	//
	////常量组如不指定类型和初始化值，该类型和值和上一行的类型一致
	//const (
	//	x int = 16
	//	y
	//	s = "abc"
	//	z
	//)
	//fmt.Println(x,y,s)
	//1. 常量的数据类型值可以是布尔，数字 和字符串
	//2. 不曾使用的常量，在编译的时候是不会报错
	//const常量的iota， 常量计数器 枚举
	const (
		Book = iota //计算器
		Cloth //行
		Phone
		DeskTop
	)
	//0,1,2 本身不重要， 这三个值不一样
	//iota 该常量的值等于上一个常量的表达式
	fmt.Println(Book, Cloth, Phone, DeskTop)
	//iota你真的懂了吗？
	// 1. iota只能在常量组中是使用
	// 2. 不同的const定义块互相不干扰
	const (
		Unknown = iota
		Female
		Male
	)
	fmt.Println(Unknown, Female, Male)

	//3. 没有表达式的常量定义复用上一行的表达式
	//4. 从第一行开始，iota从0逐行加一
	const (
		a = iota //iota  = 0
		b = 10 //iota = 1 每一行iota加一
		c // c=10 , iota = 2
		d,e = iota, iota //iota=3
		f = iota //iota=4
		//iota代表的是这里的行数
	)
	
	fmt.Println(a, b, c, d, e, f)
	//变量的作用域
}

```

---

## 常量的声明

### const

```go
const identifier [type] = value
//fg:
const a int = 10  //定义以后不能再被修改
//fg:
const a = 55     // 允许
      a = 66     // 不允许重新赋值
```

- 常量中的数据类型只可以是布尔型、数字型（整数型、浮点型和复数）和字符串型
- 常量不能再重新赋值为其他的值
- 常量的值会在编译的时候确定
- 常量也可以使用自动推导类型来定义,但是不能使用:=的方式

### 枚举iota

- 预声明标识符iota，它默认开始值是0，一组多个常量同时声明时其值逐行递增
- iota常用来表示自增的枚举变量，专门用来初始化常量

```go
package main

import "fmt"

func main() {
	// 1，iota的功能是常量生成器，每隔一行，自动累加1
	// 2，iota给常量赋值使用

	const (
		a = iota //1
		b = iota //2
		c = iota //3
	)
	fmt.Printf("a = %d, b = %d, c = %d\n", a, b, c)

	//3，多个iota时，可以只写一个。效果与上边的一样
	const (
		a1 = iota
		b1
		c1
	)
	fmt.Printf("a1 = %d, b1 = %d, c1 = %d\n", a1, b1, c1)

	//4，iota再次遇到cost，将会重置为0
	const d = iota //0
	fmt.Println("d =", d)

	//5，如果在同一行，那么值都一样
	const (
		i          = iota
		j1, j2, j3 = iota, iota, iota
		k          = iota
	)
	fmt.Printf("i = %d, j1 = %d, j2 = %d, j3 = %d, k = %d\n", i, j1, j2, j3, k)
  
  //位运算方式
  const (
    Open = 1 << iota //1
    Close //2
    Pending //4
)
}
```

---

### 常用格式化输出函数

```go
package main

import "fmt"
import "os"

type point struct {
	x, y int
}

func main() {

	// Go提供了几种打印格式，用来格式化一般的Go值，例如
	// 下面的%v打印了一个point结构体的对象的值
	p := point{1, 2}
	fmt.Printf("%v\n", p)

	// 如果所格式化的值是一个结构体对象，那么`%+v`的格式化输出
	// 将包括结构体的成员名称和值
	fmt.Printf("%+v\n", p)

	// `%#v`格式化输出将输出一个值的Go语法表示方式。
	fmt.Printf("%#v\n", p)

	// 使用`%T`来输出一个值的数据类型
	fmt.Printf("%T\n", p)

	// 格式化布尔型变量
	fmt.Printf("%t\n", true)

	// 有很多的方式可以格式化整型，使用`%d`是一种
	// 标准的以10进制来输出整型的方式
	fmt.Printf("%d\n", 123)

	// 这种方式输出整型的二进制表示方式
	fmt.Printf("%b\n", 14)

	// 这里打印出该整型数值所对应的字符
	fmt.Printf("%c\n", 33)

	// 使用`%x`输出一个值的16进制表示方式
	fmt.Printf("%x\n", 456)

	// 浮点型数值也有几种格式化方法。最基本的一种是`%f`
	fmt.Printf("%f\n", 78.9)

	// `%e`和`%E`使用科学计数法来输出整型
	fmt.Printf("%e\n", 123400000.0)
	fmt.Printf("%E\n", 123400000.0)

	// 使用`%s`输出基本的字符串
	fmt.Printf("%s\n", "\"string\"")

	// 输出像Go源码中那样带双引号的字符串，需使用`%q`
	fmt.Printf("%q\n", "\"string\"")

	// `%x`以16进制输出字符串，每个字符串的字节用两个字符输出
	fmt.Printf("%x\n", "hex this")

	// 使用`%p`输出一个指针的值
	fmt.Printf("%p\n", &p)

	// 当输出数字的时候，经常需要去控制输出的宽度和精度。
	// 可以使用一个位于%后面的数字来控制输出的宽度，默认
	// 情况下输出是右对齐的，左边加上空格
	fmt.Printf("|%6d|%6d|\n", 12, 345)

	// 你也可以指定浮点数的输出宽度，同时你还可以指定浮点数
	// 的输出精度
	fmt.Printf("|%6.2f|%6.2f|\n", 1.2, 3.45)

	// To left-justify, use the `-` flag.
	fmt.Printf("|%-6.2f|%-6.2f|\n", 1.2, 3.45)

	// 你也可以指定输出字符串的宽度来保证它们输出对齐。默认
	// 情况下，输出是右对齐的
	fmt.Printf("|%6s|%6s|\n", "foo", "b")

	// 为了使用左对齐你可以在宽度之前加上`-`号
	fmt.Printf("|%-6s|%-6s|\n", "foo", "b")

	// `Printf`函数的输出是输出到命令行`os.Stdout`的，你
	// 可以用`Sprintf`来将格式化后的字符串赋值给一个变量
	s := fmt.Sprintf("a %s", "string")
	fmt.Println(s)

	// 你也可以使用`Fprintf`来将格式化后的值输出到`io.Writers`
	fmt.Fprintf(os.Stderr, "an %s\n", "error")
}
```

---

### 常用格式化输入函数

```go
ackage main

import "fmt"

func main() {
	var a int
	fmt.Println("请输入变量a的值：")

	//fmt.Scanf("%d", &a) 阻塞等待用户的输入，别忘了&
	//对应格式写法
	// fmt.Scanf("%d", &a)

	//自动匹配格式的写法
	fmt.Scan(&a)
	fmt.Println("a =", a)
}
```























