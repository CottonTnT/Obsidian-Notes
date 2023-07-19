# 顶层 const 与底层 const


*顶层 const* ：表示指针本身是一个常量

*底层 const*：表示指针所指的对象是一个常量


```c
const int a = 10;//顶层
const int * const p = new int(10); //底层， 顶层
const int & ra = 10; //底层，引用机制类似指针
```



### 语法规则

- **当执行对象拷贝操作是，常量的顶层 const 不受什么影响，而底层的 const 必须一致**
- 引用不是对象不进行拷贝，不满足上面原则
- 常量引用在左侧，右侧可接任何有效值
- 非常量引用=常量，报错！
- 




# 用 complier 取代 preprocessor


***即用 consts, enums 和 inlines 取代  \#defines***


```c
#define ASPECT_RATIO 1.653 //No!
const double AspectRation = 1.653 //Yes!
```



### define 与 const 的优缺点

  https://blog.csdn.net/weibo1230123/article/details/81981384 





  



