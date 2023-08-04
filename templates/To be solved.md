#
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
