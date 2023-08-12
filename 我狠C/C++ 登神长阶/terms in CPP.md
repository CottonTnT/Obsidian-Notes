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

