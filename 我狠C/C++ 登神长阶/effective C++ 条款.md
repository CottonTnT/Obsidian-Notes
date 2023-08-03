
# 用 complier 取代 preprocessor


- 即用 consts, enums 和 [[关键字详解#inline | `inline`]]. 取代  [[关键字详解#define | `#define `]] 



## 用 const 替换 defines 注意情况


- 常量字符串
```c
#define str "Scott Meyers"
const char * const str = "Scott Meyers"; //放在头文件中时，肯定不希望指针本身、指针指向的内容改变
const std::string str = ("Scott Meyers");
```
  


- 涉及到 class-specific constants



## 用 enum 替代




# 只要可能就用const


## 顶层 const 与底层 const


*顶层 const* ：表示指针本身是一个常量, 放在\*右边

*底层 const*：表示指针所指的对象是一个常量\*左边


```c
const int a = 10;//顶层
const int * const p = new int(10); //底层， 顶层
const int & ra = 10; //底层，引用机制类似指针, 一个引用变量相当于一个指针常量
```



### 一个规则

- **当执行对象拷贝操作是，常量的顶层 const 不受什么影响，而底层的 const 必须一致
 



## iterator 与 const


STL iterators（迭代器）以 pointers（指针）为原型，所以一个 iterator 在行为上非常类似于一个 T* pointer（指针）


```cpp
std::vector<int> vec;
...
const std::vector<int>::iterator iter =     // iter acts like a T* const
  vec.begin();
*iter = 10;                                 // OK, changes what iter points to
++iter;                                     // error! iter is const

std::vector<int>::const_iterator cIter =    // cIter acts like a const T*
  vec.begin();
*cIter = 10;                                // error! *cIter is const
++cIter;                                    // fine, changes cIter
```


## const in function declarations


在一个 function declaration（函数声明）中，const 既可以用在函数的 return value（返回值）上，也可以用在个别的 parameters（参数）上，对于 member functions（成员函数），还可以用于整个函数。


### 利用 const 有效避错


```cpp
class Rational{ ... };
const Rational operator*{const Rational& lhs, const Rational& rhs};

Rational a, b, c;
//if no const before line 2 
(a * b) = c //right, invoke operator= on the result of a*b
//how about next?
if((a*b) = c) //missing a =,but stll work,horrible! meant to do a comparison!
```


### const member functions


- member functions（成员函数）被声明为 const 的目的是标明这个 member functions（成员函数）可能会被 const objects（对象）调用, 且该函数。


```c
//1.member functions 在只有constness不同时是可以重载的，如下
class TextBlock{
public:
	const char& operator[](std::size_t position) const{
		return text[position];}

	char& operator[](std::size_t position){
		return text[position]};

private:
	std::string text;
}
```


```c
TextBlock tb("Hello");
std::cout << tb[0];                    // calls non-const TextBlock::operator[]
const TextBlock ctb("World");
std::cout << ctb[0];                   // calls const TextBlock::operator[]
ctb[0] = 'x';                          // error! — writing a
                                       // const TextBlock
//再请注意 non-const 版本的 operator[] 的 return type（返回类型）是 reference to a char（一个 char 的引用）而不是一个 char 本身。如果 operator[] 只是返回一个简单的 char，下面的语句将无法编译：
tb[0] = 'x';
```


-  



# 对象使用前应该被初始化

 - **确保 all constructors 都初始化了 object 中的每一样东西**
 
```c
int x;//某些情况下，x会被初始为0,某些情况不会
---
class Point{
int x, y;
};
Point p;//p的data member有时会初始化为0,有时不会

//处理这种事情的表面不确定状态的最好方法就是总是在使用之前初始化你的对象。
```



## 成员初始化列表优于体内赋值
```c
class PhoneNumber { ... };

class ABEntry {                         // ABEntry = "Address Book Entry"

public:

  ABEntry(const std::string& name, const std::string& address,
          const std::list<PhoneNumber>& phones);

private:

  std::string theName;

  std::string theAddress;

  std::list<PhoneNumber> thePhones;

  int num TimesConsulted;

};

// asignment-based版本 //ABEntry::ABEntry(const std::string& name, const std::string& address,const std::list<PhoneNumber>& phones){

//  theName = name;                       // these are all assignments,
// theAddress = address;                 // not initializations
//  thePhones = phones;
//  numTimesConsulted = 0;
//}

//member initializaion list 版 
ABEntry::ABEntry(const std::string& name, const std::string& address,
                 const std::list<PhoneNumber>& phones)

: theName(name),
  theAddress(address),                  // these are now all initializations
  thePhones(phones),
  numTimesConsulted(0)    //内置类型都差不多

{}                                      // the ctor body is now empty

//assignment-based 的版本会首先调用 default constructors初始化 theName，theAddress 和 thePhones，然而很快又在 default-constructed的值之上赋予新值。那些 default constructions（缺省构造函数）所做的工作被浪费了。而 member initialization list（成员初始化列表）的方法避免了这个问题，因为 initialization list（初始化列表）中的 arguments（参数）就可以作为各种 data members（数据成员）的 constructor（构造函数）所使用的 arguments（参数）。在这种情况下，theName 从 name 中 copy-constructed（拷贝构造），theAddress 从 address 中 copy-constructed（拷贝构造），thePhones 从 phones 中 copy-constructed（拷贝构造）。对于大多数类型来说，只调用一次 copy constructor（拷贝构造函数）的效率比先调用一次 default constructor（缺省构造函数）再调用一次 copy assignment operator（拷贝赋值运算符）的效率要高（有时会高很多）。


```



- 有时，即使是 built-in types，initialization list 也必须使用。比如，const 或 references data members 是必须 be initialized（被初始化）的，它们不能 be assigned
- 初始化列表的顺序与类数据成员定义的先后有关，与初始化列表里的前后无关


## 定义在不同转换单元内的非局部静态对象的初始化的相对顺序是没有定义的 


一个 translation unit 是可以形成一个单独的 object file 的 source code（源代码）。基本上是一个单独的 source file，再加上它全部的 include 文件。


```cpp
class FileSystem{
public;
	std::size_t numDisks() const;

};

---
extern FileSystem tfs;

class Directory{
public:
	Directory(params);
};

Directory::Directory(params){
	std::size_t disks = tfs.numDisks(); //use the tfs
}

Directory tempDir(params); //directory for temporary files

//现在 initialization order（初始化顺序）的重要性变得明显了：除非 tfs 在 tempDir 之前初始化，否则，tempDir 的 constructor（构造函数）就会在 tfs 被初始化之前试图使用它。但是，tfs 和 tempDir 是被不同的人于不同的时间在不同的 source files（源文件）中创建的——它们是定义在不同 translation units（转换单元）中的 non-local static objects（非局部静态对象）。


```
```c
//改良版
class FileSystem { ... };           // as before

FileSystem& tfs()                   // this replaces the tfs object; it could be
{                                   // static in the FileSystem class
  static FileSystem fs;             // define and initialize a local static object
  return fs;                        // return a reference to it
}

--- 

class Directory { ... };            // as before

Directory::Directory( params )      // as before, except references to tfs are
{                                   // now to tfs()
  ...
  std::size_t disks = tfs().numDisks();
  ...
}

Directory& tempDir()                // this replaces the tempDir object; it
{                                   // could be static in the Directory class
  static Directory td;              // define/initialize local static object
  return td;                        // return reference to it
}
```



# 了解 c ++默认生成的函数 


```c
class Bar{
public:
	explicit Bar(std::string &name, int intValue = 10)
		:name_(name), intValue_(intValue){}

public:
	std::string &name_;
	const int intValue_;
	//std::mutex mutex; 17,18行失效，因为该变量不可拷贝与移动
}


int main(){

	std::string name1 = "bar1";
	std::string name2 = "bar2";
	Bar bar1(name1), bar2(name2);
	Bar bar3(bar1);//使用拷贝构造函数，编译器帮我们生成
	Bar bar4(std::move(bar1));//使用移动构造函数，编译器帮我们生产
	bar3 = bar2;//wrong,因为我们定义了引用和常量数据成员
	bar4 = std::move(bar2);//wrong, 同上
	return 0;
}
```


# 对于不想要的特种成员函数，明确禁止编译器自动生成

```c
//how to solve？
HomeForSale h1;
HomeForSale h2;
HomeForSale h3(h1); // attempt to copy h1 -- should not compile!
h1 = h2; //attempt to copy h2 -- should not compile!

//modern c++
class HomeForSale{
public:

HomeForSale(const HomeForSale&) = delete; //declaration only
HomeForSale& operator=(const HomeForSale&) = delete; //declaration only




}

//c ++ 98
class HomeForSale{
public:

private:

HomeForSale(const HomeForSale&); //declaration only
HomeForSale& operator=(const HomeForSale&); //declaration only
}
//编译阶段可过，链接阶段报错

//如何做到编译阶段都不行ne？

class Uncopyable {
protected:                                   // allow construction
  Uncopyable() {}                            // and destruction of
  ~Uncopyable() {}                           // derived objects...

private:
  Uncopyable(const Uncopyable&);             // ...but prevent copying
  Uncopyable& operator=(const Uncopyable&);
};

class HomeForSale: private Uncopyable {      // class no longer
  ...                                        // declares copy ctor or
};                                           // copy assign. operator

```


# 7. 在 polymorphic base classes 中将 destructor 声明为 virtual


- destructors（析构函数）的工作方式是：most derived class（层次最低的派生类）的 destructor（析构函数）最先被调用，然后调用每一个 base class（基类）的 destructors（析构函数）。所以你必须为纯虚构函数提供一个定义，不然链接程序会抗议滴！

-  C++ 规定：当一个 derived class object（派生类对象）通过使用一个 pointer to a base class with a non-virtual destructor（指向带有非虚拟析构函数的基类的指针）被删除，则结果是未定义的。运行时比较典型的后果是 derived part of the object（这个对象的派生部分）不会被析构。
```c
class TimeKeeper {
public:
  TimeKeeper();
  ~TimeKeeper();//no
	//cmp 
  virtual ~TimeKeeper();//yes
  ...
};

class AtomicClock: public TimeKeeper { ... };

class WaterClock: public TimeKeeper { ... };

class WristWatch: public TimeKeeper { ... };

TimeKeeper *ptk = getTimeKeeper();

delete ptk;    
```


- 当一个 class（类）不打算作为 base class（基类）时，将 destructor（析构函数）虚拟通常是个坏主意。
- 
```text
virtual functions（虚拟函数）的实现要求 object（对象）携带额外的信息，这些信息用于在运行时确定该 object（对象）应该调用哪一个 virtual functions（虚拟函数）。典型情况下，这一信息具有一种被称为 vptr ("virtual table pointer")（虚拟函数表指针）的指针的形式。vptr 指向一个被称为 vtbl ("virtual table")（虚拟函数表）的 array of function pointers（函数指针数组），每一个带有 virtual functions（虚拟函数）的 class（类）都有一个相关联的 vtbl。当在一个 object（对象）上调用 virtual functions（虚拟函数）时，实际的被调用函数通过下面的步骤确定：找到 object（对象）的 vptr 指向的 vtbl，然后在 vtbl 中寻找合适的 function pointer（函数指针）。
```


- **declare a virtual destructor in a class if and only if that class contains at least one virtual function（当且仅当一个类中包含至少一个虚拟函数时，则在类中声明一个虚拟析构函数）**。


- **不是设计用来作为 base classes（基类）或不是设计用于 polymorphically（多态）的 classes（类）就不应该声明 virtual destructor（虚拟析构函数）**




# 8.防止因为 exceptions 而离开 destuctors


```cpp
class DBConnection {
public:
  ...

  static DBConnection create();        // function to return
                                       // DBConnection objects; params
                                       // omitted for simplicity

  void close();                        // close connection; throw an
};       
```

```cpp
class DBConn {                          // class to manage DBConnection
public:                                 // objects
  ...
  ~DBConn()                             // make sure database connections
  {                                     // are always closed
   db.close();
  }
private:
  DBConnection db;
};
```
```cpp
{                                       // open a block
   DBConn dbc(DBConnection::create());  // create DBConnection object
                                        // and turn it over to a DBConn
                                        // object to manage
 ...                                    // use the DBConnection object
                                        // via the DBConn interface
}                                       // at end of block, the DBConn
                                        // object is destroyed, thus
                                        // automatically calling close on
                                        // the DBConnection object
```
只要能成功地调用 close 就可以了，但是如果这个调用导致一个 exception（异常），DBConn 的 destructor（析构函数）将传播那个 exception（异常），也就是说，它将离开 destructor（析构函数）。


*有如下解决办法*

- Terminate the program if close tHRows
```cpp

DBConn::~DBConn()
{
 try { db.close(); }
 catch (...) {
   make log entry that the call to close failed;
   std::abort();
 }
}

```
- Swallow the exception arising from the call to close
```cpp
DBConn::~DBConn()
{
 try { db.close(); }
 catch (...) {
      make log entry that the call to close failed;
 }
}
```
- better interface design
```c

class DBConn {
public:
  ...

  void close()                                     // new function for
  {                                                // client use
    db.close();
    closed = true;
  }

  ~DBConn()
  {
   if (!closed) {
   try {                                            // close the connection
     db.close();                                    // if the client didn't
   }
   catch (...) {                                    // if closing fails,
     make log entry that call to close failed;      // note that and
     ...                                            // terminate or swallow
   }
  }

private:
  DBConnection db;
  bool closed;
};

```


Things to Remember

- destructor（析构函数）应该永不引发 exceptions（异常）。如果 destructor（析构函数）调用了可能抛出异常的函数，destructor（析构函数）应该捕捉所有异常，然后抑制它们或者终止程序。
    
- 如果 class（类）客户需要能对一个操作抛出的 exceptions（异常）做出回应，则那个 class（类）应该提供一个常规的函数（也就是说，non-destructor（非析构函数））来完成这个操作。


# 9. 绝不要在 construction或 destruction期间调用 virtual functions


因为 base class constructors（基类构造函数）在 derived class constructors（派生类构造函数）之前执行，当 base class constructors（基类构造函数）运行时，derived class data members（派生类数据成员）还没有被初始化。如果 base class construction（基类构造）期间 virtual functions（虚拟函数）的调用 went down（向下匹配）到 derived classes（派生类），derived classes（派生类）的函数差不多总会涉及到 local data members（局部数据成员），但是那些 data members（数据成员）至此还没有被初始化。这就会为 undefined behavior（未定义行为）和通宵达旦的调试噩梦开了一张通行证。调用涉及到一个 object（对象）还没有被初始化的构件自然是危险的，所以 C++ 告诉你此路不通。直到 derived class constructor（派生类构造函数）的执行开始之前，一个 object（对象）不会成为一个 derived class object（派生类对象）。


```cpp
class Base{
public:
	Base(){
		sayHello();
	}
	virtual void sayHello(){
		std::cout<<"Hello, Base!" << std::endl;
	}
	virtual void sayBye(){
		std::cout <<"Bye, Base!" << std::endl;
	}

	virtual ~Base(){
		sayBye();
	}
}

class Derived:public Base{
public:
	Derived(){}

	void sayHello()override{
		std::cout<<"Hello, Derived!" << std::endl;
	}	
	void sayBye()override{
		std::cout<<"Bye, Derived!" << std::endl;
	}
	
}
```
- 运行结果
![[Pasted image 20230720141823.png]]


# 10. 让 assignment operators（赋值运算符）返回一个 reference to \*this



```cpp
class Widget {
public:

//right
Widget& operator=(const Widget& rhs)   // return type is a reference to
{                                      // the current class
  ...
  return *this;                        // return the left-hand object
  }
  ...
};
//wrong
void operator=(const Widget& rhs){
  ...
}
---

Widget w1(1), w2(2), w3(3);
w2 = w1;
w3 = w2 = w1; //采用wrong版就wrong了
```

这个惯例适用于所有的 assignment operators（赋值运算符），而不仅仅是上面那样的标准形式。因此：

```
class Widget {
public:
  ...
  Widget& operator+=(const Widget& rhs   // the convention applies to
  {                                      // +=, -=, *=, etc.
   ...
   return *this;
  }
   Widget& operator=(int rhs)            // it applies even if the
   {                                     // operator's parameter type
      ...                                // is unconventional
      return *this;
   }
   ...
};
```


# 在 operator=中处理 assignment to self

```
class Widget { ... };

Widget w;
...

w = w;  
```

```
class Bitmap { ... };

class Widget {
  ...

private:
  Bitmap *pb;                                     // ptr to a heap-allocated object
};
```

```
Widget&
Widget::operator=(const Widget& rhs)              // unsafe impl. of operator=
{
  delete pb;                                      // stop using current bitmap
  pb = new Bitmap(*rhs.pb);                       // start using a copy of rhs's bitmap

  return *this;                                   // see Item 10
}
```
```
Widget& Widget::operator=(const Widget& rhs)
{
  if (this == &rhs) return *this;   // identity test: if a self-assignment,
                                    // do nothing
  delete pb;
  pb = new Bitmap(*rhs.pb);

  return *this;
}
```
```
Widget& Widget::operator=(const Widget& rhs)
{
  Bitmap *pOrig = pb;               // remember original pb
  pb = new Bitmap(*rhs.pb);         // make pb point to a copy of *pb
  delete pOrig;                     // delete the original pb

  return *this;
}
```
```
class Widget {
  ...
  void swap(Widget& rhs);       // exchange *this's and rhs's data;
  ...                           // see Item 29 for details
};

Widget& Widget::operator=(const Widget& rhs)
{
  Widget temp(rhs);             // make a copy of rhs's data

  swap(temp);                   // swap *this's data with the copy's
  return *this;
}
```
```
Widget& Widget::operator=(Widget rhs)   // rhs is a copy of the object
{                                       // passed in — note pass by val

  swap(rhs);                            // swap *this's data with
                                        // the copy's
  return *this;
}
```


# 12. 拷贝一个对象的所有组成部分




# 13. 使用对象管理资源 (已过时)

- RAII：因为获取一个资源并在同一个语句中初始化资源管理对象是如此常见，所以使用对象管理资源的观念也常常被称为 Resource Acquisition Is Initialization (RAII)

# 14.谨慎考虑资源管理类的拷贝行为



![[Pasted image 20230720150111.png]]


# 15. 在资源管理类中准备访问裸资源




# 16. 使用相同形式的 new 与 delete

![[Pasted image 20230720155333.png]]

# 17. 以独立语句将 new 出来的对象存入智能指针



![[Pasted image 20230720164329.png]]

# 18. 让接口易于正确使用，难于误用


![[Pasted image 20230720165053.png]]


![[Pasted image 20230720165549.png]]


# 19. 视类设计为类型设计

- 你的新类型的对象应该如何创建和销毁？如何做这些将影响到你的类的构造函数和析构函数，以及内存分配和回收的函数（operator new，operator new[]，operator delete，和 operator delete[] ——参见 Chapter 8）的设计，除非你不写它们。
    
- 对象的初始化和对象的赋值应该有什么不同？这个问题的答案决定了你的构造函数和你的赋值运算符的行为和它们之间的不同。这对于不混淆初始化和赋值是很重要的，因为它们相当于不同的函数调用（参见 Item 4）。
    
- 以值传递（passed by value）对于你的新类型的对象意味着什么？记住，拷贝构造函数定义了一个新类型的传值（pass-by-value）如何实现。
    
- 你的新类型的合法值的限定条件是什么？通常，对于一个类的数据成员来说，仅有某些值的组合是合法的。那些组合决定了你的类必须维持的不变量。这些不变量决定了你必须在成员函数内部进行错误检查，特别是你的构造函数，赋值运算符，以及 "setter" 函数。它可能也会影响你的函数抛出的异常，以及你的函数的异常规范（exception specification）（你用到它的可能性很小）。
    
- 你的新类型是否适合放进一个继承图表中？如果你从已经存在的类继承，你将被那些类的设计所约束，特别是它们的函数是 virtual 还是 non-virtual（参见 Item 34 和 36）。如果你希望允许其他类继承你的类，将影响到你是否将函数声明为 virtual，特别是你的析构函数（参见 Item 7）。
    
- 你的新类型允许哪种类型转换？你的类型身处其它类型的海洋中，所以是否要在你的类型和其它类型之间有一些转换？如果你希望允许 T1 类型的对象隐式转型为 T2 类型的对象，你就要么在 T1 类中写一个类型转换函数（例如，operator T2），要么在 T2 类中写一个非显式的构造函数，而且它们都要能够以单一参数调用。如果你希望仅仅允许显示转换，你就要写执行这个转换的函数，而且你还需要避免使它们的类型转换运算符或非显式构造函数能够以一个参数调用。（作为一个既允许隐式转换又允许显式转换的例子，参见 Item 15。）
    
- 对于新类型哪些运算符和函数有意义？这个问题的答案决定你应该为你的类声明哪些函数。其中一些是成员函数，另一些不是（参见 Item 23、24 和 46）。
    
- 哪些标准函数不应该被接受？你需要将那些都声明为 private（参见 Item 6）。
    
- 你的新类型中哪些成员可以被访问？这个问题的可以帮助你决定哪些成员是 public，哪些是 protected，以及哪些是 private。它也可以帮助你决定哪些类和／或函数应该是友元，以及一个类嵌套在另一个类内部是否有意义。
    
- 什么是你的新类型的 "undeclared interface"？它对于性能考虑，异常安全（exception safety）（参见 Item 29），以及资源使用（例如，锁和动态内存）提供哪种保证？你在这些领域提供的保证将强制影响你的类的实现。
    
- 你的新类型有多大程度的通用性？也许你并非真的要定义一个新的类型。也许你要定义一个整个的类型家族。如果是这样，你不需要定义一个新的类，而是需要定义一个新的类模板。
    
- 一个新的类型真的是你所需要的吗？是否你可以仅仅定义一个新的继承类，以便让你可以为一个已存在的类增加一些功能，也许通过简单地定义一个或更多非成员函数或模板能更好地达成你的目标。



# 20.const 引用传递优于按值传递


- 减少不必要的拷贝构造，对于非内置数据类型而言

- 表现多态
![[Pasted image 20230720223644.png]]
Things to Remember

- 用传引用给 const 取代传值。典型情况下它更高效而且可以避免切断问题。
    
- 这条规则并不适用于内建类型及 STL 中的迭代器和函数对象类型。对于它们，传值通常更合适。


# 21. 不要错误的返回对象的引用（需再看）



# 22. 将数据成员声明为 private
![[Pasted image 20230721101715.png]]

![[Pasted image 20230721101748.png]]


# 23. 非成员非友元函数替代成员函数

![[Pasted image 20230721103118.png]]
![[Pasted image 20230721103139.png]]

# 24.当类型转换应该用于所有参数时，声明为非成员函数


![[Pasted image 20230721144212.png]]
![[Pasted image 20230721144323.png]]   


# 25.自定义 swap 函数

![[Pasted image 20230721144655.png]]

# 26. 尽可能延后变量的定义


# 27. 正确使用类型转换



# 28. 避免返回 handles 指向对象的内部成分



![[Pasted image 20230721170738.png]] 
![[Pasted image 20230721170949.png]]


# 努力写出异常安全的代码

![[Pasted image 20230721171137.png]]
![[Pasted image 20230721171157.png]]


# 30. 理解 inline

1. 让函数符号表成为 weak 型





# 34. 区别接口继承与实现继承




# 35. 考虑虚函数的替代方案




# 36. 不要重定义 non-virtual 函数



# 37. 不要重新定义重写函数的默认参数
![[Pasted image 20230723104026.png]]


# 38 通过组合建模出 has-a 关系或实现关系




# 39. 空基类优化 (Empty base optimization)


- 如果首个非静态数据成员的类型与一个空基类的类型相同或由该空基类派生，那么禁用空基类优化，因为要求来个同类型基类子对象在最终派生类型的对象表示中必须拥有不同地址


![[Pasted image 20230723105928.png]]

# 40. 谨慎使用 private 继承



# 40. 谨慎使用多重继承
![[Pasted image 20230723123305.png]]

# 41. 了解隐式接口与编译器多态



# 42. 理解 typename 的双重含义



# 44. 将与参数无关的代码抽离templates
o