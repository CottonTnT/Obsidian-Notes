> 通过本节学习如何正确使用`tcp`


- `netcat` 基本功能简介
	1. 信号发生器：发送数据。`nc > /dev/zero`、类似`chargen`服务
	2. 负载：接收数据。`nc > /dev/null`、类似`discard`服务
	3. 通过`dd`产生定量数据，通过`nc`发送测试网络带宽，dd `/dev/zero | nc`、类似`ttcp`服务
	4. 两台机器之间通过nc拷贝文件，`nc < file, nc > file`、类似scp服务
# 1 why netcat


1. read/write two(three) fd simultanenously
	不仅涉及socket, 还有标准输入输出,有并发性

2. Two simple concurrent modes are examined

	1. Thread-per-connection with blocking IO
	2. IO-multiplexing with non-blocking IO

3. How/when to close a Tcp connection?
	Ensure all data are sendt and received

4. Why IO-multiplexing must be used with non-blocking IO?



# 2 How/when to close a Tcp conn?


## 2.1 `why my TCP is not reliable`?


1. send all then close, but last few bytes are lost.
- **问题**
	如博客[The ultimate SO_LINGER page, or: why is my tcp not reliable](https://blog.netherlabs.nl/articles/2009/01/18/the-ultimate-so_linger-page-or-why-is-my-tcp-not-reliable), 如果协议栈的接受缓冲区里有数据,而应用程序还未读取就直接调用 `close` 会导致 `TCP` 协议栈发送一个 `reset` 分解(不是 `fin` ), 强行断开连接, 如果此时协议栈的 `发送缓冲区` 还有数据, 或一些数据在网络上丢失, 对方没有收到, 那么这个数据就真的丢失了
	示例代码-[sender.cc]([recipes/tpc/bin/sender.cc at master · CottonTnT/recipes](https://github.com/CottonTnT/recipes/blob/master/tpc/bin/sender.cc))
	![[{8C885C1C-C8EA-4457-B77D-1CA88EA7F003}.png]]
- **solution**
	1.  `read()` 返回0 才去 `close()`
```c++
// snippets of sender.cc
// Safe close connection
printf("Shutdown write and read until EOF\n");

stream->shutdownWrite();//

while ((nr = stream->receiveSome(buf, sizeof buf)) > 0)
{
        // do nothing
}	
```


- **总结**


![[{14B0CE43-96F1-4C57-B32A-5298CC0509B5}.png]]
*可能的Bug*（这里`read()`返回0应考虑客户端存在Bug或恶意的不返回0的情况，即Block IO 一直阻塞, Non-Block IO 一直 `EAGAIN`, 使得客户端永远不满足`read()=0`的情况。因此这里因考虑有超时机制，在`shutdown`之后若干秒内如果没有满足`read()=0`，则强制断开连接并有相应的错误处理）


*缺陷* 发送方就算采用了这里推荐的做法, 发送方也无法确切指导接受端已经收到或正确处理了数据,因为接受方有可能是直接就 `crush` 或 `close` , `read`还是会返回 `0`


# 3 Ignore SIGPIPE in any concurrent TCP server

> SIGPIPE is sent to writer when the other end of ``pipe`` is closed


程序对于SIGPIPE的默认行为是 `terminate the process` , good for command line piping
```sh
gunzip -c huge.log.gz | grep -a Succeeded. | head -10
//管道末端的程序终止,其他依次收到SIGPIPE , 要死一起死, 不做多余计算浪费CPU
```

此命令的只取压缩文件中含有 “Succeeded.” 的前10函数数据，当成功匹配到10行数据后，gunzip也会停止执行，这样就可以避免将整个大文件都解压缩。


由于缓存的存在，gunzip 解压的数据大于 grep接收的数据 大于 head 接收的数据，但是由于Unix默认是阻塞IO，在通过管道传递数据时会自带一种节流的效果，因此，gunzip实际解压的数据也不会比head接收到的数据大很多


示例2：对于网络IO，当我们关闭了一个连接时，如果尝试向他写入数据，也会收到一个 `SIGPIPE` 信号。

对于这种情况，如果在服务端没有做特殊处理，有一个客户端*意外退出*，造成连接关闭，而服务端如果此时发送消息的话，就会收到 `SIGPIPE` 信号，导致服务端被迫退出。


因此，网络程序一般在启动时，都会忽略掉 `SIGPIPE` 信号。

```c
//normal
signal(SIGPIPE, SIG_IGN);

//muduo
通过一个全局对象的构造来忽略掉SIGPIPE信号
```

另外，需要注意的是，忽略SIGPIPE后，如果对方关闭了连接，我们的程序可能不会退出，因此，我们*需要额外关注一些函数的返回值*，例如我们向一个连接请求数据后将它输出到标准输出时，我们需要额外关注printf 函数的返回值，如果它的返回值是负值，则表示对端连接已关闭。那么我们的程序刺客应该退出。


# 4 Nagle\`s  algorithm, TCP_NODELAY

`Nagle`算法主要是避免发送小的数据包，要求`TCP`连接上最多只能有一个未被确认的小分组，在该分组的确认到达之前不能发送其他的小分组。

`Nagle`算法的目的：避免发送大量的小包，网络上每次只能一个小包存在，在小包被确认之前，只能积累发送大包，`如果包长度达到MSS，则允许发送`；如果该包含有`FIN`，则允许发送；但发生了超时（一般为200ms），则立即发送， 启动`TCP_NODELAY`，就意味着禁用了Nagle算法

`Nagle`算法会严重影响请求响应式协议的延迟。
如果我们的程序设计的不够合理，`Nagle`算法可能会增加程序的延迟。

如果你的程序是 `write-write-read` 模式，在使用了`Nagle`算法后，第二个 `write` 就会被推后一个`RTT`发送而造成一个很长的`ack`等待，从而产生一个延迟。为了避免这种情况，一般建议在应用层做缓冲，将两个`write`合在一起，成为 `write-read`。

但是还有一种情况是，在一个连接上并发的有多个请求时，我们很难将数据整合在一起，它们来自程序中不同的位置。而这种情况Nagle算法会大大增加程序的延迟。

因此，如果你没有十足的把握驾驭Nagle算法的话，我们建议使用 `TCP_NODELY` 关闭Nagle算法。
# 5 使用 SO_REUSEADDR 选项

一般服务器的监听`socket`都应该打开它。它允许服务器bind一个地址，即使这个地址当前已经存在已建立的连接。


![[Pasted image 20250104145544.png]]
- 服务器启动后，与客户端建立连接，如果服务器**主动关闭**，那么和客户端的连接会处于**TIME_WAIT状态**，此将无法启动服务器进程。
- 服务器父进程监听客户端，建立连接后，fork一个子进程专门处理客户端的请求，如果父进程停止，因为子进程还和客户端有连接，所以此时重启父进程会失败。


对于以上两张情况，重启服务器都会出现`bind: : Address already in use`错误。而当我们使用了 `SO_REUSEADDR` 选项后，服务器退出后，仍然允许我们马上重启进程


# 6 `Netcat`的实现

这里提供三种netcat的实现：

1. [recipes/tpc/netcat.cc thread-per-connection]()
2. [recipes/python/netcat.py IO-multiplexing]()
3. [recipes/python/netcat-nonblock.py IO-multiplexing]
以及，为了测试以上 netcat 程序，这里还提供了对应的负载生成器：

1. [recipes/tpc/chargen.cc]()
2. [recipes/python/chargen.py]()
3. [muduo/examples/simple/chargen/*]()


## 6.1 基于多线程阻塞IO实现的 `Netcat`


`Thread-per-connection` *适用于*连接数目不太多，或者线程非常廉价(在C/C++ JAVA 中线程一般都不廉价)的情况。推荐使用这种方式。

一般情况下，使用主线程创建监听套接字等待客户端连接，建立连接后分发到工作线程进行处理。

该版本 netcat 多线程的方式实现， 一个连接对应两个线程去处理，每个线程负责连接上的一个方向，即读 或 写。


```c++
// recipes/tpc/netcat.cc 部分代码

void run(TcpStreamPtr stream)
{
  // Caution: a bad example for closing connection
  // 读socket，写stdout
  std::thread thr([&stream] () {
    char buf[8192];
    int nr = 0;
    // 接收数据，recv返回0，则代表对端关闭。因此跳出循环，关闭连接
    while ( (nr = stream->receiveSome(buf, sizeof(buf))) > 0)
    {
      int nw = write_n(STDOUT_FILENO, buf, nr);
      if (nw < nr)
      {
        break;
      }
    }
    /* 这里采用exit的方式暴力的结束程序。因为，检测到对端半关闭连接时，
    * 本端主线程可能正处于read(stdin) 的阻塞状态，为了让主线程这边
    * 也能够退出，因此采用了exit 的方案。
    */
    ::exit(0);  // should somehow notify main thread instead
  });

  // 主线程：读stdin，写socket
  char buf[8192];
  int nr = 0;
  // 阻塞IO，这里read阻塞，直至读取到数据。
  // 退出条件为，读stdin时，返回0.则主线程关闭套接字写段，发送FIN
  while ( (nr = ::read(STDIN_FILENO, buf, sizeof(buf))) > 0)
  {	// 读取到数据后，马上发送到socket上
    int nw = stream->sendAll(buf, nr);
    if (nw < nr)
    {
      break;
    }
  }
  // 关闭写段，发送FIN
  stream->shutdownWrite();
  thr.join();
}

```


## 6.2 基于 IO 复用（阻塞IO）实现的 netcat


`IO` 复用（事件驱动）使得一个线程可以处理多个连接上的请求，值得注意的是它是*同步`IO`*, 实际上复用的是线程, 一个线程可以处理多个 `IO`。

