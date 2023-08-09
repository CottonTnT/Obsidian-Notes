## 1. 概念

 makefile 关系了整个工程的编译和链接规则，通过 make 命令进行自动化编译。make 是解释 makefile 中指令的命令工具。

## 2.makefile 的规则

#### makefile 的五个法宝

Makefile 里主要包含了五个东西：显式规则、隐式规则、变量定义、指令和注释。

|   fs   |  释义    |
|:-----|:-----|
|  显示规则    | 显式规则说明了如何生成一个或多个目标文件。明显指出要生成的文件、文件的依赖文件和生成的命令     |
|   隐式规则   |   make 有自动推导的功能    |
|  变量的定义    |  makefile 中定义变量一般都是字符串，有点像 C 语言中的宏，当 Makefile 被执行时，在变量位置进行字符串替换  |
|     指令 |    包括三个部分，一是在一个 Makefile 中引用另一个 Makefile，如 C 中的 ``include``；二是指根据情况指定 Makefile 中的有效部分，如 C 的预编译 `` #if ``；三是定义一个多行的命令。  |
|     注释 |  Makefile 中只有行注释，其注释是用 # 字符，如需使用 # 字符，可以用反斜杠进行转义，如：\\# 。最后，Makefile 中的命令，必须以 Tab 键开始。    |



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

#### Make 在上述实例是如何工作的？

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
objects = main.o kbd.o command.o display.o \                                       insert.o search.o files.o utils.o
```

#### GNU Make 自动推导

make 看到一个 .o 文件，它就会自动的把 .c 文件加在依赖关系, 举例：``如果 make 找到一个 whatever.o ，那么 whatever.c 就会是 whatever.o 的依赖文件。并且 cc -c whatever.c 也会被推导 出``

*See the changes*
```makefile
objects = main.o kbd.o command.o display.o insert.o search.o files.o utils.o 

edit : $(objects) 
	cc -o edit $(objects) 
main.o : defs.h 
kbd.o : defs.h command.h 
command.o : defs.h command.h 
display.o : defs.h buffer.h 
insert.o : defs.h buffer.h 
search.o : defs.h buffer.h 
files.o : defs.h buffer.h command.h 
utils.o : defs.h

.PHONY : clean 
clean : rm edit $(objects)
```

*How about this style*
```makefile
objects = main.o kbd.o command.o display.o insert.o search.o files.o utils.o 

edit : $(objects) 
	cc -o edit $(objects)
	
$(objects) : defs.h 
kbd.o command.o files.o : command.h 
display.o insert.o search.o files.o : buffer.h 

.PHONY : clean 
clean : 
	rm edit $(objects)
```

#### 清空目录稳健的做法

```
.PHONY : clean 
clean : 
	-rm edit $(objects)
	.PHONY 表示 clean 是一个“伪目标”。而在 rm 命令前面一个小减号的意思就是，也许某些文件出现问题，但不要管，继续做后面的事。clean 的规则不要放在文件的开头，不然，这就会变成make 的默认目标
```

#### Makefile 的文件名

默认的情况下，make 命令会在当前目录下按顺序寻找文件名为 GNUmakefile 、makefile 和 Makefile 的文件。在这三个文件名中，最好使用 Makefile 这个文件名。当然，你可以使用别的文件名来书写 Makefile，比如“Make.Linux”等，如指定特定的 Makefile，你可以使用 make 的 -f 或 --file 参数，如：make -f Make.Solaris 或 make --file Make.Linux 。如果你使用多条 -f 或  --file 参数，你可以指定多个 makefile。

#### 包含其他 makefile

使用 include 指令可以把别的 Makefile 包含进来，如 C 语言的 `` #include``，被包含的文件会复制放在当前文件的包含位置。include 的语法是：
```
include <filenames> ...
```

filenames 可以是当前操作系统 Shell 的文件模式（可以包含路径和通配符）。在 include 前面可以有一些空字符，但是绝不能是 Tab 键开始。举个例子，你有这样几个 Makefile：a.mk 、b.mk 、c.mk ，还有一个文件叫 foo.make ，以及一个变量 $(bar) ，其包含了 bish 和 bash ，那么，下面的语句：

```
include foo.make *.mk $(bar)
```
 等价于：
 
```
 include foo.make a.mk b.mk c.mk bish bash
```
 make 命令开始时，会找寻 include 所指出的其它 Makefile，并把其内容安置在当前的位置。如果文件都没有指定绝对路径或是相对路径的话，make 会在当前目录下首先寻找，如果当前目录下没有找到，那么，make 还会在下面的几个目录下找：
  1. 如果 make 执行时，有 -I 或 --include-dir 参数，那么 make 就会在这个参数所指定的目录下去寻找。 
  2. 接下来按顺序寻找目录 /include （一般是 /usr/local/bin ）、/usr/gnu/include 、 /usr/local/include 、/usr/include 。环境变量 .INCLUDE_DIRS 包含当前 make 会寻找的目录列表。你应当避免使用命令行参数 -I 来寻找以上这些默认目录，否则会使得 make “忘掉”所有已经设定的包含目录，包括默认目录。如果你想让 make 不理那些无法读取的文件，而继续执行,如： -include ... 其表示，无论 include 过程中出现什么错误，都不要报错继续执行。如果要和其它版本 make 兼容，可以使用 sinclude 代替 -include 。

#### 环境变量 MAKEFILES

避免使用!!！

#### make 的工作方式

GNU 的 make 工作时的执行步骤如下：
```
1. 读入所有的 Makefile。
2. 读入被 include 的其它 Makefile。 
3. 初始化文件中的变量。 
4. 推导隐式规则，并分析所有规则。 
5. 为所有的目标文件创建依赖关系链。 
6. 根据依赖关系，决定哪些目标要重新生成。 
7. 执行生成命令。  
```

1-5 步为一阶段，6-7 为二阶段。一阶段中，如果定义的变量被使用了，那么，make 会把其展开在使用的位置。但 make 并不会完全马上展开，make 使用的是拖延战术，如果变量出现在依赖关系的规则中，那么仅当这条依赖被决定要使用了，变量才会在其内部展开。  
`