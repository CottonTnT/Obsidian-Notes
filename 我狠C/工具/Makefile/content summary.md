## 1. 概念

 makefile 关系了整个工程的编译和链接规则，通过 make 命令进行自动化编译。make 是解释 makefile 中指令的命令工具。

## 2.makefile 的规则

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

在该目录下直接输入命令 make 就可以生成执行文件 edit。如果要删除可执行文件和所有的中间目标文件，地执行一下 make clean  。*依赖关系的实质就是说明了目标文件是由哪些文件生成的，目标文件是哪些文件更新的*。

