# 属性`__attribute__`
是 GNU C 编译器提供的一种机制，用于为函数、变量或类型指定特殊属性，从而实现优化或特定的编译检查。
## 对齐属性
```c
struct C{
	char c;
    int x;
};__ALIGN(4)__;
```