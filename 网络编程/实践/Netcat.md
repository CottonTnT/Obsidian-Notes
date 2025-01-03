> 通过本节学习如何正确使用`tcp`


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