# Makefile

## Make Step
```
1、读入所有的Makefile。
2、读入被include的其它Makefile。
3、初始化文件中的变量。
4、推导隐晦规则，并分析所有规则。
5、为所有的目标文件创建依赖关系链。
6、根据依赖关系，决定哪些目标要重新生成。
7、执行生成命令。
```

## CFLAGS
CFLAGS 表示用于 C 编译器的选项，

## LIBS
 LIBS：告诉链接器要链接哪些库文件，如LIBS = -lpthread -liconv
 
 ## LDFLAGS
 LDFLAGS：gcc 等编译器会用到的一些优化参数，也可以在里面指定库文件的位置。
 
 ## Function
 
 ###  subst
 **$(subst FROM, TO, TEXT)，即将字符串TEXT中的子串FROM变为TO。**
$(subst ee,EE,feet on the street) --> fEEt on the strEEt
 
 ### patsubst
 
   (patsubst pattern,replacement,text): 也是替换，text中符合某个pattern的替换成replacement
```
$(patsubst %.c,%.o,x.c.c bar.c) --> x.c.o bar.o
$(var:pattern=replacement) == $(patsubst pattern,replacement,$(var))
$(objects:.o=.c)  == $(patsubst %.o,%.c,$(objects))
```
 
 ### strip
 strip 目的是为了去掉前后的空格。
 ```
 $(strip a b c ) --> ‘a b c’
 ```
 ### findstring
 Searches in for an occurrence of find. If it occurs, the value is find; otherwise, the value is empty. 
 ```
$(findstring a,a b c) --> a
$(findstring a,b c) -->''
 ```
### filter pattern…,text

```
sources := foo.c bar.c baz.s ugh.h
foo: $(sources)
        cc $(filter %.c %.s,$(sources)) -o foo
```
### $(filter-out pattern…,text)

### $(sort list)
Sorts the words of list in lexical order, removing duplicate words.   
```
$(sort foo bar lose)
```
 ###  word
 ```
$(word <n>,<text>)

    名称：取单词函数——word。
    功能：取字符串<text>中第<n>个单词。（从一开始）
    返回：返回字符串<text>中第<n>个单词。如果<n>比<text>中的单词数要大，那么返回空字符串。
    示例：$(word 2, foo bar baz)返回值是“bar”。 
 ```
###  $(wordlist s,e,text)
Returns the list of words in text starting with word s and ending with word e (inclusive). 
```
$(wordlist 2, 3, foo bar baz) -->  ‘bar baz’
```
### $(words text)
Returns the number of words in text. Thus, the last word of text is $(word $(words text),text). 

### $(firstword names…)
```
$(firstword foo bar)  --> foo
```
### $(lastword names…)
```
$(lastword foo bar)  --> bar
```

### export
 export 可以把一些变量传递给下一级的Makefile
 export ARCH CFLAGS LIB_INSTALL_DIR HEADER_INSTALL_DIR
 
 ### addprefix
 addprefix：用于添加前缀
 ```
  install -p -m 444 -t $(HEADER_INSTALL_DIR) $(addprefix $(mdir)/,$(HEADERS))
 ```
