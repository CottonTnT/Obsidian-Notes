# 1

## confusion code

```cpp
namespace jc {

template <typename T, typename U>
auto max(const T& a, const U& b) -> decltype(true ? a : b) {
  return a < b ? b : a;
}
}
```
- decltype
- -> ?


## knowledge point


1. **`->`**   C++14 允许 auto 作为返回类型，它通过 return 语句推断返回类型，C++11 则需要额外指定尾置返回类型，
2. **`decltype`** [[关键字详解#decltype | decltype详解]]



# 2

## confusion code

```cpp
namespace jc {

template <typename T>
const T& f(const char* s) {
  return s;
}

}  // namespace jc

int main() {
  const char* s = "downdemo";
  jc::f<const char*>(s);  // 错误：返回临时对象的引用
}
```

- 为什么错误是返回了临时对象的引用

## knowledge point

- 指针变量始终是变量，引用是别名


# 3

## confusion code

```cpp
#include <cassert>
#include <cstddef>

namespace jc {

template <typename T, typename U>
constexpr bool less(const T& a, const U& b) {
  return a < b;
}

template <typename T, std::size_t M, std::size_t N>
constexpr bool less(T (&a)[M], T (&b)[N]) {//what the fuck is this?
  for (std::size_t i = 0; i < M && i < N; ++i) {
    if (a[i] < b[i]) {
      return true;
    }
    if (b[i] < a[i]) {
      return false;
    }
  }
  return M < N;
}

}  // namespace jc

static_assert(jc::less(0, 42));
static_assert(!jc::less("down", "demo"));
static_assert(jc::less("demo", "down"));

int main() {}
```

- what is line 12 ? 

## knowledge point


- C++ 数组的引用
- 模板能在编译期推断出数组的大小 



# 

## confusion code

```cpp
template <class I>
struct iterator_traits{
	typedef typename I::value_type value_type;  //1
};
```
- 1 中的 typename 是干什么的

## knowledge point

- 我们需要使用"typename"关键字来告诉编译器"I:: value_type"是一个类型，而不是一个静态成员。

# 5

## confusion code

```cpp
template <typename T>
class A;

template <typename T>
std::ostream& operator<<(std::ostream& os, const A<T>& rhs);

template <typename T>
class A {
  friend std::ostream& operator<<<T>(std::ostream& os, const A<T>& rhs); //这里的<T>是用来干嘛的

 private:
  int n = 0;
};
```

- 9 行的 \<T> 是用来干嘛的

## knowledge point

- 在 C++模板中，`<T>`是用来*指定模板参数*的。在这个例子中，`operator<<`是一个模板函数，它需要一个模板参数来指定其操作数的类型。这里的`<T>`就是用来指定这个类型的。当你在类模板中声明友元函数模板时，你需要在函数名后面加上`<T>`来指定这个友元函数模板的模板参数。这是因为在类模板中，友元函数模板并不会自动地从类模板中继承模板参数。所以，`operator<< <T>`中的`<T>`是必须的，它告诉编译器这是一个模板函数，并且其模板参数是`T`。如果你不加`<T>`，编译器就会认为这是一个普通的非模板函数，这会导致编译错误。
