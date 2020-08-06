# go 数据类型

go数据类型包括四种 

* 基础类型 baisc type
* 聚合类型 aggregate type
* 引用类型 reference type
* 接口类型 interface type 

## 基础类型

基础类型包括 数值(number),字符串(string)和布尔(boolean)

### 整数

有符号整数

8位 int8 16位 int16 32位 int32 64位 int64

无符号整数

8位uint8 16位 uint16 32位 uint132 64位 uint64



int/uint 在特定平台上与原生的有符号整数\无符号整数相同,或者等同于该平台上的运算效率最高的值.

int是目前使用最广泛的数值类型.

rune是int32类型的同义词

byte是int8类型的同义词

### 浮点数

float32 float64

十进制下,float32的有效数字大约是6位,float64的有效位数大约是15位.绝大多数情况下,应优先选用float64,因为除非非常小心,否则float32的运算会迅速积累误差.另外,float32能精确表示的正整数范围有限.