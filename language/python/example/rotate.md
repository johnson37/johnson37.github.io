# Rotate

## For
```python
for letter in 'Python':     # 第一个实例
   print (letter)
 
fruits = ['banana', 'apple',  'mango']
for fruit in fruits:        # 第二个实例
   print (fruit)
 
print ("Good bye!")

```
## While
```python
count = 0
while (count < 9):
   print (count)
   count = count + 1
 
print ("Good bye!")

```

## break
```python
for letter in 'Python':     # 第一个实例
   if letter == 'h':
      break
   print '当前字母 :', letter
  
var = 10                    # 第二个实例
while var > 0:              
   print '当前变量值 :', var
   var = var -1
   if var == 5:   # 当变量 var 等于 5 时退出循环
      break
 
print "Good bye!"
```
## continue
```python
for letter in 'Python':     # 第一个实例
   if letter == 'h':
      continue
   print (letter)
 
var = 10                    # 第二个实例
while var > 0:              
   var = var -1
   if var == 5:
      continue
   print (var)
print ("Good bye!")
```
