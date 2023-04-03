## 1. 代码求解

```c
for(int i = 2, j = 0; i <= m; i++) //字符串从数组1下标开始
{
    while(j && p[i] != p[j+1]) j = next[j];

    if(p[i] == p[j+1]) j++;

    next[i] = j;
}
```


### 2. 关于 next 数组的周期性质

##### 引理

```thoery
  对于某一字符串 S[1 ~ N]，在它众多的next[i]的“候选项”中，如果存在某一个next[i]，使得: 
i % (i−next[i])==0，那么 S[1 ~ (i−next[i])] 可以为 S[1 ~ i] 的循环元而 i/(i−next[i]) 即是它的循环次数 K。
```
