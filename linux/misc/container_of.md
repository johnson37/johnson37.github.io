# container of

## basic Introduction
container_of is one Linux kernel function. It aims to find the whole structure pointer according to one member pointer.

## code Implementation
```c
#define container_of(ptr, type, member) ({			\
	const typeof( ((type *)0)->member ) *__mptr = (ptr);	\
	(type *)( (char *)__mptr - offsetof(type,member) );})

#define offsetof(TYPE, MEMBER)	((size_t)&((TYPE *)0)->MEMBER)
```
