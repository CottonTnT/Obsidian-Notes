>`tcp`自连接：`tcp`连接两端使用了同一地址进行连接，而tcp并没有报错并且连接成功。即`localhost:x --> localhost:x` 


					# 1 为何
如果本地`tcp`程序本地通信，且客户端先于服务端启动，那么有极大可能会产生一种自连接现象。


而产生这种现象的原因源自于tcp内部的一种特性，而处理自连接的方法也很简单。

下面使用一个示例来复现这种自连接的情况
```python
#!/usr/bin/python
import errno
import socket
import sys
import time

if len(sys.argv) < 2:
    print "Usage: %s port" % sys.argv[0]
    print "port should in net.ipv4.ip_local_port_range"
else:
    port = int(sys.argv[1])	# 命令行输入端口号
    for i in range(65536):
        try:
            sock = socket.create_connection(('localhost', port)) # 尝试对端口号发起连接
            print "connected", sock.getsockname(), sock.getpeername() # 连接成功，打印本机地址和对端地址
            time.sleep(60*60)	# 休眠一小时
        except socket.error, e:
            if e.errno != errno.ECONNREFUSED:
                break
```


![[Pasted image 20241231160027.png]]

`tcp`在发起连接时，会从 `ip_local_port_range` 中选取一个临时端口号，选定端口号后再向服务器的端口发送一个`syn`的请求。如果指定端口在侦听，那么这个随机端口就不会选取到这个端口（连接到22端口，不会发生自连接），*如果指定端口没有在侦听，那么就有可能发生自连接*。

究其原因为，客户端不断地尝试连接 `31000` 端口，每次都会随机分配一个临时端口（临时端口分配规则是，给上一次分配的端口号+1）。在多次尝试后，会出现分配的临时端口和目的端口一样，此时就会出现自连接情况了。 详细步骤大致为：

客户端在选取了一个临时端口 x 后，该端口就被加入内核中，客户端在发送syn报文到31000端口（x to 31000），但因为 31000 没有在监听，因此 连接失败 。
在此尝试连接，重新选取临时端口号 x + 1 ，再次尝试连接 … 失败。
第n次尝试时，选取的临时端口号刚好是 31000 端口，然后向 31000 端口发起syn报文，此时在内核中检测到了刚加入的端口，然后连接成功。


- 解决自连接问题：

解决方案也很简单，在连接成功后检测一下是否为自连接，如果是自连接断开即可。

参考 ： [recipes/tpc/TcpStream.cc]([recipes/python/self-connect.py at master · CottonTnT/recipes](https://github.com/CottonTnT/recipes/blob/master/python/self-connect.py))	 网络库封装的连接函数：

// 检测是否为自连接：判断本机地址是否等于对端地址
```cpp
bool isSelfConnection(const Socket& sock)
{
  return sock.getLocalAddr() == sock.getPeerAddr();
}

// 连接的处理：是自连接则断开连接
TcpStreamPtr TcpStream::connectInternal(const InetAddress& serverAddr, const InetAddress* localAddr)
{
  TcpStreamPtr stream;
  Socket sock(Socket::createTCP());
  if (localAddr)
  {
    sock.bindOrDie(*localAddr);
  }
  if (sock.connect(serverAddr) == 0 && !isSelfConnection(sock)) // 连接成功 且 不是自连接
  {
    // FIXME: do poll(POLLOUT) to check errors
    stream.reset(new TcpStream(std::move(sock))); // 设置stream，将连接移动到strean中
  }
  return stream;  // 如果不设置stream等于放弃sock的连接（sock出作用域后会自动释放，连接自动断开）
}

```

