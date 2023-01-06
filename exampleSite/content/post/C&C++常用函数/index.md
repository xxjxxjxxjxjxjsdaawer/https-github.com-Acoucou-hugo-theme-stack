+++
author = "coucou"
title = "C/C++常用函数"
date = "2022-11-25"

description = "这里有C/C++常用的函数"

categories = [
    "C语言"
]

tags = [
    "C语言"
]

+++

![](1.jpg)

## C语言常用函数

### 数学函数

```c
#include <math.h>
幂函数: pow

取整函数:round/floor/ceil
四舍五入函数:round()函数
向下取整函数:floor()函数
向上取整函数:ceil()函数

绝对值函数:abs/fabs
fabs为实数绝对值函数。传入的参数为一个实数，返回值是这个实数求完绝对值以后的结果。
abs为整数绝对值函数。 传入的参数为一个整数，返回值是这个正数求完绝对值以后的结果。
abs的头文件为#include <stdlib.h>，这点比较特殊，不同于其他的数学函数。
    
平方根函数:sqrt
```

### 字符串函数

```c
int atoi(const char *str);
//函数作用: 把参数 str 所指向的字符串转换为一个整数
//头文件: stdlib.h
long int atol(const char *str);
//函数作用: 把参数 str 所指向的字符串转换为一个长整数
//头文件: stdlib.h
 
char *itoa( int value, char *string,int radix);
//函数作用: 把一个整数转换为字符串
//头文件: stdlib.h
```

9. ~~~~~C/C++常用函数汇总

https://blog.csdn.net/m0_37602827/article/details/88685213

https://www.cnblogs.com/fkissx/p/4610947.html
