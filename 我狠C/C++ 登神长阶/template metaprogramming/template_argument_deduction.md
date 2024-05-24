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
```cpp
struct B {
    B() = default;
    B(int a)
        : m_a { a } {};
    void f(const int* p) const noexcept
    {
        printf("call B::f %d %d", m_a, *p);
    }
    int m_a;
};

template <typename RT, typename T, typename... Args>
void fk(RT (T::*fucptr)(Args...) const)
{
    T object { 2 };
    int ia = 3;
    std::invoke(fucptr, &object, &ia);
}

int main()
{
    fk(&T::B::f);
    return 0;
}
```

- 参数包的推断
```cpp
template <typename T, typename U>
struct A {};

template <typename T, typename... Args>
void f(const A<T, Args>&...);

template <typename... T, typename... U>
void g(const A<T, U>&...);

int main(){
  f(A<int, bool>{}, A<int, char>{});   // T = int, Args = [bool,char]
  g(A<int, bool>{}, A<int, char>{});   // T = [int, int], U = [bool, char]
  g(A<int, bool>{}, A<char, char>{});  // T = [int, char], U = [bool, char]
  // f(A<int, bool>{}, A<char, char>{});  // 错误，T 分别推断为 int 和 char
return 0;
}
```

- 完美转发处理空指针常量时，整型值会被当做常量值0



