# POD(Plain Old Data)

> 用于说明类/结构体的属性，指其没有使用面向对象的特性来设计的类/结构体，**Plain表明它是一个普通的类型，没有虚函数虚继承等特性；Old表明它与C兼容**


 POD类型在C++ 有两个独立的特性

   * 支持静态初始化
   - 拥有和C语言一样的内存布局
# 左值与右值


*不要试图自己去定义它，only have to know what is lvalue or rvalue*


# 声明与定义(declare vs define)

## declare
>When you declare a variable, a function, or even a class all you are doing is saying: there is something with this name, and it has this type. The compiler can then handle most (but not all) uses of that name without needing the full definition of that name. Declaring a value--without defining it--allows you to write code that the compiler can understand without having to put all of the details. 


- This is particularly useful if you are working with multiple source files, and you need to use a function in multiple files. You don't want to put the body of the function in multiple files, but you do need to provide a declaration for it.
## define
>Defining something means providing all of the necessary information to create that thing in its entirety. 

- Defining a function means providing a function body; 
- defining a class means giving all of the methods of the class and the fields. 
- Once something is defined, that also counts as declaring it; so you can often both declare and define a function, class or variable at the same time. But you don't have to

# 内部链接(static linkage) 与 外部链接(extern)


## static linkage
>如果一个名称对于它的编译单元来说是局部的, 并且在连接的时候不可能与其它编译单元中的同样的名称相冲突,则这个名称具有内部连接.即具有内部连接的名称不会被带到目标文件中.


## 外部链接
>外部连接 在一个多文件程序中,如果一个名称在连接时可以和其他编译单元交互,那么这个名称就具有外部连接. 即具有外部连接的名称会引入到目标文件中,由连接程序进行处理.这种符号在整个程序中必须是惟一的.



## material


[C++的内部链接和外部链接总结]( https://blog.csdn.net/xiexievv/article/details/8491494#:~:text=%E5%86%85%E9%83%A8%E8%BF%9E%E6%8E%A5%20%E5%A6%82%E6%9E%9C%E4%B8%80%E4%B8%AA%E5%90%8D%E7%A7%B0%E5%AF%B9%E4%BA%8E%E5%AE%83%E7%9A%84%E7%BC%96%E8%AF%91%E5%8D%95%E5%85%83%E6%9D%A5%E8%AF%B4%E6%98%AF%E5%B1%80%E9%83%A8%E7%9A%84%2C,%E5%B9%B6%E4%B8%94%E5%9C%A8%E8%BF%9E%E6%8E%A5%E7%9A%84%E6%97%B6%E5%80%99%E4%B8%8D%E5%8F%AF%E8%83%BD%E4%B8%8E%E5%85%B6%E5%AE%83%E7%BC%96%E8%AF%91%E5%8D%95%E5%85%83%E4%B8%AD%E7%9A%84%E5%90%8C%E6%A0%B7%E7%9A%84%E5%90%8D%E7%A7%B0%E7%9B%B8%E5%86%B2%E7%AA%81%2C%E5%88%99%E8%BF%99%E4%B8%AA%E5%90%8D%E7%A7%B0%E5%85%B7%E6%9C%89%E5%86%85%E9%83%A8%E8%BF%9E%E6%8E%A5.%E5%8D%B3%20%E5%85%B7%E6%9C%89%E5%86%85%E9%83%A8%E8%BF%9E%E6%8E%A5%E7%9A%84%E5%90%8D%E7%A7%B0%E4%B8%8D%E4%BC%9A%E8%A2%AB%E5%B8%A6%E5%88%B0%E7%9B%AE%E6%A0%87%E6%96%87%E4%BB%B6%E4%B8%AD.)

[what does const static mean in C and C ++](https://stackoverflow.com/questions/177437/what-does-const-static-mean-in-c-and-c/177781#177781?newreg=5e127facde354737af6946e99af9ae62)


# 数组名


## material

[is an array name a pointer](https://stackoverflow.com/questions/1641957/is-an-array-name-a-pointer)
