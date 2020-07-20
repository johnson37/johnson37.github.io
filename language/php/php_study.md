# PHP Study Note

## Grammer

### Basic Grammer
php 脚本以<?php 开始，以 ?> 结束
```php
<?php
// PHP 代码
?>
```

### Comment
Comment 方式与C语言相同。

### PHP Variable
- php不需要声明变量。
- 变量以 $ 符号开始，后面跟着变量的名称
- 变量名必须以字母或者下划线字符开始
- 变量名只能包含字母数字字符以及下划线（A-z、0-9 和 _ ）
- 变量名不能包含空格
- 变量名是区分大小写的（$y 和 $Y 是两个不同的变量

#### PHP 弱类型语言
C语言是一门强类型语言，在定义变量时需要说明变量的类型，PHP会根据变量赋值情况自动。

#### PHP 变量域
php有四种不同的作用域：
- local
- global
- static
- parameter

global：在函数外部定义的为全局变量，可以在函数外的任何地方使用，如果想在函数内部使用，需要加global
local： 函数内部定义的为局部变量，只能在函数内部使用。
static：static的含义跟C语言之函数内static变量含义相同。

#### echo 与 print
echo和print是两个php中的输出语句。
echo可以输出一个或者多个字符串，echo比print快
print只能输出一个字符串，但是有返回值。

### 数据类型
String（字符串）, Integer（整型）, Float（浮点型）, Boolean（布尔型）, Array（数组）, Object（对象）, NULL（空值）
#### String
```php
<?php 
$x = "Hello world!";
echo $x;
echo "<br>"; 
$x = 'Hello world!';
echo $x;
?>
```
#### 整形
```php
<?php 
$x = 5985;
var_dump($x);
echo "<br>"; 
$x = -345; // 负数 
var_dump($x);
echo "<br>"; 
$x = 0x8C; // 十六进制数
var_dump($x);
echo "<br>";
$x = 047; // 八进制数
var_dump($x);
?>
```
#### 浮点型
```php
<?php 
$x = 10.365;
var_dump($x);
echo "<br>"; 
$x = 2.4e3;
var_dump($x);
echo "<br>"; 
$x = 8E-5;
var_dump($x);
?>
```
#### 布尔型
```php
$x=true;
$y=false;
```

#### PHP 数组

```php
<?php 
$cars=array("Volvo","BMW","Toyota");
var_dump($cars);
?>
```
#### PHP 对象

#### PHP NULL
NULL 值表示变量没有值。NULL 是数据类型为 NULL 的值。

NULL 值指明一个变量是否为空值。 同样可用于数据空值和NULL值的区别。

### PHP 大小比较
- ==：只比较大小，不比较类型
- ===：大小类型都比较


### PHP 常量
bool define ( string $name , mixed $value [, bool $case_insensitive = false ] )
- name
- value
- case_insensitive: optional

```php
<?php
// 区分大小写的常量名
define("GREETING", "欢迎访问 Runoob.com");
echo GREETING;    // 输出 "欢迎访问 Runoob.com"
echo '<br>';
echo greeting;   // 输出 "greeting"
?>
```

```php

<?php
// 不区分大小写的常量名
define("GREETING", "欢迎访问 Runoob.com", true);
echo greeting;  // 输出 "欢迎访问 Runoob.com"
?>

```

```php

<?php
define("GREETING", "欢迎访问 Runoob.com");
 
function myTest() {
    echo GREETING;
}
 
myTest();    // 输出 "欢迎访问 Runoob.com"
?>

```

### 字符串

```php
 <?php
$txt="Hello world!";
echo $txt;
?> 
```

#### php 字符串的并置运算符
并置运算符 (.) 用于把两个字符串值连接起来。
```php
 <?php
$txt1="Hello world!";
$txt2="What a nice day!";
echo $txt1 . " " . $txt2;
?> 
```
#### PHP strlen() 函数
```php
 <?php
echo strlen("Hello world!");
?> 
```

#### PHP strpos() 函数
在字符串中查找一个指定文本
```php
 <?php
echo strpos("Hello world!","world");
?> 
```

### 运算符
#### 算术运算符

- x + y 	加 	x 和 y 的和 	2 + 2 	4
- x - y 	减 	x 和 y 的差 	5 - 2 	3
- x * y 	乘 	x 和 y 的积 	5 * 2 	10
- x / y 	除 	x 和 y 的商 	15 / 5 	3
- x % y 	模（除法的余数） 	x 除以 y 的余数 	5 % 2
- x 	取反 	x 取反 	
- a . b 	并置 	连接两个字符串 	"Hi" . "Ha" 	HiHa
```php

<?php 
$x=10; 
$y=6;
echo ($x + $y); // 输出16
echo '<br>';  // 换行
 
echo ($x - $y); // 输出4
echo '<br>';  // 换行
 
echo ($x * $y); // 输出60
echo '<br>';  // 换行
 
echo ($x / $y); // 输出1.6666666666667
echo '<br>';  // 换行
 
echo ($x % $y); // 输出4
echo '<br>';  // 换行
 
echo -$x;
?>

```

#### 赋值运算符
- x = y 	x = y 	左操作数被设置为右侧表达式的值
- x += y 	x = x + y 	加
- x -= y 	x = x - y 	减
- x *= y 	x = x * y 	乘
- x /= y 	x = x / y 	除
- x %= y 	x = x % y 	模（除法的余数）
- a .= b 	a = a . b 	连接两个字符串

```php

<?php 
$x=10; 
echo $x; // 输出10
 
$y=20; 
$y += 100;
echo $y; // 输出120
 
$z=50;
$z -= 25;
echo $z; // 输出25
 
$i=5;
$i *= 6;
echo $i; // 输出30
 
$j=10;
$j /= 5;
echo $j; // 输出2
 
$k=15;
$k %= 4;
echo $k; // 输出3
?> 
```
#### PHP 递增/递减运算符

- ++ x 	预递增 	x 加 1，然后返回 x
- x ++ 	后递增 	返回 x，然后 x 加 1
- -- x 	预递减 	x 减 1，然后返回 x
- x -- 	后递减 	返回 x，然后 x 减 1

```php

<?php
$x=10; 
echo ++$x; // 输出11
 
$y=10; 
echo $y++; // 输出10
 
$z=5;
echo --$z; // 输出4
 
$i=5;
echo $i--; // 输出5
?>
```

#### PHP 比较运算符

- x == y 	等于 	如果 x 等于 y，则返回 true 	5==8 返回 false
- x === y 	绝对等于 	如果 x 等于 y，且它们类型相同，则返回 true 	5==="5" 返回 false
- x != y 	不等于 	如果 x 不等于 y，则返回 true 	5!=8 返回 true
- x <> y 	不等于 	如果 x 不等于 y，则返回 true 	5<>8 返回 true
- x !== y 	绝对不等于 	如果 x 不等于 y，或它们类型不相同，则返回 true 	5!=="5" 返回 true
- x > y 	大于 	如果 x 大于 y，则返回 true 	5>8 返回 false
- x < y 	小于 	如果 x 小于 y，则返回 true 	5<8 返回 true
- x >= y 	大于等于 	如果 x 大于或者等于 y，则返回 true 	5>=8 返回 false
- x <= y 	小于等于 	如果 x 小于或者等于 y，则返回 true 	5<=8 返回 true

```php

<?php
$x=100; 
$y="100";
 
var_dump($x == $y);
echo "<br>";
var_dump($x === $y);
echo "<br>";
var_dump($x != $y);
echo "<br>";
var_dump($x !== $y);
echo "<br>";
 
$a=50;
$b=90;
 
var_dump($a > $b);
echo "<br>";
var_dump($a < $b);
?>
```

#### PHP 逻辑运算符
- x and y 	与 	如果 x 和 y 都为 true，则返回 true 	x=6
- x or y 	或 	如果 x 和 y 至少有一个为 true，则返回 true 	x=6
- x xor y 	异或 	如果 x 和 y 有且仅有一个为 true，则返回 true 	x=6 y=3
- x && y 	与 	如果 x 和 y 都为 true，则返回 true 	x=6
- x || y 	或 	如果 x 和 y 至少有一个为 true，则返回 true 	x=6
- ! x 	非 	如果 x 不为 true，则返回 true 	x=6

#### PHP 数组运算符
- x + y 	集合 	x 和 y 的集合
- x == y 	相等 	如果 x 和 y 具有相同的键/值对，则返回 true
- x === y 	恒等 	如果 x 和 y 具有相同的键/值对，且顺序相同类型相同，则返回 true
- x != y 	不相等 	如果 x 不等于 y，则返回 true
- x <> y 	不相等 	如果 x 不等于 y，则返回 true
- x !== y 	不恒等 	如果 x 不等于 y，则返回 true

```php

<?php
$x = array("a" => "red", "b" => "green"); 
$y = array("c" => "blue", "d" => "yellow"); 
$z = $x + $y; // $x 和 $y 数组合并
var_dump($z);
var_dump($x == $y);
var_dump($x === $y);
var_dump($x != $y);
var_dump($x <> $y);
var_dump($x !== $y);
?>

```
#### 三元运算符
(expr1) ? (expr2) : (expr3) 

```php

<?php
$test = '菜鸟教程';
// 普通写法
$username = isset($test) ? $test : 'nobody';
echo $username, PHP_EOL;
 
// PHP 5.3+ 版本写法
$username = $test ?: 'nobody';
echo $username, PHP_EOL;
?>

```

### if-else
```php

<?php
$t=date("H");
if ($t<"20")
{
    echo "Have a good day!";
}
?>

```

```php

<?php
$t=date("H");
if ($t<"20")
{
    echo "Have a good day!";
}
else
{
    echo "Have a good night!";
}
?>

```

```php
if (条件)
{
    if 条件成立时执行的代码;
}
elseif (条件)
{
    elseif 条件成立时执行的代码;
}
else
{
    条件不成立时执行的代码;
}
```

### switch

```php

<?php
switch (n)
{
case label1:
    如果 n=label1，此处代码将执行;
    break;
case label2:
    如果 n=label2，此处代码将执行;
    break;
default:
    如果 n 既不等于 label1 也不等于 label2，此处代码将执行;
}
?>

```

```php

<?php
$favcolor="red";
switch ($favcolor)
{
case "red":
    echo "你喜欢的颜色是红色!";
    break;
case "blue":
    echo "你喜欢的颜色是蓝色!";
    break;
case "green":
    echo "你喜欢的颜色是绿色!";
    break;
default:
    echo "你喜欢的颜色不是 红, 蓝, 或绿色!";
}
?>

```

### PHP 数组
