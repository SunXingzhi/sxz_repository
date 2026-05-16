RTOS(Real Time Operation Sytem)在嵌入式中有广泛的应用. 相比于普通的逻辑开发(STM32, ESP32), 加入了RTOS, 可以最大化的增加单片机的运行效率, 不会导致一个任务"阻塞"式运行(1个while循环, 所有任务必须等待前面的任务完成, 才能进行下一个任务), 增强了系统的实时性.

# Kernel Implemention


# System Scheduler


# Inter Task communication

# System Usage
使用FreeRTOS的一般步骤是:
1) 明确任务需求与通信. 
2) 创建对应任务函数.
3) 开辟栈空间

# 基于CubeMX CMSISv2 的 FreeRTOS配置
如果在CubeMX使用ST官方的CMSISv2接口, 就是相当于对FreeRTOS做了一个二次封装, 目的就是为了做一个RTOS的兼容层, 无缝移植项目.
