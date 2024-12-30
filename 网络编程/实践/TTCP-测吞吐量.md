![[Pasted image 20241215142935.png]]
> `TTCP(test ttcp)`is a utility for measuring netword throughput
# 1 why ttcp 
1. 使用了基本的sockets APIs：socket，listen， bind， accept，connect，read/recv，write/send，shutdown，close 等等
2. 协议带有格式，不只是字节流，相较于echo具有tcp分包处理等
3. ttcp 本身是由 tcp/ip 实现的程序，具有一些典型的行为。可以阅读其代码学习它的一些优秀实现
4. 协议简单，可以由多种语言实现，针对测试结果对比个语言实现的runtime开销
5. 无并发连接，client与server之间只有一个tcp socket


相比 不接受ack的测试方法，肯定受网络延迟影效果大，但是很多服务就是要接受ack才有下一步
