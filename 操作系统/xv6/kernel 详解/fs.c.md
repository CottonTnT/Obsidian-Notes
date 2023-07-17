## iget

### 函数原型

```c
static struct inode*
iget(uint dev, uint inum)
{
  struct inode *ip, *empty;

  acquire(&itable.lock);

  // Is the inode already in the table?
  empty = 0;
  for(ip = &itable.inode[0]; ip < &itable.inode[NINODE]; ip++){
    if(ip->ref > 0 && ip->dev == dev && ip->inum == inum){
      ip->ref++;
      release(&itable.lock);
      return ip;
    }
    if(empty == 0 && ip->ref == 0)    // Remember empty slot.
      empty = ip;
  }

  // Recycle an inode entry.
  if(empty == 0)
    panic("iget: no inodes");

  ip = empty;
  ip->dev = dev;
  ip->inum = inum;
  ip->ref = 1;
  ip->valid = 0;
  release(&itable.lock);

  return ip;
}

```

- 在设备 dev 上找到编号为 inum 的 inode，并返回内存中的副本。
- 不锁定 inode，也不从磁盘读取它。
#### 参数介绍
```c
```

#### 返回值
```c
```
## namex

### 函数原型

```c
static struct inode*
namex(char *path, int nameiparent, char *name)
{
  struct inode *ip, *next;

  if(*path == '/')
    ip = iget(ROOTDEV, ROOTINO);
  else
    ip = idup(myproc()->cwd);

  while((path = skipelem(path, name)) != 0){
    ilock(ip);
    if(ip->type != T_DIR){
      iunlockput(ip);
      return 0;
    }
    if(nameiparent && *path == '\0'){
      // Stop one level early.
      iunlock(ip);
      return ip;
    }
    if((next = dirlookup(ip, name, 0)) == 0){
      iunlockput(ip);
      return 0;
    }
    iunlockput(ip);
    ip = next;
  }
  if(nameiparent){
    iput(ip);
    return 0;
  }
  return ip;
}
```
- 查找并返回索引 inode 以获取路径名。
- 如果 parent != 0，则返回父节点的 inode，并将最后的 path 元素复制到 name 中，该元素必须有容纳 DIRSIZ 字节的空间。
- 必须在事务中调用，因为它调用 input()。
#### 参数介绍
```c
```

#### 返回值
```c
```
## nameiparent

### 函数原型

```c
struct inode*
nameiparent(char *path, char *name)
{
  return namex(path, 1, name);
}
```

#### 参数介绍
```c
```

#### 返回值
```c
```