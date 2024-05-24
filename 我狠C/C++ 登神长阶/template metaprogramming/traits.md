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


