# 语法


 cpp lambda 表达式语法具体定义如下:
```cpp
[ captures ] ( params ) specifiers exception -> ret {body}
```


- [captures] -- 捕获列表
- (params)   -- 可选参数列表, 不需要参数时*可省略*
- specifiers  -- 可选限定符, C11 可以用$mutable$
- exception -- 可选异常说明符, 可用$noexcept$来指明lambda 是否抛异常
- ret            -- 可选返回值类型，如无retValue, 可以*省略->在内的整个部分*, 在有返回值时，也可不指定，此时*编译器会为我们推导出一个retValue*
- {body}      -- 与普通函数无一

# 捕获列表


- 捕获列表的捕获方式有两种 *按值捕获* 和 *引用捕获*， 标准规定能捕获的变量一定是*自动存储类型*
### 捕获值


```cpp
int main(){
	int x = 5, y = 8;
	auto foo = [x, y]{return x * y};
}
```

- 捕获值是将函数作用域的x与y*复制到lambda表达式的作用域内*,如果构造函数传参
- lambda表达式的一个*特性*: 捕获的变量默认常为*常量*, 或者说lambda是一个*常量函数(类似于常量成员函数)*
- 标准规定，捕获的变量必须是一个*自动存储类型*

- 使用$mutable$ 说明符可以移除lambda表达式的常量性
``` cpp
void bar3(){
	int x = 5, y = 8;
	auton foo = [x, y] () mutable { // 语法规定lambda 表达式如果存在specifier,形参列表不可省
		x += 1;
		y += 2;
		return x * y;
	}
}
```


#### 特殊的捕获方法


![[Pasted image 20231128143103.png]]

# lambda的实现原理

> lambda 表达式在编译期由编译器自动生成一个闭包类，在运行时由闭包类产生一个对象，我们称其为闭包
```cpp
#include <iostream>
class Bar { 
public: 
	Bar(int x, int y) : x_(x), y_(y) {} 
	int operator () () { return x_ * y_; } 
private: 
	int x_; int y_; 
}; 

int main() { 
	int x = 5, y = 8; 
	auto foo = [x, y] { return x * y; }; 
	Bar bar(x, y); 
	std::cout << "foo() = " << foo() << std::endl; 
	std::cout << "bar() = " << bar() << std::endl;
	}
```

上述两者明显区别如下:

1. 使用lambda不需要显示定义类，快捷方便
2. 使用函数对象在初始化时操作丰富，如 `Bar bar(x+y, x * y)`  但C11的lambda表达式不允许


### 小窥gcc 下 lambda的美

```cpp
#include <iostream>
int main(){
	int x = 5, y = 8;
	auto foo = [=] {return x * y};
	int z = foo();
}
```


![[Pasted image 20231128145615.png]]
![[Pasted image 20231128145628.png]]

### 无状态lambda表达式

```cpp
void f(void(*)()){}

void g() { f([] {});}
```

- 无状态lambda表达式可以隐式转换为函数指针

