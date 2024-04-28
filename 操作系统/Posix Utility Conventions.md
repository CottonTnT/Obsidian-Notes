
# 1 Utility Argument Syntax
This section describes **the argument syntax** of the standard utilities and **introduces terminology** used throughout POSIX.1-2017 for describing the arguments processed by the utilities


Unless otherwise noted, all utility descriptions use this notation, which is illustrated by this example

$$\begin{aligned}
utility\_name[-a][-b][-c\space option\_argument] [-d|-e] \\ [-f[option\_argument]][operand...]
\end{aligned}
$$

- `utility_name`: 实用程序的名称。
- `[-a]`, `[-b]`: 方括号表示这些选项*是可选的*。`-a` 和 `-b` 是不带参数的选项，用于开启特定的功能或模式。
- `[-c option_argument]`: `-c` 是一个*需要紧随其后的参数*（`option_argument`）的选项，用于提供额外的必要信息或值。
- `[-d|-e]`: 表示 `-d` 和 `-e` 是*互斥*的选项，即在同一命令中只能使用其中一个。
- `[-f[option_argument]]`: 表示 `-f` 是一个*可选的*选项，它可以单独使用，也*可以带有一个紧随其后的可选参数*。
- `[operand...]`: 方括号和省略号表示操作数是可选的，并且可以指定一个或多个。操作数通常是命令作用的目标，如文件名、URL等。

```shell
`cp source destination` 里的 `source` 和 `destination` 就是操作数operand
“option_argument”是跟在某些选项后面的值，用来为那些选项提供必要的信息。
`ls -l /home` 中，`-l` 是一个选项，它告诉 `ls` 以长格式列出信息，而 `/home` 是操作数，指明了 `ls` 命令的目标目录。如果一个选项需要特定的值来具体指导其行为，这个值就是选项参数。如 `-c config.txt` 中的 `config.txt` 是 `-c` 选项的参数，指定了配置文件。
```



