# 1 Overrall Options

- -c
	表示只进行编译并生成目标文件，而不进行链接操作。编译过程将把源文件编译成目标文件（通常是`.o`文件）

- --verbose
	 可以完整的显示编译和链接过程
	 

# 2 C Language Option

- -fno-builtin
	GCC 提供了一些内建函数，这些函数在编译器级别实现了一些常见的操作和功能。它们通常以内联的方式提供，可以在编译时直接嵌入到生成的代码中，以提高执行效率。

	使用 -fno-builtin 选项可以告诉编译器禁用内建函数的自动内联。这意味着编译器会将内建函数视为普通的函数调用，并生成对应的函数调用指令。禁用内建函数可能会导致一些微小的性能损失，但也提供了更大的灵活性

- -ffreestanding
	通过使用 `-ffreestanding` 选项，GCC 编译器将根据自由站立的需求进行编译，不依赖于操作系统提供的标准库和运行时环境。这会禁用某些标准库函数和特性，并要求程序显式提供必要的功能。
# 3 Debugging Options
- -g
	告诉gcc在编译过程中生成调试信息
	
	- gdb 进一步指定生成符合GDB调试器的调试信息。

	

# 4 Warining Options

- -Wall

	启用编译器的大多数警告信息。它会开启一系列警告，帮助您发现潜在的问题和不规范的代码。使用 -Wall 可以提高代码质量和可靠性
- -Werror
	选项用于将所有警告视为错误


# 5 linker Options

- -Wl, option
	选项允许将参数传递给连接器，以便控制连接过程的行为。您可以使用 -Wl 将连接器选项传递给连接器，例如链接库、指定输出文件名等


# 6 Directory Options
> These options specify directories to search for header files, for libraries and for parts of the compiler:

- `-I dir`

- `-iquote dir`

- `-isystem dir`

- `-idirafter dir`