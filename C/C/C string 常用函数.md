```c
#include <string>
```

- 1. `strrchr`

```c
char *strrchr(const char *str, int c);

//strrchr() 函数从字符串的末尾开始向前搜索，直到找到指定的字符或搜索完整个字符串。如果找到字符，它将返回一个指向该字符的指针，否则返回 NULL。
```


- 2. `strcpy`
```c
char *strcpy(char *dest, const char *src);

// 需要注意的是如果目标数组 dest 不够大，而源字符串的长度又太长，可能会造成缓冲溢出的情况,返回一个指向最终的目标字符串 dest 的指针
```


```c
char *strncpy(char *dest, const char *src, size_t n);

//最多复制 n 个字符。当 src 的长度小于 n 时，dest 的剩余部分将用空字节填充。
```


