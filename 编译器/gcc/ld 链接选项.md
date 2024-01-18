### 0.1.1 -T
>用于指定链接脚本文件，控制可执行文件或共享库的内存布局。链接脚本定义了可执行文件的各个节（sections）的排列方式、起始地址和大小等信息
	
以下是`ld`使用`-T`选项的示例命令：

```shell
ld -T linker_script.ld object_file.o -o output_file

ld mbr.o -Ttext Ox7c00
//目标文件 `mbr.o` 链接为可执行文件，并将其起始地址设置为 0x7c00。在这个特定的示例中，它可能是用于链接引导扇区代码的命令，以生成可引导的镜像文件
```

在这个示例中，`linker_script.ld`是链接脚本文件的路径，`object_file.o`是要链接的目标文件，`output_file`是生成的可执行文件或共享库的输出文件名。

链接脚本文件中定义了各个节的起始地址、大小和属性等。它可以使用链接脚本语言的语法来描述内存布局，例如：


```ld
ENTRY(start)
SECTIONS {
    . = 0x1000;
    .text : { *(.text) }
    .data : { *(.data) }
    .bss : { *(.bss) }
}
```

上述链接脚本定义了程序的入口点为`start`，并按顺序排列了`.text`、`.data`和`.bss`节。每个节的内容由输入目标文件中对应的节填充。


请注意，链接脚本的语法和功能因链接器而异，因此具体的用法和语法细节可能会有所不同。可以参考所使用的链接器的文档和参考资料，以了解其支持的链接脚本语法和选项。


### 0.1.2 -dynamic-linker -lc

```shell
ld -dynamic-linker /lib64/ld-linux-x86-64.so.2 \
  /usr/lib/x86_64-linux-gnu/crt1.o \
  /usr/lib/x86_64-linux-gnu/crti.o \
  main.o say.o -lc \
  /usr/lib/x86_64-linux-gnu/crtn.o
```


- `dynamic-linker /lib64/ld-linux-x86-64.so.2`：指定动态链接器的路径。这个路径指向 ld-linux-x86-64.so.2 文件，它是 Linux 系统上的默认动态链接器,负责动态库的加载。
    
- `/usr/lib/x86_64-linux-gnu/crt1.o` 和 `/usr/lib/x86_64-linux-gnu/crti.o`：是`C runtime`的缩写这两个文件是 C 运行时启动代码，用于初始化 C 程序的运行环境,例如程序的入口函数 `_start` (二进制文件并不是从 `main` 开始执行的！)、`atexit` 注册回调函数的执行等。
    
- `main.o` 和 `say.o`：这两个是编译器生成的目标文件，分别包含 `main` 函数和 `say` 函数的实现。
    
- `-lc`：这个选项指定链接器链接 C 标准库（libc）。`-lc` 表示链接`glibc`库。
    
- `/usr/lib/x86_64-linux-gnu/crtn.o`：这个文件是 C 运行时结束代码，用于清理 C 程序的运行环境。


### 0.1.3 -m
- -melf_x86_64
	指定链接为 x86_64 ELF 格式；


### 0.1.4 -N 
标记 `.text` 和 `.data` 都可写，这样它们可以一起加载 (而不需要对齐到页面边界)，减少可执行文件的大小；