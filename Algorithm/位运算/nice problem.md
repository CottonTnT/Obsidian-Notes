# 1 快速将一个数向$2^x$ 进行round
设被圆整的数为$num$, $round = 2^x$ 
```c
int down_rounded_num =  num & ~(round - 1) //向下圆整
int up_rounded_num = (num + round - 1) & ~(round - 1);//向上圆整
```


# 2 lowbit
>获得一个整数最低的二进制位的数值大小


```c
int lowbit(int x)
	return x & -x;
```

- 参考CSAPP的整数章节

