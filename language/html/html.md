# html

## html sample
```html

<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>
 
<h1>我的第一个标题</h1>
 
<p>我的第一个段落。</p>
 
</body>
</html>

```

- <!DOCTYPE html> 声明为 HTML5 文档
- <html> 元素是 HTML 页面的根元素
- <head> 元素包含了文档的元（meta）数据，如 <meta charset="utf-8"> 定义网页编码格式为 utf-8。
- <title> 元素描述了文档的标题
- <body> 元素包含了可见的页面内容
- <h1> 元素定义一个大标题
- <p> 元素定义一个段落

只有Body的部分会跟用户展示。

## html basic

### html 标题
```html

<h1>这是一个标题</h1>
<h2>这是一个标题</h2>
<h3>这是一个标题</h3>

```
### html 段落
```html

<p>这是一个段落。</p>
<p>这是另外一个段落。</p>

```

### html 链接
```html

<a href="https://www.runoob.com">这是一个链接</a>

```

### HTML 图像
```html

<img src="/images/logo.png" width="258" height="39" />

```

### 段落与换行的区别
<p></p>
<br>
段落和换行都能够新起一行，区别在于段落和段落之间有更大的间距。

## HTML 元素

- <p> 元素： 段落
- <body> 元素:
- <html> 元素

## HTML 属性

- HTML 元素可以设置属性
- 属性可以在元素中添加附加信息
- 属性一般描述于开始标签
- 属性总是以名称/值对的形式出现，比如：name="value"。


HTML 链接由 <a> 标签定义。链接的地址在 href 属性中指定： 
```html
<a href="http://www.runoob.com">这是一个链接</a>
```

### HTML 属性参考手册
| --- | --- |
|属性 	|描述 |
|class 	|为html元素定义一个或多个类名（classname）(类名从样式文件引入) |
|id 	|定义元素的唯一id |
|style 	|规定元素的行内样式 （inline style） |
|title 	|描述了元素的额外信息 (作为工具条使用) |

## html 标题
### html 标题
```html

<h1>这是一个标题。</h1>
<h2>这是一个标题。</h2>
<h3>这是一个标题。</h3>

```

### html 水平线
<hr> 标签在 HTML 页面中创建水平线。
```html

<p>这是一个段落。</p>
<hr>
<p>这是一个段落。</p>
<hr>
<p>这是一个段落。</p>

```

### html 注释
```html

<!-- 这是一个注释 -->

```

## html 段落
段落是通过 <p> 标签定义的。
```html

<p>这是一个段落 </p>
<p>这是另一个段落 </p>

```

### HTML 折行
如果您希望在不产生一个新段落的情况下进行换行（新行），请使用 <br> 标签：
```html

<p>这个<br>段落<br>演示了分行的效果</p>

```

## html 超链接
```html
<a href="https://www.runoob.com/">访问菜鸟教程</a>
```

## html 头部

<head> 元素包含了所有的头部标签元素。在 <head>元素中你可以插入脚本（scripts）, 样式文件（CSS），及各种meta信息。

可以添加在头部区域的元素标签为: <title>, <style>, <meta>, <link>, <script>, <noscript> 和 <base>。

### title
 <title> 标签定义了不同文档的标题。

<title> 在 HTML/XHTML 文档中是必须的。

<title> 元素:

- 定义了浏览器工具栏的标题
- 当网页添加到收藏夹时，显示在收藏夹中的标题
- 显示在搜索引擎结果页面的标题

### base

<base> 标签描述了基本的链接地址/链接目标，该标签作为HTML文档中所有的链接标签的默认链接:
```html
<head>
<base href="http://www.runoob.com/images/" target="_blank">
</head>
```

### link
 <link> 标签定义了文档与外部资源之间的关系。

<link> 标签通常用于链接到样式表:

```html
<head>
<link rel="stylesheet" type="text/css" href="mystyle.css">
</head>
```

### HTML <style> 
<style> 标签定义了HTML文档的样式文件引用地址.

在<style> 元素中你也可以直接添加样式来渲染 HTML 文档:
```html
<head>
<style type="text/css">
body {background-color:yellow}
p {color:blue}
</style>
</head>
```

### HTML <meta> 元素
meta标签描述了一些基本的元数据。
<meta> 标签提供了元数据.元数据也不显示在页面上，但会被浏览器解析。
META 元素通常用于指定网页的描述，关键词，文件的最后修改时间，作者，和其他元数据。
元数据可以使用于浏览器（如何显示内容或重新加载页面），搜索引擎（关键词），或其他Web服务。
<meta> 一般放置于 <head> 区域

```html
<meta name="keywords" content="HTML, CSS, XML, XHTML, JavaScript">
<meta name="description" content="免费 Web & 编程 教程">
<meta name="author" content="Runoob">
<meta http-equiv="refresh" content="30">
```

### HTML <script> 元素
<script>标签用于加载脚本文件，如： JavaScript。
<script> 元素在以后的章节中会详细描述。

## HTML CSS
CSS 是在 HTML 4 开始使用的,是为了更好的渲染HTML元素而引入的.

CSS 可以通过以下方式添加到HTML中:

- 内联样式- 在HTML元素中使用"style" 属性
- 内部样式表 -在HTML文档头部 <head> 区域使用<style> 元素 来包含CSS
- 外部引用 - 使用外部 CSS 文件

#### 内联样式
