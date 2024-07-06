## 0.1 空基类优化(EBCO, Empty Base Class Optimization)
- 为了保证给类动态分配内存时有不同的地址，C++ 规定空类大小必须大于 0

```c
namespace jc {

struct A {};
struct B {};

static_assert(sizeof(A) > 0);
static_assert(sizeof(B) > 0);

}  // namespace jc

int main() {
  jc::A a;
  jc::B b;
  static_assert((void*)&a != (void*)&b);
}
```


- 一般编译器将空类大小设为 1 字节，对于空类存在继承关系的情况，如果支持 EBCO，可以优化派生类的空间占用大小

```cpp
/* 不支持 EBCO 的内存布局：
 * [    ] } A } B } C
 * [    ]     }   }
 * [    ]         }
 *
 * 支持 EBCO 的内存布局：
 * [    ] } A } B } C
 */

namespace jc {

struct A {
  using Int = int;
};

struct B : A {};
struct C : B {};

static_assert(sizeof(A) == 1);
static_assert(sizeof(A) == sizeof(B));
static_assert(sizeof(A) == sizeof(C));

}  // namespace jc

int main() {}
```

- 模板参数可能是空类

```c
namespace jc {

struct A {};
struct B {};

template <typename T, typename U>
struct C {
  T a;
  U b;
};

static_assert(sizeof(C<A, B>) == 2);

}  // namespace jc

int main() {}
```

- 为了利用 EBCO 压缩内存空间，可以将模板参数设为基类

```c
namespace jc {

struct A {};
struct B {};

template <typename T, typename U>
struct C : T, U {};

static_assert(sizeof(C<A, B>) == 1);

}  // namespace jc

int main() {}
```

- 但模板参数可能是相同类型，或者不一定是类，此时将其设为基类在实例化时会报错。如果已知一个模板参数类型为空类，把可能为空的类型参数与一个不为空的成员利用 EBCO 合并起来，即可把空类占用的空间优化掉

```cpp
namespace jc {

template <typename Base, typename Member>
class Pair : private Base {
 public:
  Pair(const Base& b, const Member& m) : Base(b), member_(m) {}

  const Base& first() const { return (const Base&)*this; }

  Base& first() { return (Base&)*this; }

  const Member& second() const { return this->member_; }

  Member& second() { return this->member_; }

 private:
  Member member_;
};

template <typename T>
struct Unoptimizable {
  T info;
  void* storage;
};

template <typename T>
struct Optimizable {
  Pair<T, void*> info_and_storage;
};

}  // namespace jc

struct A {};

static_assert(sizeof(jc::Unoptimizable<A>) == 2 * sizeof(void*));
static_assert(sizeof(jc::Optimizable<A>) == sizeof(void*));

int main() {}
```

## 0.2 奇异递归模板模式（CRTP）
> The Curiously Recurring Template Pattern

- CRTP 的实现手法是将派生类作为基类的模板参数