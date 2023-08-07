# 六大部件之间的联系
![[Pasted image 20230409115246.png]]

# Containters
## 容器结构及分类

![[Pasted image 20230409115903.png]]   
![[Pasted image 20230409115948.png]]

![[Pasted image 20230805104709.png]]
## 深度探索 list


```cpp
template<class T>
struct _list_node{
	_list_node prev;
	_list_node next;
	T data;
};

template<class T, class Alloc = alloc>
class list{
protected:
	typedef _list_node<T> list_node;

public:
	typedef list_node* link_type;
	typedef _list_iterator<T> iterator;
protected:
	link_type node;
... 
};


template<class T>
struct _list_iterator{
	typedef _list_iterator<T> self;
	typedef T value_type;
	typedef T* pointer;
	typedef T& reference;
	typedef _list_node<T>* link_type;
	typedef ptrdiff_t difference_type;

	link_type node;
	reference operator*() const{ return (*node).data;}
	pointer operator->() const{return &(operator*())};
	self& operator++(){//前置++
		node = (link_type)((*node).next);
		return *this;
	}
	self operator++(int){//后置++,通过参数列表区分
		self tmp = *this; ++*this; return tmp;
	}
	...
};
```


![[Pasted image 20230805114405.png]]
![[Pasted image 20230805120711.png]]


## vector

- vector 的内存增长，每次要重新申请一块更大的连在一起的内存，还要对以前的值进行复制
![[Pasted image 20230805144613.png]]
```cpp
template <class T, class Alloc = alloc>
class vector{
public:
		typedef T value_type;
		typedef value_type* iterator;
		typedef value_type& reference;
		typedef size_t size_type;
protected:
		iterator start;
		iterator finish;
		iterator end_of_storage;
public:
		iterator begin(){return start};
		iterator end(){return finish}；
		size_type size() const{
			return size_type(end_of_storage-begin());
		}
		bool empty() const{return begin() == end();}
		reference operator[](size_type n){
			return *(begin() + n);
		}
		reference front(){
			return *begin();
		}
		reference back(){
			return *(end() - 1)；
		}
}
```


## array

- 模拟普通数组

 ![[Pasted image 20230805153525.png]]


## deque


- deque 是一种伪连续的容器, 具体存储结构如下
 ![[Pasted image 20230805154218.png]]
```cpp


template <class T, class Allo = alloc, size_t BufSiz=0>//BufSize是指每个buffer容纳的元素个数
class deque{
public:
		typedef T value_type;
		typedef __deque_iterator<T, T&, T*, BufSize> Iterator;
protected:
		typedef pointer* map_pointer; //T**
protected:
		iterator start;
		iterator finish;
		map_pointer map;
		size_type map_size;
public:
		iterator begin(){return start;}
		iterator end(){return finish;}
		size_type size() const {return finish - start;}
...
}
```
# Iterator 
## 设计原则


![[Pasted image 20230805122210.png]]

 - Iterator 必须提供 5 种 associated types

 
```cpp
template<typename I>
inline void algorithm(I first, I last){//算法提问
...
	I::iterator_category;
	I::pointer;
	I::reference;
	I::value_type;
	I::difference_type
...
}
```

##  Iterator Traits

> 用以分离 `class iterator` 和 `non-class iterator` <sup>1</sup>

![[Pasted image 20230805135714.png]]



-  利用 partial specialization 来分辨
 

```cpp
template <class I>
struct iterator_traits{
		typedef typename I::value_type value_type;
};

template <class I>
struct iterator_traits<T*>{
	typedef T value_type
}
```

--- 
<sup>1</sup>  Non class (template)  iterators 即 native pointer, 无法定义 associated type. 但其 associated type 




# Algorithm


 - Algorithm 看不见 containers, 所需要的一切信息来自 Iterators, 



# tuple


- G 4.8 源码节录并简化

```cpp
template<typename... Values> class tuple;
template<>class tuple<>{};
---

template<typename Head, typename... Tail>
class tuple<Head, Tail...>: private tuple<Tail...>{

	typedef tuple<Tail...> inherited;
public:
	tuple(){}
	tuple(Head v, Tail... vtail):m_head(v), inherited(vtail...){}
	typename Head::type head(){ return m_head;}
	inherited& tail(){ return *this;}
protected:
	Head m_head;
}


```




# traits


## type traits

  




 