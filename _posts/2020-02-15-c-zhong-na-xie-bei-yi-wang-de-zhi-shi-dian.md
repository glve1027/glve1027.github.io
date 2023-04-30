---
layout:       post
title:        "C中那些被遗忘的知识点"
author:       "GH"
header-style: text
catalog:      true
tags:
    - C
---

### 因为最近在看C++的知识点，发现很多C中的知识点有点不记得了，又把书拿出来翻了一下，整理了一些容易忽略的知识点。

<!-- more -->

* 函数的定义和实现可以分类，定义的时候可以省略形参的参数名，例如下面的代码：
```c
int minus(int, int); // 定义
int minus(int a, int b) { return a - b; } // 实现
```

* 如果函数只有定义，没有实现的时候, 编译器在编译的时候就会报错。

* 如果声明了一个局部变量, 但是没有赋值, 可能在不同的平台上会有不同的表现，目前在MAC OS上，会打印0, 而在Windows上则会直接报错，也会随机打印一个垃圾数据, 如果是全局变量, 例如int a, 默认会赋值0。

* 汉字或者字符串, 必须要用字符数组来表示，因为1个汉字 = 2个字符。

* 同时使用两种修饰符
```c
signed short int a;
unsigned long long int a;
```

* 在64位编译器环境下，short占2个字节(16位)，int占4个字节(32位)，long占8个字节(64位

* signed代表有符号，包括正数、负数和0；unsigned代表无符号，只包括正数和0。比如，signed的取值范围是-3276832767，那么unsigned的取值范围是065535

* 输出格式占位符号

```c
%d = 带符号的十进制形式输出整数（正数不带+）
%c = 输出一个字符
%s = 输出一个或者多个字符
%p = 地址内存
```

* printf("The price is %4d.", 14); => 4d可能会有空格来控制宽度（从左边开始算宽度） 【-4d则会从右边开始控制宽度】

* printf("My height is %f", 179.95f);默认会输出6位小数 【控制小数的位数%8.2f => 2位小数，宽度为8】

* 逗号表达式：表达式1, 表达式2, … …, 表达式n = 【从左到右的顺序，先计算表达式1，接着计算表达式2，...，最后计算表达式n，整个逗号表达式的值是最后一个表达式的值】

* 数组的定义： 类型 数组名[数组个数]

1. 数组的个数也可以是常量表达式int a[3+4], 也可以是字符常量int a['A']
2. a = &a[0]
3. 初始化: 类型 数组名[数组个数] = { 元素1，元素2 ...} = int a[2] = { 2, 3 }
4. c语言中对数组的越界不会检查
5. 当数组都赋值了，可以省略个数int a[] = { 1, 2, 3 }✅
6. int a[3] = {1,2,3}; => a是数组名，代表数组的地址，它是常量
7. 二维数组a[1][1]中a[0]、a[1]也是数组，是一维数组，而且a[0]、a[1]就是数组名，因此a[0]、a[1]就代表着这个一维数组的地址

* puts函数的使用

```c
char name[] = "gonghuan"
puts(name) // 从name的首地址开始一直输出到`\0`为止
```

* scanf函数

```c
char name[10];
scanf("%s", name) // !! 这里不需要&name 因为name就已经代表了数组的地址了
```

* putchar字符输出

```c
putchar(65) = A
putchar('A') = A
```

* 指针的定义：类名标识符 * 指针变量名

```c
int a = 10; // 定义一个int变量a
int *p; // 定义一个指针类型p
p = &a; // 将变量a的地址赋值给指针变量p
```

* 指针赋值

```c
int a = 10;
int *p = &a; // 将变量a的地址复制给指针变量p
*p = 9;      // 通过p存储的地址值，找到地址指向的具体指，将指修改为9, 同时a也就为9

int value = *p; // *p的意思是：根据p值(即变量a的地址)访问对应的存储空间，并取出存储的内容(即取出变量a的值)，赋值给value
```

* 遍历数组

```c
int a[4] = { 1, 2, 3, 4 };
    
for (int i = 0; i < 4 ; i++) {
    printf("%p\n", &a[i]);
}
    
printf("====\n");
    
int *p = a;
for (int i = 0 ; i < 4; i++) {
    printf("%d\n", *p); // 1,2,3,4
    p +=1;
}
    
printf("====\n");
for (int i = 0; i < 4; i++) {
    printf("%p---%d\n", a+i, *(a+i));
}
```

* 字符数组的遍历

```c
char names[3] = { 'g', 'h', '\0' };
char *p = names;
    
for (; *p != '\0'; p++) {
    printf("%c\n", *p);
}
```

* 定义字符
```c
char *names = "gh";
char name[] = "gh";
char name[3] = { 'g', 'h', '\0' };
```

* 下面这样赋值是错误的

```c
char name[3];
name = "gh"; // 这样是❌，因为name是数组的首地址，是常量，不能赋值
```

* 字符的注意点：

```c
char a[] = "gh";// 定义的是一个字符串变量！
char *p2 = "gh";// 定义的是一个字符串常量！
```

* 预编译的时候判断是否定义过某个宏

```c
#if defined(xxx)
...code...
#endif

// 等效
#ifdef xxx
#endif

// 未定义
#ifndef xxx // => (等效) #if !defined(xxx)
#endif
```

* 自动变量auto
> 所有的局部变量在默认情况下都是自动变量
> 在程序执行到声明自动变量的代码块(函数)时，自动变量才被创建

* 静态变量：
> 所有的全局变量都是静态变量
> 被关键字static修饰的局部变量也是静态变量
> 生命周期：静态变量在程序运行之前创建，在程序的整个运行期间始终存在，直到程序结束

* 哪些值可以存放在寄存器中
> 被关键字register修饰的自动变量都是寄存器变量
> 只有自动变量才可以是寄存器变量，全局变量和静态局部变量不行
> 寄存器变量只限于int、char和指针类型变量使用

* 外部函数：
> 不能存在同名的外部函数
> 使用extern[ 同时也可以省略 ]
> 如果在定义函数时省略extern，则隐含为外部函数
> extern是用来声明已经定义过而且能够访问的变量
> extern可以用来声明一个全局变量，但是不能用来定义变量

* 内部函数 [静态函数]
> 使用static修饰
> 这样该函数就只能在其定义所在的文件中使用
> 如果在不同的文件中有同名的内部函数，则互不干扰

* 声明一个外部变量 => extern int a;

* 定义一个变量 => int a;

* 定义结构体的时候可以定义变量

```c
struct Student {
    char *name;
    int age;
} stu;

// 也可以省略
struct {
    char *name;
    int age;
} stu;

// 使用的时候一定要加上struct
struct Student stu; // 这里和c++ 有点不同
```

* 结构体初始化

```c
struct Student stu = { "22", "gh" };  // 大括号按循序赋值

// 下面这样初始化是错误的❌，应该分开赋值
struct Student stu;
stu = { "gh", 12 };
```

* 结构体数据的初始化
```c
struct {
    char *name;
    int age;
} stu[2] = { {"M", 27}, {"N", 30} };
```

* 结构体当做函数参数
> 都是值传递
> 不会影响之前的值

* 枚举的定义和结构体很相似
```c
//先定义枚举类型，再定义枚举变量
enum Season { spring, summer, autumn, winter };
enum Season s;
//定义枚举类型的同时定义枚举变量
enum Season { spring, summer, autumn, winter } s;
//省略枚举名称，直接定义枚举变量
enum {spring, summer, autumn, winter} s;
```

* typeof

```c
// 定义一个结构体，顺便起别名
typedef struct MyPoint {
    float x;
    float y;
} Point;

// 可以省略结构体名称
typedef struct {
    float x;
    float y;
} Point;
```
