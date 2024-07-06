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



## 0.3 SFINAE
> subsitution failure is not a error


- SFINAE 用于禁止不相关函数模板在重载解析时造成错误，当替换返回类型无意义时，会忽略（SFINAE out）匹配而选择另一个更差的匹配 
```cpp

#include <vector>

namespace jc {

template <typename T, std::size_t N>
T* begin(T (&a)[N]) {
  return a;
}

template <typename Container>
typename Container::iterator begin(Container& c) {
  return c.begin();
}

}  // namespace jc

int main() {
  std::vector<int> v;
  int a[10] = {};

  jc::begin(v);  // OK：只匹配第二个，SFINAE out 第一个
  jc::begin(a);  // OK：只匹配第一个，SFINAE out 第二个
}

```

- SFINAE 只发生与函数模板替换的及时上下文中即时上下文中，对于模板定义中不合法的表达式，不会使用`SFINAE`机制

```cpp
namespace jc {
template <typename T, typename U>
auto f(T t, U u) -> decltype(t + u) {
  return t + u;
}

void f(...) {}

template <typename T, typename U>
auto g(T t, U u) -> decltype(auto) {  // 必须实例化 t 和 u 来确定返回类型
  return t + u;  // 不是即时上下文，不会使用 SFINAE
}

void g(...) {}

struct X {};

using A = decltype(f(X{}, X{}));  // OK：A 为 void
using B = decltype(g(X{}, X{}));  // 错误：g<X, X> 的实例化非法,即可以匹配模板函数，但是实例化后的表达式出错，

}  // namespace jc

int main() {}
```

- - 一个简单的 `SFINAE` 技巧是使用尾置返回类型，用 `decltype` 和逗号运算符定义返回类型，在 `decltype` 中定义必须有效的表达式
```cpp
namespace C {
template <typename T>
auto size(T&& t) -> decltype(t.size(), T::size_type())
{
    return t.size();
}
template <typename U>
auto size(const U&)
{
    return 1;
}
}

int main(){

C::size(1);
}
```
- 如果替换时使用了类成员，则会实例化类模板，此期间发生的错误不在即时上下文中，即使另一个函数模板匹配无误也不会使用 `SFINAE` 
```cpp
namespace jc {

template <typename T>
class Array {
 public:
  using iterator = T*;
};

template <typename T>
void f(typename Array<T>::iterator) {}

template <typename T>
void f(T*) {}

}  // namespace jc

int main() {
  jc::f<int&>(0);  // 错误：第一个模板实例化 Array<int&>，创建引用的指针是非法的
}
```
- - SFINAE 最出名的应用是 [std::enable_if](https://en.cppreference.com/w/cpp/types/enable_if)

## 0.4 Deduction Guides

- 字符串字面值传引用是推断为字符数组，传值时推断为const char*,- C++17 可以定义 deduction guides 对特定类型的实参指定其推断类型
```cpp

```