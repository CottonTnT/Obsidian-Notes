## 一堂很棒的入门课

https://www.bilibili.com/video/BV1Wh4y1s7Lj/?spm_id_from=333.999.0.0&vd_source=9ae91b4d2d5e405fc28a9000eb9ba3cd

## git 中的底层数据模型
![[Pasted image 20230715092434.png]]
```c
// some actual types
type blob = array<byte> // file, 一推字节

type tree = map<string, tree | blob> // folders, 目录名到实际内容的映射

type commit = struct { // 一推属性，
	parents: array<commit>
	autuor:string
	message:string
	snapshot:tree  //存了实际的内容，是一课树，代表相应的顶层目录
}
//how gits actually stores and address this actual data ?
type object = blob | tree | commit

//在git中所有objects都是内容寻址的，即id是根据内容生产的
//git 在磁盘存储的是一堆objects

objects = map<string, object>

def store(o):
	id = sha1(o)
	objects[id] = o

def load(id):
	return objects[id]

// id 过长？来个human-friendly name

type references = map<string, string> //即 name->id
```
git 所有操作基本变换的都是 objects 和references


## git 命令

### 初始化本地仓库

```c
git init //会默认创建一个master分支，通常代表日常开发中主分支
```
会生产. git 文件夹，其中包含的内容就是历史的完整副本，包含 objects 和 refs，和之前所有的快照

```c
git init --bare //裸仓库，同常用来做服务器仓库
```
- 裸库往往被创建用于作为大家一起工作的共享库，每一个人都可以往里面 push 自己的本地修改。
- 裸仓库一个惯用的命名方式是在库名后加上.git。-
- 裸仓库初始化后，其项目目录下就是标准仓库.git 目录里的内容，没有工作空间。  
- 这个仓库只保存 git 历史提交的版本信息，而不允许用户在上面进行各种 git 操作（如：push、commit 操作）。
- 依旧可以使用 git show 命令查看提交内容。
### 查看当前仓库信息

```c
git status
```


### 创建快照

```c
//git让用户在创建快照时能够自由选择需要上传哪些更改，
//staging area：告诉git下一次创建快照应该包含哪些更改
git add [files] // 将当前文件加入staging area
	-p //允许交互地暂存文件的片段

git commit  //git snapshot command,即提交更改
	-a 提交所有git跟踪的文件的更改
```


### 可视化提交历史
```c
git log //查看版本历史记录
	--all --graph --decorate // 图型化且优雅的查看
	--oneline //显示更紧凑
```

### 一些特殊引用 

```c
HEAD  //基本用于指向你当前正在查看的commit，默认指向最后一次snapshot
```

### 改变当前工作目录状态

```c
git checkout [commit-id] 
git checkout [brance-name]
git checkout [files] //将file的内容回溯到HEAD指向的快照
	-f //忽略当前分支的更改，强制切换分支
// checkout 命令实质移动了HEAD指针，然后改变当前工作目录的内容，有可能会销毁掉未提交的更改

git stash //将工作目录恢复到上一次提交的状态，但会保持当前的更改
git stash pop //重新展示之前保存的更改

```

### 查看当前目录具体变化

```c
git diff [files] //即与HEAD的指向相比
git diff [id/branch] [files] //与特定快照相比
git diff [id/brance] [id/brance] [files] //特定的两个快照，file的不同
 --cached //显示暂存区里的实际保留的更改
```

### 分支与合并

分支是为了实行并行开发
```c
git branch //查看当前分支
	-vv  //详细信息关于所有分支
	--set-upstream-to=<repo_name/brance_name> //为当前分支push指定特定的远程仓库与分支

git branch [newbranch-name]//创建一个name分支，实质上是创建了一个ref，和HEAD指向同一个commit

git checkout [branch-name] //切换分支,即让HEAD指向该branch-name

git merge [branch-name] //合并branch-name分支到当前head所指分支
	--abort //回到之前git merge 时的状态
	//当合并时遇到冲突时,需要手动去选择合并哪些内容
	//手动解决冲突后需要重新git add [conflict file]
	--continue //解决冲突后继续合并分支
```

### 操作远程仓库
远程仓库的目的，即让其他人可以拥有整个 Git 仓库的副本，让你的本地仓库知道其他克隆副本的存在

```c
git remote //列出当前本地仓库所知道的所有远程仓库

git remote add <repo_name> <url> //如果没有远程repo默认名字为origin

git push <remote> <local_branch>:<remote_branch>//将local_brance的更改发送到remote的remote_branch，用--set-upstream-to设置后，可以直接git push

git fetch <remote> //同步远程仓库的更改，但不会更改本地历史记录、本地引用、分支等
//想要切换到远程仓库的状态还需要 本地git merge

git pull <remote> //相当于 git fetch 然后 git merge

git clone <url> <folder name>//克隆一个仓库到本地，并以此初始化本地仓库,拥有完整的历史记录
	--shallow //只克隆最新的快照

```

## 配置 git

### .gitconfig
```c
git config
or
vim .gitconfig 
//copy 同行最方便
```

### .gitignore
让 git 不要多管闲事
```c
*.o //忽略所有.o文件
```
## git internal command

### 查看 ref 的键值
```c
git cat-file -p [id]
```



## git 与 github

github 只是 git 一个仓库托管平台，loose def 即一个可用的远程仓库