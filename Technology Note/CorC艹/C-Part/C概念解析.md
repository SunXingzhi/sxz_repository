# const 修饰符
## const 修饰指针
1. const放在\*的左边，修饰的是指针指向的内容，保证指针指向的内容不能通过指针来改变。
（但是指针变量本身的内容可变）
```C
const int *p = &n;        n里面的值不能被改变
*p = 20;        //会报错
```
2. const放在\*的右边，修饰的是指针变量本身，保证了指针变量的内容不能修改。
```C
int *const p = &n;        指向n的行为不能被改变
p = &m;        //会报错
```
## 与内存的关系
我们可以知道的是，被const修饰的变量，不一定会存放在`.constdata`数据段中（在MCU中一般是只读数据段,`.rodata`。这跟变量的类型，const 作用对象，以及变量的定义区有关。

# 链表（Linked List)
[数据结构——单向循环链表和双链表与双向循环链表-ITADN社区](https://itadn.com/i0_39026476340/2935189)
## 单向循环链表
### 头插
### 尾插
### 头删
### 尾删
### 遍历
### eg: 约瑟夫环
将 n 个个体排列成一个环形结构，从某一特定位置开始依次报数 1, 2, …, m，当计数达到 m 时，对应的个体将被移出环外。随后，从被移除个体的下一位继续进行相同的操作，直至所有个体均被移出环外，最终形成一个新的出圈顺序序列。

例如，当 n=8、m=4 时，若初始计数从第一个位置开始，则最终生成的出圈顺序为 4, 8, 5, 2, 1, 3, 7, 6。
## 双向循环链表

双向循环链表表示链表可以通过任意一个节点向前previous，向后next开始遍历，同时朝着一个方向遍历一定可以到达任意一个节点（因为是一个环）。在双向链表中的一个标志现象是采用了哨兵节点，我们可以看FreeRTOS对于双向链表的实现：
```c
// List Item data structure
typedef struct xLIST_ITEM
{
	TickType_t xItemValue;
	struct xLIST_ITEM* pxNext;
	struct xLIST_ITEM* pxPrevious;
	void* pvOwner;
	void* pvContainer;
	
} ListItem_t;

// Mini List node data structure
typedef struct xMINI_LIST_ITEM
{
	TickType_t xItemValue;			// Auxiliary value
	struct xLIST_ITEM*	pxNext;		// the next list node
	struct xLIST_ITEM*	pxPrevious;	// the previous list node
} MiniListItem_t;

// List's root node data structure
typedef struct xLIST
{
	UBaseType_t	uxNumberOfItems;	// List node counter
	ListItem_t*	pxIndex;		// List node index pointer
	MiniListItem_t	xListEnd;		// the last node of the list
} List_t;
```
其中List_t的`xListEnd`就是哨兵节点,在整个环形链表中可看作是(0,0)的参考点.向前一个元素可以看成是**双向非循环链表**的尾节点(`tail node`),而向后就是 `head node`. 这么做的好处是可以按照一个方向到达任意一个节点.
还有一个需要理解的问题: `pxIndex`是干嘛的? 这里就要区分一下我们说的逻辑队列和物理队列的概念了.物理队列就是说的是通过`xListEnd`可以获取到的队尾/队首. 不过在实际的FreeRTOS应用中,使用的是**逻辑队列**,即通过`pxIndex`来实现. 区分于物理队列满足一个顺序(通过`xItemValue`大/小进行排列), 逻辑队列是按照`pxIndex->next`为逻辑队首,`previous`为逻辑队尾实现的. 我们插入一个新节点(不是升序插入),只需要加入到`pxIndex-previous`,然后更新链表逻辑即可, 因为FreeRTOS中`pxIndex->previous`就是代表的队尾,与`pxList`概念一致.