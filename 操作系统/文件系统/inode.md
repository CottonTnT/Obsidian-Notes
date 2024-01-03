## Ext2 inode(第0版rev.0)
![[Pasted image 20240103152641.png]]
- 一个`inode`在磁盘上是`128Byte`
- 一个`block`能存放8个`inode`
- 一个`inode`总共有60个字节存放`direct blocks number` 和 `indirect blocks number`



### file is appendable, but not easy-insertable


 ![[Pasted image 20240103155200.png]]


## Ext4
![[Pasted image 20240103155333.png]]