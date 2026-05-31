RTOS(Real Time Operation Sytem)在嵌入式中有广泛的应用. 相比于普通的逻辑开发(STM32, ESP32), 加入了RTOS, 可以最大化的增加单片机的运行效率, 不会导致一个任务"阻塞"式运行(1个while循环, 所有任务必须等待前面的任务完成, 才能进行下一个任务), 增强了系统的实时性.

# Kernel
## Kernel Implemention

## Kernel Object 
FreeRTOS中的内核对象主要有消息队列(Message Queue), TCB(Task Control Block), 线程(Thread, CMSIS中称为任务)
### 任务和任务管理
#### 任务状态

#### 任务切换
##### Q1: 中进行上下文切换时是如何保留上下文状态的？
实际上内核把每个状态下的任务都存储为链表/列表, 即有一下几个列表: 阻塞列表, 挂起列表, 延时列表 就绪列表等等. 当任务从XX状态->另一个状态, 那么就是在操作这几张表.

##### Q2: 如何记录一个任务此时的上下文信息?
##### 延时函数
延时函数分为相对延时(从调用延时函数开始计算指定延时时间), 以及绝对延时(即定时器)
freertos自带的相对延时函数是:
###### `vTaskDelay(const TickType_t xTicksToDelay)`

###### `vTaskDelayUntil( TickType_t * const pxPreviousWakeTime,const TickType_t xTimeIncrement );`
绝对延时

### 消息队列(Message Queue)
Reference:[(39 封私信 / 80 条消息) FreeRTOS源码探析之——消息队列 - 知乎](https://zhuanlan.zhihu.com/p/341506772)
消息队列被创建后实际上会开辟一块大小为(消息队列控制块 头+队列长度+每个数据结构的字节大小或者叫做消息空间大小)的内存, 后续管理队列就通过消息队列控制块进行管理.
#### Source Code
```C
QueueHandle_t xQueueGenericCreate( const UBaseType_t uxQueueLength,
								   const UBaseType_t uxItemSize,
								   const uint8_t ucQueueType )

{
	Queue_t * pxNewQueue = NULL;
	size_t xQueueSizeInBytes;
	uint8_t * pucQueueStorage;
	traceENTER_xQueueGenericCreate( uxQueueLength, uxItemSize,ucQueueType );

	if( ( uxQueueLength > ( UBaseType_t ) 0 ) &&
		/* Check for multiplication overflow. */
		( ( SIZE_MAX / uxQueueLength ) >= uxItemSize ) &&
		/* Check for addition overflow. */
		/* MISRA Ref 14.3.1 [Configuration dependent invariant] */
		/* More details at: https://github.com/FreeRTOS/FreeRTOS-Kernel/blob/main/MISRA.md#rule-143. */
		/* coverity[misra_c_2012_rule_14_3_violation] */
		( ( SIZE_MAX - sizeof( Queue_t ) ) >= ( size_t ) ( ( size_t ) uxQueueLength * ( size_t ) uxItemSize ) ) )

	{
		/* Allocate enough space to hold the maximum number of items that
		 * can be in the queue at any time.  It is valid for uxItemSize to be
		 * zero in the case the queue is used as a semaphore. */

		xQueueSizeInBytes = ( size_t ) ( ( size_t ) uxQueueLength * ( size_t ) uxItemSize );

		/* MISRA Ref 11.5.1 [Malloc memory assignment] */
		/* More details at: https://github.com/FreeRTOS/FreeRTOS-Kernel/blob/main/MISRA.md#rule-115 */
		/* coverity[misra_c_2012_rule_11_5_violation] */

		pxNewQueue = ( Queue_t * ) pvPortMalloc( sizeof( Queue_t ) + xQueueSizeInBytes );

		if( pxNewQueue != NULL ){
			/* Jump past the queue structure to find the location of the queue
			 * storage area. */

			pucQueueStorage = ( uint8_t * ) pxNewQueue;
			pucQueueStorage += sizeof( Queue_t );

			prvInitialiseNewQueue( uxQueueLength, uxItemSize, pucQueueStorage, ucQueueType, pxNewQueue );

		}
		else{
			traceQUEUE_CREATE_FAILED( ucQueueType );
			mtCOVERAGE_TEST_MARKER();
		}
	}
	else{
		configASSERT( pxNewQueue );
		mtCOVERAGE_TEST_MARKER();
	}
	traceRETURN_xQueueGenericCreate( pxNewQueue );
	return pxNewQueue;
}
```
消息队列的管理可分为读队列和写队列. 无论读还是写, 都可以分为
### 信号量(Semaphore)
信号量一般用于实现同步, 即中断和任务之间or任务和任务之间的资源/状态同步等.
#### 任务优先级反转
##### 解决方案
#### Q1: 全局变量和信号量的本质区别?
裸机开发经常会用到全局变量来实现每个任务间的数据交流通信, 同步等, 但是即使使用`volatile`也会导致数据会发生更改的情况导致判断不成立.
**单核的情况下**
```c
if (global == 1) ...

// 对应汇编程序
mov eax, [_global_addr]
cmp eax, 1
jz xxxx
```
在进入cmp指令前, 进入到中断中了, 中断更改状态值, 那么此时if判断就进入中间态了, 导致还是用的之前变量存入寄存器中的数据, 造成中断更新失败的结果.
##### 解决方案
在裸机开发中, 常用的解决方案就是添加中断临界区(`taskCRITICAL()`or `__disabled_irq()`),等等. 即:
```C
while(1){
	// 判断标志位前进入临界区, 防止中断更改状态
	__disable_irq();
	if(flag==1){
		// 相关逻辑
	}
	__enable_irq();
}
```
### 互斥量(Mutex)
互斥量实际上是加了锁的信号量. 


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
|                       |                  |     |

### Semaphore
### Message Queue
