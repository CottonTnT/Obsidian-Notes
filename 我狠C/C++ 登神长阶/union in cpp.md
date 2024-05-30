# 1 正确使用union
>如果尝试访问未在其生命周期内的联合体成员，则会导致未定义行为。

```cpp
#include <iostream> 
#include <string>
#include <type_traits> 
union Example { 
	int intValue;
	float floatValue; 
	std::string strValue;
	 Example() { new (&intValue) int(); } 
	 ~Example() { intValue.~int(); }
 }; 
 int main() { 
	 Example e; 
	 // 正确使用：初始化 int 成员 new (&e.intValue) int(42); 
	 std::cout << "intValue: " << e.intValue << std::endl; 
	 e.intValue.~int(); 
	 // 正确使用：切换到 float 成员 
	 new (&e.floatValue) float(3.14); 
	 std::cout << "floatValue: " << e.floatValue << std::endl; 
	 e.floatValue.~float(); 
	 // 正确使用：切换到 std::string 成员 
	 new (&e.strValue) std::string("Hello, Union!"); 
	 std::cout << "strValue: " << e.strValue << std::endl; 
	 e.strValue.~std::string(); 
	 // 错误使用：尝试访问已经销毁的 int 成员 
	 // std::cout << "intValue: " << e.intValue << std::endl; 
	 // 未定义行为 
	 return 0;
}
```
或者可以简化一下
```cpp
#include <iostream>
#include <string> 
#include <type_traits> 
union Example { 
	int intValue; 
	float floatValue; 
	std::string strValue; 
	enum class Type { None, Int, Float, String }; 
	Type currentType; 
	Example() : currentType(Type::None) {} 
	~Example() { destroyCurrent(); }
	 void setInt(int value) { 
		 destroyCurrent(); 
		 new (&intValue) int(value); 
		 currentType = Type::Int;
	} 
	void setFloat(float value) { 
		destroyCurrent(); 
		new (&floatValue) float(value); 
		currentType = Type::Float;
	} 
	void setString(const std::string& value) { 
		destroyCurrent(); 
		new (&strValue) std::string(value); 
		currentType = Type::String; 
	} 
	void destroyCurrent() { 
		switch (currentType) { 
			case Type::Int: 
				intValue.~int(); 
				break; 
			case Type::Float: 
				floatValue.~float(); 
				break; 
			case Type::String: 
				strValue.~std::string(); 
				break; 
			default: 
				break; 
		} 
		currentType = Type::None; 
	} 
}; 
int main() { 
	Example e; 
	// 设置并使用 int 成员 
	e.setInt(42); 
	std::cout << "intValue: " << e.intValue << std::endl; 
	// 设置并使用 float 成员 
	e.setFloat(3.14f); 
	std::cout << "floatValue: " << e.floatValue << std::endl; 
	// 设置并使用 string 成员 
	e.setString("Hello, Union!"); 
	std::cout << "strValue: " << e.strValue << std::endl; 
	// 析构时会自动调用 destroyCurrent()，安全析构活动成员 
	return 0; 
}
```

- 在 C++17 中，引入了 `std::variant`，这是一个更安全、更现代的方式来处理类似联合体的情况。`std::variant` 确保只有一个成员是活动的，并提供了类型安全的访问方式。以下是使用 `std::variant` 的示例：
```cpp
#include <iostream> 
#include <variant> 
#include <string>
int main() { 
	std::variant<int, float, std::string> v; 
	v = 42; 
	std::cout << "intValue: " << std::get<int>(v) << std::endl; 
	v = 3.14f; 
	std::cout << "floatValue: " << std::get<float>(v) << std::endl; 
	v = std::string("Hello, Variant!");
	 std::cout << "strValue: " << std::get<std::string>(v) << std::endl; 
	 return 0; 
}
```