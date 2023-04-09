### 简介







### 运算规则
|  法则    | 释义     |
|:-----|:-----|
|  分配律    | ![[Pasted image 20230409093836.png]]     |
|    交换律  |      ![[Pasted image 20230409093728.png]] |
|    吸收率  |      ![[Pasted image 20230409093938.png]]
|    结合律  |      ![[Pasted image 20230409093807.png]]
|    摩根律 (反演律)  |      ![[Pasted image 20230409094015.png]]
|      |      |





### 妙用


#### 1.lowbit (x)

用于求 x 的比特位的最有一位1
```C
int lowbit(int x) return x & -x;
```

*证明*
```Pro
因    x + ~x = -1, x + -x = 0;
所    -x = ~x + 1 
故 x & -x = x & (~x + 1)
```