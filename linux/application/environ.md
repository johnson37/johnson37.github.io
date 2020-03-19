# environment

## Basic introduction
Linux中有环境变量。环境变量是位于进程level。即每个进程都有自己的一组环境变量。在创建子进程时，子进程会默认继承父进程的环境变量。
子进程中可以通过api的形式改变自己的环境变量。

## Usage
```c
#include <stdio.h>
#include <unistd.h>

extern char **environ;
// env 和 environ 是一样的，都是指向的存储着环境变量字符串指针的数组
int main(int argc, char **argv, char **env)
{
    for (int i=0; env[i] != NULL; i++) {
        printf("env[%d] at %p = %s\n", i, env[i], env[i]);
    }
    printf("End of Env\n");
    getchar();
    return 0;
}
```
