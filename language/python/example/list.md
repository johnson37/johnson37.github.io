# List

```python
list1 = [ 'runoob', 786 , 2.23, 'john', 70.2 ]
tinylist = [123, 'john']
 
print list1               # 输
print list1[0]            # 输出列表的第一个元素
print list1[1:3]          # 输出第二个至第三个元素 
print list1[2:]           # 输出从第三个开始至列表末尾的所有元素
print tinylist * 2       # 输出列表两次
print tinylist
tinylist = tinylist*2
print tinylist
print list1 + tinylist    # 打印组合的列表

list1[1] = "villa"
print list1
print list1[-1]
print list1[2:-2]

```