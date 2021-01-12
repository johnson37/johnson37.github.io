# Object-c

## Basic Introdcution
Object-c 是C语言的严格超集，任何C语言可以不经修改直接通过Object-C编译器。

### 文件扩展名
- .h -- 头文件
- .m -- 源代码文件，可以包含Object-c和C代码
- .mm -- 源代码文件，除了可以包含源代码和C代码以外，还可以包含C++代码。


## 语法

### 消息传递
C++ 中obj.method(argument);

Object-c:
```objc
[obj method: argument];
```

### 类
```objc
@interface MyObject : NSObject {
    int memberVar1; // 实体变量
    id  memberVar2;
}

+(return_type) class_method; // 类方法

-(return_type) instance_method1; // 实例方法
-(return_type) instance_method2: (int) p1;
-(return_type) instance_method3: (int) p1 andPar: (int) p2;
@end
```

方法前面的 +/- 号代表函数的类型：加号（+）代表类方法（class method），不需要实例就可以调用，与C++ 的静态函数（static member function）相似。减号（-）即是一般的实例方法（instance method）。

### xiahuaxian
property 声明变量时，会生成setter，getter和一个带——前缀的变量。

### block 函数指针

