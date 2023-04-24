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