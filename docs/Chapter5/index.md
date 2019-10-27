# 散列

散列表的实现常常叫做散列（hashing）。散列是一种用于以常数平均时间执行插入，删除和查找的技术。但是，那些需要元素间任何排序信息的操作将不会得到有效的支持。因此，诸如 FindMin，FindMax 以及以线性时间按排序顺序将整个表进行打印的操作都是散列所不支持的。

## 一般想法

理想的散列表数据结构只不过是一个包含关键字（key）的具有固定大小的数组。典型情况下，一个关键字就是一个带有相关值（例如工资信息）的字符串。我们把表的大小记作`Table-Size`，并将其理解为散列数据结构的一部分而不仅仅是浮动于全局的某个变量。通常的习惯是让表从`0`到`Table-Size - 1`变化。

每个关键字被映射到从`0`到`Table-Size - 1`这个范围中的某个数，并且被放到适当的单元中。这个映射就叫做`散列函数`（`hash function`）。理想情况下它应该运算`简单`并且应该保证任何两个不同的关键字映射到不同的单元。不过，这是不可能的，因为单元的数目是有限的，而关键字实际上是用不完的。因此，我们寻找一个散列函数，该函数要在单元之间`均匀`地分配关键字。这就是散列的基本想法。剩下的问题则是选择一个函数，决定当两个关键字散列到同一个值的时候（称为冲突`collision`）应该做什么以及如何确定散列表的大小。

## 散列函数

### 输入整数关键字

如果输入的关键字是整数，则一般合理的方法就是直接返回`“Key mod TableSize”`（关键字对表大小取模）的结果，除非`Key`碰巧具有某些不理想的性质。例如，若表的大小是`10`，而关键字都以`0`为个位，这意味所有关键字取模运算的结果都是`0`（都能被10整除）。这种情况，好的办法通常是保证表的大小是素数（也叫质数，只能被1和自身整除）。当输入的关键字是随机的整数时，散列函数不仅算起来`简单`而且关键字的分配也很`均匀`。

### 输入字符串关键字 

通常，关键字是字符串；在这种情形下，散列函数需要仔细地选择。

一种方法是把字符串中字符的`ASCII`码值加起来。以下该方法的C语言实现。

``` c
typedef unsigned int Index;

Index
Hash(const char *Key, int TableSize)
{
    unsigned int HashVal = 0;
    while(*Key != '\0')
        HashVal += *Key++;
    return HashVal % TableSize;
}
```

这种方法实现起来简单而且能很快地计算出答案。不过，如果表很大，则函数将不会很好地分配关键字。例如，设$TableSize = 10007$（10007是素数），并设所有的关键字至多8个字符长。由于`char`型量的值最多是`127`，因此散列函数只能取在0和1016之间，其中$1016=127\times8$。这显然不是一种均匀分配。

第二种方法。假设`Key`至少有两个字符外加`NULL`结束符。值`27`表示英文字母表的字母个数外加一个空格，而$729=27^2$。该函数只考查前三个字符，假如它们是随机的，而表的大小像前面那样还是`10007`，那么我们就会得到一个合理的均衡分配。

``` c
Index
Hash(const char *Key, int TableSize)
{
    return (Key[0] + 27*Key[1] + 729*Key[2]) % TableSize;
}
```

可是不巧的是，英文并不是随机的。虽然3个字符（忽略空格）有$26^3 = 17576$种可能组合，但查验词汇量足够大的联机词典却揭示：3个字母的不同组合数实际只有`2851`。即使这些组合没有冲突，也不过只有表的`28%`被真正散列到。因此，虽然很容易计算，但是当散列表足够大的时候这个函数还是不合适的。

一个更好的散列函数。这个散列函数涉及到关键字中的所有字符，并且一般可以分布得很好。计算公式如下：

$$
\sum_{i=0}^{KeySize-1} Key[KeySize-1-i]\cdot32^i
$$

它根据`Horner`法则计算一个（32的）多项式。例如，计算$h_k=k_1 + 27k_2 + 27^2k_3$的另一种方式是借助于公式$h_k=(k_3 \times 27 + k_2)\times 27 + k_1$进行。`Horner`法则将其扩展到用于$n$次多项式。

``` c
Index
Hash(const char *Key, int TableSize)
{
    unsigned int HashVal = 0;

    while(*Key != '\0')                     /* 1 */
        HashVal = (HashVal << 5) + *Key++;  /* 2 */
    return HashVal % TableSize;             /* 3 */
}
```

这里之所以用 32 替代27，是因为用32作乘法不是真的去乘，而是移动二进制的5位。为了运算更快，程序第2行的加法可以用按位异或来代替。虽然就表的分布而言未必是最好的，但确实具有及其简单的优点。如果关键字特别长，那么该散列函数计算起来将会花费过多的时间，不仅如此，前面的字符还会左移出最终的结果。这种情况，通常的做法是不使用所有字符。此时关键字的长度和性质将影响选择。

## 冲突解决

解决了关键字均匀映射的问题，剩下的主要编程细节是解决冲突的消除问题。如果当一个元素被插入时另一个元素已经存在（散列值相同），那么就产生了冲突，这种冲突需要消除。解决这种冲突的方法有几种。最简单的两种是：分离链接法和开放定址法。

### 分离链接法（separate chaining）

分离链接法是将散列到同一个值的所有元素保留到一个表中。为了方便起见，这些表都有表头，实现方法与`表ADT`相同。如果空间很紧，则更可取的方法是避免使用这些表头。

类型声明：

``` c
#ifndef _HashSep_H

struct ListNode;
typedef struct ListNode *Position;
struct HashTbl;
typedef struct HashTbl *HashTable;

HashTable InitializeTable(int TableSize);
void DestroyTable(HashTable H);

ElementType Retrieve(Position P);

#endif

struct ListNode
{
    ElementType Element;
    Position Next;
};

typedef Position List;

struct HashTbl
{
    int TableSize;
    List *TheLists;
};
```

初始化函数：

``` c
HashTable
InitializeTable(int TableSize)
{
    HashTable H;
    int i;

    if (TableSize < MinTableSize)
    {
        Error("Table size too small");
        return NULL;
    }

    H = malloc(sizeof(struct HashTbl));
    if (H == NULL)
        FatalError("out of space!!!");

    H->TableSize = NextPrime(TableSize);                /* 1 设置素数大小 */
    H->TheLists = malloc(sizeof(List) * H->TableSize);  /* 2 */
    if (H->TheLists == NULL)
        FatalError("Out of space!!!");
    
    /**
      * 分配链表表头
      * 
      * 给每一个表设置一个表头，并将 Next 指向 NULL。如果不用表头，以下代码可省略。
      */
    for(i = 0; i < H->TableSize; i++)
    {
        H->TheLists[i] = malloc(sizeof(struct ListNode)); /* 3 */
        if (H->TheLists[i] == NULL)
            FatalError("Out of space!!!");
        else
            H->TheLists[i]->Next = NULL;
    }

    return H;
}
```

以上程序低效之处是标记为3处`malloc`执行了`H->TableSize`次。这可以通过循环开始之前调用一次`malloc`操作：

``` c
H->TheLists = malloc(sizeof(struct ListNode) * H->TableSize);
```

Find 操作：

``` c
Position
Find(ElementType Key, HashTable H)
{
    Position P;
    List L;

    L = H->TheLists[ Hash(Key, H->TableSize) ];
    P = L->Next;
    while(P != NULL && P->Element != Key)   /* ElementType 为 int时比较。字符串比较使用 `strcmp` */
        P = P->Next;

    return P;
}
```

> 注意，判断 `P->Element != Key`，这里适用于整数。字符串比较用 `strcmp` 替换。

Insert 操作：

``` c
void
Insert(ElementType Key, HashTable H)
{
    Position Pos, NewCell;
    List L;

    Pos = Find(Key, H);
    if (Pos == NULL)
    {
        NewCell = malloc(sizeof(struct ListNode));
        if(NewCell == NULL)
            FatalError("Out of space!!!");
        else
        {
            L = H->TheLists[ Hash(Key, H->TableSize) ];
            NewCell->Next = L->Next;
            NewCell->Element = Key;
            L->Next = NewCell;
        }
    }
}
```

> 如果在散列中诸例程中不包括删除操作，那么最好不要使用表头。因为这不仅不能简化问题而且还要浪费大量的空间。

### 开放定址法（Open addressing hashing）

类型声明：

``` c
#ifndef _HashQuad_H

typedef unsigned int Index;
typedef Index Position;

struct HashTbl;
typedef struct HashTbl *HashTable;

HashTable InitializeTable(int TableSize);
void DestroyTable(HashTable H);
Position Find(ElementType Key, HashTable H);
void Insert(ElementType Key, HashTable H);
ElementType Retrieve(Position P, HashTable H);
HashTable Rehash(HashTable H);

#endif

enum KindOfEntry { Legitimate, Empty, Deleted };

struct HashEntry
{
    ElementType Element;
    enum KindOfEntry Info;
};

typedef struct HashEntry Cell;

struct HashTbl
{
    int TableSize;
    Cell *TheCells;
};
```

初始化开放定址散列表:

``` c
HashTable
InitializeTable(int TableSize)
{
    HashTable H;
    int i;

    if (TableSize < MinTableSize)
    {
        Error("Table size too small");
        return NULL;
    }

    /* 给散列表分配内存 */
    H = malloc(sizeof(struct HashTbl));
    if (H == NULL)
        FatalError(Out of space!!!);
    
    H->TableSize = NextPrime(TableSize);    /* 大于 TableSize 的第一个素数 */

    /* 给数组所有单元分配内存 */
    H->TheCells = malloc(sizeof(Cell) * H->TableSize);
    if(H->TheCells == NULL)
        FatalError("Out of space!!!");
    
    for (i = 0; i < H->TableSize; i++)
        H->TheCells[i].Info = Empty;

    return H;
}
```

Find 操作：

``` c
Position
Find(ElementType Key, HashTbl H)
{
    Position CurrentPos;
    int CollisionNum;

    CollisionNum = 0;
    CurrentPos = Hash(Key, H->TableSize);
    while(H->TheCells[CurrentPos].Info != Empty &&
          H->TheCells[CurrentPos].Element != Key)
          /* 这里可能需要使用 strcmp 字符串比较函数 !! */
    {
        CurrentPos += 2 * ++CollisionNum - 1;
        if (CurrentPos >= H->TableSize)
            CurrentPos -= H->TableSize;
    }

    return CurrentPos;
}
```

Insert 操作：

``` c
void Insert(ElementType Key, HashTable H)
{
    Position Pos;
    Pos = Find(Keu, H);
    if (H->TheCells[Pos].Info != Legitimate)
    {
        H->TheCells[Pos].Info = Legitimate;
        H->TheCells[Pos].Element = Key; /* 字符串类型需要使用 strcpy 函数 */
    }
}
```

## 散列表的应用

散列有着丰富的应用。编译器使用散列表跟踪源代码中声明的变量。这种数据结构叫做`符号表(symbol table)`。散列表是这种问题的理想应用，因为只有`Insert`和`Find`操作。标识符一般都不长，因此其散列函数能够迅速被算出。

散列表常见的用途也出现在为游戏编写的程序中。当程序搜索游戏的不同的行时，它跟踪通过计算机基于位置的散列函数而看到的一些位置。如果同样的位置再出现，程序通常通过简单移动变换来避免昂贵的重复计算。游戏程序的这种一般特点叫做`变换表(transposition table)`。

另外一个用途是在线拼写检验程序。如果错拼检测（与纠正错误相比）更重要，那么整个词典可以被预先散列，单词则可以在常数时间内被检测。散列表很适合这项工作，因为以字母顺序排列单词并不重要；而以它们在文件中出现的顺序显示出错误拼写当然是可以接受的。
