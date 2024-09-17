# 1 线程安全的对象声明周期管理


## 1.1 对象创建的线程安全:简单

对象构造要做到线程安全，唯一的要求是在构造期间不要泄露this 指针，即 

- 不要在构造函数中注册任何回调； 
- 也不要在构造函数中把this传给跨线程的对象； 
- 即便在构造函数的最后一行也不行。

原因如下:

1. 在构造函数执行期间对象还没有完成初始化，如果`this`被泄露（escape）给了其他对象（其自身创建的子对象除 外），那么别的线程有可能访问这个半成品对象，这会造成难以预料的后果


2. 即使构造函数的最后一行也不要泄露`this`，因为`Foo`有可能是个*基类*，*基类先于派生类构造*，执行完`Foo::Foo()`的最后一行代码还会继续 执行派生类的构造函数，这时`most-derived class`的对象还处于构造中，仍然不安全。 

```c++
.// don`t do this
class Foo:public Observer {
public:
	Fool(Observable* s){
	    s->register_(this); //错误非线程安全
	}
	vitual void update();
}
```


应当使用 ==二段式构造==, 即`构造函数+initialize()`, 这虽然不符合C++教条，但是多线程下别无选择。另外，既然允 许二段式构造，那么构造函数不必主动抛异常，调用方靠`initialize()`的返 回值来判断对象是否构造成功，这能简化错误处理。

```c++
class Foo:public Observer {
public:
	Foo() ;
	//定义另外一个函数，处理回调函数的注册工作
	void observe(Observable* s){
		s->register_(this);
	}
	vitual void update();
}
```




## 1.2 当析构函数遇到多线程:困难


> [!NOTE] 析构安全三问
> 1. 在即将析构一个对象时，从何而知此刻是否有别的线程正在执行 该对象的成员函数？ 
> 2. 如何保证在执行成员函数期间，对象不会在另一个线程被析构？
> 3. 在调用某个对象的成员函数之前，如何得知这个对象还活着？它 的析构函数会不会碰巧执行到一半



对象析构，这在单线程里不构成问题，最多需要注意 避免空悬指针 和 野指针 。而在多线程程序中，存在了太多的竞态条件。对一般成员 函数而言，做到线程安全的办法是让它们顺次执行，而不要并发执行 （关键是不要同时读写共享状态），也就是让每个成员函数的临界区不重叠。这是显而易见的，不过有一个*隐含条件*或许不是每个人都能立刻 想到：*成员函数用来保护临界区的互斥器本身必须是有效的*。而析构函数破坏了这一假设，它会把mutex成员变量销毁掉。


考虑如下代码:

![[Pasted image 20240909003056.png]]

尽管线程`A`在销毁对象之后把指针置为了`NULL`，尽管线程B在调用`x`的 成员函数之前检查了指针`x`的值，但还是无法避免一种`race condition`：

1. 线程A执行到了析构函数的(1)处，已经持有了互斥锁，即将继续往下执行。 

2. 线程B通过了if (x)检测，阻塞在(2)处。 接下来会发生什么，只有天晓得。因为析构函数会把mutex_销毁， 那么(2)处有可能永远阻塞下去，有可能进入“临界区”，然后`core dump`，或者发生其他更糟糕的情况。 这个例子至少说明delete对象之后把指针置为NULL根本没用


==故作为数据成员的`mutex`不能保护析构==


**作为`class`数据成员的`MutexLock`只能用于同步本 `class`的其他数据成员的读和写**，它不能保护安全地析构。因为 `MutexLock`成员的生命期最多与对象一样长，而析构动作可说是发生在 对象身故之后（或者身亡之时）。另外，对于基类对象，那么调用到基 类析构函数的时候，派生类对象的那部分已经析构了，那么基类对象*拥有的`Mutexlock`不能保护整个析构过程*。*

**再说，析构过程本来也不应该需要保护，因为只有别的线程都访问不到这个对象时，析构才是安全的*。**



 另外如果要同时读写一个`class`的两个对象，有潜在的死锁可能。比方说有`swap()`这个函数：

```c++
void swap(Counter& a, Counter& b){
	MutexLockGuard aLock(a.mutex_);//potential deal lock
	MutexLockGuard aLock(b.mutex_);
	...//swap
}
```
如果线程A执行`swap(a, b)`;而同时线程B执行`swap(b, a)`;，就有可能 死锁。`operator=()`也是类似的道理.

*一个函数如果要锁住相同类型的多个对象，为了保证始终按相同的顺序加锁，我们可以比较mutex对象的地址，始终先加锁地址较小的 mutex*。


## 1.3 实现线程安全的observer


### 1.3.1 原始指针的问题

*无法通过指针(引用)判断一个动态创建的对象是否还活着*。

1. 指针指向的一块内存如果这块内存上的对象已经销毁，那么就根本不能访问（就像free(3)之后的地址不能访问一样），既然不能访问又如何知道对象的状态呢？
2. 换句话说，判断一个指针是不是合法指针没有高效的办法，这是C/C++指针问题的根源  。（万一原址又创建了一个新的对象呢？再万一这个新的对象的类型 异于老的对象呢？）

> [!note] 在Java中，一个reference只要不为null，它一定指向有效的对象


观察者模式中对象关系为 association（关联／联系），是一种很宽泛的关系，*它表示一个对象a用到了另一个对象b，调用了后者的成员函数*。从代码形式上看，a 持有b的指针（或引用），但是b的生命期不由a单独控制。


考虑如下代码:

```cpp
class Observer{
public:
	virtual ~Observer();
	virtual void update() = 0;
};

class Observable{
public:
	void register(Observer* x);
	void unregister(Observer* x);
	void notifyObserver(){
		for(Observer* x : observer){
			x->update();
		}
	}
private:
	std::vector<Observer*> observer;
};
```

当`Observable`通知每一个`Observer`时(L17)，它从何得知`Observe`r对象 x还活着？


试试在Observer的析构函数里调用unregister()来解注册？ 恐难奏效.
![[Pasted image 20240917003707.png]]

我们试着让`Observer`的析构函数去调用`unregister(this)`，这里有两个 `race conditions`。其一：L32如何得知`subject_`还活着？其二：就算 subject_指向某个永久存在的对象，那么还是险象环生：
- 1．线程A执行到L32之前，还没有来得及unregister本对象。 
- 2．线程B执行到L3，x正好指向是L32正在析构的对象。


### 1.3.2 使用std::shared_ptr

![[Pasted image 20240917004611.png]]

但还有以下几个疑点。

 1. 侵入性。强制要求Observer必须以shared_ptr来管理。 不是完全线程安全`Observer`的析构函数会调用`subject_->unregister(this)`，万一`subject_`已经不复存在了呢？为了解决它，又要求 `Observable`本身是用`shared_ptr`管理的，并`且subject_`多半是个 `weak_ptr`。 
 2. 锁争用`lock contention`。即`Observable`的三个成员函数都用了互斥器来同步，这会造成`register_()`和`unregister()`等待`notifyObservers()`，而 后者的执行时间是无上限的，因为它同步回调了用户提供的`update()`函 数。我们希望`register_()`和`unregister()`的执行时间不会超过某个固定的上限，以免殃及无辜群众。 
 3. 死锁　万一`L62`的`update()`虚函数中调用了`(un)register`呢？如果 `mutex_`是不可重入的，那么会死锁；如果mutex_是可重入的，程序会面临迭代器失效（core dump是最好的结果），因为`vector observers_`在遍 历期间被意外地修改了。这个问题乍看起来似乎没有解决办法，除非在文档里做要求。（一种办法是：用可重入的mutex_，把容器换为 std::list，并把++it往前挪一行。） 我个人倾向于使用不可重入的mutex，例如Pthreads默认提供的那 个，因为“要求mutex可重入”本身往往意味着设计上出了问题 ）。



## 1.4 shared_ptr的技术与陷阱


 1. 意外延长对象的生命期

 2. 析构动作在创建时被捕获　这是一个非常有用的特性，这意味着： 
   -   虚析构不再是必需的。 
   - shared_ptr可以持有任何对象，而且能安全地释放。 ·
   - shared_ptr对象可以安全地跨越模块边界，比如从DLL里返回，而 不会造成从模块A分配的内存在模块B里被释放这种错误。 ·
   - 二进制兼容性，即便Foo对象的大小变了，那么旧的客户代码仍然 可以使用新的动态库，而无须重新编译。前提是Foo的头文件中不出现 访问对象的成员的inline函数，并且Foo对象的由动态库中的Factory构 造，返回其shared_ptr。
   - 析构动作可以定制。 
1. 析构所在的线程 
  - 对象的析构是同步的，当最后一个指向x的 shared_ptr离开其作用域的时候，x会同时在同一个线程析构。这个线程 不一定是对象诞生的线程。这个特性是把双刃剑：如果对象的析构比较 耗时，那么可能会拖慢关键线程的速度（如果最后一个shared_ptr引发 的析构发生在关键线程）；同时，我们可以用一个单独的线程来专门做 析构，通过一个`BlockingQueue<std::shared_ptr<void>>`把对象的析构都转移 到那个专用线程，从而解放关键线程。

2. 现成的RAII handle


## 1.5 弱回调与线程安全的对象池
![[Pasted image 20240917013740.png]]
![[Pasted image 20240917013800.png]]
本节的StockFactory只有针对单个Stock对象的操作，如果程序需要 遍历整个stocks_，稍不注意就会造成死锁或数据损坏（§2.1），请参考 §2.8的解决办法。



## 1.6 使用信号与槽




# 2 线程同步精要

并发编程有两种基本模型，一种是`message passing`，另一种是 `shared memory`。


线程同步的四项原则，按重要性排列：
- 1．首要原则是尽量最低限度地共享对象，减少需要同步的场合。 一个对象能不暴露给别的线程就不要暴露；如果要暴露，优先考虑 immutable对象；实在不行才暴露可修改的对象，并用同步措施来充分 保护它。
- 2．其次是使用高级的并发编程构件，如`TaskQueue、Producer Consumer Queue、CountDownLatch`等等。
- 3．最后不得已必须使用底层同步原语（primitives）时，只用非递 归的互斥器和条件变量，慎用读写锁，不要用信号量。 
- 4．除了使用`atomic`整数之外，不自己编写`lock-free`代码用“内核级”同步原语，也不要 。不凭空猜测“哪种做法性能会更好”，比如`spin lock` vs. `mutex`。



## 2.1 互斥器

陈硕前辈的主要原则:
1. 用RAII手法封装mutex的创建、销毁、加锁、解锁这四个操作。
2. 只用非递归的mutex（即不可重入的mutex）。
3. 不手工调用lock()和unlock()函数，一切交给栈上的Guard对象的构 造和析构函数负责。Guard对象的生命期正好等于临界区（分析对象在 什么时候析构是C++程序员的基本功）。这样我们保证始终在同一个函 数同一个scope里对某个mutex加锁和解锁。避免在foo()里加锁，然后跑 到bar()里解锁；也避免在不同的语句分支中分别加锁、解锁。这种做法 被称为Scoped Locking 。


次要原则:
1. 不使用跨进程的mutex，进程间通信只用TCP sockets。
2. 加锁、解锁在同一个线程，线程a不能去unlock线程b已经锁住的 mutex（RAII自动保证）
3. 别忘了解锁（RAII自动保证）
4. 不重复解锁（RAII自动保证）
5. 必要的时候可以考虑用`PTHREAD_MUTEX_ERRORCHECK`来排错。




[使用shared_ptr实现copy-on-write_原始指针赋值给shared ptr-CSDN博客](https://blog.csdn.net/qiuguolu1108/article/details/115285574)