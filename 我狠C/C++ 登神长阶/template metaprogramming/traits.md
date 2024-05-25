- [std::is_same](https://en.cppreference.com/w/cpp/types/is_same)

```c
#include <type_traits>

namespace jc {

template <typename T, typename U>
struct is_same {
  static constexpr bool value = false;
};

template <typename T>
struct is_same<T, T> {
  static constexpr bool value = true;
};

template <typename T, typename U>
constexpr bool is_same_v = is_same<T, U>::value;

}  // namespace jc

static_assert(jc::is_same_v<int, int>);
static_assert(!jc::is_same_v<int, double>);
static_assert(!jc::is_same_v<int, int&>);

int main() {}
```

- 获取元素类型
```cpp
template <typename T>
struct get_element {
    using type = T;
};

template <typename T>
struct get_element<T[]> {
    // using type = T;error, if T[] = int[][][],T = T[][]
    using type = typename get_element<T>::type;
};

template <typename T, std::size_t N>
struct get_element<T[N]> { //partial specialization for T[N][N][N]
    using type = typename get_element<T>::type;
};

template <typename T>
using get_element_t = typename get_element<T>::type;

static_assert(is_same_v<get_element_t<int>, int>);
static_assert(is_same_v<get_element_t<int[]>, int>);
static_assert(is_same_v<get_element_t<int[3][4][5]>, int>);
```

- - [std::remove_reference](https://en.cppreference.com/w/cpp/types/remove_reference)
```cpp
template <typename T>
struct remove_ref {
    using type = T;
};

template <typename T>
struct remove_ref<T&> {
    using type = T;
};

template <typename T>
struct remove_ref<T&&> {
    using type = T;
};

template <typename T>
using remove_ref_t = typename remove_ref<T>::type;

static_assert(is_same_v<remove_ref_t<int>, int>);
static_assert(is_same_v<remove_ref_t<int&>, int>);
static_assert(is_same_v<remove_ref_t<int&&>, int>);
```

- [std::enable_if](https://en.cppreference.com/w/cpp/types/enable_if)

```cpp
namespace AA {
struct Base {
};
struct Derived1 : Base {
};
struct Derived2 : Base {
};

template <typename T, template <typename...> class V>
void impl(const V<T>&)
{
    static_assert(std::is_constructible_v<Base*, T*>);
}

template <typename T, template <typename...> class V, typename... Args, ::enable_if_t<std::is_constructible_v<Base*, T*>, void*> = nullptr>
void f(const V<T>& t, Args&&... args)
{
    impl(t);
    if constexpr (sizeof...(args) > 0)
    {
        f(std::forward<Args>(args)...);
    }
}
}

int main()
{
    f(std::vector<AA::Derived1> {}, std::list<AA::Derived2> {});
    return 0;
}
```



## 0.1 元函数转发
- Traits 可以视为对类型做操作的函数，称为元函数，元函数一般包含一些相同的成员，将相同成员封装成一个基类作为基本元函数，继承这个基类即可使用成员，这种实现方式称为元函数转发，标准库中实现了 [std::integral_constant](https://en.cppreference.com/w/cpp/types/integral_constant) 作为基本元函数

```cpp
#include <cassert>
#include <type_traits>

namespace jc {

template <class T, T v>
struct integral_constant {
  static constexpr T value = v;
  using value_type = T;
  using type = integral_constant<T, v>;
  constexpr operator value_type() const noexcept { return value; }//定义了该类可以隐式转换为T
  constexpr value_type operator()() const noexcept { return value; }//可以函数调用一样返回其中的值

};

constexpr int to_int(char c) {
  // hexadecimal letters:
  if (c >= 'A' && c <= 'F') {
    return static_cast<int>(c) - static_cast<int>('A') + 10;
  }
  if (c >= 'a' && c <= 'f') {
    return static_cast<int>(c) - static_cast<int>('a') + 10;
  }
  assert(c >= '0' && c <= '9');
  return static_cast<int>(c) - static_cast<int>('0');
}

template <std::size_t N>
constexpr int parse_int(const char (&arr)[N]) {
  int base = 10;   // to handle base (default: decimal)
  int offset = 0;  // to skip prefixes like 0x
  if (N > 2 && arr[0] == '0') {
    switch (arr[1]) {
      case 'x':  // prefix 0x or 0X, so hexadecimal
      case 'X':
        base = 16;
        offset = 2;
        break;
      case 'b':  // prefix 0b or 0B (since C++14), so binary
      case 'B':
        base = 2;
        offset = 2;
        break;
      default:  // prefix 0, so octal
        base = 8;
        offset = 1;
        break;
    }
  }
  int res = 0;
  int multiplier = 1;
  for (std::size_t i = 0; i < N - offset; ++i) {
    if (arr[N - 1 - i] != '\'') {
      res += to_int(arr[N - 1 - i]) * multiplier;
      multiplier *= base;
    }
  }
  return res;
}

template <char... cs>
constexpr auto operator"" _c() {
  return integral_constant<int, parse_int<sizeof...(cs)>({cs...})>{};
}

static_assert(std::is_same_v<decltype(2_c), integral_constant<int, 2>>);
static_assert(std::is_same_v<decltype(0xFF_c), integral_constant<int, 255>>);
static_assert(
    std::is_same_v<decltype(0b1111'1111_c), integral_constant<int, 255>>);

}  // namespace jc

static_assert(jc::integral_constant<int, 42>::value == 42);
static_assert(std::is_same_v<int, jc::integral_constant<int, 0>::value_type>);
static_assert(jc::integral_constant<int, 42>{} == 42);

int main() {
  jc::integral_constant<int, 42> f;
  static_assert(f() == 42);
}
```

- 利用元函数转发实现[std::is_same](https://en.cppreference.com/w/cpp/types/is_same)

```cpp
template <bool B>

using bool_constant = integral_constant<bool, B>;
using true_type = bool_constant<true>;
using false_type = bool_constant<false>;

template <typename T, typename U>
struct is_same2 : false_type {};

template <typename T>
struct is_same2<T, T> : true_type {};

template <typename T, typename U>
constexpr bool is_same_v = is_same<T, U>::value;
```



## 0.2 SFINAE-based traits

- [std::is_default_constructible](https://en.cppreference.com/w/cpp/types/is_default_constructible)
```cpp
namespace CC {
template <typename T>
struct is_default_constructible {
private:
    template <typename U, typename = decltype(U())>
    static std::true_type test(void*);

    template <typename>
    static std::false_type test(...);

public:
    static constexpr bool value = decltype(test<T>(nullptr))::value;
};

template <typename T>
constexpr bool is_default_constructible_v = is_default_constructible<T>::value;

}  
struct CannotDefaultCons {
    CannotDefaultCons() = delete;
};
static_assert(!CC::is_default_constructible_v<CannotDefaultCons>);
```

- [std::void_t](https://en.cppreference.com/w/cpp/types/void_t)

```c
#include <type_traits>

namespace jc {

template <typename...>
using void_t = void;

template <typename, typename = void_t<>>
struct is_default_constructible : std::false_type {};

template <typename T>
struct is_default_constructible<T, void_t<decltype(T())>> : std::true_type {};

template <typename T>
constexpr bool is_default_constructible_v = is_default_constructible<T>::value;

}  // namespace jc

struct A {
  A() = delete;
};

static_assert(!jc::is_default_constructible_v<A>);

int main() {}
```

