# Scala 

Scala是一门多范式语言，对面向对象和面向过程都有很好的支持。
Scala的源文件后缀为xxx.scala, 经过scala编译器的编译之后生成xxx.class, 形成JAVA字节码，这也就意味着scala形成的执行文件能够在JVM上运行。

## Scala Helloworld

```c
object HelloWorld {
    def main(args: Array[String]): Unit = {
        println("Hello, world!")
    }
}
```

## Scala 特性
- 面向对象特性
- 函数式编程
- 静态类型
	- 泛型类
    - 协变和逆变
    - 标注
    - 类型参数的上下限约束
    - 把类别和抽象类型作为对象成员
    - 复合类型
    - 引用自己时显式指定类型
    - 视图
    - 多态方法
- 扩展性
- 并发性

### Scala的基础语法
- 对象 : 对象有属性和行为。例如：一只狗的状属性有：颜色，名字，行为有：叫、跑、吃等。对象是一个类的实例。
- 类 : 类是对象的抽象，而对象是类的具体实例。
- 方法 : 方法描述的基本的行为，一个类可以包含多个方法。
- 字段 : 每个对象都有它唯一的实例变量集合，即字段。对象的属性通过给字段赋值来创建。


- 区分大小写 :  Scala是大小写敏感的，这意味着标识Hello 和 hello在Scala中会有不同的含义。
- 类名 : 对于所有的类名的第一个字母要大写。
  如果需要使用几个单词来构成一个类的名称，每个单词的第一个字母要大写。

  示例：class MyFirstScalaClass
- 方法名称 : 所有的方法名称的第一个字母用小写。
  如果若干单词被用于构成方法的名称，则每个单词的第一个字母应大写。
  示例：def myMethodName()

- 程序文件名 : 程序文件的名称应该与对象名称完全匹配(新版本不需要了，但建议保留这种习惯)。
  保存文件时，应该保存它使用的对象名称（记住Scala是区分大小写），并追加".scala"为文件扩展名。 （如果文件名和对象名称不匹配，程序将无法编译）。

  示例: 假设"HelloWorld"是对象的名称。那么该文件应保存为'HelloWorld.scala"

  def main(args: Array[String]) - Scala程序从main()方法开始处理，这是每一个Scala程序的强制程序入口部分。

### Scala 包

```c
package com.runoob
class HelloWorld
```

```c
package com.runoob {
  class HelloWorld 
}
```
#### Scala 引用包
```c
import java.awt.Color  // 引入Color
 
import java.awt._  // 引入包内所有成员
 
def handler(evt: event.ActionEvent) { // java.awt.event.ActionEvent
  ...  // 因为引入了java.awt，所以可以省去前面的部分
}
```

### Scala 数据类型
| --- | --- | 
|数据类型 |	描述 |
|Byte 	 | 8位有符号补码整数。数值区间为 -128 到 127 |
|Short 	 | 16位有符号补码整数。数值区间为 -32768 到 32767 |
|Int 	 | 32位有符号补码整数。数值区间为 -2147483648 到 2147483647 |
|Long 	 | 64位有符号补码整数。数值区间为 -9223372036854775808 到 9223372036854775807 |
|Float 	 | 32 位, IEEE 754 标准的单精度浮点数 |
|Double  |	64 位 IEEE 754 标准的双精度浮点数 |
|Char 	 | 16位无符号Unicode字符, 区间值为 U+0000 到 U+FFFF |
|String  |	字符序列 |
|Boolean | 	true或false |
|Unit 	 |表示无值，和其他语言中void等同。用作不返回任何结果的方法的结果类型。Unit只有一个实例值，写成()。 |
|Null 	 |null 或空引用 |
|Nothing 	| Nothing类型在Scala的类层级的最底端；它是任何其他类型的子类型。 |
|Any 	| Any是所有其他类的超类 |
|AnyRef 	| AnyRef类是Scala里所有引用类(reference class)的基类 |

#### Scala 基础字面量

##### 整形字面量
整型字面量用于 Int 类型，如果表示 Long，可以在数字后面添加 L 或者小写 l 作为后缀。：

```c
0
035
21 
0xFFFFFFFF 
0777L
```
##### 浮点型字面量
如果浮点数后面有f或者F后缀时，表示这是一个Float类型，否则就是一个Double类型的。实例如下：
```c
0.0 
1e30f 
3.14159f 
1.0e100
.1
```
##### 布尔型字面量
布尔型字面量有 true 和 false。

##### 字符字面量 
在 Scala 字符变量使用单引号 ' 来定义，如下：
```c
'a' 
'\u0041'
'\n'
'\t'
```

##### 字符串字面量
在 Scala 字符串字面量使用双引号 " 来定义，如下：
```c
"Hello,\nWorld!"
"菜鸟教程官网：www.runoob.com"
```
##### 多行字符串的表示方法
```c
val foo = """菜鸟教程
www.runoob.com
www.w3cschool.cc
www.runnoob.com
以上三个地址都能访问"""
```

### Scala 变量
在 Scala 中，使用关键词 "var" 声明变量，使用关键词 "val" 声明常量。
声明变量实例如下：
```c
var myVar : String = "Foo"
var myVar : String = "Too"
```
声明常量实例如下：
```c
val myVal : String = "Foo"
```

#### 变量类型声明
```c
var VariableName : DataType [=  Initial Value]

或

val VariableName : DataType [=  Initial Value]
```

#### 变量类型引用
在 Scala 中声明变量和常量不一定要指明数据类型，在没有指明数据类型的情况下，其数据类型是通过变量或常量的初始值推断出来的。

所以，如果在没有指明数据类型的情况下声明变量或常量必须要给出其初始值，否则将会报错。 
```c
var myVar = 10;
val myVal = "Hello, Scala!";
```