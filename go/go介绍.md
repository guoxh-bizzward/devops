# GO 介绍

go能用干什么?



## go程序结构

### 名称

go关键字

go预生命的常量,类型和函数

```
实体声明在函数内,则它是函数内可见的,声明在包内,它将对包里面的所有源文件可见;
实体第一个字母的大小写决定其可见性是否跨包,如果是大写开头,它是导出的,意味着它对包外是可见和可访问的.
```

### 声明

声明给一个程序实体命名,并设定其部分或者全部属性.有4个主要的声明

* 变量(var)
* 常量(const)
* 函数(func)
* 类型(type)

```
//输出水的沸点
package main
import "fmt"
const boilingF = 212.0
func main() {
	var f = boilingF
	var c = (f-32)*5/9
	fmt.Printf("boiling pont = %gF or %gC \n",f,c)
}
```

### 变量

var声明创建一个具体类型的变量,然后给它附加一个名字,设置它的初始值.

> var name type = expression

类型和表达式部分可以省略一个,但不能全部省略

省略类型会根据初始表达式决定

省略表达式其初始值是对应类型的零值 数字 0 ,布尔 false,字符串 "",接口和引用类型是nil

#### 短变量声明

一般用于局部变量

> name := expression

`:=`表示声明;`=`表示赋值

短变量声明不需要声明所有在左边的变量,如果一些变量在同一个词法块中声明,那么对于那些变量,短变量的声明等同于赋值

#### 指针

指针存储的是变量的地址,一个指针指示**值所保存的位置**.**不是所有的值都有地址,但是所有的变量都有** 

使用指针,可以在无需知道变量名字的情况下,间接读取或者更新变量的值.

```
i := 1
p := &i //p是整型指针,指向i
fmt.Println(&p) //输出地址
fmt.Println(*p) //输出值
*p = 2          //结果i=2
fmt.Println(i) //输出i=2
fmt.Println(p != nil) //true 说明p指向一个变量
```

指针的零值是nil.指针是可比较的,两个指针当且仅当指向同一个变量或者两者都不是nil的情况下才相等

```
var x,y int
fmt.Println(&x == &x,&x == &y,&x == nil) //输出true false false
```

函数返回局部变量的地址是非常安全的

```
func main() {
	p := f()
	fmt.Println(*p) // 1
	fmt.Println(f()) // 函数的地址
	fmt.Println(f() == f()) //false
}

func f() *int{
	v := 1
	return &v
}
```

因为一个指针包含变量的地址,所以传递一个指针参数给函数,能够让函数更新间接传递的变量值

```
func main() {
	v := 1
	incr(&v) // v= 2
	fmt.Println(incr(&v)) //v=3
}

func incr(p *int) int{
	fmt.Println(&p) //0xc000110018
	*p++ //递增p指向的值,p自身保持不变
	fmt.Println(&p) //0xc000110018
	return *p
}
```

*p 是 v的别名,指针别名允许我们不用变量的名字来访问变量.

指针对于flag包是很关键的,它使用程序的命令行参数来设置整个程序内的某些变量的值

```
import (
	"flag"
	"fmt"
	"strings"
)

var n = flag.Bool("n",false,"omit trailing newline")
var seq = flag.String("s"," ","separator")
func main() {
	flag.Parse()
	fmt.Print(strings.Join(flag.Args(),*seq))
	if *n {
		fmt.Print("$")
	}
}
```

测试

```
go run echo2.go a b def => a b def
go run echo2.go -s / a b def => a/b/def
go run echo2.go -n a b def => a b def$
go run echo2.go --help
```

#### new函数

另一种创建变量的方式是使用内置的new函数

new(T)创建一个未命名的T类型变量,初始化为T类型的零值,并返回其地址(地址类型为*T)

```
func main() {
	p := new(int)
	fmt.Println(&p) //内存地址
	fmt.Println(*p) //0
	*p = 2
	fmt.Println(&p) //内存地址不变
	fmt.Println(*p) //2
}
```

使用new创建的变量和取其地址的普通局部变量没有什么不同,只是不需要引入(和声明)一个虚拟的名字,通过new(T)就可以直接在表达式中使用.**new只是语法上的便利,不是一个基础概念**

```
//两种方法具有相同的行为
func newInt1() *int {
	return new(int)
}
func newInt2() *int{
	var tmp int
	return &tmp
}
```

每一次调用new返回一个具有唯一地址的不同变量

```
p := new(int)
q := new(int)
fmt.Println(&p == &q) => fmt.Println(p == q) //false
```

这个规则有一个例外:两个变量的类型不携带任何信息且是零值,例如struct{}或者[0]int,当前的实现里面,它们有相同的地址.

因为最常见的未命名变量都是结构体类型,它的语法比较复杂,所以**new函数使用的相对较少**.

new是一个预声明的函数,不是关键字,所以他可以重定义为另外的其他类型.

```
func delta(old,new int)int {return new-old}
```

#### 变量的生命周期

生命周期是指程序执行过程中变量存在的时间段.

包级别变量的生命周期是整个程序的执行时间.

局部变量有一个动态的生命周期:每次执行声明语句时创建一个新的实体,变量一直生存到它变得不可访问,这时它占用的内存空间被回收.

函数的参数和返回值也是局部变量,在闭包函数被调用的时候创建.

变量的生命周期是通过它是否可达来确定的.所以局部变量可在包含它的循环的一次迭代之外继续存活,即使包含它的循环已经返回,它的存在还可能继续.

编译器可以选择使用堆或者栈上的空间来分配

```
var global *int
func f(){
	var x int
	x = 1
	global = &x
}
func g(){
	y := new(int)
	*y = 1
}
```

x使用的是堆内存,因为它在f函数执行完成以后仍然可以通过global变量访问,尽管它被声明为一个局部变量.这种情况下我们成x从f中**逃逸**.相反,g函数返回时,y变量不可访问,被回收.因为*y没有从g中逃逸.

任何情况下,逃逸的概念使你不需要额外费心来写正确的代码,但是记住它在性能优化的时候是有好处的,因为每次变量逃逸都需要一次额外的内存分配过程.

### 赋值

#### 多重赋值

允许多个变量一次性被赋值

```
x,y = y,x+y
a[i],a[j] = a[j],a[i]
```

#### 可赋值性

显式赋值

隐式赋值

### 类型声明

type 声明定义一个新的命名类型,它和某个已有类型使用相同的底层类型.

> type name underlaying-type

命名类型提供了一种方式来区分底层类型的不同或者不兼容使用,这样它们就不会在无意中混用

```
type Celsius float64
type Fahrenheit float64

const (
	AbsoluteZeroC Celsius = -273.15
	FreezingC Celsius = 0
	BoilingC Celsius = 100
)

func FtoC(fahrenheit Fahrenheit) Celsius{
	return Celsius((fahrenheit - 32) * 5 / 9) //需要进行显示的类型转换
}

func CtoF(celsius Celsius) Fahrenheit {
	return Fahrenheit(celsius*9*5 - 32)
}
```

对于每个类型T,都有一个对应的类型转换操作T(x)将值x转换为类型T

### 包和文件

#### 导入

#### 包初始化

### 作用域



​																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																						

