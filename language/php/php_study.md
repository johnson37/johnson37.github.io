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
在 PHP 中，array() 函数用于创建数组：array();

#### 数值数组 - 带有数字 ID 键的数组
这里有两种创建数值数组的方法：

自动分配 ID 键（ID 键总是从 0 开始）：

```php
$cars=array("Volvo","BMW","Toyota");
```

人工分配 ID 键：
```php
$cars[0]="Volvo";
$cars[1]="BMW";
$cars[2]="Toyota";
``` 

```php

<?php
$cars=array("Volvo","BMW","Toyota");
echo "I like " . $cars[0] . ", " . $cars[1] . " and " . $cars[2] . ".";
?>

```
##### 获取数组的长度 - count() 函数
```php

<?php
$cars=array("Volvo","BMW","Toyota");
echo count($cars);
?>

```

##### 遍历数值数组
```php

<?php
$cars=array("Volvo","BMW","Toyota");
$arrlength=count($cars);
 
for($x=0;$x<$arrlength;$x++)
{
    echo $cars[$x];
    echo "<br>";
}
?>

```

#### 关联数组 - 带有指定的键的数组，每个键关联一个值
这里有两种创建关联数组的方法：
```php
$age=array("Peter"=>"35","Ben"=>"37","Joe"=>"43");
```
or:
```php
$age['Peter']="35";
$age['Ben']="37";
$age['Joe']="43"; 
```

```php

<?php
$age=array("Peter"=>"35","Ben"=>"37","Joe"=>"43");
echo "Peter is " . $age['Peter'] . " years old.";
?>

```

##### 遍历关联数组
```php

<?php
$age=array("Peter"=>"35","Ben"=>"37","Joe"=>"43");
 
foreach($age as $x=>$x_value)
{
    echo "Key=" . $x . ", Value=" . $x_value;
    echo "<br>";
}
?>

```

#### 多维数组 - 包含一个或多个数组的数组

TBD

#### 数组排序

##### sort 升序排列
```php
<?php
$cars=array("Volvo","BMW","Toyota");
sort($cars);
?> 
```

```php
<?php
$numbers=array(4,6,2,22,11);
sort($numbers);
?> 
```
##### rsort 降序排列

```php
<?php
$cars=array("Volvo","BMW","Toyota");
rsort($cars);
?> 
```

```php
<?php
$numbers=array(4,6,2,22,11);
rsort($numbers);
?> 
```

### 超级全局变量

#### $GLOBALS
$GLOBALS 是PHP的一个超级全局变量组，在一个PHP脚本的全部作用域中都可以访问。

$GLOBALS 是一个包含了全部变量的全局组合数组。变量的名字就是数组的键。

以下实例介绍了如何使用超级全局变量 $GLOBALS:

```php

<?php 
$x = 75; 
$y = 25;
 
function addition() 
{ 
    $GLOBALS['z'] = $GLOBALS['x'] + $GLOBALS['y']; 
}
 
addition(); 
echo $z; 
?>

```
#### $_SERVER
$_SERVER 是一个包含了诸如头信息(header)、路径(path)、以及脚本位置(script locations)等等信息的数组。这个数组中的项目由 Web 服务器创建。不能保证每个服务器都提供全部项目；服务器可能会忽略一些，或者提供一些没有在这里列举出来的项目。

以下实例中展示了如何使用$_SERVER中的元素:

|  元素 | 描述 | 
| :---: | :---:|
| $_SERVER['PHP_SELF'] | 当前执行脚本的文件名，与 document root 有关。例如，在地址为 http://example.com/test.php/foo.bar 的脚本中使用 $_SERVER['PHP_SELF'] 将得到 /test.php/foo.bar。 |
| $_SERVER['GATEWAY_INTERFACE']	|服务器使用的 CGI 规范的版本；例如，"CGI/1.1"。|
|$_SERVER['SERVER_ADDR']		| 	当前运行脚本所在的服务器的 IP 地址。		|
|$_SERVER['SERVER_NAME']		|当前运行脚本所在的服务器的主机名。如果脚本运行于虚拟主机中，该名称是由那个虚拟主机所设置的值决定。(如: www.runoob.com)		|
|$_SERVER['SERVER_SOFTWARE']		|服务器标识字符串，在响应请求时的头信息中给出。 (如：Apache/2.2.24)		|
|$_SERVER['SERVER_PROTOCOL']		|请求页面时通信协议的名称和版本。例如，"HTTP/1.0"。		|
|$_SERVER['REQUEST_METHOD']		|访问页面使用的请求方法；例如，"GET", "HEAD"，"POST"，"PUT"。		|
|$_SERVER['REQUEST_TIME'] 	|请求开始时的时间戳。从 PHP 5.1.0 起可用。 (如：1377687496)|
|$_SERVER['QUERY_STRING'] 	|query string（查询字符串），如果有的话，通过它进行页面访问。|
|$_SERVER['HTTP_ACCEPT'] 	|当前请求头中 Accept: 项的内容，如果存在的话。|
|$_SERVER['HTTP_ACCEPT_CHARSET'] 	|当前请求头中 Accept-Charset: 项的内容，如果存在的话。例如："iso-8859-1,utf-8"。|
|$_SERVER['HTTP_HOST'] 	|当前请求头中 Host: 项的内容，如果存在的话。|
|$_SERVER['HTTP_REFERER'] 	|引导用户代理到当前页的前一页的地址（如果存在）。由 user agent 设置决定。并不是所有的用户代理都会设置该项，有的还提供了修改 HTTP_REFERER 的功能。简言之，该值并不可信。)|
|$_SERVER['HTTPS'] 	|如果脚本是通过 HTTPS 协议被访问，则被设为一个非空的值。|
|$_SERVER['REMOTE_ADDR'] 	|浏览当前页面的用户的 IP 地址。|
|$_SERVER['REMOTE_HOST'] 	|浏览当前页面的用户的主机名。DNS 反向解析不依赖于用户的 REMOTE_ADDR。|
|$_SERVER['REMOTE_PORT'] 	|用户机器上连接到 Web 服务器所使用的端口号。|
|$_SERVER['SCRIPT_FILENAME'] 	|当前执行脚本的绝对路径。|
|$_SERVER['SERVER_ADMIN'] 	|该值指明了 Apache 服务器配置文件中的 SERVER_ADMIN 参数。如果脚本运行在一个虚拟主机上，则该值是那个虚拟主机的值。(如：someone@runoob.com)|
|$_SERVER['SERVER_PORT'] 	|Web 服务器使用的端口。默认值为 "80"。如果使用 SSL 安全连接，则这个值为用户设置的 HTTP 端口。|
|$_SERVER['SERVER_SIGNATURE'] 	|包含了服务器版本和虚拟主机名的字符串。|
|$_SERVER['PATH_TRANSLATED'] 	|当前脚本所在文件系统（非文档根目录）的基本路径。这是在服务器进行虚拟到真实路径的映像后的结果。|
|$_SERVER['SCRIPT_NAME'] 	|包含当前脚本的路径。这在页面需要指向自己时非常有用。__FILE__ 常量包含当前脚本(例如包含文件)的完整路径和文件名。|
|$_SERVER['SCRIPT_URI'] 	|URI 用来指定要访问的页面。例如 "/index.html"。 |

#### $_REQUEST
PHP $_REQUEST 用于收集HTML表单提交的数据。

以下实例显示了一个输入字段（input）及提交按钮(submit)的表单(form)。 当用户通过点击 "Submit" 按钮提交表单数据时, 表单数据将发送至<form>标签中 action 属性中指定的脚本文件。 在这个实例中，我们指定文件来处理表单数据。如果你希望其他的PHP文件来处理该数据，你可以修改该指定的脚本文件名。 然后，我们可以使用超级全局变量 $_REQUEST 来收集表单中的 input 字段数据:
```php

<html>
<body>
 
<form method="post" action="<?php echo $_SERVER['PHP_SELF'];?>">
Name: <input type="text" name="fname">
<input type="submit">
</form>
 
<?php 
$name = $_REQUEST['fname']; 
echo $name; 
?>
 
</body>
</html>

```

#### $_POST
PHP $_POST 被广泛应用于收集表单数据，在HTML form标签的指定该属性："method="post"。

以下实例显示了一个输入字段（input）及提交按钮(submit)的表单(form)。 当用户通过点击 "Submit" 按钮提交表单数据时, 表单数据将发送至<form>标签中 action 属性中指定的脚本文件。 在这个实例中，我们指定文件来处理表单数据。如果你希望其他的PHP文件来处理该数据，你可以修改该指定的脚本文件名。 然后，我们可以使用超级全局变量 $_POST 来收集表单中的 input 字段数据:

```php

<html>
<body>
 
<form method="post" action="<?php echo $_SERVER['PHP_SELF'];?>">
Name: <input type="text" name="fname">
<input type="submit">
</form>
 
<?php 
$name = $_POST['fname']; 
echo $name; 
?>
 
</body>
</html>

```
#### $_GET
PHP $_GET 同样被广泛应用于收集表单数据，在HTML form标签的指定该属性："method="get"。

$_GET 也可以收集URL中发送的数据。

假定我们有一个包含参数的超链接HTML页面：
```php
<html>
<body>

<a href="test_get.php?subject=PHP&web=runoob.com">Test $GET</a>

</body>
</html>
```

```php

<html>
<body>
 
<?php 
echo "Study " . $_GET['subject'] . " @ " . $_GET['web'];
?>
 
</body>
</html>

```
#### $_FILES

#### $_ENV

#### $_COOKIE

#### $_SESSION

### PHP 循环

#### while 循环
```php
while (条件)
{
    要执行的代码;
}
```

```php
<html>
<body>

<?php
$i=1;
while($i<=5)
{
    echo "The number is " . $i . "<br>";
    $i++;
}
?>

</body>
</html>
```
#### do...while 语句

```php
do
{
    要执行的代码;
}
while (条件);
```

```php
<html>
<body>

<?php
$i=1;
do
{
    $i++;
    echo "The number is " . $i . "<br>";
}
while ($i<=5);
?>

</body>
</html>

```

#### For 循环

```php
for (初始值; 条件; 增量)
{
    要执行的代码;
}
```

```php

<?php
for ($i=1; $i<=5; $i++)
{
    echo "The number is " . $i . "<br>";
}
?>

```

#### Foreach 循环
```php
foreach ($array as $value)
{
    要执行代码;
}
```

```php

<?php
$x=array("one","two","three");
foreach ($x as $value)
{
    echo $value . "<br>";
}
?>
```

### 函数

如要在页面加载时执行脚本，您可以把它放到函数里。

函数是通过调用函数来执行的。

你可以在页面的任何位置调用函数。

```php

<?php
function functionName()
{
    // 要执行的代码
}
?>

```

```php
function writeName()
{
    echo "Kai Jim Refsnes";
}
 
echo "My name is ";
writeName();
```

#### 带参数
```php

<?php
function writeName($fname,$punctuation)
{
    echo $fname . " Refsnes" . $punctuation . "<br>";
}
 
echo "My name is ";
writeName("Kai Jim",".");
echo "My sister's name is ";
writeName("Hege","!");
echo "My brother's name is ";
writeName("Ståle","?");
?>

```

#### 返回值

```php

<?php
function add($x,$y)
{
    $total=$x+$y;
    return $total;
}
 
echo "1 + 16 = " . add(1,16);
?>

```

### PHP 魔术常量

- __LINE__: 文件中的当前行号
- __FILE__: 文件的完整路径和文件名。如果用在被包含文件中，则返回被包含的文件名。
- __DIR__: 文件所在的目录。
- __FUNCTION__: 函数名称（PHP 4.3.0 新加）。自 PHP 5 起本常量返回该函数被定义时的名字（区分大小写）。在 PHP 4 中该值总是小写字母的。
- __METHOD__: 类的方法名（PHP 5.0.0 新加）。返回该方法被定义时的名字（区分大小写）。


## 表单

### PHP 表单和用户输入
有一点很重要的事情值得注意，当处理 HTML 表单时，PHP 能把来自 HTML 页面中的表单元素自动变成可供 PHP 脚本使用。
```html

<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>
 
<form action="welcome.php" method="post">
名字: <input type="text" name="fname">
年龄: <input type="text" name="age">
<input type="submit" value="提交">
</form>
 
</body>
</html>

```

```php

欢迎<?php echo $_POST["fname"]; ?>!<br>
你的年龄是 <?php echo $_POST["age"]; ?>  岁。

```

### PHP 获取下拉菜单的数据

#### PHP 下拉菜单单选
```php
<?php
$q = isset($_GET['q'])? htmlspecialchars($_GET['q']) : '';
if($q) {
        if($q =='RUNOOB') {
                echo '菜鸟教程<br>http://www.runoob.com';
        } else if($q =='GOOGLE') {
                echo 'Google 搜索<br>http://www.google.com';
        } else if($q =='TAOBAO') {
                echo '淘宝<br>http://www.taobao.com';
        }
} else {
?>
<form action="" method="get"> 
    <select name="q">
    <option value="">选择一个站点:</option>
    <option value="RUNOOB">Runoob</option>
    <option value="GOOGLE">Google</option>
    <option value="TAOBAO">Taobao</option>
    </select>
    <input type="submit" value="提交">
    </form>
<?php
}
?>
```

#### php 下拉菜单多选
```php

<?php
$q = isset($_POST['q'])? $_POST['q'] : '';
if(is_array($q)) {
    $sites = array(
            'RUNOOB' => '菜鸟教程: http://www.runoob.com',
            'GOOGLE' => 'Google 搜索: http://www.google.com',
            'TAOBAO' => '淘宝: http://www.taobao.com',
    );
    foreach($q as $val) {
        // PHP_EOL 为常量，用于换行
        echo $sites[$val] . PHP_EOL;
    }
      
} else {
?>
<form action="" method="post"> 
    <select multiple="multiple" name="q[]">
    <option value="">选择一个站点:</option>
    <option value="RUNOOB">Runoob</option>
    <option value="GOOGLE">Google</option>
    <option value="TAOBAO">Taobao</option>
    </select>
    <input type="submit" value="提交">
    </form>
<?php
}
?>

```

#### 单选按钮表单

php_form_radio.php

```php

<?php
$q = isset($_GET['q'])? htmlspecialchars($_GET['q']) : '';
if($q) {
        if($q =='RUNOOB') {
                echo '菜鸟教程<br>http://www.runoob.com';
        } else if($q =='GOOGLE') {
                echo 'Google 搜索<br>http://www.google.com';
        } else if($q =='TAOBAO') {
                echo '淘宝<br>http://www.taobao.com';
        }
} else {
?><form action="" method="get"> 
    <input type="radio" name="q" value="RUNOOB" />Runoob
    <input type="radio" name="q" value="GOOGLE" />Google
    <input type="radio" name="q" value="TAOBAO" />Taobao
    <input type="submit" value="提交">
</form>
<?php
}
?>

```

#### checkbox 复选框

php_form_select_checkbox.php

```php

<?php
$q = isset($_POST['q'])? $_POST['q'] : '';
if(is_array($q)) {
    $sites = array(
            'RUNOOB' => '菜鸟教程: http://www.runoob.com',
            'GOOGLE' => 'Google 搜索: http://www.google.com',
            'TAOBAO' => '淘宝: http://www.taobao.com',
    );
    foreach($q as $val) {
        // PHP_EOL 为常量，用于换行
        echo $sites[$val] . PHP_EOL;
    }
      
} else {
?><form action="" method="post"> 
    <input type="checkbox" name="q[]" value="RUNOOB"> Runoob<br> 
    <input type="checkbox" name="q[]" value="GOOGLE"> Google<br> 
    <input type="checkbox" name="q[]" value="TAOBAO"> Taobao<br>
    <input type="submit" value="提交">
</form>
<?php
}
?>

```

### include/require

在 PHP 中，您可以在服务器执行 PHP 文件之前在该文件中插入一个文件的内容。

include 和 require 语句用于在执行流中插入写在其他文件中的有用的代码。

include 和 require 除了处理错误的方式不同之外，在其他方面都是相同的：

- require 生成一个致命错误（E_COMPILE_ERROR），在错误发生后脚本会停止执行。
- include 生成一个警告（E_WARNING），在错误发生后脚本会继续执行。

menu.php
```php
<?php
echo '<a href="/">主页</a>
<a href="/html">HTML 教程</a>
<a href="/php">PHP 教程</a>';
?>
```

```php
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<div class="leftmenu">
<?php include 'menu.php'; ?>
</div>
<h1>欢迎来到我的主页!</h1>
<p>一些文本。</p>

</body>
</html>
```

### File 处理
fopen() 函数用于在 PHP 中打开文件。

此函数的第一个参数含有要打开的文件的名称，第二个参数规定了使用哪种模式来打开文件：
```php
<html>
<body>

<?php
$file=fopen("welcome.txt","r");
?>

</body>
</html> 
```

|---|---|
|模式 |描述 |
|r 	|只读。在文件的开头开始。 |
|r+ |	读/写。在文件的开头开始。 |
|w 	|只写。打开并清空文件的内容；如果文件不存在，则创建新文件。|
|w+ 	|读/写。打开并清空文件的内容；如果文件不存在，则创建新文件。|
|a 	|追加。打开并向文件末尾进行写操作，如果文件不存在，则创建新文件。|
|a+ 	|读/追加。通过向文件末尾写内容，来保持文件内容。|
|x 	|只写。创建新文件。如果文件已存在，则返回 FALSE 和一个错误。|
|x+ 	|读/写。创建新文件。如果文件已存在，则返回 FALSE 和一个错误。|

```php
<html>
<body>

<?php
$file=fopen("welcome.txt","r") or exit("Unable to open file!");
?>

</body>
</html> 
```

#### 关闭文件
```php
<?php
$file = fopen("test.txt","r");

//执行一些代码

fclose($file);
?> 
```

#### 逐行读取文件

```php
<?php
$file = fopen("welcome.txt", "r") or exit("无法打开文件!");
// 读取文件每一行，直到文件结尾
while(!feof($file))
{
    echo fgets($file). "<br>";
}
fclose($file);
?> 

```

#### 逐字符读取文件
```php
<?php
$file=fopen("welcome.txt","r") or exit("无法打开文件!");
while (!feof($file))
{
    echo fgetc($file);
}
fclose($file);
?> 
```

