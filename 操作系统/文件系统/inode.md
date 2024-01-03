## Ext2 inode(第0版rev.0)
![[Pasted image 20240103152641.png]]
- 一个`inode`在磁盘上是`128Byte`
- 一个`block`能存放8个`inode`
- 一个`inode`总共有60个字节存放`direct blocks number` 和 `indirect blocks number`



### file is appendable, but not easy-insertable


 ![[Pasted image 20240103155200.png]]


## Ext4
![[Pasted image 20240103155333.png]]


## Filesystem abstraction refined

### inode = file

![[Pasted image 20240103230830.png]]

```cpp
class FileSystem{
	public:
		using inode_num_t = uint32_t;
		explict FileSystem(BlockDevice*dev);
		inode_num_t lookup(string_view path);
		shared_ptr<Inode> getInode(inode_num_t)
	private:
		BlockDevice* dev_;
		SuperBlock sb_;
		Map<string, inode_num_t> dirs_;
		Array<Inode> inodes_;
}

struct Inode{
	using block_num_t = uint64_t;

	bool isDir() const;
	bool isFile() const;
	bool is SystemLink() const;
	int64_t file_size;
	int ref_count;
	block_num_t getBlockNum(uint32 idx);
	bool appendBlock(block_num_t blk);
private:
	vector<block_num_t> blocks;
}
```
