## sleep
### 函数原型

```c
void
sleep(void *chan, struct spinlock *lk)
{
  struct proc *p = myproc();
  
  // Must acquire p->lock in order to
  // change p->state and then call sched.
  // Once we hold p->lock, we can be
  // guaranteed that we won't miss any wakeup
  // (wakeup locks p->lock),
  // so it's okay to release lk.

  acquire(&p->lock);  //DOC: sleeplock1
  release(lk);

  // Go to sleep.
  p->chan = chan;
  p->state = SLEEPING;

  sched();

  // Tidy up.
  p->chan = 0;

  // Reacquire original lock.
  release(&p->lock);
  acquire(lk);
}
```

#### 参数介绍
```c
```

#### 返回值
```c
```


## wake up

### 函数原型

```c
// Wake up all processes sleeping on chan.
// Must be called without any p->lock.
void
wakeup(void *chan)
{
  struct proc *p;

  for(p = proc; p < &proc[NPROC]; p++) {
    if(p != myproc()){
      acquire(&p->lock);
      if(p->state == SLEEPING && p->chan == chan) {
        p->state = RUNNABLE;
      }
      release(&p->lock);
    }
  }
}

```

在唤醒一个进程之前，必须先获得一个进程的锁

#### 参数介绍
```c
```

#### 返回值
```c
```


## scheduler

### 函数原型

```c
void
scheduler(void)//启动后调度的第一个进程为initcode
{
  struct proc *p;
  struct cpu *c = mycpu();
  
  c->proc = 0;
  for(;;){
    // Avoid deadlock by ensuring that devices can interrupt.
    intr_on();

    for(p = proc; p < &proc[NPROC]; p++) {
      acquire(&p->lock);
      if(p->state == RUNNABLE) {
        // Switch to chosen process.  It is the process's job
        // to release its lock and then reacquire it
        // before jumping back to us.
        p->state = RUNNING;
        c->proc = p;
        swtch(&c->context, &p->context);//交换后ret到ra寄存器里面的值 

        // Process is done running for now.
        // It should have changed its p->state before coming back.
        c->proc = 0;
      }
      release(&p->lock);
    }
  }
}

```

- 每个 cpu 都有一个自己的进程调度器。
- 每个 cpu 在设置好自己之后调用 scheduler()。
- 调度程序永远不会返回。它循环，做:
	-选择要运行的进程。
	-交换机启动该进程。
	-最终，这个过程转移了控制权
- 通过 switch 返回到 scheduler。

#### 参数介绍
```c
```

#### 返回值
```c
```

## sched

### 函数原型

```c
void
sched(void)
{
  int intena;
  struct proc *p = myproc();

  if(!holding(&p->lock))
    panic("sched p->lock");
  if(mycpu()->noff != 1)
    panic("sched locks");
  if(p->state == RUNNING)
    panic("sched running");
  if(intr_get())
    panic("sched interruptible");

  intena = mycpu()->intena;
  swtch(&p->context, &mycpu()->context);//会将ra改成scheduler的地址 
  mycpu()->intena = intena;
}

```

- xv6 只能通过 sched 切换到调度器。
- 必须只持有 p->锁并且已经更改了 proc->状态。如果还持有其他锁，会可能导致死锁
- 保存和恢复 intena，因为 intena 是内核线程的属性，而不是 CPU 的属性。
- 它应该是 proc-> intena 和 proc-> noff，但在少数几个锁持有但没有进程的地方，这将会中断。

#### 参数介绍
```c
```

#### 返回值
```c
```
## yield

### 函数原型

```c
// Give up the CPU for one scheduling round.
void
yield(void)
{
  struct proc *p = myproc();
  acquire(&p->lock);//防止将p->state改为runnable后，两个cpu同时运行该进程
  p->state = RUNNABLE;
  sched();
  release(&p->lock);
}
```
- the kernel handler 自动让出 cpu 给其他程序
- 抢占式让出cpu
#### 参数介绍
```c
```

#### 返回值
```c
```


## exit

### 函数原型


```c
void exit(int status){
  struct proc *p = myproc();
  if(p == initproc) //init进程不允许退出
    panic("init exiting");
  for(int fd = 0; fd < NOFILE; fd++){//关闭该进程所有的文件引用
    if(p->ofile[fd]){
      struct file *f = p->ofile[fd];
      fileclose(f);
      p->ofile[fd] = 0;
    }
  }
	//将该进程的目录的引用释放给文件系统
  begin_op();
  iput(p->cwd);
  end_op();
  p->cwd = 0;

  acquire(&wait_lock);

  //我们可以将该进程的子进程重新指派父进程为init。我们不能精确地唤醒init，因为一旦我们获得了任何其他进程的锁，我们就无法获得它的锁。不管是否需要，都要唤醒init。它可能会错过这个唤醒，但这似乎是无害的。init的工作就是在一个循环中不停的调用init
  reparent(p);

  //唤醒当前进程的父进程，因为当前的父进程或许正在等待当前进程退出
  wakeup(p->parent);
  
  acquire(&p->lock);

  p->xstate = status;
  p->state = ZOMBIE;//现场的进程还没完全释放它所占的资源,可能需要重用,如fork

  release(&wait_lock);

  // Jump into the scheduler, never to return.
  sched();
  panic("zombie exit");
}

```

#### 参数介绍
```c
```

#### 返回值
```c
```

## kill

### 函数原型

```c
// Kill the process with the given pid.
// The victim won't exit until it tries to return
// to user space (see usertrap() in trap.c).
int
kill(int pid)
{
  struct proc *p;

  for(p = proc; p < &proc[NPROC]; p++){
    acquire(&p->lock);
    if(p->pid == pid){
      p->killed = 1;
      if(p->state == SLEEPING){
        // Wake process from sleep().
        p->state = RUNNABLE;
      }
      release(&p->lock);
      return 0;
    }
    release(&p->lock);
  }
  return -1;
}

```
- kill 没有停止其他进程的退出、运行等等, 只是使 p->killed = 1
- 目标程序运行到内核代码能安全停止运行的位置时，会检查自己的 killed 标志位，if 1, 其他进程会自动的执行 exit 系统调用
#### 参数介绍
```c
```

#### 返回值
```c
```
## wait

### 函数原型

```c
// Wait for a child process to exit and return its pid.
// Return -1 if this process has no children.
int
wait(uint64 addr)
{
  struct proc *np;
  int havekids, pid;
  struct proc *p = myproc();

  acquire(&wait_lock);

  
  for(;;){
	//当一个进程调用wait时，扫描进程清单，寻找父进程是当前当前proc的proc,并且当前状态为ZOMBIE
    // Scan through table looking for exited children.
    havekids = 0;
    for(np = proc; np < &proc[NPROC]; np++){
      if(np->parent == p){
        // make sure the child isn't still in exit() or swtch().
        acquire(&np->lock);

        havekids = 1;
        if(np->state == ZOMBIE){
          // Found one.收集子进程的退出状态
          pid = np->pid;
          if(addr != 0 && copyout(p->pagetable, addr, (char *)&np->xstate,
                                  sizeof(np->xstate)) < 0) {
            release(&np->lock);
            release(&wait_lock);
            return -1;
          }
          freeproc(np);//由父进程最后来释放子进程的资源
          release(&np->lock);
          release(&wait_lock);
          return pid;
        }
        release(&np->lock);
      }
    }

    // No point waiting if we don't have any children.
    if(!havekids || p->killed){
      release(&wait_lock);
      return -1;
    }
    
    // Wait for a child to exit.
    sleep(p, &wait_lock);  //DOC: wait-sleep
  }
}

```
-wait 不只是方便知道子进程什么时候退出，实际上也是进程退出的一个重要组成部分，在 unix 中每一个进程的退出都需要由一个对应的 wait 系统调用
#### 参数介绍
```c
```

#### 返回值
```c
```




## proc_freepagetable 

### 函数原型

```c
// Free a process's page table, and free the physical memory it refers to.
void
proc_freepagetable(pagetable_t pagetable, uint64 sz)
{
  uvmunmap(pagetable, TRAMPOLINE, 1, 0);
  uvmunmap(pagetable, TRAPFRAME, 1, 0);
  uvmfree(pagetable, sz);
}
```
- 释放一个进程的页表，并释放页表所占的物理内存
#### 参数介绍
```c
```

#### 返回值
```c
```

## freeproc

### 函数原型

```c
// free a proc structure and the data hanging from it,
// including user pages.
// p->lock must be held.
static void freeproc(struct proc *p){
	if(p->trapframe)
		kfree((void*)p->trapframe);
	p->trapframe = 0;
	if(p->pagetable)
		proc_freepagetable(p->pagetable, p->sz);

	p->pagetable = 0;
	p->sz = 0;
	p->pid = 0;
	p->parent = 0;
	p->name[0] = 0;
	p->chan = 0;
	p->killed = 0;
	p->xstate = 0;
	p->state = UNUSED;
}
```
- 释放进程结构及其挂起的数据，包括用户页面。
- 必须持有 P ->锁。
- 因为内核栈的 guard page，没有必要再释放一次内核栈
#### 参数介绍
```c
```

#### 返回值
```c
```