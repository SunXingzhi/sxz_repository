# 大端模式和小端模式

大端模式是最高地址的有效字节(MSB)放在内存的低地址, 后续字节按重要性依次存放在高地址上, 例如`0x12345678`在地址中从低到高的字节数据依次为`0x12`, `0x34`, `0x56`, `0x78`.小端同理

## example

写一个程序验证当前机器内存大端还是小端存储.

```c
#include <stdio.h>
#include <stdbool.h>
#include <stdint.h>

typedef enum {
    BIG_ENDIAN  = 0,
    LITTLE_ENDIAN,
    UNKNOWN    
} endian_type_t;

endian_type_t xGet_memory_endian(void)
{
    // 使用 uint32_t 保证大小明确
    uint32_t test_num = 0x01020304;
    unsigned char *p = (unsigned char*)&test_num;

    if (p[0] == 0x04) {
        return LITTLE_ENDIAN;   // 低地址存储低字节
    } else if (p[0] == 0x01) {
        return BIG_ENDIAN;      // 低地址存储高字节
    } else {
        // 理论上不会发生，但可添加异常处理
        return UNKNOWN;
    }
}

int main(void)
{
    int32_t a = 0x12345678;
    endian_type_t result = xGet_memory_endian();

    // 友好输出
    const char* endian_str;
    switch (result) {
        case LITTLE_ENDIAN: endian_str = "Little Endian"; break;
        case BIG_ENDIAN:    endian_str = "Big Endian";    break;
        default:            endian_str = "Unknown";       break;
    }
    printf("The memory endian type is: %s\n", endian_str);

    return 0;
}
```

