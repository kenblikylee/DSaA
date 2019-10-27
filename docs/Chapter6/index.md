# 优先队列（堆）

## 为什么需要优先队列？

队列是一种先进先出的表ADT，正常来说，先入队的元素，会先出队，意味没有那个元素是特殊的，拥有“插队”的优先权。这种平等，并不试用所有场景。有时，我们希望队列中某类元素拥有比其他元素更高的优先级，以便能提前得到处理。因此，我们需要有一种新的队列来满足这样的应用，这样的队列叫做“优先队列（priority queue）”。

## 优先队列模型

优先队列允许至少两种操作：Insert(插入) ，以及 DeleteMin(删除最小者)。Insert 操作等价于 Enqueue(入队)，而 DeleteMin 则是队列中 Dequeue(出队) 在优先队列中的等价操作。DeleteMin 函数也变更它的输入。

## 简单实现

### 单链表

在表头以$O(1)$执行插入操作，并遍历该链表以删除最小元，这又需要$O(N)$时间。另一种做法是，始终让表保持排序状态；这使得插入代价高昂（$O(N)$）而DeleteMin花费低廉（$O(1)$）。基于DeleteMin的操作次数从不多于插入操作次数的事实，前者也许是更好的办法。

### 二叉查找树

使用二叉查找树，Insert 和 DeleteMin 这两种操作的平均运行时间都是$O(log N)$。

## 二叉堆（binary heap）

实现优先队列更加普遍的方法是`二叉堆`，以至于当`堆`(heap)这个词不加修饰地使用时一般都是指该数据结构(优先队列)的这种实现。因此，我们单独说堆时，就是指二叉堆。同二叉查找树一样，堆也有两个性质，即`结构性`和`堆序性`。正如AVL树一样，对堆的一次操作可能破坏这两个性质的一个，因此，堆的操作必须要到堆的所有性质都被满足时才能终止。事实上这并不难做到。

### 结构性质

堆是一棵被完全填满的二叉树，有可能的例外是在底层，底层上的元素从左到右填入。这样的树称为完全二叉树（complete binary tree）。因为完全二叉树很有规律，所以它可以用一个数组表示而不需要指针。对于数组任意位置$i$上的元素，其左儿子在位置$2i$上，右儿子在左儿子后的单元$2i+1$上，它的父亲则在位置$\lfloor i/2 \rfloor$上。因此，不仅指针这里不需要，而且遍历该树所需要的操作也极简单，在大部分计算机上运行很可能非常快。

一个堆数据结构由一个数组，一个代表最大值的整数以及当前的堆大小组成。

优先队列声明：

``` c
#ifndef _BinHeap_H

struct HeapStruct;
typedef struct HeapStruct *PriorityQueue;

PriorityQueue Initialize(int MaxElements);
void Destroy(PriorityQueue H);
void MakeEmpty(PriorityQueue H);
void Insert(ElementType X, PriorityQueue H);
ElementType DeleteMin(PriorityQueue H);
ElementType FindMin(PriorityQueue H);
int IsEmpty(PriorityQueue H);
int IsFull(PriorityQueue H);

#endif

struct HeapStruct
{
    int Capacity;
    int Size;
    ElementType *Elements;
}
```

### 堆序性质

使操作被快速执行的性质是`堆序`(heap order)性。由于我们想要快速地找出最小元，因此，最小元应该在根上。如果我们考虑任意子树也应该是一个堆，那么任意节点就应该小于它的所有后裔。

``` c
PriorityQueue
Initialize(int MaxElements)
{
    PriorityQueue H;

    if (MaxElements < MinPQSize)
        Error("Priority queue size is too small");
    
    H = malloc(sizeof(struct HeapStruct));
    if (H == NULL)
        FatalError("Out of space!!!");
    
    H->Elements = malloc((MaxElements + 1)
                         * sizeof(ElementType));
    
    if (H->Elements == NULL)
        FatalError("Out of space!!!");

    H->Capacity =  MaxElements;
    H->Size = 0;
    H->Elements[0] = MinData;

    return H;
}
```

根据堆序性质，最小元总是可以在根处找到。因此，我们以常数时间完成附加运算FinMin。

### 基本操作

#### Insert (插入)

``` c
void
Insert(ElementType X, PriorityQueue H)
{
    int i;

    if (IsFull(H))
    {
        Error("Priority queue is full");
        return;
    }
    for (i = ++H->Size; H->Elements[i / 2] > X; i /= 2)
        H->Elements[i] = H->Elements[i / 2];
    H->Elements[i] = X;
}
```

#### DeleteMin (删除最小元)

``` c
ElementType
DeleteMin(PriorityQueue H)
{
    int i, Child;
    ElementType MinElement, LastElement;

    if (IsEmpty(H))
    {
        Error("Priority queue is empty");
        return H->Elements[0];
    }
    MinElement = H->Elements[1];
    LastElement = H->Elements[H->Size--];

    for(i = 1; i * 2 <= H->Size; i = Child)
    {
        Child = i * 2;
        if (Child != H->size && H->Elements[Child + 1]
                              < H->Elements[Child])
            Child++;

        if (LastElement > H->Elements[ Child ])
            H->Elements[i] = H->Elements[Child];
        else
            break;
    }
    H->Elements[i] = LastElement;
    return MinElement;
}
```

## 优先队列的应用

- 操作系统设计：进程调度
- 图论算法
- 选择问题：从N个元素中找出第k个最大的元素
- 事件模拟
