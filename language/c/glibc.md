# LIBC & GLIBC

## Introduction
LIBC is one library for Linux c functions, which exists in Linux source code.
glibc is one library for Linux c function, which is provided by GNU Organization. It doesn't exist in Linux source code.
If Linux system wants to make use of glibc, it need to add the related so file, like libc.so.6
However, glibc is quite popular, it has become the standard Linux c Function.

## What do libc&glibc contain?
Such as: open, read, write, close and so on.
In this glibc, we can devide all C functions to two kinds. 
- One: This function implement some functions and don't need the kernel.
- Two: This function implement some functions and need sys_call to kernel, kernel will help finish the functions.

## uclibc
uclibc is one kind of Linux c function library, which is more smaller for Embeded Linux system.
