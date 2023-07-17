## spinlock. h

### 函数原型

```c
struct spinlock {
  uint locked;       // Is the lock held?

  // For debugging:
  char *name;        // Name of lock.
  struct cpu *cpu;   // The cpu holding the lock.
};

```


## push_off 

### 函数原型

```c
void
push_off(void)
{
  int old = intr_get();

  intr_off();
  if(mycpu()->noff == 0)
    mycpu()->intena = old;
  mycpu()->noff += 1;
}

```

#### 参数介绍
```c
```

#### 返回值
```c
```

## pop_off 

### 函数原型

```c
void
pop_off(void)
{
  struct cpu *c = mycpu();
  if(intr_get())
    panic("pop_off - interruptible");
  if(c->noff < 1)
    panic("pop_off");
  c->noff -= 1;
  if(c->noff == 0 && c->intena)
    intr_on();
}
```

Push_off /pop_off 类似于 intr_off()/intr_on()，只是它们是匹配的:撤销两个 Push_off()需要两个 pop_off()。同样，如果中断最初是关闭的，那么 push_off 和 pop_off 将关闭它们。

#### 参数介绍
```c
```

#### 返回值
```c
```

## acquire

### 函数原型

```c
// Acquire the lock.
// Loops (spins) until the lock is acquired.
void
acquire(struct spinlock *lk)
{
  push_off(); // disable interrupts to avoid deadlock.
  if(holding(lk))
    panic("acquire");

  // On RISC-V, sync_lock_test_and_set turns into an atomic swap:
  //   a5 = 1
  //   s1 = &lk->locked
  //   amoswap.w.aq a5, a5, (s1)
  while(__sync_lock_test_and_set(&lk->locked, 1) != 0)
    ;

  // Tell the C compiler and the processor to not move loads or stores
  // past this point, to ensure that the critical section's memory
  // references happen strictly after the lock is acquired.
  // On RISC-V, this emits a fence instruction.
  __sync_synchronize();

  // Record info about lock acquisition for holding() and debugging.
  lk->cpu = mycpu();
}

```

 Xv6 的 **acquire** 使用了可移植的 C 库调用 **__sync_lock_test_and_set** ，它本质上为 **amoswap** 指令；返回值是 **lk->locked** 的旧（交换）内容。 **acquire** 函数循环交换，重试（旋转）直到获取了锁。每一次迭代都会将 1 交换到**lk->locked** 中，并检查之前的值；如果之前的值为 0 ，那么我们已经获得了锁，并且交换将**lk->locked** 设置为 1 。如果之前的值是 1 ，那么其他 CPU 持有该锁，而我们原子地将 1 换成 **lk->locked** 并没有改变它的值。 
 
#### 参数介绍
```c
```

#### 返回值
```c
```


## release

### 函数原型

```c
// Release the lock.
void
release(struct spinlock *lk)
{
  if(!holding(lk))
    panic("release");

  lk->cpu = 0;

  // Tell the C compiler and the CPU to not move loads or stores
  // past this point, to ensure that all the stores in the critical
  // section are visible to other CPUs before the lock is released,
  // and that loads in the critical section occur strictly before
  // the lock is released.
  // On RISC-V, this emits a fence instruction.
  __sync_synchronize();

  // Release the lock, equivalent to lk->locked = 0.
  // This code doesn't use a C assignment, since the C standard
  // implies that an assignment might be implemented with
  // multiple store instructions.
  // On RISC-V, sync_lock_release turns into an atomic swap:
  //   s1 = &lk->locked
  //   amoswap.w zero, zero, (s1)
  __sync_lock_release(&lk->locked);

  pop_off();
}
```

函数 **release (kernel/spinlock. C:47)** 与 **acquire** 相反：它清除 **lk->cpu** 字段，然后释放锁。从概念上讲，释放只需要给 **lk->locked** 赋值为 0 。*C 标准允许编译器用多条存储指令来实现赋值，所以 C 赋值对于并发代码来说可能是非原子性的*。相反，release 使用 C 库函数 **__sync_lock_release** 执行原子赋值。这个函数也是使用了 RISC-V 的 **amoswap** 指令。

#### 参数介绍
```c
```

#### 返回值
```c
```