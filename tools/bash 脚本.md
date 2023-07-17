## Shebang
- (#!)字符序列被称为 Shebang。它后面是解释器 (或程序)的路径，用来*运行 (或解释)文本文件中其余的行*。(对于 Bash 脚本，它将是 Bash 的路径，但是还有许多其他类型的脚本，它们都有自己的解释器。)

- shebang 必须位于文件的第一行 (第 2 行不行，即使第一行是空的)。   # 前面和 !之间也不能有空格。以及通往解释器的路径。

- 虽然可以为解释器使用相对路径，但大多数情况下您将希望使用绝对路径。您可能会从各种位置运行脚本，因此绝对路径是最安全的 (在这种特殊情况下，通常也比相对路径短)。

- 省略带有 shebang 的行并仍然运行脚本是可能的，但这是不明智的。如果您在终端上运行 Bash shell，并且执行了一个不带 shebang 的脚本，那么 Bash 将假定它是一个 Bash 脚本。因此，这只能在运行脚本的用户在 Bash shell 中运行脚本的情况下工作，并且有各种原因导致这种情况可能不存在，这是危险的。

- 还可以运行 Bash，将脚本作为参数传递。

## Variables 

### 赋值

```c
foo=bar
// `foo = bar` 不要使用空格隔开，因为解释器会调用程序`foo` 并将 `=` 和 `bar`作为参数。在shell脚本中使用空格会起到分割参数的作用


```

### 字符串
```c
foo=bar
echo "$foo" #打印 bar
echo '$foo'#打印 $foo
//Bash中的字符串通过' 和 "分隔符来定义，但是它们的含义并不相同。以'定义的字符串为原义字符串，其中的变量不会被转义，而 "定义的字符串会将变量值进行替换。
```

### 命令替换


```c
myvar=$(ls /etc | wc -l)
echo There are $myvar entries in the directory /etc 
```
- 命令替换允许我们获取命令或程序的输出(通常会打印到屏幕上)并将其保存为变量的值。为了做到这一点，我们把它放在括号里，前面有一个$符号。


### 特殊变量

```c
$0     -脚本名
$1到$9 -脚本的参数。$1 是第一个参数，依此类推。
$@     -所有参数
$#     -参数个数
$?     -前一个命令的返回值
$$     -当前脚本的进程识别码
!!     -完整的上一条命令，包括参数。常见应用：当你因为权限不足执行命令失败时，可以使用 `sudo !!`再尝试一次。
$_     -上一条命令的最后一个参数。如果你正在使用的是交互式 shell，你可以通过按下 `Esc` 之后键入 . 来获取这个值。
```


### 导出变量


```c
export var1 //Make the variable var1 available to child processes.
```

- 导出变量是一个单向过程。原始进程可以将变量传递给新进程，但进程对变量副本所做的任何操作都不会影响原始变量。


## input

### command line arguments
- 略

### Read input during script execution 

```c
read var1 //ask the user for input. This command takes the input and will save it into a var1.
	-p  <prompt> //allows you to specify a prompt
	-s //makes the input silent

//how about read more ?

read var1 var2 var3

```


### Accept data that has been redirected into Bash script via STDIN

```c
#### summary

1. #!/bin/bash
2. # A basic summary of my sales report

4. echo Here is a summary of the sales data:
5. echo ====================================
6. echo

8. cat /dev/stdin | cut -d' ' -f 2,3 | sort
```



```c
1. cat salesdata.txt
2. Fred apples 20 July 4
3. Susy oranges 5 July 7
4. Mark watermelons 12 July 10
5. Terry peaches 7 July 15

7. cat salesdata.txt | ./summary
8. Here is a summary of the sales data:
9. ====================================

11. apples 20
12. oranges 5
13. peaches 7
14. watermelons 12
```



## Arithmetic

### let

**let** is a builtin function of Bash that allows us to do simple arithmetic.

```c
//let_example.sh

#!/bin/bash
# Basic arithmetic using let

let a=5+4
echo $a # 9

let "a = 5 + 4"
echo $a # 9

let a++
echo $a # 10

let "a = 4 * 5"
echo $a # 20

let "a = $1 + 30"
echo $a # 30 + first command line argument


//output
user@bash ./let_example.sh 15
9
9
10
20
45
```


### expr
```c
expr item1 operator item2
```
- **expr** is similar to let except instead of saving the result to a variable it instead prints the answer. 
- Unlike **let** you *don't need to enclose the expression in quotes*. 
- You also *must have spaces between the items of the expression*. 
- It is also common to use **expr** within command substitution to save the output to a variable.

```c
#### expr_example.sh

1. #!/bin/bash
2. # Basic arithmetic using expr

expr 5 + 4

expr "5 + 4"

expr 5+4

expr 5 \* $1

12. expr 11 % 2

14. a=$( expr 10 - 3 )
15. echo $a 
```

```c
user@bash ./expr_example.sh 12
9
5 + 4
5+4
60
1
7
```


### Double Parentheses

在关于变量的一节中，我们看到可以很容易地将命令的输出保存到变量中。事实证明，如果我们稍微调整一下语法，这个机制也可以为我们做基本的算术。我们使用双括号这样做:
```c
$(( expression ))
```



### Length of a Variable

find out the length of a variable

```c
${#var}
```





## If Statements


```c
if [<some test>]
then
	<command>
elif [<elif test1>] || [<elif test2>]
then
	<elif command>
else
	<other command>
fi
```

### Test

some of the more common ones are listed below

![[Pasted image 20230717133606.png]]

- **=** is slightly different to **-eq**. [ 001 = 1 ] will return false as = does a string comparison (ie. character for character the same) whereas -eq does a numerical comparison meaning [ 001 -eq 1 ] will return true.
- - When we refer to FILE above we are actually meaning a *path*. Remember that a path may be absolute or relative and may refer to a file or a directory.


### indenting

缩进，是编写良好、整洁代码的重要组成部分(在任何语言中，不仅仅是 Bash 脚本)。在 Bash 中没有任何关于缩进的规则，所以你可以缩进或不缩进，但你的脚本仍然会完全一样地运行。强烈建议缩进代码(特别是当你的脚本变得更大)，否则你会发现越来越难以看到你的脚本结构。


### Case Statements

```c
case <variable> in  
<pattern 1>)  
	<commands>  
	;;  
<pattern 2>)  
	<other commands>  
	;;  
esac
```


```c
#!/bin/bash
# case example

case $1 in
    start)
		echo starting
	;;
    stop)
		echo stoping
	;;
    restart)
		echo restarting
    ;;
    *)
		echo don\'t know
	;;
esac
```



## Loops!


### While Loops

```c
while [<some test>]
do
	<commands>
done
```

### Until Loops

```c
until [ <some test> ]  
do  
	<commands>  
done
```

### For loops

```c
for var in <list>
do
	<commands>
done
```

- The list is defined as a series of strings, separated by spaces.

#### Ranges

```c
for value in {1..5}
do
	echo $value
done 

for value in {10..0..2}
do
	echo $value
done

```

- It's important when specifying a range like this that there are no spaces present between the curly brackets { }.  If there are then it will not be seen as a range but as a list of items.

- It is also possible to specify a value to increase or decrease by each time. You do this by adding another two dots ( .. ) and the value to step by.

### Controls

```c
The **break** statement tells Bash to leave the loop straight away.
he **continue** statement tells Bash to stop running through this iteration of the loop and begin the next iteration.
//同c语言
```


### Select

```c
select var in <list>  
do  
	<commands>  
done

```




## Functions

```c
function_name(){
	<commands>
}
// or

function function_name(){
	<commands>
}

```


- 上述任意一种指定函数的方法都是有效的。两者的运作方式相同，两者之间没有优劣之分。这只是个人喜好。
- 在其他编程语言中，通常将参数传递给括号 ()内列出的函数。在 Bash 中，它们只是用来装饰的，你永远不会在里面放任何东西。
- 函数定义 (实际函数本身)必须在对函数的任何调用之前出现在脚本中


### 传递参数

```c
#!/bin/bash
# Passing arguments to a function

print_something () {
    echo Hello $1
}

print_something Mars
print_something Jupiter
```


### Return Values

Bash 函数没有函数返回值的概念，但是，它们允许我们设置返回状态。类似于一个程序或命令如何退出，退出状态表明它是否成功。我们使用关键字 return 来指示返回状态。


```c
#!/bin/bash
# Setting a return status for a function

print_something () {
    echo Hello $1
	return 5
}

print_something Mars
print_something Jupiter
echo The previous function has a return value of $?
```

### Variable Scope

 - By default a variable is **global**. 
 - We may also create a variable as a **local** variable.  When we create a local variable within a function, it is only visible within that function. To do that we use the keyword **local** in front of the variable the first time we set it's value.

```c
#!/bin/bash
# Experimenting with variable scope

var_change () {
	local var1='local 1'
    echo Inside function: var1 is $var1 : var2 is $var2
	var1='changed again'
	var2='2 changed again'
}

var1='global 1'
var2='global 2'

echo Before function call: var1 is $var1 : var2 is $var2

var_change

echo After function call: var1 is $var1 : var2 is $var2
```


```c
user@bash: ./local_variables.sh
Before function call: var1 is global 1 : var2 is global 2
Inside function: var1 is local 1 : var2 is global 2
After function call: var1 is global 1 : var2 is 2 changed again
```

### Overriding Commands

```c
#!/bin/bash
# Create a wrapper around the command ls

ls () {
	command ls -lh
}
ls
```

- 在上面的例子中，如果我们没有在第5行的 ls 前面加上关键字 command，我们就会陷入一个无限循环。即使我们在函数 ls 内部，当我们调用 ls 时，它也会调用函数 ls 的另一个实例，这个实例反过来也会做同样的事情，以此类推。



## User Interface

