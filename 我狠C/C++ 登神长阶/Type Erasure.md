`Type Erasure` 即类型擦除，why we need it ? 当我们*想代码具备多态性质*时,我们没办法保留对象本身的类型，而需要用一种通用的类型去使用他们，这时就需要擦除对象原有的类型


# 1 void*


C 语言中很多通用算法函数都会使用`void *`作为参数类型,如
```c
void qsort(void *base, size_t num, size_t size, int(*compare)(const void*, const void*));
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