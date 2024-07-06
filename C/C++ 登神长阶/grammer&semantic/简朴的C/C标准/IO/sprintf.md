> 发送格式化输出到 `str` 指向的字符串
## 函数声明
```cpp
int sprintf(char * str, const char * format, ...)
```

- **str** 指向字符数组的指针,存储C字符串
- **format** 一个格式化C字符串，format 标签可被随后的附加参数中指定的值替换，并按需求进行格式化。format 标签属性是 **%\[flags\]\[width\]\[.precision\]\[length\]specifier**


#### 实例
```cpp
#include <stdio.h>  
#include <math.h>  
  
int main()  
{  
   char str[80];  
  
   sprintf(str, "Pi 的值 = %f", M_PI);  
   puts(str);  //Pi 的值 = 3.141593
     
   return(0);  
}
```

```cpp
Pi 的值 = 3.141593
```
