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

``




