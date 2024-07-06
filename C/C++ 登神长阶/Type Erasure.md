`Type Erasure` 即类型擦除，why we need it ? 当我们*想代码具备多态性质*时,我们没办法保留对象本身的类型，而需要用一种通用的类型去使用他们，这时就需要擦除对象原有的类型


# 1 void*


C 语言中很多通用算法函数都会使用`void *`作为参数类型,如
```c
void qsort(void *base, size_t num, size_t size, int(*compare)(const void*, const void*));

int int_compare(const void* a, const void *b){
	return *(const int*)a - *(const int *)b;
}

int str_compare(const void* a, const void* b){
	return strcmp((const char*)a, (const char*)b);
}




```


缺点是:
1. 不能保证类型安全。假设我们传递了错误的`compare`，谁能知道这件事？编译器不知道，因为你把类型擦除掉了。你自己也不知道，因为代码就是你写的。测试程序可能知道，也可能不知道，因为这个时候程序的行为是未定义的


# 2 继承

在面向对象语言中，继承是最常见的Type Erasure。

```java
interface Counter{
	public void Increase(int v);	
	public void Decrease(int v);
	public int Value();
};

public class Test{
	public static void down(Counter c){
		int count = 0;
		int oldValue = c.Value();
		while(c.Value() != 0){
			c.Decrease(1);
			count ++;
		}
	}
}
```
在上述代码中，我们只知道`c`的类型，不知道具体的哪个实现类型，这里它的类型就被擦除了

优点是:

1. 比`void*`来说，因为操作对象时，调用的是对象具体的实现API，即只擦除了调用处对象的类型，实际上它并没有丢掉自己的类型，也保证了类型安全性

缺点是:
1. 要求每个具体实现类型继承某个基类或接口
2. 有额外开销，如虚表虚指针等

# 3 Duck Typing

>如果一个东西，走路像鸭子，叫声也像鸭子，那么它就是鸭子。

如果一个`T`，满足我们对`X`的所有要求，那么它就是`X`。这就是`duck typing`，即鸭子类型。


Python中大量应用了`duck typing`
```python
class RedApple:
	def color(self):
		return "red"
	def round_like(self):
		return True;

def map_by_color(items):
	ret = defaultdict(list)
	for item in items:
	ret[item.color()].append(item)
	return ret
```
在 `map_by_color`中，我们对`items`有两项要求：
1. 可迭代
2. 其中每个元素都有`color`方法

C++ 的模板也是一种`duck typing`:

```cpp
template <typename C>  
int CountByColor(const C& container, Color color) {  
    int count = 0;  
    for (const auto& item: container) {  
        if (item.Color() == color) {  
            ++count;  
        }  
    }  
    return count;  
}
```


优点：

1. 完整的类型安全性，没有任何环节丢掉了类型信息
2. 不需要动态绑定，所有环节都是静态的，没有运行时性能损失
坏处:
1. 模板类型会作为模板函数或模板类的原型的一部分，即`vector<int>`和`vector<double>`是两个类型，没办法用一个类型来表示

2. 每次用不同的参数类型来实例化模板时，都会新生成一份代码，导致编译出来的二进制文件臃肿


# 4 C++ 中结合继承与Template 的Type Erasure


在C++中我们可以结合继承与Template，实现出一种Type Erasure，

- *既有duck typing的优点*，

- 又可以将不同类型用同一种类型表示

```cpp
class CounterBase{
public:
	virtual ~CounterBase{}
	virtual void Increase(int v) = 0;
	virtual void Decrease(int v) = 0;
	virtual void Count() const = 0;
};

template <typename T>
class CounterImpl:public CounterBase{
public:
	explicit CounterImpl(T t):mImpl(std::move(t)){}
	void Increase(int v) override{
		mImpl.Increase(v);
	}
	void Decrease(int v) override{
		mImpl.Decrease(v);
	}
	 int Count() const override{
		return mImpl.Count();
	 }
private:
	T mImpl;
};

class Counter{
	tempalte<typename T>
	Counter(T t):mptr(new CounterImpl(std::forward<T>(t))){}

	void Increase(int v){
		mptr->Increase(int v);
	}

	void Decrease(int v){
		m->ptr->Decrease(v);
	}

	int Count() const {
		return mptr->Count();
	}
private:
	std::unique_ptr<CounterBase> mPtr;
}
```


k

