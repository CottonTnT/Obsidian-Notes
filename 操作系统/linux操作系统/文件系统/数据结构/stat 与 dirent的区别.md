## Struct stat
### 1.定义
- 用于储存文件的属性信息，类型、大小、权限、所有者、创建时间、修改时间等
```c
struct stat { 
	dev_t st_dev; /* ID of device containing file */ 
	ino_t st_ino; /* inode number */ 
	mode_t st_mode; /* protection */ 
	nlink_t st_nlink; /* number of hard links */ 
	uid_t st_uid; /* user ID of owner */ 
	gid_t st_gid; /* group ID of owner */ 
	dev_t st_rdev; /* device ID (if special file) */ 
	off_t st_size; /* total size, in bytes */ 
	blksize_t st_blksize; /* blocksize for file system I/O */ 
	blkcnt_t st_blocks; /* number of 512B blocks allocated */ 
	time_t st_atime; /* time of last access */ 
	time_t st_mtime; /* time of last modification */ 
	time_t st_ctime; /* time of last status change */ };
```

### 2.相关函数

####   1.stat

##### 函数原型

```c
int stat(const char *path, struct stat *buf);
```

##### 参数介绍
-  其中，path 参数为文件路径，buf 参数为存储文件状态信息的结构体指针。调用成功后，文件状态信息将被存储在 buf 指向的结构体中。 


##### 返回值
- stat 函数返回值为0表示调用成功，返回-1表示调用失败。调用失败时，可以通过 errno 变量获取错误码。 

####  2. fstat

##### 函数原型
```c
int fstat(int fd, struct stat *buf);
```

##### 参数介绍
- 其中，fd 参数为已打开文件的文件描述符，buf 参数为存储文件状态信息的结构体指针。调用成功后，文件状态信息将被存储在 buf 指向的结构体中。

##### 返回值
- fstat 函数返回值为0表示调用成功，返回-1表示调用失败。调用失败时，可以通过 errno 变量获取错误码。


## struct dirent

### 1. 定义

- 用于储存目录中的文件信息，包括文件名、文件类型等等。它的定义如下
```c
struct dirent { 
	ino_t d_ino; /* inode number 如果一个目录项的d_ino字段为0，表示该目录项无效或已被删除。这通常发生在目录项被删除但是目录文件尚未被更新的情况下。在这种情况下，d_ino字段为0，但是d_name字段仍然存在，因此需要特别注意处理这种情况，以避免出现错误   */ 
	off_t d_off; /* offset to the next dirent */ 
	unsigned short d_reclen; /* length of this record */ 
	unsigned char d_type; /* type of file; not supported by all file system types */ 
	char d_name[256]; /* filename */ 
};
```