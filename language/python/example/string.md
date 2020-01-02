# String

## String basic usage
```python
a = "Hello"
b = "Python"
 
print ("a + b 输出结果：", a + b )
print ("a * 2 输出结果：", a * 2 )
print ("a[1] 输出结果：", a[1] )
print ("a[1:4] 输出结果：", a[1:4] )
 
if( "H" in a) :
    print ("H 在变量 a 中" )
else :
    print ("H 不在变量 a 中") 
 
if( "M" not in a) :
    print ("M 不在变量 a 中" )
else :
    print ("M 在变量 a 中")
```

## String builtin Function
[String method](https://docs.python.org/3/library/stdtypes.html#str.capitalize)
- string.capitalize()
- string.upper()
- string.strip([obj])
- string.swapcase()
- string.title()
- string.split(str="", num=string.count(str))
- string.replace(str1, str2,  num=string.count(str1))
- string.lower()
- string.find(str, beg=0, end=len(string))
