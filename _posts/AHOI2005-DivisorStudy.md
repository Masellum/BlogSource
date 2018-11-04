---
title: '[AHOI2005] 约数研究'
date: 2018-09-12 08:35:55
tags:
  - 数学
categories: 题解
summary: ' '
---

# 题目

[题目链接](https://www.luogu.org/problemnew/show/P1403)

## 题目描述

科学家们在Samuel星球上的探险得到了丰富的能源储备，这使得空间站中大型计算机“Samuel II”的长时间运算成为了可能。由于在去年一年的辛苦工作取得了不错的成绩，小联被允许用“Samuel II”进行数学研究。

小联最近在研究和约数有关的问题，他统计每个正数 $N$ 的约数的个数，并以 $f(N)$ 来表示。例如 $12$ 的约数有 $1$、$2$、$3$、$4$、$6$、$12$。因此 $f(12)=6$。下表给出了一些 $f(N)$ 的取值：

![img](https://cdn.luogu.org/upload/pic/1645.png)

$f(n)$ 表示 $n$ 的约数个数，现在给出 $n$，要求求出 $f(1)$ 到 $f(n)$ 的总和。

## 输入输出格式

### 输入格式：

输入一行，一个整数 $n$。

### 输出格式：

输出一个整数，表示总和。

## 说明

对于 $20\%$ 的数据，$N \le 5000$；

对于 $100\%$，$N \le 1000000$。

# 题解

对于一个 $n$，
$$
f(n) = \sum_{i = 1}^{n} \lfloor \frac{n}{i} \rfloor.
$$
打表可以发现 $\lfloor\frac{n}{i}\rfloor$ 的值呈现块状分布，对于某一个区间内的 $i$ ，$\lfloor\frac{n}{i}\rfloor$ 的值相同，这个区间的右端点为 $\frac{n}{n/i}$。

这样的区间最多有 $2\sqrt{n}$ 个。于是我们进行整除分块，总复杂度 $O(n\sqrt{n})$。

# 代码

```cpp
#include <algorithm>
#include <cstdio>

int n, ans;

int main() {
    scanf("%d", &n);
    for (int l = 1, r; l <= n; l = r + 1) {
        r = n / (n / l);
        ans += (r - l + 1) * (n / l);
    }
    printf("%d", ans);
    return 0;
}
```

Finita la comedia.