# vscode 的布局

![[Pasted image 20230802211944.png]]
- 活动栏: 从上到下依次为，打开侧边栏，搜索，使用 git，debug，使用插件

- 侧边栏: 新建项目文件和文件夹

- 编辑栏: 编写代码的区域

- 面板栏: 从左到右依次为，问题，输出，调试栏，终端（terminal），最重要的是 terminal，用来输入相关命令

- 状态栏: 该区域可以调出面板栏

- 需要注意的为下图红框所示，分别表示鼠标光标所在位置和 tab 缩进字符，这里为缩进 4 个字符


# Predefined variables

- **${userHome}** - the path of the user's home folder
- **${workspaceFolder}** - the path of the folder opened in VS Code
- **${workspaceFolderBasename}** - the name of the folder opened in VS Code without any slashes (/)
- **${file}** - the current opened file
- **${fileWorkspaceFolder}** - the current opened file's workspace folder
- **${relativeFile}** - the current opened file relative to `workspaceFolder`
- **${relativeFileDirname}** - the current opened file's dirname relative to `workspaceFolder`
- **${fileBasename}** - the current opened file's basename
- **${fileBasenameNoExtension}** - the current opened file's basename with no file extension
- **${fileExtname}** - the current opened file's extension
- **${fileDirname}** - the current opened file's folder path
- **${fileDirnameBasename}** - the current opened file's folder name
- **${cwd}** - the task runner's current working directory upon the startup of VS Code
- **${lineNumber}** - the current selected line number in the active file
- **${selectedText}** - the current selected text in the active file
- **${execPath}** - the path to the running VS Code executable
- **${defaultBuildTask}** - the name of the default build task
- **${pathSeparator}** - the character used by the operating system to separate components in file paths

# 窗口改变

## zen mode

- `C + k z`

## 打开(隐藏)边栏
- `C + b` 
- `C + S + b`
# 代码编辑快捷键


## 代码报错跳转

- `C + S + m` 打开 problem view
## 代码重构


### 函数重命名

-  `f2`


### 代码快速修复
- `C + .`


### 快速格式化代码
- `C + S + i`


## 代码结构查看
- `C + k C + i`
## 快速注释 

- `C + /`



# 文件相关操作快捷键


## 浏览所有打开文件并跳转

`C + T`


## 搜索文件并打开

-  `C + p`


# 终端操作快捷键

## 切换终端聚焦

```
C + `
C + S + ` //新建终端
```

