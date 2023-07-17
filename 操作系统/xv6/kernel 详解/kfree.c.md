## kfree

### 函数原型

```c
void
kfree(void *pa)
{
  struct run *r;

  if(((uint64)pa % PGSIZE) != 0 || (char*)pa < end || (uint64)pa >= PHYSTOP)
    panic("kfree");

  // Fill with junk to catch dangling refs.
  memset(pa, 1, PGSIZE);

  r = (struct run*)pa;

  acquire(&kmem.lock);
  r->next = kmem.freelist;
  kmem.freelist = r;
  release(&kmem.lock);
}
```

- 释放由 pa 指向的物理内存页面
- 该页面通常应该通过调用 kalloc()返回。(在初始化分配器时例外;见 kinit。)

#### 参数介绍
```c
```

#### 返回值
```c
```