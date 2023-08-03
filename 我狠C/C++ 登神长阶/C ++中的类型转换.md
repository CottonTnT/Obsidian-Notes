## static_cast


```cpp
//1.执行基本的内置数据类型转换
float data = static_cast<float>(1);

//2.用于指针之间的类型转换
//指针之间的隐式转换是不合法的，除了void*
int type_int = 10;
float f_ptr1 = &type_int;//wrong,隐式转换无效
void* void_ptr = &type_int;//成！
float f_ptr2 = &type_int;//显式转换有效

//3.用于类之间的类型转换
class A{
public: int a;
};

class C: public A{
public: int a;
};
---
A a = static_cast<A>(c);//上行转换正常
C c_a = static_cast<C>(a);//下行转换无效
```

- static_cast 不做类型转换之间的类型检查



## const_cast

```c
//用于去除变量的const 或 volitale 属性

```


## reinterpret_cast

- 使用 reinterpret_cast 强制转换过程仅仅只是比特位的拷贝, 和 c 风格类似

- type-id 和 expression 中必须有一个是指针或引用类型
- reinterpret 不能转换掉 expression 的 const 或 volitale 属性
- reinterpret_cast 可以将指针或引用转为一个不为指针或引用的数据类型，但该数据类型与当前操作系统指针所占字节数必须一致

```c
reinterpret_cast<typeid>(expression)
```


## dynamic_cast


- dynamic_cast 是运行时处理的，运行时会进行类型检查
- dynamic_cast 不用于内置基本数据类型的强制转换，只能对指针或引用进行强制转换
- dynamic_cast 转换成功返回的是指向类的指针或引用，*转换失败返回nullptr*

