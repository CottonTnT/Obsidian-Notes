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


下面进行一个测试：

- server：chargen.cc,，只发数据不收数据
- client: nc， nc < /dev/zero

如下代码是 `python` 实现的基于阻塞IO的多路复用

```python
def relay(sock:socket):
    poll = select.poll()
    poll.register(sock, select.POLLIN)
    poll.register(sys.stdin, select.POLLIN)

    done = False
    while not done:
        events = poll.poll(10000)  # 10 seconds
        for fileno, event in events:
            if event & select.POLLIN:
                if fileno == sock.fileno():
                    data = sock.recv(8192)
                    if data:
                        sys.stdout.write(str(data))
                    else:
                        done = True
                else:
                    assert fileno == sys.stdin.fileno()
                    data = os.read(fileno, 8192)
                    if data:
                        sock.sendall(data)
                    else:
                        sock.shutdown(socket.SHUT_WR) #需要等待socket 读完数据,才能关闭连接
                        poll.unregister(sys.stdin)
    sock.close() #也可以不关, 反正运行完这个函数,就退出了
```

`chargen` 程序只发送数据而不读取，如果使用 `nc` 时有标准输入，即 `nc` 会向 `chargen` 发送数据，将最终导致 `chargen` 的接收缓冲区被填满，而 `nc` 无法再发送数据。

因此，`nc` 的实现方式就显得尤为重要了。如果 `nc` 这边是网络读写没有分离开，那么由于对端缓冲区满将会导致本端写动作阻塞，进而阻塞整个程序。

示例一：使用`python`实现的 `nc` 与 `chargen` 测试。可以看到，`nc` 单向接收时，吞吐量可以达到 `9894MiB/s` ，而 `nc` 端有输入时，引起了阻塞。

![[{7F1191A3-1FFD-4702-891F-AA802F69A0BC}.png]]

使用 **strace** 调试一下程序，发现 `nc` 阻塞在`sendto()`上

![[{F782603F-677C-4BDD-8EFD-BEE877265473}.png]]

查看连接上的缓冲区可以发现，由于 `chargen` 没有读数据，它的输入缓冲区被填满了，导致 `nc` 阻塞在 `sendto` 上，同时由于 `sendto` 的阻塞，`nc` 也不会再读取输入缓冲区的数据。

![[{D124463C-630F-47B7-B403-EE52B858DD5B}.png]]


示例二:使用多线程实现的 ` recipes/tpc/netcat.cc` 与 `chargen` 测试。可以看到这次即使 nc 有数据输入，程序依然正常运行

![[{E98A08BA-7D28-4AA1-87AA-D75C97D065E4}.png]]


因此，**阻塞IO 如果和 IO复用 配合使用，一旦发生阻塞就会影响到同一事件循环下的其他IO事件。**


## 6.3 基于 IO复用（非阻塞IO）实现的 netcat

使用非阻塞IO可以有效避免上述情况的发生。但非阻塞IO在编程上要比阻塞IO更难，并且在程序的维护上比较痛苦。一般使用非阻塞IO编程时建议使用一些封装好的网络库比较容易编写。

下面是python实现的基于IO复用的非阻塞IO的netcat 。

```python
import errno
import fcntl
import os
import select
import socket
import sys

def setNonBlocking(fd):
    flags = fcntl.fcntl(fd, fcntl.F_GETFL)
    fcntl.fcntl(fd, fcntl.F_SETFL, flags | os.O_NONBLOCK)

def nonBlockingWrite(fd, data):
    try:
        nw = os.write(fd, data)
        return nw
    except OSError as e:
        if e.errno == errno.EWOULDBLOCK:
            return -1
def relay(sock:socket):
    socketEvents = select.POLLIN
    poll = select.poll()
    poll.register(sock, socketEvents)
    poll.register(sys.stdin, select.POLLIN)
    setNonBlocking(sock)
    # setNonBlocking(sys.stdin)
    # setNonBlocking(sys.stdout) #简化处理流程, 对于standardout用阻塞写
    done = False
    socketOutputBuffer = bytes()
    while not done:
        events = poll.poll(10000)  # 10 seconds
        for fileno, event in events:
            if event & select.POLLIN:
                if fileno == sock.fileno(): # if fd is socket fd
                    data = sock.recv(8192)
                    if data:
                        nw = sys.stdout.write(data)  # stdout does support non-blocking write, though
                    else:
                        done = True
                else:
                    assert fileno == sys.stdin.fileno() #if fd is stdin
                    data = os.read(fileno, 8192)
                    if data:
                        assert len(socketOutputBuffer) == 0 #不为空, 数据会乱序
                        nw = nonBlockingWrite(sock.fileno(), data)
                        if nw < len(data):
                            if nw < 0:
                                nw = 0
                            socketOutputBuffer = data[nw:]
                            socketEvents |= select.POLLOUT #为什么不一直关注POLLOUT事件, 因为会造成BUSY LOOP, 电平触发的缺点
                            poll.register(sock, socketEvents)
                            poll.unregister(sys.stdin) #这一行不是通用的写法, 使专门针对netcat 的 stdin做的, 在别的应用不一定需要去unregister这个事件
                    else:
                        sock.shutdown(socket.SHUT_WR)
                        poll.unregister(sys.stdin)
            if event & select.POLLOUT: # if sockfd is writable
                if fileno == sock.fileno():
                    assert len(socketOutputBuffer) > 0
                    nw = nonBlockingWrite(sock.fileno(), socketOutputBuffer)
                    if nw < len(socketOutputBuffer):
                        assert nw > 0
                        socketOutputBuffer = socketOutputBuffer[nw:]
                    else:
                        socketOutputBuffer = bytes()
                        socketEvents &= ~select.POLLOUT #一定要取消注册pollout事件, 不要会造成BUSYLOOP, 网络库一般会填这个坑
                        poll.register(sock, socketEvents)
                        poll.register(sys.stdin, select.POLLIN) # netcat 的特殊处理

def main(argv):
    if len(argv) < 3:
        binary = argv[0]
        print("Usage:\n  %s -l port\n  %s host port",argv[0], argv[0])
        print (sys.stdout.write)
        return
    port = int(argv[2])
    if argv[1] == "-l":
        # server
        server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        server_socket.bind(('', port))
        server_socket.listen(5)
        (client_socket, client_address) = server_socket.accept()
        server_socket.close()
        relay(client_socket)
    else:
        # client
        sock = socket.create_connection((argv[1], port))
        relay(sock)

if __name__ == "__main__":
    main(sys.argv)
```

## 6.4 Why non-blocking IO is must in IO multiplexing?


对于 前面的 [[#6.2 基于 IO 复用（阻塞IO）实现的 netcat]], 可以看到阻塞是因为我们没有检测其 `pollout` 事件, 所以是否可以用 *阻塞IO 每次先检查一下 可读再去读, 可写再去写* ? 
	**No, too naive!**


缘由如下
1. Example from Unix Network Programming
	Calling `accept(2)` after a listening socket is "ready for reading" could block, because *client could have disconnected in between* 2. [man page of select()](http://man7.org/linux/man-pages/man2/select.2.html) On Linux, **select**() may report a socket file descriptor as "ready   for reading", while nevertheless a subsequent read blocks.  This could for example happen when data has arrived but upon examination has the wrong checksum and is discarded.  There may be other circumstances in which a file descriptor is *spuriously* reported as ready.  Thus it may be safer to use **O_NONBLOCK** on sockets that should not block.


## 6.5 非阻塞IO中需要关注的问题


### 6.5.1 如何处理`非阻塞IO`中的 `short write` ?


一般来说在非阻塞编程中：

- 对于非阻塞的读，如果读数据不全，我们需要将数据缓存，等凑够一条完整消息再触发消息处理逻辑。
- 对于非阻塞的写，通常是网络库需要实现的功能，我们需要做的是告诉网络库我们需要发送多少数据，至于网络库内部如何处理事件我们在编程阶段不关心。



以发数据为例：

- 如果数据发送不完整，剩下的数据需要放置到一个发送的缓冲区中。 如果缓冲区非空，则我们不能对新数据`write`。因为这会造成数据乱序。只有等上一条消息全部发送成功后才可以对新消息进行发送。
	*方案一*：先尝试发送一次数据，如果数据发送不完整，将剩余数据存放在发送缓冲区（应用层的），然后注册`POLLOUT`事件用于处理剩余的数据发送。
	
  - 或者始终从缓冲区发送数据。
	*方案二*：将所有数据都存放在发送缓冲区，然后再去注册`POLLOUT`事件，**只**通过`POLLOUT`事件处理数据的发送。(现在网络库一般采用方案一)


- 向套接字中写数据，关注`POLLOUT`事件。
  - 当`POLLOUT`准备好时，开始从`发送缓冲区`（应用层）中取数据向`sock`中写
  - 当`发送换缓冲区`（应用层）为空时，停止关注`POLLOUT`事件。
	注意，如果忘记取消关注`POLLOUT`事件，则认为`sock`一直可写（LT模式持续通知我们有事件），而实际上我们并没有数据向`sock`写，会进入一种`busy loop`状态，大量空耗`cpu`过度占用资源。


如上 是一个标准的 `short write`在非阻塞IO的电平触发下的 处理办法


### 6.5.2 如何处理非阻塞IO中对方接受缓慢。


设想，
1. 本端在发送一个文件时，将文件加载到内存中然后通过网络向对端发送，而如果对方接受缓慢，那本端的发送方就要迁就对方从而缓慢发送，但是本端内存中缓存的数据就会持续占用在内存中。

2. 比如你的程序运行在一个 服务器机房 类似网关的位置, 然后它和内部是千兆网连接, 客户进来是一个10兆甚至更小的家用带宽,然后他问你通过代理向内部的服务器请求一个大的数据, 但客户带宽很小, 只能慢慢给他发, 就很容易造成这个内存使用生长很快

这些都是`非阻塞IO` 需要*从设计上考虑的问题*, 如果待发送的数据很多的话，那么一味的将文件读取到内存中。等待向对端发送显然是不明智的。因为这将会占用大量的内存资源。

​

**解决方案**:设定一个`最高/低水位(hight/low water mark)`，如果水池内的水位*高于最高水位则停止注水*，如果水位*低于最低水位则开始注水*。这样使得水位在 `hwm ~ lwm` 之间浮动，而水池出水率始终是最大值。


一个具体的例子就是 `非阻塞的echo`: `echo`读的时候会放到本地的缓冲区当中, 如果对方一直 *只发不收*
	1. 你拼命的往缓存区度,导致本地内存爆掉
	2. 如果你的 `写` 是 `阻塞`的话, 就影响了 `echo`上的其他的客户端, 
可行的 *解决方案如下*:
	1. 对于这个连接的发送缓冲区, 设置 `HWM`, 超过一定水位, 就不去读.(这对于简单的协议是可行的, 如 `echo`只是简单的转发一下数据, 不关协议内容, 但是对于一些复杂的协议, 比如 `RPC` , 他发过来是一个很小的请求, 如 `get 1 个 image` ,  但发回去的响应很大, 因为是一个 `image` 几十几百k的图片, 他不断的发请求, 发送缓冲区很快就堆积起来, 但是这交给网络库有不妥了, 应该交给 `RPC`那一层来做才对)
同理，我们可以参考高低水位的方式去设计。例如在向对端发送数据时，内存中缓冲的数据已经高出规定阈值时，我们可以考虑不去读对方的下一个请求直到本次发送完成，并且可以限制从本地读取待发送文件的速率。

但这终究不是一个完美的解决方案，对于接收端大量频繁的请求而言，我们不 read() 这些请求并不是一个好的解决方案，最好的方式是接收两方进行协议层面的商定，通过滑动窗口的思想告知接收端是否可以开启下一次的请求。这样方可避免由于接收方大量的数据请求而造成发送端发缓冲区数据的大量堆积（比如，接收端每次get image请求，发送端将对应image数据发送给接收端。对于接收端而言发送一次请求耗费的数据量很少，而发送发要回应的每个请求的数据量等很大，如果发送方一次接收到多个请求则这些应答数据将会占满发送发的缓冲区）。



### 6.5.3 Use LT（level trigger） or ET（edge trigger） ?

`select()` 与 `poll()` 都是 `LT` 模式。 如果有事件到达，或事件没有处理完，则它会一直通知，直到事件被处理。
`epoll()` 即 `edge-poll`, 但 它同时支持 `LT` 模式与 `ET` 模式。

>No up-to-date benchmarks show which is faster

能否结合两种模式的特点，分别在不同的场景使用不同的模式


对于`ET`模式而言， 从编码的角度, 更适用于`write`事件和`accept`事件（`accept`如果文件描述符用完，会陷入死循环中，因此使用模式更好）
`LT`模式 更适用于`read()`事件，它不会造成接收的饥饿，`ET`模式可能会造成数据接收不完整的情况。

可惜的是，*目前内核中使用同一种数据结构表示读和写事件，读写事件放在一个就绪列表中，在读出后才判断是读事件还是写事件*。因此，我们无法实现在不同的场景使用不同的模式。 值得注意的是，许多第三方网络库都使用的是LT模式，一般来说为了互相的兼容推荐使用LT模式。
