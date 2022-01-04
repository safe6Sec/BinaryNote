## 前言

菜鸡的我又来学c了，之前以为自己已经懂了。一去学习win32，简直一塌糊涂。简单记一些之前没学到的点。





## 变量

声明变量的格式：

```
变量类型 变量名;

注：变量类型用来说明数据宽度是多大

例：int 4字节、short 2字节、char 1字节
```

c中变量类型有这几种：

1. 基本类型：整数、浮点
2. 构造类型：数组、结构体、共用体（联合）
3. 指针类型
4. 空类型：void





**全局变量**：

1. 在函数体外定义，并且作用于全局；
2. 在程序编译完成后，内存地址和宽度就已经确定下来了，**变量名就是内存地址的别名**；
3. 只要程序启动，全局变量就已经存在，如若变量在一开始声明时没有赋值，则初始值为0；
4. 如果不重新编译，全局变量的内存地址永远都不会变；
5. 全局变量中的值任何程序都可以改，其最终存储的就是最后一次修改的值。



```c
int a = 123;
 
int plus() {
    return a+1;
}
 
void main() {
    plus();
    return;
}
```

**局部变量**：

1. 在函数体内定义，作用于当前所在函数；
2. 局部变量的内存是在**堆栈**中分配的，程序执行时才分配，不执行则不会分配，我们无法预知程序何时执行，也就意味着我们无法知道局部变量的内存地址；
3. 不确定局部变量的内存地址，所以其也就只能作用于当前函数内部，其他函数不能使用。



```c
int plus() {
    int a = 123;
    return a+1;
}
 
void main() {
    plus();
    return;
}
```

结论：

1. 全局变量是可以没有初始值直接使用的，因为系统默认给其0为初始值；
2. 局部变量在使用前必须要赋值，因为系统不会初始化它，而只有在其赋值时才会分配内存。



**在C语言中不添加如下关键词，默认就是有符号数**：

```
signed char x = -1; // 有符号数
 
unsigned char a = 1; // 无符号数
```

什么时候去使用有符号、无符号呢？

1. 有符号：涉及负数的领域，如炒股
2. 无符号：无负数的领域，如年龄



## 结构体



当需要一个容器能够存储**5个数据，这5个数据中有1字节的，2字节的有10字节的**...你会怎么做？

这时候就需要结构体，用来存放不同的数据类型，**相对于把多种不同的基础类型组成一个新的数据类型**。

```c
struct 类型名{
    // 可以定义多种类型
    int a;
    char b;
    short c;
};
```

结构体的特点是什么呢？

1. char/int/数组 等类型是编译器已知类型，我们称之为内置类型；但结构体编译器并不认识，当我们使用的时候需要告诉编译器一声，我们也称之为自定义类型；
2. 如上代码所示我们仅仅是告诉编译器，我们定义的类型是什么样的，这段代码本身并不会占用内存空间；
3. 结构体声明的位置和变量一样，都存在全局和局部的属性；
4. 结构体在定义的时候，除了本身以外可以使用任何类型。

```c
struct stStudent
{
    int stucode;
    char stuName[20];
    int stuAge;
    char stuSex;
};
 
stStudent student = {101,"张三",18,'M'};
```



赋值

```c
struct stPoint
{
    int x;
    int y;
};
 
stPoint point = {10,20};
int x;
int y;
 
// read
x = point.x;
y = point.y;
 
// write
point.x = 100;
point.y = 200;
```



**新语法，定义结构体类型的时候，直接定义变量**

```c
struct stPoint
{
    int x;
    int y;
}point1,point2,point3;
 
point1.x = 1;
point1.y = 2;
 
point2.x = 3;
point2.y = 4;
 
point3.x = 5;
point3.y = 6;
```



**打印数据宽度可用函数sizeof()**



结构体和int、char等本质是没有区别的，也可以有数组。



```c
// 定义结构体类型
struct stStudent
{
    int Age;
    int Level;
};
 
// 定义结构体变量
struct stStudent st;
// 定义结构体数组
struct stStudent arr[10]; 或者 stStudent arr[10];
```



```c
// 结构体数组名[下标].成员名
 
arr[0].Age = 10;
int age = arr[0].Age;
```



```c
struct stStudent{
    int Age;
    char Name[0x20];
};
struct stStudent arr[3] = {{0,"张三"},{1,"李四"},{2,"王五"}};
 
int x = arr[0].Age;
```



## 指针



在C语言里面指针是一种数据类型，是给编译看的，也就是说指针与int、char、数组、结构体是平级的，都是一个类型。



指针类型的变量宽度永远是4字节，无论类型是什么，无论有几个星号。



指针定义

```c
char* x;
short* y;
int* a;
float* b;
```



&符号是取地址符，任何变量都可以使用&来获取地址，但不能用在常量上。



```c
#include <stdio.h>
 
struct Point {
    int x;
    int y;
};
 
char a;
short b;
int c;
Point p;
 
int main()
{
    printf("%x %x %x %x \n", &a, &b, &c, &p);
    return 0;
}
```



字符串的几种表现形式：

```c
char str[6] = {'A','B','C','D','E','F'};
char str[] = "ABCDE";
char* str = "ABCDE";
```



字符串函数

```c
int strlen(char* s); // 返回类型是字符串s的长度，不包含结束符号\0
char* strcpy(char* dest, char* src); // 复制字符串src到dest中，返回指针为dest的值
char* strcat(char* dest, char* src); // 将字符串src添加到dest尾部，返回指针为dest的值
int strcmp(char* s1, char* s2); // 比较s1和s2，一样则返回0，不一样返回非0
```



### 结构体指针



```c
#include <stdio.h>
 
struct Point {
    int a;
    int b;
};
 
int main()
{
    Point p;
    
    Point* px = &p;
    
    printf("%d \n", sizeof(px));
 
    return 0;
}
```

我们打印结构体指针的宽度，最终结果是4（默认4）,这时候我们需要知道不论你是什么类型的指针，其特性就是我们之前说的指针的特性，并不会改变。

如下代码就是使用结构体指针：

```c
// 创建结构体
Point p;
p.x=10;
p.y=20;
 
// 声明结构体指针
Point* ps;
 
// 为结构体指针赋值
ps = &p;
 
// 通过指针读取数据
printf("%d \n",ps->x);
 
// 通过指针修改数据
ps->y=100;
 
printf("%d\n",ps->y);
```

**结构体指针,调用成员变量需要用->进行调用**。





## 调用约定

函数调用约定就是告诉编译器怎么传递参数，怎么传递返回值，怎么平衡堆栈。

常见的几种调用约定:

一般情况下自带库默认使用 __stdcall，我们写的代码默认使用 __cdecl

| 调用约定   | 参数压栈顺序                           | 平衡堆栈     |
| ---------- | -------------------------------------- | ------------ |
| __cdecl    | 从右至左入栈                           | 调用者清理栈 |
| __stdcall  | 从右至左入栈                           | 自身清理堆栈 |
| __fastcall | ECX/EDX 传送前两个，剩下：从右至左入栈 | 自身清理堆栈 |

更换调用约定就是在函数名称前面加上关键词：

```c
int __stdcall method(int x,int y)
{
    return x+y;
}
 
method(1,2);
```



### 函数指针

好奇怪的语法，不知道这样做的意义在哪？直接调用函数不行吗？

函数指针变量定义的格式：

```c
// 返回类型 (调用约定 *变量名)(参数列表);
 
int (__cdecl *pFun)(int,int);
```

函数指针类型变量的赋值与使用：

```c
#include <stdio.h>
//返回两个数中较大的一个
int maxFun(int a, int b){
    return a>b ? a : b;
}
int main(){
    int x, y, maxval;
    //定义函数指针
    int (*pmax)(int, int) = maxFun;  //也可以写作int (*pmax)(int a, int b)
    printf("Input two numbers:");
    scanf("%d %d", &x, &y);
    maxval = (*pmax)(x, y);
    printf("Max value: %d\n", maxval);
    return 0;
}
```



## 宏

预处理一般是指在程序源代码被转换为二进制代码之前，由预处理器对程序源代码文本进行处理，处理后的结果再由编译器进一步编译。

预处理功能主要包括宏定义、文件包含、条件编译三部分。

### 宏定义的注意事项

1. 只做字符序列的替换工作，不做任何语法检测，在编译前处理
2. 宏名标识符与左圆括号之前不允许有空白符，应紧接在一起
3. 为了避免出错，宏定义中给形参加上括号
4. 多行声明时，回车换行前要加上字符'\'，注意字符'\'后要紧跟回车键，中间不能有空格或其他字符
5. 末尾不需要分号



### 宏定义

感觉就是常量，约定成俗用大写。

简单的宏：**#define 标识符 字符序列**

```c
#define FALSE 0
#define NAME "LUODAOYI"
#define __IN
#define __OUT
```



极端例子：

用宏定义一个函数，分别用abcd占位。

```c
#define NAME "LUODAOYI"
#define A int method() {
#define B char buffer[0x10];
#define C strcpy(buffer,NAME);
#define D return 0;}
#define E method();
 
// use
A
B
C
D
 
int main()
{
    E
    return 0;
}
```

带参数的宏：**#define 标识符(参数表) 字符序列**

```c
#define MAX(A,B)((A)>(B)?(A):(B))
 
int method()
{
    int x = 1;
    int y = 2;
    int z = MAX(x,y);
    return 0;
}
```

多行定义，'\\' 后不可有空格

```c
#define A for(int i=0;i<length;i++)\
{\
    printf("%d \n",arr[i]);\
}\
 
int method(int arr[],int length)
{
     A
     return 0;
}
int main()
{
    int arr[] = {1,2,3,4,5,6,7,8,9,0};
    method(arr,10);
}
```

直接使用宏定义函数

```c
#define MYPRINT(X,Y) for(int i=0;i<(Y);i++)\
{\
    printf("%d \n",(X)[i]);\
}\
return 0;\
 
int main()
{
    int arr[] = {1,2,3,4,5,6,7,8,9,0};
    MYPRINT(arr,10);
}
```

使用宏定义函数和普通函数的区别：使用宏比较节省空间，因为使用宏定义函数，没有堆栈提升操作，也就是不会作为函数调用而是直接内联到代码内。



### 条件编译与文件包含

条件编译，就是当满足条件时才会要求编译器进行编译；如下代码当if成立则编译printf，否则就不编译：



```c
#define DEBUG 0
 
int main()
{
#if DEBUG
    printf("--------")
#endif
    return 0;
}
```

**if define**之类的，我们都称之为预处理指令，如下是常用的。

**预处理指令：条件编译是通过预处理指令实现的**

| 指令     | 用途                                                      |
| -------- | --------------------------------------------------------- |
| #define  | 定义宏                                                    |
| #undef   | 取消已定义的宏                                            |
| #if      | 如果给定条件为真，则编译下面代码                          |
| #endif   | 如果前面的#if给定条件不为真，当前条件为真，则编译下面代码 |
| #else    | 同else                                                    |
| #endif   | 结束一个#if…#else条件编译块                               |
| #ifdef   | 如果宏已经定义，则编译下面代码                            |
| #ifndef  | 如果宏没有定义，则编译下面代码                            |
| #include | 包含文件                                                  |

### 文件包含

文件包含有两种格式，分别是 #include "file" 和 #include <file>

**使用双引号**：系统首先到当前目录下查找被包含的文件，如果没找到，再到系统指定的包含文件目录(由用户在配置环境时设置)去找
**使用尖括号**：直接到系统指定的包含文件目录去查找

所以**系统文件用 <> 尖括号，自己定义的文件用 "" 双引号**。

文件包含可能会存在重复包含的情况，我们可以使用条件编译、前置声明的方式避免。

