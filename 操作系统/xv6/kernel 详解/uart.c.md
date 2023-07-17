## 一些definition
### 函数原型
```c
// the UART control registers are memory-mapped
// at address UART0. this macro returns the
// address of one of the registers.
#define Reg(reg) ((volatile unsigned char *)(UART0 + reg))

// the UART control registers.
// some have different meanings for
// read vs write.
// see http://byterunner.com/16550.html
#define RHR 0                 // receive holding register (for input bytes)
#define THR 0                 // transmit holding register (for output bytes)
#define IER 1                 // interrupt enable register
#define IER_RX_ENABLE (1<<0)
#define IER_TX_ENABLE (1<<1)
#define FCR 2                 // FIFO control register
#define FCR_FIFO_ENABLE (1<<0)
#define FCR_FIFO_CLEAR (3<<1) // clear the content of the two FIFOs
#define ISR 2                 // interrupt status register
#define LCR 3                 // line control register
#define LCR_EIGHT_BITS (3<<0)
#define LCR_BAUD_LATCH (1<<7) // special mode to set baud rate
#define LSR 5                 // line status register
#define LSR_RX_READY (1<<0)   // input is waiting to be read from RHR
#define LSR_TX_IDLE (1<<5)    // THR can accept another character to send

#define ReadReg(reg) (*(Reg(reg)))
#define WriteReg(reg, v) (*(Reg(reg)) = (v))

// the transmit output buffer.
struct spinlock uart_tx_lock;
#define UART_TX_BUF_SIZE 32
char uart_tx_buf[UART_TX_BUF_SIZE];//uart用来传送数据的buffer
uint64 uart_tx_w; // write next to uart_tx_buf[uart_tx_w % UART_TX_BUF_SIZE]，写指针与下面的读指针来构建一个环形的buffer
uint64 uart_tx_r; // read next from uart_tx_buf[uart_tx_r % UART_TX_BUF_SIZE]
```


## uartinit 
### 函数原型

```c
void
uartinit(void)
{
  // disable interrupts.关闭中断
  WriteReg(IER, 0x00);

  // special mode to set baud rate.设置波特率,即串口线的传输速率
  WriteReg(LCR, LCR_BAUD_LATCH);

  // LSB for baud rate of 38.4K.
  WriteReg(0, 0x03);

  // MSB for baud rate of 38.4K.
  WriteReg(1, 0x00);

  // leave set-baud mode,
  // and set word length to 8 bits, no parity.设置字符长度
  WriteReg(LCR, LCR_EIGHT_BITS);

  // reset and enable FIFOs.重置且打开UART内部的FIFO
  WriteReg(FCR, FCR_FIFO_ENABLE | FCR_FIFO_CLEAR);

  // enable transmit and receive interrupts.重新打开中断
  WriteReg(IER, IER_TX_ENABLE | IER_RX_ENABLE);

  initlock(&uart_tx_lock, "uart");
}
```

初始化 uart 设备

#### 参数介绍
```c
```

#### 返回值
```c
```


## uartstart
### 函数原型

```c
void
uartstart()
{
  while(1){
    if(uart_tx_w == uart_tx_r){//检查当前缓存是否为空
      // transmit buffer is empty.
      return;
    }
    
    if((ReadReg(LSR) & LSR_TX_IDLE) == 0){//检查当前设备是否空闲,THR是否能接受另一个字符
      // the UART transmit holding register is full,
      // so we cannot give it another byte.
      // it will interrupt when it's ready for a new byte.
      return;
    }
    
    int c = uart_tx_buf[uart_tx_r % UART_TX_BUF_SIZE];
    uart_tx_r += 1;
    
    // maybe uartputc() is waiting for space in the buffer.
    wakeup(&uart_tx_r);
    
    WriteReg(THR, c);//将字符送入THR传入
  }
}
```

- 如果 UART 是空闲的，并且一个字符正在等待发送,在发送缓冲区中发送。
- 调用方必须持有 uart_tx_lock。
- 从上半部分和下半部分调用。

#### 参数介绍
```c
```

#### 返回值
```c
```


## uartputc

### 函数原型

```c
void
uartputc(int c)
{
  acquire(&uart_tx_lock);

  if(panicked){
    for(;;)
      ;
  }

  while(1){
    if(uart_tx_w == uart_tx_r + UART_TX_BUF_SIZE){//检查环形buf是否已满,初始时写指针与读指针相等，buf为空
      // buffer is full.
      // wait for uartstart() to open up space in the buffer.
      sleep(&uart_tx_r, &uart_tx_lock);
    } else {//如果buf未满
      uart_tx_buf[uart_tx_w % UART_TX_BUF_SIZE] = c;
      uart_tx_w += 1;
      uartstart();
      release(&uart_tx_lock);
      return;
    }
  }
}
```

- 向输出缓冲区中添加一个字符并告诉UART 开始发送，如果它还没有。
- 如果输出缓冲区已满，则阻塞。
- 因为它可能会阻塞，所以不能调用从中断; 它只适合使用通过写 ()。
#### 参数介绍
```c
```

#### 返回值
```c
```

## uartgetc

### 函数原型

```c
int
uartgetc(void)
{
  if(ReadReg(LSR) & 0x01){//从第一寄存器获取数据
    // input data is ready.
    return ReadReg(RHR);
  } else {
    return -1;
  }
}
```
- 从 UART 读入一个输入的字符
- return -1 如果没有输入的话

#### 参数介绍
```c
```

#### 返回值
```c
```


## uartintr 

### 函数原型

```c
void
uartintr(void)
{
  // read and process incoming characters.
  while(1){
    int c = uartgetc();//是否是因为键盘中断,或者键盘是否还在读入
    if(c == -1)
      break;
	    consoleintr(c);//触发cosoleintr()中断
  }
//无论是键盘中断或是UART完成输入的中断,都需将trans下一个字符
  // send buffered characters.
  acquire(&uart_tx_lock);
  uartstart();
  release(&uart_tx_lock);
}
```

- 处理一个 intr，因为输入有已到达，或者部件准备好进行更多输出，或者两者都有。
- 从 trap.C中调用。

#### 参数介绍
```c
```

#### 返回值
```c

```


## wakeup

### 函数原型

```c
// Wake up all processes sleeping on chan.
// Must be called without any p->lock.
void
wakeup(void *chan){
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

- 唤醒所有获有 chan 的处于休眠状态的进程。
- 必须在没有 p->lock的情况下调用。

#### 参数介绍
```c

```

#### 返回值
```c
```


## sleep

### 函数原型

```c
void sleep(void *chan, struct spinlock *lk){
	struct proc *p = myproc();
	// Must acquire p->lock in order to
	// change p->state and then call sched.
	// Once we hold p->lock, we can be
	// guaranteed that we won't miss any wakeup
	// (wakeup locks p->lock),
	// so it's okay to release lk.
	acquire(&p->lock); //DOC: sleeplock1
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

- 自动释放锁并休眠 on chan。
- 唤醒时需要重新获取锁。

#### 参数介绍
```c
```

#### 返回值
```c
```