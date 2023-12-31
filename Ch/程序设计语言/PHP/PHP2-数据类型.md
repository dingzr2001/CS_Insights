# PHP2

### 0x01.数据类型

datatype，存储的数据本身的类型，而不是变量的类型。



### 0x02.PHP的八种数据类型

#### 基本数据类型(4种)：

整型：int，系统分配4个字节存储，表示整数类型（有范围，若过大则会变成浮点型）。

浮点型：float/double，系统分配8个字节存储，表示小数或者整型存不下的整数。

字符串型：string，系统根据实际长度分配，表示字符串。

布尔类型：bool，表示布尔类型，只有两个值，true和false。

#### 复合数据类型(2种)：

对象类型：object，存放对象（面向对象）。

数组类型：array，存储多个数据。

#### 特殊数据类型(2种)：

资源类型：resource，存放资源数据（PHP外部数据，如数据库，文件）。

空类型：Null，不能运算。



### 0x03.类型转换

#### 两种转换方式：

自动转换：系统根据需求自己转换。

手动转换：自己通过命令转换。

#### 其他类型转数值的说明

布尔true为1，false为0。

以字母开头的字符串为0。

以数字开头的字符串，取到碰到字符为止，不会同时包含两个小数点。

例：int('123ab')输出123    float('123.12.2ab')输出123.12



### 0x04.类型判断

通过一组类型判断函数来判断变量，最终返回这个变量所保存数据的数据类型。是一组以is_开头后面跟类型名字的函数。如is_int，is_strintg。相同为true，不同为false。

#Bool类型不能用echo查看，因为不知道返回的是bool类型的true/false还是string类型的true/false。可以用var_dump结构查看。

var_dump(变量1,变量2,变量3,...)

例：var_dump(is_int('123'))输出bool(false)



### 0x05.类型更改

#### 两个函数

gettype(变量名)：获取类型，得到的是该类型名字对应的字符串。

settype(变量名,数据类型)：设定数据类型。转换成功返回true，失败转换false。

#settype和强制转换的区别：

强制转换是对变量的数据值做改变，对变量本身不改变，即$b=int($a)，$b的值是将$a转换成int后的值，而$a本身的类型不变，不会变为int。

settype是$a的值就变成了int。



### 0x06.进制转换

$a=120   十进制

$a=0b120  二进制

$a=0120  八进制

$a=0x120  十六进制

十进制转二进制   decbin(变量名/数字)

十进制转八进制  decoct(变量名/数字)

十进制转十六进制  dechex(变量名/数字)

十六进制转十进制  hexdcec(变量名/数字)

八进制转十进制  ocdec(变量名/数字)

二进制转十进制  bindec(变量名/数字)



### 0x07.浮点类型

$f=1.23e10  表示1.23乘e^10。



### 0x08.布尔类型

empty()：判断数据的值是否为空，如果为空返回true，不为空返回false。

isset()：判断数据存储的变量本身是否存在，存在变量返回true，不存在返回false。只要变量被定义过就视为存在。
