# 步进电机S型/梯形算法优化

 `fastInvSqrt` 是游戏界著名的 **Quake III Arena 快速倒数平方根算法**：

```c
float fastInvSqrt(const float x) {
    const float xhalf = x * 0.5f;
    union { float x; int32_t i; } u;
    u.x = x;
    u.i = 0x5f3759df - (u.i >> 1);  // 魔法数字：牛顿迭代的初始猜测
    return u.x * (1.5f - xhalf * u.x * u.x);  // 一次牛顿迭代精化
}
```

### 浮点数的其他存储方式
