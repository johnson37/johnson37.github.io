# Googletest

## Usage


| Fatal assertion 	| Nonfatal assertion 	| Verifies |
| ----------------- | --------------------- | --------- |
| ASSERT_TRUE(condition); 	| EXPECT_TRUE(condition); 	| condition is true |
| ASSERT_FALSE(condition); 	| EXPECT_FALSE(condition); 	| condition is false |
| ASSERT_EQ(expected, actual); 	| EXPECT_EQ(expected, actual); 	| expected == actual |
| ASSERT_NE(val1, val2); 	| EXPECT_NE(val1, val2); 	| val1 != val2 |
| ASSERT_LT(val1, val2); 	| EXPECT_LT(val1, val2); 	| val1 < val2 |
| ASSERT_LE(val1, val2); 	| EXPECT_LE(val1, val2); 	| val1 <= val2 |
| ASSERT_GT(val1, val2); 	| EXPECT_GT(val1, val2); 	| val1 > val2 |
| ASSERT_GE(val1, val2); 	| EXPECT_GE(val1, val2); 	| val1 >= val2 |
| ASSERT_STREQ(expected_str, actual_str); 	| EXPECT_STREQ(expected_str, actual_str); 	| the two C strings have the same content |
| ASSERT_STRNE(str1, str2); 	| EXPECT_STRNE(str1, str2); 	| the two C strings have different content |
| ASSERT_STRCASEEQ(expected_str, actual_str); 	| EXPECT_STRCASEEQ(expected_str, actual_str); 	| the two C strings have the same content, ignoring case |
| ASSERT_STRCASENE(str1, str2); 	| EXPECT_STRCASENE(str1, str2); 	| the two C strings have different content, ignoring case  |

## Example

### example.c
```c
#include <stdio.h>

int add(int x, int y)
{
    return (x+y);
}

int multiply(int x, int y)
{
    return (x*y);
}

```
### example.h
```c
#ifndef _EXAMPLE_H_
#define _EXAMPLE_H_

extern int add(int x, int y);
extern int multiply(int x, int y);
#endif

```

### test.cpp
```c
#include <gtest/gtest.h>
#include "example.h"

class MyTest: public testing::Test {
protected:
    static void SetUpTestCase() {
    }
    static void TearDownTestCase() {
    }
};


TEST_F(MyTest,test_add)
{
    int x = 10;
    int y = 20;
    EXPECT_EQ(30, add(10,20));
}
TEST_F(MyTest,test_multiply)
{
    int x = 10;
    int y = 20;
    EXPECT_EQ(200, multiply(10,20));
}

```

### Makefile

```
test: test.cpp
        g++ -I/opt/tools/ut_environment/include -lpthread test.cpp example.c  gtest_main.a  -o test

clean:
        rm test -f

```