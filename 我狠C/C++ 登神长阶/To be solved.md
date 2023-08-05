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