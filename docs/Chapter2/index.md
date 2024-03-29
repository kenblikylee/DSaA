# 算法分析

## 前言

在数据量不大时，我们的程序只需要考虑能否实现功能。但随着用户量的增长以及随之而来的用户行为数据和各种业务数据的急剧增加，那些仅仅能满足功能的代码，就不得不再考虑它的运行效率。如果运行过于漫长，就算实现了功能，这样的程序在实际生产中也是不能用的，必须对程序算法进行分析，给出时间复杂度更低的改进算法。本文从初学者角度介绍算法分析的数学基础，以及如何使用大 $O$ 法分析程序或算法的时间复杂度和常用的分析法则。 

## 1. 为什么要做算法分析？

设想解决同一个问题有两个算法，算法 1 花费的时间为 $T_1(N)=100N$，算法 2 花费的时间为 $T_2(N)=N^2$ 。其中 $N$ 代表解决问题要处理的数据量，当 $N$ 相等时，要判断哪个算法更快，取决于 $N$ 的大小，或者说要处理的数据量大小。当数据量很小时，算法1要比算法2花费更多时间；当$N$超过某个临界值后（这里是 100），算法2的花费时间急剧增加，远超算法1，相对而言，算法1却增长平缓。　 

![](http://cdn.kenblog.top/algorithm_runtime_alg1_alg2.png)

由此可见，单纯看算法实际运行时间，即使是在同样的环境下，我们也无法得到一个一致的答案来回答哪个算法更好的问题。为了给算法提供统一评判标准，我们需要一个更具一般性的分析方法。

## 2. 四大数学定义

我相信很多读者看到“数学”这两个字，就开始有点慌了，况且这一下还来四个，就更受不了了。但是别慌，数学定义虽然看起刻板生硬，实际要表达的意思却是一句人话就能说明白的。当然，我们也不能怪数学家们，他们这样定义也是为了追求严谨。这里，除了第一个大$O$定义，其他三个定义，笔者为了能更加清晰看出各定义间的区别，在意思不变的前提下，对符号格式和语言顺序做了调整。仔细对比，不难发现，其实我们只需要理解第一个大 $O$ 定义，其他的实际是在说同一个事的不同情况。

### 2.1 大 $O$ 定义

如果存在正常数 $c$ 和 $n_0$，使得当$N >= n_0$时，$T(N)<=cf(N)$，则记为 $T(N)=O(f(N))$ 。

### 2.2 大 $\Omega$ 定义

如果存在正常数 $c$ 和 $n_0$，使得当$N >= n_0$时，$T(N)>=cf(N)$，则记为 $T(N)=\Omega(f(N))$ 。

### 2.3 大 $\Theta$ 定义

当 $T(N)=O(f(N))$ 且 $T(N)=\Omega(f(N))$ 时，则记为 $T(N)=\Theta(f(N))$ 。

### 2.4 小 $o$ 定义

当 $T(N)=O(f(N))$ 且 $T(N) \neq \Omega(f(N))$ 时，则记为 $T(N)=o(f(N))$ 。

### 2.5 说句人话

看完四大定义，再放到具体场景中。例如，$T(N）$ 代表第一节中的算法运行时间，N依然指代算法处理的数据量。当数据量非常大时，大 $O$ 代表算法运行时间的上限，大 $\Omega$ 是下限，大$\Theta$代表两个算法的时间复杂度是一样的，小$o$与大$O$的区别是，小$o$不能等于上限，而大$O$可以。 

对于实际需求，与其给算法运行时间定一个**下限**，或者说算法**至少**需要花多时间才能完成，其意义不如给算法定一个**上限**，即算法**最多**花费多少时间，它可以提前完成，也可以正好等于我们的预期(上限)，但是绝不能超出上限。由此可见，大$O$正好满足这样的分析需求。

## 3. 用大 $O$ 法分析算法时间复杂度

我们已经知道大 $O$ 是给算法定义一个时间上限（函数）$f(N)$，只要算法运行时间不超出这个上限，都可以说算法的时间复杂度为 $T(N) = O(f(N))$ 。因此，使用大 $O$ 法分析算法的时间复杂度，本质就是给出一个上限函数，来评估算法的运行时间。当然数学上，这样的上限函数不只一个。为了简化分析，我们将采纳如下约定：不存在特定的时间单位。因此，分析中，我们将抛弃`常数系数`和`低阶项`，抛弃`常数系数`和`低阶项`， 抛弃`常数系数`和`低阶项`。重要的事情说三遍。

### 3.1 举个栗子

计算 $\sum_{i=1}^{N}i^3$ 

JavaScript:

``` js
function Sum(N) {
    var sum = 0                 /* 1 */
    for (i = 1; i <= N; i++)    /* 2 */
        sum += i * i *i         /* 3 */
    return sum                  /* 4 */
}
```

Python:

``` python
def Sum(N):
    sum = 0                     # 1
    for i in range(1, N+1):     # 2
        sum += i * i *i         # 3
    return sum                  # 4
```

分析这段程序很简单。标注第 1 行和第 4 行各占一个时间单元；第 3 行执行占用 4 个时间单元（2 次乘法 $+$ 1 次加法 $+$ 1 次赋值）而执行 $N$ 次共占用 $4N$ 个时间单元；第 2 行在初始化 `i` 占用一个时间单元，测试 `i <= N` 占用 $N+1$ 个时间单元，以及所有自增运算 $N$ 个时间单元，共 $2N+2$ 。忽略调用函数和返回值的开销，得到总量是 $6N+4$ 。抛弃常数系数 `6` 和低阶项 `4` (可以看作是 $4N^0$)，我们说该函数的时间复杂度是 $O(N)$。

如果你觉得这样逐行代码进行分析的方式过于繁琐。那么值得高兴的是，我们不需要每次都采取这样笨拙的方法。事实上，通过分析得出大$O$，可以发现很多地方对时间单元的精确定量是没有必要的甚至多余的，因为这些精确的数值恰恰是我们最后要抛弃的`常数系数`和`低阶项`。因此，我们得到若干通用法则。

### 3.2 通用法则

#### 3.2.1 法则1——FOR 循环

一次 `for` 循环的运行时间至多是该 `for` 循环内语句（包括测试）的运行时间乘以迭代的次数。

也就是收如果 `N` 是 `for` 循环的迭代次数，那么它的时间复杂度就是 $O(N)$ 。例如：

JavaScript:

``` js
for (var i = 0; i < N; i++) {
    count++
}
```

Python:

``` python
for i in range(N):
    count += 1
```

#### 3.2.2 法则2——嵌套的FOR 循环

从里向外分析这些循环。在一组嵌套循环内部的一条语句总的运行时间为该语句的运行时间乘以以该组所有的 `for` 循环的大小的乘积。

作为一个例子，下列程序片段的时间复杂度为 $O(N^2)$:

JavaScript:

``` js
for (var i = 0; i < N; i++) {
    for (var j = 0; j < N; j++) {
        k++
    }
}
```

Python:

``` python
for i in range(N):
    for j in range(N):
        k += 1
```

#### 3.2.3 法则3——顺序语句

将各个语句的运行时间求和，最大值项(抛弃`低阶项`)就是所得运行时间。

例如，下面的程序片段先用去 $O(N)$ ,再花费 $O(N^2)$，总的开销也是 $O(N^2)$ 。

JavaScript:

``` js
for (var i = 0; i < N; i++) {
    count++
}

for (var i = 0; i < N; i++) {
    for (var j = 0; j < N; j++) {
        k++
    }
}
```

Python:

``` python
for i in range(N):
    count += 1

for i in range(N):
    for j in range(N):
        k += 1
```

#### 3.2.4 法则4——IF/ELSE 语句

对于程序片段

JavaScript:

``` js
if (condition) {
    statement_block1
} else {
    statement_block2
}
```

Python:

``` python
if condition:
    statement_block1
else:
    statement_block2
```

一个 `if/else` 语句的运行时间从不超过判断再加上语句块 `statement_block1` 和 `statement_block2` 中运行时间长者的总的运行时间。显然在某些情形下这么估计有些过高，但绝对不会估计过低。

分析的基本策略是从内部向外展开，如果有函数调用，那么这些调用就要首先分析。例如：

JavaScript:

``` js
function Factorial(N) {
    if (N <= 1) {                   /* O(1) */
        return 1                    /* O(1) */
    } else {
        return N * Factorial(N - 1) /* O(N) */
    }
}
```

Python:

``` python
def Factorial(N):
    if N <= 1:                      # O(1)
        return 1                    # O(1)
    else:
        return N * Factorial(N - 1) # O(N)
```

求阶乘函数 `Factorial` 中，判断 `N <= 1` 为 $O(1)$，`if` 分支语句块为 $O(1)$，`else` 分支语句块是一个递归调用，但是其实复杂度相当于一个 `for` 循环，因此为 $O(N)$，所以整个程序片段的时间复杂度是 $O(N)$ 。

## The End

