RTOS(Real Time Operation Sytem)在嵌入式中有广泛的应用. 相比于普通的逻辑开发(STM32, ESP32), 加入了RTOS, 可以最大化的增加单片机的运行效率, 不会导致一个任务"阻塞"式运行(1个while循环, 所有任务必须等待前面的任务完成, 才能进行下一个任务), 增强了系统的实时性.

# Kernel
## Kernel Implemention
## Kernel Object
FreeRTOS中的内核对象主要有消息队列(Message Queue), TCB(Task Control Block), 线程(Thread, CMSIS中称为任务), 
### 信号量(Semaphore)
#### 任务优先级反转

### 互斥量(Mutex)



# System Scheduler


# Inter Task communication

# System Usage
使用FreeRTOS的一般步骤是:
1) 明确任务需求与通信. 
2) 创建对应任务函数.
3) 开辟栈空间

# 基于CubeMX CMSISv2 的 FreeRTOS配置
如果在CubeMX使用ST官方的CMSISv2接口, 就是相当于对FreeRTOS做了一个二次封装, 目的就是为了做一个RTOS的兼容层, 无缝移植项目.

Good Article:[CubeMX使用FreeRTOS编程指南_cubemx freertos-CSDN博客](https://blog.csdn.net/qq_45396672/article/details/120877303)

## Reference API
具体提供的接口如下:
### Thread

| 函数                    | 功能               |     |
| --------------------- | ---------------- | --- |
| osThreadNew           | 创建新任务            |     |
| \*osThreadGetName     | 获取任务名称           |     |
| osThreadGetId         | 获取当前任务的控制块（TCB）  |     |
| osThreadGetState      | 获取当前任务的运行状态      |     |
| osThreadGetStackSize  | 获取任务的堆栈大小        |     |
| osThreadGetStackSpace | 获取任务剩余的堆栈大小      |     |
| osThreadSetPriority   | 设定任务优先级          |     |
| osThreadGetPriority   | 获取任务优先级          |     |
| osThreadYield         | 切换控制权给下一个任务      |     |
| osThreadSuspend       | 挂起任务             |     |
| osThreadResume        | 恢复任务（挂起多少次恢复多少次） |     |
| osThreadDetach        | 分离任务，方便任务结束进行回收  |     |
| osThreadJoin          | 等待指定的任务停止        |     |
| osThreadExit          | 停止当前任务           |     |
| osThreadTerminate     | 停止指定任务           |     |
| osThreadGetCount      | 获取激活的任务数量        |     |
| osThreadEnumerate     | 列举激活的任务          |     |
### Semaphore
### Message Queue
