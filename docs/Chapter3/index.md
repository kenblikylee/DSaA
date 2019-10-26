# 表，栈和队列

## 抽象数据类型（ADT）

抽象数据类型是一些操作的集合，它是数学的抽象，不涉及如何实现操作的集合。对于每种ADT并不存在什么法则来告诉我们必须要有哪些操作，这只是一个设计决策。

## 表ADT

### 表的简单数组实现

操作时间复杂度:

- PrintList: $O(N)$
- Find: $O(N)$
- FindKth: $O(1)$
- Insert: $O(N)$
- Delete: $O(N)$

当插入和删除$N$个元素时，需要花费$O(N^2)$的时间，运行慢且表的大小必须事先已知，所以简单数组一般不用来实现表这种数据结构。

### 链表实现

为了避免插入和删除的线性开销，我们需要允许表可以不连续存储，否则表的部分或全部需要整体移动。因此，出现链表(linked list)这种数据结构。

链表由一系列不必在内存中相连的结构组成。每一个结构均含有表元素和指向包含该元素后继元的结构的指针。我们称之为`Next`指针。最后一个单元的`Next`指针指向`Null`。

#### 双链表

#### 循环链表

## 栈ADT

栈（stack）是限制插入和删除只能在一个位置上进行的表，该位置是表的末端，叫做栈的顶（top）。对栈的基本操作有`Push`(进栈)和`Pop`(出栈)，前者相当于插入，后者则是删除最后插入的元素。最后插入的元素可以通过使用`Top`操作在执行`Pop`之前读取。

由于栈是一个表，因此任何实现表的方法都能实现栈。通常使用数组是一个较为简便的方法。

## 队列ADT

和栈一样，队列（queue）也是表。不同的是，使用队列时插入在一端进行而删除则在另一端进行。

队列的基本操作是`Enqueue` (入队)，它是在表的末端（rear 队尾）插入一个元素，还有 `Dequeue` （出队），它是删除（或返回）在表的开头（front 对头）的元素。

## 链表的 C 语言实现

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
