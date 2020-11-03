### Shell编程入门目录

- [一、第一个shell脚本](#一、第一个shell脚本)
    - [1、作为可执行程序](#1、作为可执行程序)
    - [2、作为解释器参数](#2、作为解释器参数)
- [二、shell变量](#二、shell变量)
- [三、Shell传递参数](#三、Shell传递参数)
- [四、Shell数组](#四、Shell数组)
- [五、Shell基本运算符](#五、Shell基本运算符) 
    -  [1、算数运算符](#1、算数运算符)
    - [2、关系运算符](#2、关系运算符)
    - [3、布尔运算符](#3、布尔运算符)
    - [4、逻辑运算符](#4、逻辑运算符)
    - [5、字符串运算符](#5、字符串运算符)
    - [6、文件测试运算符](#6、文件测试运算符)
- [六、Shell echo命令](#六、Shellecho命令)
- [七、Shell printf命令](#七、Shellprintf命令) 
  - [printf的转义序列](#printf的转义序列)
- [八、Shell test命令](#八、Shelltest命令) 
    - [1、数值测试](#1、数值测试)
    - [2、字符串测试](#2、字符串测试)
    - [3、文件测试](#3、文件测试)
    - [4、示例](#4、示例)
- [九、Shell流程控制](#九、Shell流程控制) 
    - [1、if else语法格式](#1、ifelse语法格式)
    - [2、if else-if else语法格式](#2、ifelse-ifelse语法格式)
    - [3、for循环一般格式为](#3、for循环一般格式为)
    - [4、while语句](#4、while语句)
    - [5、无限循环语法格式](#5、无限循环语法格式)
    - [6、until循环](#6、until循环)
    - [7、case语句](#7、case语句)
    - [8、跳出循环](#8、跳出循环)
- [十、Shell函数](#十、Shell函数)
- [十一、Shell 输入/输出重定向](#十一、Shell输入和输出重定向)
- [十二、Shell文件包含](#十二、Shell文件包含)

### 一、第一个shell脚本

```shell
#!/bin/bash
echo "Hello World!"
#! 是一个约定的标记，告诉系统这个脚本需要什么解释器来执行
```

运行shell脚本有两种方法：

#### 1、作为可执行程序

将上面的代码保存为test.sh，并cd到相应目录；

```shell
chmod +x ./test.sh #使脚本具有执行权限./test.sh #执行脚本
```

注意，一定要写成./test.sh，而不是test.sh，运行其他二进制的程序也一样，直接写test.sh，linux系统会去PATH里寻找有没有叫test.sh的，而只有/bin, /sbin, /usr/bin，/usr/sbin等在PATH里，你的当前目录通常不在PATH里，所以写成test.sh是会找不到命令的，要用./test.sh告诉系统说，就在当前目录找。

#### 2、作为解释器参数

这种运作方式是，直接运行解释器，其参数就是shell脚本的文件名，如：

> sh test.sh

这种方式运行的脚本，不需要在第一行指定解释器信息，写了也没用。

### 二、shell变量

定义变量时，变量名不加美元符号（$），如：`your_name="TestName"`，注意，变量名和等号之间不能有空格，这个和大多数变成语言都是不同的。变量名的命名必须遵循如下规则：

- 首个字符必须为字母（a-z，A-Z）
- 中间不能有空格，可以使用下划线（_）
- 不能使用标点符号
- 不能使用bash里的关键字

除了显式地直接赋值，还可以用语句给变量赋值，如：

```shell
for file in 'ls /etc'
# 上面语句将/etc下目录的文件名循环出来
```

**使用变量**

使用一个定义过的变量，只要在变量名前面加美元符号即可，如：

```shell
your_name="TestName"
echo $your_name
```

变量名外面的花括号是可选的，加花括号是为了帮助解释器识别变量的边界，比如下面这种情况：

```shell
# echo ${your_name}

your_name="TestName"
echo "Your name is $your_nameand your age is 20"
```

这个时候解释器就会将`$your_nameand`识别为一个变量，脚本就无法输出预期的结果了。因此，在使用变量时务必加上花括号。

已定义的变量，可以被重新定义，如：

```shell
your_name="tom"
echo $your_name
your_name="alibaba"
echo $your_name
```

这样写是合法的，但注意，第二次赋值的时候不能写`$your_name="alibaba"`，使用变量的时候才加美元符`($)`。

### 三、Shell传递参数

我们可以在执行shell脚本时，向脚本传递参数，脚本内获取参数的格式为：`$n`。n代表一个数字，1为执行脚本的第一个参数，2为执行脚本的第二个参数，以此类推。。。。。

以下实例向脚本传递三个参数，并分别输出，其中`$0`为执行的文件名：

```shell
#!/bin/bash

#Shell传参举例
echo "shell 传递参数实例！"
echo "执行的文件名：$0"
echo "第一个参数为：$1"
echo "第二个参数为：$2"
echo "第三个参数为：$3"
echo "参数个数为：$#"
echo "传递的参数作为一个字符串显示：$*"
```

另外，还有几个特殊字符用来处理参数：

| 参数处理 | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| $#       | 传递到脚本的参数个数                                         |
| $*       | 以一个单字符串显示所有向脚本传递的参数。如`"$*"`用「"」括起来的情况、以`"$1 $2 … $n"`的形式输出所有参数。 |
| $$       | 脚本运行的当前进程ID号                                       |
| $!       | 后台运行的最后一个进程的ID号                                 |
| $@       | 与`$*`相同，但是使用时加引号，并在引号中返回每个参数。如`"$@"`用「"」括起来的情况、以`"$1" "$2" … "$n"` 的形式输出所有参数。 |
| $-       | 显示Shell使用的当前选项，与set命令功能相同。                 |
| $?       | 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。 |

### 四、Shell数组

数组中可以存放多个值。Bash Shell只支持一维数组，初始化时不需要定义数组大小。数组元素的下标从0开始。Shell数组用括号来表示，元素用“空格”符号分隔开，语法格式如下：

```shell
array_name=(value1 ... valuen)

#读取数组元素的一般格式为：
${array_name[index]}
```

使用`@或*`可以获取数组中的所有元素，获取数组长度的方法与获取字符串长度的方法相同。

```shell
#!/bin/bash

#创建一个一位数组，并填充数据
array1=(13 2 3)
#创建一个一维数组，不填充数据
array2=()

#为空的数组赋值
array2[0]=4
#读取数组的一个元素
echo ${array1[0]}
echo ${array2[0]}

#输出数组所有的元素
echo ${array1[@]} echo ${array1[*]}

#计算数组长度
length1=${#array1[@]}
length2=${#array2[*]}
#计算数组元素的长度
length3=${#array1[0]}
echo ${length1}
echo ${length2}
echo ${length3}
```

### 五、Shell基本运算符

Shell和其他编程语言一样，支持多种运算符，包括：算数运算符、关系运算符、布尔运算符、逻辑运算符、字符串运算符和文件测试运算符。expr是一款表达式计算工具，使用它能完成表达式的求值操作。

需要注意的是：**表达式和运算符之间要有空格**，例如2+2是错误的，必须写成2 + 2；另外，完整的表达式要被反引号``包含。

#### 1、算数运算符

| 运算符 | 说明                                          | 举例                           |
| ------ | --------------------------------------------- | ------------------------------ |
| +      | 加法                                          | `expr $a + $b` 结果为 30。     |
| -      | 减法                                          | `expr $a - $b` 结果为 -10。    |
| *      | 乘法                                          | `expr $a \* $b` 结果为  200。  |
| /      | 除法                                          | `expr $b / $a` 结果为 2。      |
| %      | 取余                                          | `expr $b % $a` 结果为 0。      |
| =      | 赋值                                          | `a=$b` 将把变量 b 的值赋给 a。 |
| ==     | 相等。用于比较两个数字，相同则返回 true。     | `[ $a == $b ]` 返回 false。    |
| !=     | 不相等。用于比较两个数字，不相同则返回 true。 | `[ $a != $b ]` 返回 true。     |

示例：

```shell
#!/bin/bash

#运算符操作示例

#两个数相加
a=10
b=20
val_add=`expr $a + $b`
echo "a + b : ${val_add}"

#两个数相减
val_subtract=`expr $a - $b`
echo "a - b : ${val_subtract}"

#两个数相乘，乘号（*）前边必须加反斜杠（\）才能实现乘法运算；
val_multiply=`expr $a \* $b`
echo "a * b : ${val_multiply}"

#两个数相除
val_divide=`expr $b / $a`
echo "b / a : ${val_divide}"

#两个数取余
val_remainder=`expr $b % $a`
echo "b % a : ${val_remainder}"

#条件表达式要放在方括号之间，并且要有空格，例如[$a==$b]是错误的，必须写成[ $a == $b ]
#if...then...fi是条件语句
if [ $a == $b ]
then
 echo "a等于b"
fi
```

#### 2、关系运算符

关系运算符只支持数字，不支持字符串，除非字符串的值是数字。下表列出了常用的关系运算符，假定变量a为10，变量b为20：

| 运算符 | 说明                                                  | 举例                         |
| ------ | ----------------------------------------------------- | ---------------------------- |
| -eq    | 检测两个数是否相等，相等返回 true。                   | `[ $a -eq $b ]` 返回 false。 |
| -ne    | 检测两个数是否相等，不相等返回 true。                 | `[ $a -ne $b ]` 返回 true。  |
| -gt    | 检测左边的数是否大于右边的，如果是，则返回 true。     | `[ $a -gt $b ]` 返回 false。 |
| -lt    | 检测左边的数是否小于右边的，如果是，则返回 true。     | `[ $a -lt $b ]` 返回 true。  |
| -ge    | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | `[ $a -ge $b ]` 返回 false。 |
| -le    | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | `[ $a -le $b ]` 返回 true。  |

示例：

```shell
#!/bin/bash

a=10
b=20

if [ $a -eq $b ]
then
echo "a等于b"
fi

if [ $a -ne $b ]
then
echo "a不等于b"
fi

if [ $a -gt $b ]
then
echo "a大于b"
fi

if [ $a -lt $b ]
then
echo "a小于b"
fi

if [ $a -ge $b ]
then
echo "a大于等于b"
fi

if [ $a -le $b ]
then
echo "a小于等于b"
fi
```

#### 3、布尔运算符

下表列出了常用的布尔运算符，假定变量a为10，变量b为20：

| 运算符 | 说明                                                | 举例                                       |
| ------ | --------------------------------------------------- | ------------------------------------------ |
| !      | 非运算，表达式为 true 则返回 false，否则返回 true。 | `[ ! false ]` 返回 true。                  |
| -o     | 或运算，有一个表达式为 true 则返回 true。           | `[ $a -lt 20 -o $b -gt 100 ]` 返回 true。  |
| -a     | 与运算，两个表达式都为 true 才返回 true。           | `[ $a -lt 20 -a $b -gt 100 ]` 返回 false。 |

示例：

```shell
#!/bin/bash

#bool运算符
a=10
b=20

if [ $a -gt 20 -o $b -gt 10 ]
then
        echo "$a -gt 20 -o $b -gt 10 : 返回true"
fi

if [ $a -lt 20 -a $b -gt 10 ]
then
        echo "$a -lt 20 -a $b -gt 10 : 返回true"
fi
```

#### 4、逻辑运算符

下表列出了Shell的逻辑运算符，假定变量a为10，变量b为20：

| 运算符 | 说明       | 举例                                        |
| ------ | ---------- | ------------------------------------------- |
| &&     | 逻辑的 AND | `[[ $a -lt 100 && $b -gt 100 ]]` 返回 false |
| \|\|   | 逻辑的 OR  | `[[ $a -lt 100 || $b -gt 100 ]]` 返回 true  |

示例：

```shell
#!/bin/bash

#bool运算符
a=10
b=20

#逻辑运算符
if [[ $a -gt 20 || $b -gt 10 ]]
then
        echo "$a - gt 20 || $b -gt 10 : 返回true"
fi

if [[ $a -lt 20 && $b -gt 10 ]]
then
        echo "$a -lt 20 && $b -gt 10 : 返回true"
fi
```

#### 5、字符串运算符

下表列出了常用的字符串运算符，假定变量a为“abc”，变量b为“efg”：

| 运算符 | 说明                                      | 举例                       |
| ------ | ----------------------------------------- | -------------------------- |
| =      | 检测两个字符串是否相等，相等返回 true。   | `[ $a = $b ]` 返回 false。 |
| !=     | 检测两个字符串是否相等，不相等返回 true。 | `[ $a != $b ]` 返回 true。 |
| -z     | 检测字符串长度是否为0，为0返回 true。     | `[ -z $a ]` 返回 false。   |
| -n     | 检测字符串长度是否为0，不为0返回 true。   | `[ -n $a ]` 返回 true。    |
| str    | 检测字符串是否为空，不为空返回 true。     | `[ $a ]` 返回 true。       |

示例：

```shell
#!/bin/bash

a="abc"
b="efg"

if [ $a == $b ]
then
        echo "$a 等于 $b"
else
        echo "$a 不等于 $b"
fi

if [ $a != $b ]
then
        echo "$a 不等于 $b"
else
        echo "$a 等于 $b"
fi

if [ -z $a ]
then
        echo "$a 长度为0"
else
        echo "$a 长度不为0"
fi
```

#### 6、文件测试运算符

文件测试运算符用于检测Linux文件的各种属性。属性检测描述如下：

| 操作符  | 说明                                                         | 举例                        |
| ------- | ------------------------------------------------------------ | --------------------------- |
| -b file | 检测文件是否是块设备文件，如果是，则返回 true。              | `[ -b $file ]` 返回 false。 |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。            | `[ -c $file ]` 返回 false。 |
| -d file | 检测文件是否是目录，如果是，则返回 true。                    | `[ -d $file ]` 返回 false。 |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | `[ -f $file ]` 返回 true。  |
| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。            | `[ -g $file ]` 返回 false。 |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  | `[ -k $file ]` 返回 false。 |
| -p file | 检测文件是否是有名管道，如果是，则返回 true。                | `[ -p $file ]` 返回 false。 |
| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。            | `[ -u $file ]` 返回 false。 |
| -r file | 检测文件是否可读，如果是，则返回 true。                      | `[ -r $file ]` 返回 true。  |
| -w file | 检测文件是否可写，如果是，则返回 true。                      | `[ -w $file ]` 返回 true。  |
| -x file | 检测文件是否可执行，如果是，则返回 true。                    | `[ -x $file ]` 返回 true。  |
| -s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true。     | `[ -s $file ]` 返回 true。  |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。          | `[ -e $file ]` 返回 true。  |

示例：

```shell
#!/bin/bash

file=/home/wjy/Files/shellscript/test5.sh
catalog=/home/wjy/Files/shellscript

if [ -f $file ]
then
        echo "$file 是普通文件"
else
        echo "$file 不是普通文件"
fi

if [ -r $file ]
then
        echo "$file 可读"
else
        echo "$file 不可读"
fi

if [ -w $file ]
then
        echo "$file 可写"
else
        echo "$file 不可写"
fi

if [ -x $file ]
then
        echo "$file 可执行"
else
        echo "$file 不可执行"
fi

if [ -d $catalog ]
then
        echo "$catalog 是目录"
else
        echo "$catalog 不是目录"
fi
```

### 六、Shellecho命令

```shell
#!/bin/bash

#显示普通字符串
echo "Hello World!"

#省略引号
echo Hello World!

#显示转义字符
echo "\"It is a test\""

#取消换行
echo -e "OK!\c"
echo "This is a test!"

#显示换行
echo -e "OK!\n"
echo "This is a test!"

#显示的结果定向至文件
echo "结果重定向到文件中" > test111.txt

#read命令从标准输入中读取一行，并把输入行的每个字段的值指定给Shell变量
read name
echo "My name is $name"

#原样输出字符串，不进行转义或取变量（用单引号）
echo '$name\"'
```

### 七、Shellprintf命令

```shell
#!/bin/bash

printf "%-10s %-8s %-4s\n" 姓名 性别 体重kg
printf "%-10s %-8s %-4.2f\n" 赵一一 男 56.55555
printf "%-10s %-8s %-4.2f\n" 钱二二 女 48.56
printf "%-10s %-8s %-4.2f\n" 孙三 男 56

#format-string为双引号
printf "%d %s\n" 333 "abc"
#单引号与双引号效果一样
printf '%d %s\n' 333 "abc"

#没有引号也可以输出，需要注意的是，这里的\n是不可缺少的，否则将无法输出
#如果只指定了一个参数，多出的参数仍会按照该格式输出，format-string被重用
printf "%s\n" abcd edfg
printf "\f"
printf "a"
```

说明：

- %s %c %d %f和在C语言中的用法一样，都是格式替换符。
- %-10s指一个宽度为10个字符（-表示左对齐，没有则表示右对齐），任何字符都会被显示在10个字符宽的字符内，如果不足则自动以空格填充，超过也会将内容全部显示出来。
- %-4.2f指格式化为小数，其中.2指保留2位小数。

#### printf的转义序列

| 序列  | 说明                                                         |
| ----- | ------------------------------------------------------------ |
| \a    | 警告字符，通常为ASCII的BEL字符                               |
|       | 后退                                                         |
| \c    | 抑制（不显示）输出结果中任何结尾的换行字符（只在%b格式指示符控制下的参数字符串中有效），而且，任何留在参数里的字符、任何接下来的参数以及任何留在格式字符串中的字符，都被忽略 |
| \f    | 换页（formfeed）                                             |
| \n    | 换行                                                         |
| \r    | 回车（Carriage return）                                      |
| \t    | 水平制表符                                                   |
| \v    | 垂直制表符                                                   |
| \\    | 一个字面上的反斜杠字符                                       |
| \ddd  | 表示1到3位数八进制值的字符。仅在格式字符串中有效             |
| \0ddd | 表示1到3位的八进制值字符                                     |

示例：

```shell
$ printf "a string, no processing:<%s>\n" "A\nB"
a string, no processing:<A\nB>
 
$ printf "a string, no processing:<%b>\n" "A\nB"
a string, no processing:<A
B>
 
$ printf "test \a"
test $   #不换行
```

### 八、Shelltest命令

Shell的test命令用于检查某个条件是否成立，它可以进行数值、字符和文件三方面的测试。

#### 1、数值测试

| 参数 | 说明           |
| ---- | -------------- |
| -eq  | 等于则为真     |
| -ne  | 不等于则为真   |
| -gt  | 大于则为真     |
| -ge  | 大于等于则为真 |
| -lt  | 小于则为真     |
| -le  | 小于等于则为真 |

#### 2、字符串测试

| 参数      | 说明                   |
| --------- | ---------------------- |
| =         | 等于则为真             |
| !=        | 不相等则为真           |
| -z 字符串 | 字符串长度为零则为真   |
| -n 字符串 | 字符串长度不为零则为真 |

#### 3、文件测试

| 参数      | 说明                                 |
| --------- | ------------------------------------ |
| -e 文件名 | 如果文件存在则为真                   |
| -r 文件名 | 如果文件存在且可读则为真             |
| -w 文件名 | 如果文件存在且可写则为真             |
| -x 文件名 | 如果文件存在且可执行则为真           |
| -s 文件名 | 如果文件存在且至少有一个字符则为真   |
| -d 文件名 | 如果文件存在且为目录则为真           |
| -f 文件名 | 如果文件存在且为普通文件则为真       |
| -c 文件名 | 如果文件存在且为字符型特殊文件则为真 |
| -b 文件名 | 如果文件存在且为块特殊文件则为真     |

另外，Shell还提供了与（-a）、或（-o）、非（!）三个逻辑操作符用于将测试条件连接起来，其优先级为：“!”最高，“-a”次之，“-o”最低。

#### 4、示例

```shell
#!/bin/bash

num1=100
num2=200
if test $[num1] -eq $[num2]
then
 echo "$num1 $num2 两个数相等"
else
 echo "$num1 $num2 两个数不相等"
fi

str1="Hello"
str2="Hello"
if test str1=str2
then
 echo "$str1 $str2 两个字符串相等"
else
 echo "$str1 $str2 两个字符串不相等"
fi

if test $[num1] -eq $[num2] -a str1=str2
then
 echo "$num1 $num2 两个数相等，$str1 $str2 两个字符串相等"
elif test $[num1] -eq $[num2]
then
 echo "$num1 $num2 两个数相等，$str1 $str2 两个字符串不相等"
elif test str1=str2
then
 echo "$num1 $num2 两个数不相等，$str1 $str2 两个字符串相等"
else
 echo "$num1 $num2 两个数不相等，$str1 $str2 两个字符串不相等"
fi
```

### 九、Shell流程控制

在sh/bash里面，如果else分支没有语句执行，就不要写这个else。if语句语法格式：

```shell
if condition
then
	command1
	command2
	...
	commandN
fi

#写成一行（适用于终端命令提示符）：
if [ $(ps -ef | grep -c "ssh") -gt 1 ]; then echo "true"; fi
```

#### 1、ifelse语法格式

```shell
if condition
then
	command1
	command2
	...
	commandN
elif
	command
fi
```

#### 2、ifelse-ifelse语法格式

```shell
if condition1
then
	command1
elif condition2
	command2
else
	commandN
fi
```

#### 3、for循环一般格式为

```shell
for var in item1 item2 ... itemN
do
	command1
	command2
	...
	commandN
done
```

当变量值在列表里，for循环即执行一次属所有命令，适用变量名获取列表中的当前取值。命名可为任何有效的shell命令和语句。in列表可以包含替换、字符串和文件名。in列表是可选的，如果不用它，for循环适用命令行的位置参数。

例如：顺序输出当前列表中的数字，输出字符串。

```shell
#!/bin/bash
for loop in 1 2 3 4 5
do
        echo "The value is:$loop"
done

for str in "Hello World!"
do
        echo "$str"
done
```

#### 4、while语句

while循环用于不断执行一系列命令，也用于从输入文件中读取数据；命令通常为测试条件。格式如下：

```shell
while condition
do
	command
done

#示例
number=1
while (( $number <= 5 ))
do
    echo $number
    let "number=number+1"
    #或者可以使用let "number++"
done
```

while循环可用于读取键盘信息。下面的例子中，输入信息被设置为变量FILM，按`<Ctrl+D>`结束循环。

```shell
#!/bin/bash

echo "按下<Ctrl+D>退出"
echo -n "输入你最喜欢的电影名："
while read Film
do
	echo "是的！$Film是一个好电影"
done
```

#### 5、无限循环语法格式

```shell
while :
do
	command
done

#或者
while true
do
	command
done

#或者
for (( ; ;))
```

#### 6、until循环

until循环执行一系列命令直至条件为真时停止。until循环与while循环在处理方式上刚好相反。一般while循环优于until循环。语法格式：

```shell
untile condition
do
	command
done
```

#### 7、case语句

Shell case语句为多选择语句。可以用case语句匹配一个值与一个模式，如果匹配成功，则执行相匹配的命令。格式如下：

```shell
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2）
    command1
    command2
    ...
    commandN
    ;;
esac
```

取值后面必须为in，每一模式必须以右括号结束。取值可以为变量或者常数。匹配发现取值符合某一模式后，其间所有命令开始执行直至 ;;

取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。如果无一匹配模式，使用星号*捕获该值，再执行后面的命令。

示例：

```shell
#!/bin/bash

echo "请输入1到3之间的数字："
echo "你输入的数字为："
read number

case $number in
1) 
	echo "你输入了1"
	;;
2) 
	echo "你输入了2"
	;;
3) 
	echo "你输入了3"
	;;
esac
```

#### 8、跳出循环

在循环执行过程中，有时候需要在未达到循环结束条件时强制跳出循环，Shell使用两个命令来实现该功能：break和continue。

break命令允许跳出所有循环（终止执行后面的所有循环）。

```shell
#!/bin/bash

while :
do
	echo -n "输入1到5之间的数字："
	read num
	
	case $num in
	1|2|3|4|5) 
		echo "你输入的数字为$num!"
		;;
	*) 
		echo "你输入的数字不是1到5之间的！游戏结束"
		break
		;;
	esac
done
```

continue命令与break命令类似，只有一点差别，它不会跳出所有循环，仅仅跳出当前循环。

```shell
#!/bin/bash
while :
do
        echo -n "输入1到5之间的数字："
        read num
        case $num in
                1|2|3|4|5) 
                	echo "你输入的数字为$num!"
                	;;
                *) 
                	echo "你输入的数字不是1到5之间的!"
                    continue
                    echo "游戏结束"
                	;;
        esac
done
```

esac是case的反写，作为case语句的结束标记，每个case分支用右圆括号，用两个分号来表示break。

### 十、Shell函数

Shell中函数的定义格式如下：

```shell
[ function ] funname [()]
{
    action;
 
    [return int;]
}
```

说明：

- 可以带function fun()定义，也可以直接fun()定义，不带任何参数。
- 参数返回，可以显式加return返回，如果不加，将以最后一条命令运行的结果作为返回值。

示例：

```shell
#!/bin/bash

demoFun(){
	echo "这是我的第一个Shell函数！"
}
echo "-----函数开始执行-----"
demoFun
echo "-----函数执行完毕-----"

funWithReturn(){
    echo "这个函数会对输入的两个数字进行相加运算。。。"
    echo "输入第一个数字："
    read numa
    echo "输入第二个数字："
    read numb
    echo "两个数字分别为 $numa 和 $numb !"
    return $(($numa+$numb))
}
funWithReturn
echo "输入的两个数字之和为：$?"

function funWithParam(){
    echo "第一个参数为：$1 "
    echo "第二个参数为：$2 "
    echo "第三个参数为：$3 "
    echo "第四个参数为：$4 "
    echo "第五个参数为：$5 "
    echo "第十个参数为：${10} "
    echo "第十一个参数为：${11} "
    echo "参数总数有 $# 个"
    echo "作为一个字符串输出所有参数 $* "
}
funWithParam 1 2 3 4 5 6 7 8 9 900 1999
```

注意：所有函数在使用前必须定义。这意味着**必须将函数放在脚本开始部分**，直至Shell解释器首次发现它时才可以使用。调用函数仅使用其函数名即可。

在Shell中，**调用函数时可以向其传递参数**。在函数体内部，通过`$n`的形式来获取参数的值，例如`$1`表示第一个参数，`$2`表示第二个参数，但是当n>=10时，必须要使用`${n}`来获取参数。

### 十一、Shell输入和输出重定向

大多数UNIX系统命令从你的终端接受输入并将所产生的输出发送回终端。一个命令通常从一个叫做标准输入的地方读取输入，默认情况下，这恰好是我们的终端。同样，一个命令通常将其输出写入到标准输出，默认情况下，这也是我们的终端。**重定向命令列表**如下：

| 命令            | 说明                                               |
| --------------- | -------------------------------------------------- |
| command > file  | 将**输出**重定向到 file。                          |
| command < file  | 将**输入**重定向到 file。                          |
| command >> file | 将**输出以追加**的方式重定向到 file。              |
| n > file        | 将文件描述符为 n 的文件重定向到 file。             |
| n >> file       | 将文件描述符为 n 的文件以追加的方式重定向到 file。 |
| n >& m          | 将输出文件 m 和 n 合并。                           |
| n <& m          | 将输入文件 m 和 n 合并。                           |
| << tag          | 将开始标记 tag 和结束标记 tag 之间的内容作为输入。 |

注意：文件描述符0通常是标准输入（STDIN），1是标准输出（STDOUT），2是标准错误输出（STDERR）。

**重定向深入理解**

一般情况下，每个Unix/Linux命令运行时都会打开三个文件：

- 标准输入文件（stdin）：stdin的文件描述符为0，Unix程序默认从stdin读取数据；
- 标准输出文件（stdout）：stdout的文件描述符为1，Unix程序默认向stdout输出数据；
- 标准错误文件（stderr）：stderr的文件描述符为2，Unix程序会向stderr流中写入错误信息。

默认情况下，`command > file`将stdout重定向到file，`command < file`将stdin重定向到file。

- 如果希望stderr重定向到file，可以写成`command 2 > file`，如果希望stderr追加到file文件末尾，可以写成`command 2 >> file`，2表示标准错误文件（stderr）。
- 如果希望将stdout和stderr合并后重定向到file，可以写成 `command > file 2>&1`或者`command >> file 2>&1`。
- 如果希望对stdin和stdout都重定向，可以写成`command < file1 > file2`，将stdin重定向到file1，将stdout重定向到file2。

**Here Document**

Here Document是Shell中的一种特殊的重定向方式，用来将输入重定向到一个交互式Shell脚本或程序。基本格式如下：

```shell
command << delimiter
    document
delimiter
```

它的作用是将两个delimiter之间的内容（document）作为输入传递给command。注意：结尾的delimiter一定要顶格写，前面不能有任何字符。

**/dev/null 文件**

如果希望执行某个命令，但又不希望在屏幕上显示输出结果，那么可以将输出重定向到`/dev/null`，如`command > /dev/null`。

`/dev/null` 是一个特殊的文件，写入到它的内部的内容都会被丢弃；如果尝试从该文件读取内容，那么什么也读取不到。但是`/dev/null` 文件非常有用，将命令的输出重定向到它，可以**起到“禁止输出”的效果**。如果希望屏蔽stdout和stderr，可以写成 `command > /dev/null 2>&1`。

### 十二、Shell文件包含

和其他语言一样，Shell也可以包含外部脚本。这样可以很方便的封装一些公用的代码作为一个独立的文件。Shell文件包含的语法格式如下：

```shell
. filename #注意点号（.）和文件名中间有一个空格
#或
source filename
```

示例：

```shell
#!/bin/bash

#这个实例执行后将先执行suanshu.sh.backup脚本，然后再输出Hello World!
. ../suanshu.sh.backup  #suanshu.sh.backup是一个执行算数运算的脚本
echo "Hello World!"
```

注意：被引用的文件不需要可执行权限。







