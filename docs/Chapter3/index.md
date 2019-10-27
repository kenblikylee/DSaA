# 表，栈和队列

表，栈和队列是计算机科学中最简单和最基本的三种底层数据结构。事实上，每一个有意义的程序都将明晰地至少使用一种这样的数据结构，而栈则在程序中总是要间接地用到，不管你在程序中是否做了声明。

## 抽象数据类型（ADT）

在计算机软件编程中，我们会接触到诸如整型，浮点型，字符型，布尔型等基本数据类型，也有一些更为复杂的复合数据类型，如数组，字典（散列表），元组等。如果我们抛开这些数据类型具体实现，进一步抽象，给出更一般的定义，即每一种数据类型实际是一些特定`操作`的集合。我们称这些`操作的集合`为`抽象数据类型`（`abstract data type, ADT`）。ADT 是数学意义上的抽象，它不约束各个操作的具体实现，对于每种 ADT 并不存在什么法则来告诉我们必须要有哪些操作，这只是一个设计决策。

## 表ADT

表是一种形如 $A_1, A_2, A_3, \ldots, A_N$ 的数据结构。我们说这个表的大小是 $N$ 。我们称大小为$0$的表为空表(empty list)。对于除空表以外的任何表，我们说$A_{i+1}(i<N)$后继$A_i$（或继$A_i$之后）并称$A_{i-1}(i>1)$前驱$A_i$。定义表ADT的操作集合通常包含：

- PrintList: 打印表
- MakeEmpty: 返回空表
- Find: 返回关键字(key)首次出现的位置
- Insert: 从表的指定位置插入关键字(key)
- Delelte: 从表的指定位置删除关键字(key)
- FindKth: 返回指定位置上的元素

### 简单数组实现

我们最容易想到实现表的方法就是数组。使用数组实现表的各项操作，显而易见的时间复杂度是:

- PrintList: $O(N)$
- Find: $O(N)$
- FindKth: $O(1)$
- Insert: $O(N)$
- Delete: $O(N)$

我们不难发现，当插入和删除$N$个元素时，需要花费$O(N^2)$的时间，运行慢且表的大小必须事先已知。因此当需要具有插入和删除操作时，通常不使用简单数组来实现。

### 链表实现

为了避免插入和删除的线性开销，我们需要允许表可以不连续存储，否则表的部分或全部需要整体移动。因此，这种情况下更好的实现方式是链表(linked list)。

链表由一系列不必在内存中相连的结构组成。每一个结构均含有表元素和指向包含该元素后继元的结构的指针。我们称之为`Next`指针。最后一个单元的`Next`指针指向`Null`。链表分为：单链表，双链表，循环链表。

链表的 C 语言实现：

``` c
#include <stdlib.h>

struct Node
{
    int Element;
    Node* Next;
};

int
IsEmpty(Node* L)
{
    return L->Next == NULL;
}

int
IsLast(Node* P, Node* L)
{
    return P->Next == NULL;
}

Node*
Find(int x, Node* L)
{
    Node* P;
    P = L->Next;
    while(P != NULL && P->Element != x)
        P = P->Next;
    return P;
}

Node*
FindPrevious(int x, Node* L)
{
    Node* P;
    
    P = L;
    while(P->Next != NULL && P->Next->Element != x)
        P = P->Next;
    
    return P;
}

void
Delete(int x, Node* L)
{
    Node* P;
    Node* TmpCell;
    
    P = FindPrevious(x, L);
    
    if (!IsLast(P, L)) {
        TmpCell = P->Next;
        P->Next = TmpCell->Next;
        free(TmpCell);
    }
}

void
Insert(int x, Node* L, Node* P)
{
    Node* TmpCell;
    
    TmpCell = (Node*)malloc(sizeof(struct Node));
    
    if (TmpCell != NULL)
        printf("Out of space!!!");
    
    TmpCell->Element = x;
    TmpCell->Next = P->Next;
    P->Next = TmpCell;
}

void
DeleteList(Node* L)
{
    Node* P;
    Node* Tmp;
    P = L->Next;
    L->Next = NULL;
    while(P != NULL)
    {
        Tmp = P->Next;
        free(P);
        P = Tmp;
    }
}
```

## 栈ADT

栈（stack）是限制插入和删除只能在一个位置上进行的表，该位置是表的末端，叫做栈的顶（top）。对栈的基本操作有`Push`(进栈)和`Pop`(出栈)，前者相当于插入，后者则是删除最后插入的元素。最后插入的元素可以通过使用`Top`操作在执行`Pop`之前读取。

由于栈是一个表，因此任何实现表的方法都能实现栈。通常使用数组是一个较为简便的方法。

## 队列ADT

和栈一样，队列（queue）也是表。不同的是，使用队列时插入在一端进行而删除则在另一端进行。

队列的基本操作是`Enqueue` (入队)，它是在表的末端（rear 队尾）插入一个元素，还有 `Dequeue` （出队），它是删除（或返回）在表的开头（front 对头）的元素。
