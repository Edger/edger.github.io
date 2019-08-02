---
layout:     post
title:      "C++ 学习笔记 "
subtitle:   " 学习 C++ 时做的一些笔记 "
date:       2019-01-07 17:27:32
author:     "Edger"
header-img: "img/post-bg-C++-learnin.jpg"
catalog:    true
tags:

    - C/C++ 
---


#### 1. 为什么尽量不要使用 using namespace std ？

主要是为了避免命名冲突，但是不宜因噎废食。

详见 [【知乎】为什么尽量不要使用 using namespace std？](https://www.zhihu.com/question/26911239)

#### 2. #include <> 和 #include \"\" 的区别

**#include <>** 先到系统目录中找头文件，如果没找到，再到当前目录下寻找。如果要引用像 stdio.h、stdlib.h 等标准的头文件，用 **#include <>** 这个方法。

**#include \"\"** 先到当前目录下寻找头文件，如果没找到，再到系统目录下寻找。如果要使用当前目录下自定义的头文件，则使用 **#include \"\"** 这个方法。

#### 3. 在 C++ 中 main 函数前面为什么要加上数据类型，比如：int void ?

**main** 函数的返回值是返回给主调进程，使主调进程得知被调用程序的运行结果。

标准规范中规定 **main** 函数的返回值为 **int**，一般约定返回 0 值时代表程序运行无错误，其它值均为错误号，但该约定并非强制。

如果程序的运行结果不需要返回给主调进程，或程序开发人员确认该状态并不重要，比如所有出错信息均在程序中有明确提示的情况下，可以不写 **main** 函数的返回值。在一些检查不是很严格的编译器中，比如 VC, VS 等，**void** 类型的 **main** 函数是允许的。不过在一些检查严格的编译器下，比如 g++ , 则要求 **main** 函数的返回值必须为 int 型。

所以在编程时，区分程序运行结果并以 **int** 型返回，是一个良好的编程习惯。

#### 4. # if condition ... # endif

**# if condition ... # endif** 可以用来实现注释，且可以实现嵌套，格式为：

```cpp
# if condition
     code
# endif 
```

当 **condition** 为 0 时不执行 **code** 代码，当 **condition** 为 1 时执行 **code** 代码。

**# if condition ... # endif** 有助于程序调试，测试时使用 **# if 1** 来执行测试代码，发布时使用 **# if 0** 来屏蔽测试代码。

> **condition** 可以是任意的条件语句。

**# if condition ... # endif** 还可以配合 **else** 使用。下面的代码如果 **condition** 为 **true** 执行 **code1** ，否则执行 **code2**。

```cpp
# define condition 1

# if condition
     code1
# else
     code2
# endif
```
#### 5. 常量和字面量

常量是固定值，在程序执行期间不会改变。这些固定的值，又叫做字面量。

#### 6. 变量的声明和定义

定义包含了声明，但是声明不包含定义，如

```cpp
int a = 0;     // 定义并声明了变量 a
extern int a;  // 只是声明了有一个变量 a 存在，具体 a 在哪定义的，需要编译器编译的时候去找。
```

函数也是类似，定义的时候同时声明。但如果只是声明，编译器只知道有这么个函数，具体函数怎么定义的要编译器去找。

```cpp
void fun1();  // 函数声明

void fun1()   // 函数定义
{  
    cout << "fun1" << endl;
}
```
#### 7. 变量左值放在等式的左边用来判空，以防出现逻辑错误

```cpp
#include "stdafx.h"

int  *a = NULL;

int main()
{

    if (a = NULL) // 赋值语句，并非判断语句
    {
        return false;
    }
    
    if (NULL = a) // !!!!ERROR  此处在程序编译阶段不通过，“=” 右边不能为变量名
    {
        return false;
    }
    
    if (a == NULL) // 可行，判断指针 a 是否为空
    {
        return false;
    }
    
    if (NULL == a) // 可行，判断指针 a 是否为空。在实际项目中，
                   // 为了防止将 “==” 误写作 “=” 推荐讲变量名写在右侧，
                   // 编译器可以帮助寻找错误
    {
        return false;
    }

    return 0;

}
```
#### 8. 前 ++ 和后 ++ 总结

前 ++ 是先自加再使用，而后 ++ 是先使用再自加！
