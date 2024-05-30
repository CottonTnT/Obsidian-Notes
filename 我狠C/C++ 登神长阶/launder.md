> defined in <new> since c++ 17


```cpp

#include <iostream> 

#include <new> 

#include <string> 

struct Data { 

	std::string value; 

	Data(const std::string& val) : value(val) {} 

	~Data() { 

		std::cout << "Data with value \"" << value << "\" is destroyed." << std::endl; 

	} 

}; 

int main() { 

	alignas(Data) char buffer[sizeof(Data)]; // 缓冲区对齐到 Data 的对齐要求 

	Data* p1 = new (buffer) Data("First"); // 在缓冲区中构造 Data 对象 

	std::cout << "p1 points to: " << p1->value << std::endl; 

	p1->~Data(); // 手动析构 Data 对象 

	Data* p2 = new (buffer) Data("Second"); // 在相同的缓冲区中构造新的 Data 对象 std::cout << "p2 points to: " << p2->value << std::endl; // 使用 std::launder 获取新的指针 

	Data* p3 = std::launder(reinterpret_cast<Data*>(buffer)); 

	std::cout << "p3 points to: " << p3->value << std::endl; // 检查 p2 和 p3 是否相同 

	if (p2 == p3) { std::cout << "p2 and p3 point to the same object." << std::endl; } else {

		 std::cout << "p2 and p3 point to different objects." << std::endl; 

	} 

	p3->~Data(); // 手动析构新的 Data 对象 

	return 0; 

}
```
