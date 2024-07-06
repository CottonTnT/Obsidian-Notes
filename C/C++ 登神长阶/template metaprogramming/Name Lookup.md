# 1 Name Lookup 
## 1.1 ADL
> Argument-Dependent Lookup


 - [Name lookup](https://en.cppreference.com/w/cpp/language/lookup) 是当程序中出现一个名称时，将其与引入它的声明联系起来的过程，它分为 [qualified name lookup](https://en.cppreference.com/w/cpp/language/qualified_lookup) 和 [unqualified name lookup](https://en.cppreference.com/w/cpp/language/lookup)，[unqualified name lookup](https://en.cppreference.com/w/cpp/language/lookup) 对于函数名查找会使用 [ADL](https://en.cppreference.com/w/cpp/language/adl)


```cpp
namespace T {
struct A {
};
void f1(int) { }
void f1(A a){}
}

namespace M {

void f1(int)
{
    f1(2); //只看见M::f1,故无限递归
}

void f1(T::A a)
{
    f1(a); // error, 通过 a的类型T::A,也看到T,也看到A::f1
        //因为 M::f1也可见，此处产生二义性调用错误
}

}

auto main() -> int
{
    return 0;
}
```

