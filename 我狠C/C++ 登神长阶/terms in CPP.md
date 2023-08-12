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





