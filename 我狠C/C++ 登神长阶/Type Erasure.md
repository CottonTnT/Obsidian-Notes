`Type Erasure` 即类型擦除，why we need it ? 当我们*想代码具备多态性质*时,我们没办法保留对象本身的类型，而需要用一种通用的类型去使用他们，这时就需要擦除对象原有的类型


# 1 void*


C 语言中很多通用算法函数都会使用`void *`作为参数类型,如
```c
void qsort(void *base, size_t num, size_t size, int(*compare)(const void*, const void*));
```

