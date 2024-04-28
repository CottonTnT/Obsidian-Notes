# 1 指针

- C语言的指针只是一个Integer, 不包含类型信息，可以随意转换


# 2 可变参数


## 2.1 什么是C中可变参数

```c
int printf(const char* format, ...)
int scanf(const char* format, ...)
```

- 任何一个可变参数的函数都可以分为两部分：固定参数和可选参数。*至少要有一个固定参数*；可选参数可以是0个或多个，声明时用"…"表示。固定参数和可选参数[]()共同构成可变参数函数的参数列表。

## 2.2 实现原理

C语言中使用 `va_list` 系列变参宏实现变参函数，va means variable-argument(可变参数)。


x86平台VC6.0编译器中，stdarg.h头文件内变参宏定义如下：
```c
typedef char * va_list;

// 把 n 向上round为 sizeof(int) 的倍数
#define INTSIZEOF(n)       ( (sizeof(n)+sizeof(int)-1) & ~(sizeof(int)-1) )


// 初始化 ap 指针，使其指向第一个可变参数。v 是变参列表的前一个参数
#define va_start(ap,v)      ( ap = (va_list)&v + _INTSIZEOF(v) )

// 该宏返回当前变参值,并使 ap 指向列表中的下个变参
#define va_arg(ap, type)    ( *(type *)((ap += _INTSIZEOF(type)) - _INTSIZEOF(type)) )

// /将指针 ap 置为无效，结束变参的获取
#define va_end(ap)             ( ap = (va_list)0 )
```


### 2.2.1 INTSIZEOF(n)
`INTSIZEOF`考虑某些系统需内存地址对齐。从宏名应按`sizeof(int)`即栈粒度对齐，参数在内存中的地址均为`sizeof(int)=4`的倍数。

例如，若`1≤sizeof(n)≤4`，则`INTSIZEOF(n)＝4`；若`5≤sizeof(n)≤8`，则`INTSIZEOF(n)=8`。


### 2.2.2 va_start(ap,v)
`va_start`宏首先根据`(va_list)&v`得到参数 `v` 在栈中的内存地址，加上`INTSIZEOF(v)`即`v`所占内存大小后，使 `ap` 指向 `v` 的下一个参数。在使用的时候，一般用这个宏初始化 `ap` 指针，`v` 是变参列表的前一个参数，即最后一个固定参数，初始化的结果是 `ap` 指向第一个变参。

### 2.2.3 va_arg(ap, type)
这个宏取得 `type` 类型的可变参数值。首先 `ap += INTSIZEOF(type)`，即 `ap` 跳过当前可变参数而指向下个变参的地址；然后`ap-INTSIZEOF(type)`得到当前变参的内存地址，类型转换后解引用，最后返回当前变参值。

### 2.2.4 va_end(ap)

`va_end` 宏使 `ap` 不再指向有效的内存地址。该宏的某些实现定义为`((void*)0)`，编译时不会为其产生代码，调用与否并无区别。但某些实现中 `va_end` 宏用于在函数返回前完成一些必要的清理工作：如 `va_start` 宏可能以某种方式修改栈，导致返回操作无法完成，`va_end` 宏可将有关修改复原；又如 `va_start` 宏可能为参数列表动态分配内存以便于遍历，`va_end` 宏可释放此内存。因此，从使用 `va_star`t 宏的函数中退出之前，必须调用一次 `va_end` 宏。



