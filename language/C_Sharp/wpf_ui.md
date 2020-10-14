# WPF UI

## Column/Row Definition.
```c

```

## 如何查看当前使用C#的版本。
```
C# 1.0 with Visual Studio.NET
C# 2.0 with Visual Studio 2005
C# 3.0 with Visual Studio 2008，2010（bai.net 2.0, 3.0, 3.5）du
C# 4.0 with Visual Studio 2010（.net 4）
C# 5.0 with Visual Studio 2012（.net 4.5），2013（.net 4.5.1, 4.5.2）
C# 6.0 with Visual Studio 2015（.net 4.6, 4.6.1, 4.6.2）
C# 7.0 with Visual Studio 2017（.net 4.6.2, 4.7）
```

## 如何创建一个新的Win
Add--> Window

## Win show/showDialog
[Difference](https://www.cnblogs.com/uniquefrog/archive/2013/01/07/2849676.html)

## 如何完成窗口之间的切换。
```
Window1 _newWin = new Window1(this);
this.Hide();
_newWin.Show();
```