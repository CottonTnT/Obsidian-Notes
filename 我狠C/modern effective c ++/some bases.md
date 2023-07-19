
# 用 complier 取代 preprocessor


***即用 consts, enums 和 inlines 取代  \#defines***


```c
#define ASPECT_RATIO 1.653 //No!
const double AspectRation = 1.653 //Yes!
```



### define 与 const 的优缺点

  https://blog.csdn.net/weibo1230123/article/details/81981384 




## 用 const 替换 defines 注意情况


- 常量字符串
```c
#define str "Scott Meyers"
const char * const str = "Scott Meyers"; //放在头文件中时，肯定不希望指针本身、指针指向的内容改变
const std::string str = ("Scott Meyers");
```
  


- 涉及到 class-specific constants



## 用 enum 替代




## 用 inline 替代

```c
//call f with the maximum of a and b
// #define版

#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))
int a = 5, b = 0;
CALL_WITH_MAX(++a, b); // a is incremented twice
CALL_WITH_MAX(++a, b+10); // a is incremented once

// inline 版
template<typename T>                               // because we don't
inline void callWithMax(const T& a, const T& b)    // know what T is, we
{                                                  // pass by reference-to-
  f(a > b ? a : b);                                // const - see Item 20
}

```



# 只要可能就用const


## 顶层 const 与底层 const


*顶层 const* ：表示指针本身是一个常量, 放在\*右边

*底层 const*：表示指针所指的对象是一个常量\*左边


```c
const int a = 10;//顶层
const int * const p = new int(10); //底层， 顶层
const int & ra = 10; //底层，引用机制类似指针
```



### 一个规则

- **当执行对象拷贝操作是，常量的顶层 const 不受什么影响，而底层的 const 必须一致*



## iterator 与 const


