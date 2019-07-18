# Why do we use do while 0 ?

Example:
```c
#define FOO(x) foo(x); bar(x)

if (condition)
    FOO(x);
else // syntax error here
    ...;
```

Even using braces doesn't help:
```c
#define FOO(x) { foo(x); bar(x); }
```

Using this in an if statement would require that you omit the semicolon, which is counterintuitive:
```c
if (condition)
    FOO(x)
else
    ...
```
If you define FOO like this:
```c
#define FOO(x) do { foo(x); bar(x); } while (0)
```

then the following is syntactically correct:

```c
if (condition)
    FOO(x);
else
    ....
```