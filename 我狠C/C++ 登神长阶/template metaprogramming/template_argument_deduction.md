## 0.1 Deduced Context

- 复杂的类型声明的匹配过程从最顶层构造开始，然后不断递归子构造，即各种组成元素，这些构造被称为 deduced context，*non-deduced context 不会参与推断，而是使用其他处推断的结果*，受限类型名称如 `A<T>::type` 不能用来推断 T，非类型表达式如 `A<N + 1>` 不能用来推断 N

```c
namespace jc {

template <int N>
struct A {
  using T = int;

  void f(int) {}
};

template <int N>  // A<N>::T 是 non-deduced context，X<N>::*p 是 deduced context
void f(void (A<N>::*p)(typename A<N>::T)) {}

}  // namespace jc

int main() {
  using namespace jc;
  f(&A<0>::f);  // 由 A<N>::*p 推断 N 为 0，A<N>::T 则使用 N 变为 A<0>::T
}
```

- 默认实参不能用于推断
```cpp
template <typename T>
void g(T a = 32)
{
    std::cout << a;
}
int main()
{
    // T::g();错误，无法推断T
    T::g<int>();
    return 0;
}
```

## 0.2 特殊的推断情况

- 成员函数的推断
```


```