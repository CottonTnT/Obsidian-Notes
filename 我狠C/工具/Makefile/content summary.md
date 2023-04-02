## 1. 概念

 makefile 关系了整个工程的编译和链接规则，通过 make 命令进行自动化编译。make 是解释 makefile 中指令的命令工具。

## 2.makefile 的规则

#### makefile 的书写

```makefile粗略工具规则
target ... : prerequisites ... 
	recipe 
	... 
	...
```


|   字段   |  作用    |
|:-----|:-----|
| target     |   可以是一个 object file（目标文件），也可以是一个可执行文件，还可以是一个标签（label）   |
| prerequisites     | 生成 target 依赖的文件     |
| recipe | 该 target 要执行的命令（任意 shell 命令)


> [!NOTE] 直白的说 
> 
prerequisites 中如果有一个以上的文件比 target 文件要新的话，recipe 所定义的命令就会被执行。

*一个实例*
```makefile 实例
edit : main.o kbd.o command.o display.o \  
		insert.o search.o files.o utils.o                                                     
	cc -o edit main.o kbd.o command.o display.o insert.o search.o files.o utils.o 
	 
main.o : main.c defs.h 
		cc -c main.c 
kbd.o : kbd.c defs.h command.h 
		cc -c kbd.c 
command.o : command.c defs.h command.h 
		cc -c command.c 
display.o : display.c defs.h buffer.h 
		cc -c display.c 
insert.o : insert.c defs.h buffer.h 
		cc -c insert.c 
search.o : search.c defs.h buffer.h 
		cc -c search.c 
files.o : files.c defs.h buffer.h command.h 
		cc -c files.c 
utils.o : utils.c defs.h 
		cc -c utils.c 
clean : 
	rm edit main.o kbd.o command.o display.o \ 
	insert.o search.o files.o utils.o
```
``

在该目录下直接输入命令 make 就可以生成执行文件 edit。如果要删除可执行文件和所有的中间目标文件，地执行一下 make clean ,clean 如同 C 语言中的 label，通过 make 显示指定该 label 的名字执行后面定义的命令, 可以在 clean 定义不用的编译或是和编译无关的命令，比如程序的打包，程序的备份，等等。*依赖关系的实质就是说明了目标文件是由哪些文件生成的，目标文件是哪些文件更新的*。

#### Makefile 是如何工作的？

```default
make 会在当前目录下找名字叫“Makefile”或“makefile”的文件。 
如果找到，它会找文件中的第一个目标文件（target），在上面的例子中，他会找到“edit”这个文 件，并把这个文件作为最终的目标文件。
如果 edit 文件不存在，或是 edit 所依赖的后面的 .o 文件的文件修改时间要比 edit 这个文件新， 那么，他就会执行后面所定义的命令来生成 edit 这个文件。 
如果 edit 所依赖的 .o 文件也不存在，那么 make 会在当前文件中找目标为 .o 文件的依赖性，如 果找到则再根据那一个规则生成 .o 文件。（这有点像一个堆栈的过程） 
当然，你的 C 文件和头文件是存在的啦，于是 make 会生成 .o 文件，然后再用 .o 文件生成 make 的终极任务，也就是可执行文件 edit 了。
在找寻的过程中，如果出现错误，比如最后被依赖的文件找不到，那么 make 就会直接退出，并 报错，而对于所定义的命令的错误，或是编译不成功，make 根本不理。make 只管文件的依赖性，即，如 果在我找了依赖关系之后，冒号后面的文件还是不在，那么对不起，我就不工作啦。
```



#### Makefile 使用变量

makefile 的变量也就是一个字符串，理解成 C 语言中的宏可能会更好, 例如：
```
objects = main.o kbd.o command.o display.o \                                      insert.o search.o files.o utils.o
```

#### GNU Make 自动推导

make 看到一个 .o 文件，它就会自动的把 .c 文件加在依赖关系, 举例：``如果 make 找到一个 whatever.o ，那么 whatever.c 就会是 whatever.o 的依赖文件。并且 cc -c whatever.c 也会被推导 出``
