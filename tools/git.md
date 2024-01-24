# 1 一堂很棒的入门课

https://www.bilibili.com/video/BV1Wh4y1s7Lj/?spm_id_from=333.999.0.0&vd_source=9ae91b4d2d5e405fc28a9000eb9ba3cd

# 2 Git 状态模型

![[Pasted image 20240124112354.png]]a


## 2.1 工作区(workspace)

就是我们当前工作空间，也就是我们当前能在本地文件夹下面看到的文件结构。初始化工作空间或者工作空间 clean 的时候，文件内容和 index 暂存区是一致的，随着修改，工作区文件在没有 add 到暂存区时候，工作区将和暂存区是不一致的。


## 2.2 暂存区(index)

老版本概念也叫 Cache 区，就是文件暂时存放的地方，所有暂时存放在暂存区中的文件将随着一个 commit 一起提交到 local repository 此时 local repository 里面文件将完全被暂存区所取代。暂存区是 git 架构设计中非常重要和难理解的一部分。


## 2.3 本地仓库

git 是分布式版本控制系统，和其他版本控制系统不同的是他可以完全去中心化工作，你可以不用和中央服务器 (remote server) 进行通信，在本地即可进行全部离线操作，包括 log，history，commit，diff 等等。完成离线操作最核心是因为 git 有一个几乎和远程一样的本地仓库，所有本地离线操作都可以在本地完成，等需要的时候再和远程服务进行交互。

## 2.4 远程仓库

中心化仓库，所有人共享，本地仓库会需要和远程仓库进行交互，也就能将其他所有人内容更新到本地仓库把自己内容上传分享给其他人。结构大体和本地仓库一样。




文件在不同的操作下可能处于不同的 git 生命周期，下面看看一个文件变化的例子。


![[Pasted image 20240124112932.png]]
# 3 git 中的数据对象模型


## 3.1 仓库结构

git 分布式的一个重要体现是 git 在本地是有一个完整的 git 仓库也就是 .git 文件目录，通过这个仓库，git 就可以完全离线化操作。在这个本地化的仓库中存储了 git 所有的模型对象。下面是 git 仓库的 tree 和相关说明：

![[Pasted image 20240124114207.png]]

git 主要有四个对象，分别是 `Blob，Tree， Commit， Tag` 他们都用 `SHA-1` 进行命名。

你可以用 `git cat-file -t` 查看每个 `SHA-1` 的类型，用 `git cat-file -p` 查看每个对象的内容和简单的数据结构。git cat-file 是 git 的瑞士军刀，是底层核心命令。




![[Pasted image 20230715092434.png]]
## 3.2 blob对象
```c
// some actual types
type blob = array<byte> // file, 只用于存储单个文件内容，一般都是二进制的数据文件，不包含任何其他文件信息，比如不包含文件名和其他元数据。
```


## 3.3 tree对象

```c
type tree = map<string, tree | blob> // folders, 目录名到实际内容的映射,对应文件系统的目录结构，里面主要有：子目录 (tree)，文件列表 (blob)，文件类型以及一些数据文件权限模型等。
```

如下图输出
```shell
→ git cat-file -t ed807a4d010a06ca83d448bc74c6cc79121c07c3
tree
→ git cat-file -p ed807a4d010a06ca83d448bc74c6cc79121c07c3

100644 blob 36a982c504eb92330573aa901c7482f7e7c9d2e6    .cise.yml
100644 blob c439a8da9e9cca4e7b29ee260aea008964a00e9a    .eslintignore
100644 blob 245b35b9162bec4ef798eb05b533e6c98633af5c    .eslintrc
100644 blob 10123778ec5206edcd6e8500cc78b77e79285f6d    .gitignore
100644 blob 1a48aa945106d7591b6342585b1c29998e486bf6    README.md
100644 blob 514f7cb2645f44dd9b66a87f869d42902174fe40    abc.json
040000 tree 8955f46834e3e35d74766639d740af922dcaccd3    cli_list100644 blob f7758d0600f6b9951cf67f75cf0e2fabcea55771    dep.json
040000 tree e2b3ee59f6b030a45c0bf2770e6b0c1fa5f1d8c7    doc
100644 blob e3c712d7073957c3376d182aeff5b96f28a37098    index.js
040000 tree b4aadab8fc0228a14060321e3f89af50ba5817ca    lib040000 tree 249eafef27d9d8ebe966e35f96b3092d77485a79    mock
100644 blob 95913ff73be1cc7dec869485e80072b6abdd7be4    package.json
040000 tree e21682d1ebd4fdd21663ba062c5bfae0308acb64    src
040000 tree 91612a9fa0cea4680228bfb582ed02591ce03ef2    static
040000 tree d0265f130d2c5cb023fe16c990ecd56d1a07b78c    task100644 blob ab04ef3bda0e311fc33c0cbc8977dcff898f4594    webpack.config.js
100644 blob fb8e6d3a39baf6e339e235de1a9ed7c3f1521d55    webpack.dll.config.js
040000 tree 5dd44553be0d7e528b8667ac3c027ddc0909ef36    webpack
```

详细解释如下:



```
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


## 3.4 git 命令

### 3.4.1 初始化本地仓库

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
### 3.4.2 查看当前仓库信息

```c
git status
```


### 3.4.3 创建快照

```c
//git让用户在创建快照时能够自由选择需要上传哪些更改，
//staging area：告诉git下一次创建快照应该包含哪些更改
git add [files] // 将当前文件加入staging area
	-p //允许交互地暂存文件的片段

git commit  //git snapshot command,即提交更改
	-a 提交所有git跟踪的文件的更改
```


### 3.4.4 可视化提交历史
```c
git log //查看版本历史记录
	--all --graph --decorate // 图型化且优雅的查看
	--oneline //显示更紧凑
```

### 3.4.5 一些特殊引用 

```c
HEAD  //基本用于指向你当前正在查看的commit，默认指向最后一次snapshot
```

### 3.4.6 改变当前工作目录状态

```c
git checkout [commit-id] 
git checkout [brance-name]
git checkout [files] //将file的内容回溯到HEAD指向的快照
	-f //忽略当前分支的更改，强制切换分支
// checkout 命令实质移动了HEAD指针，然后改变当前工作目录的内容，有可能会销毁掉未提交的更改

git stash //将工作目录恢复到上一次提交的状态，但会保持当前的更改
git stash pop //重新展示之前保存的更改

```

### 3.4.7 查看当前目录具体变化

```c
git diff [files] //即与HEAD的指向相比
git diff [id/branch] [files] //与特定快照相比
git diff [id/brance] [id/brance] [files] //特定的两个快照，file的不同
 --cached //显示暂存区里的实际保留的更改
```

### 3.4.8 分支与合并

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

### 3.4.9 操作远程仓库
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

## 3.5 配置 git

### 3.5.1 .gitconfig
```c
git config
or
vim .gitconfig 
//copy 同行最方便
```

### 3.5.2 .gitignore
让 git 不要多管闲事
```c
*.o //忽略所有.o文件
```
## 3.6 git internal command

### 3.6.1 查看 ref 的键值
```c
git cat-file -p [id]
```



## 3.7 git 与 github

github 只是 git 一个仓库托管平台，loose def 即一个可用的远程仓库