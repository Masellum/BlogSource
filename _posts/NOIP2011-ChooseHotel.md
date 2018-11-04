---
title: "[NOIP2011] 选择客栈"
date: 2018-08-24 20:55:44
tags:
  - 递推
categories: 题解
summary: ' '
---

# 题目

[题目链接](https://www.luogu.org/problemnew/show/P1311)

## 题目描述

丽江河边有 $n$ 家很有特色的客栈，客栈按照其位置顺序从 $1$ 到 $n$ 编号。每家客栈都按照某一种色调进行装饰（总共 $k$ 种，用整数 $0 \ldots k-1$ 表示），且每家客栈都设有一家咖啡店，每家咖啡店均有各自的最低消费。

两位游客一起去丽江旅游，他们喜欢相同的色调，又想尝试两个不同的客栈，因此决定分别住在色调相同的两家客栈中。晚上，他们打算选择一家咖啡店喝咖啡，要求咖啡店位于两人住的两家客栈之间（包括他们住的客栈），且咖啡店的最低消费不超过 $p$。

他们想知道总共有多少种选择住宿的方案，保证晚上可以找到一家最低消费不超过 $p$ 元的咖啡店小聚。

## 输入输出格式

### 输入格式

共 $n+1$ 行。

第一行三个整数 $n ,k ,p$， 每两个整数之间用一个空格隔开，分别表示客栈的个数，色调的数目和能接受的最低消费的最高值；

接下来的 $n$ 行，第 $i+1$ 行两个整数，之间用一个空格隔开，分别表示 $i$ 号客栈的装饰色调和 $i$ 号客栈的咖啡店的最低消费。

### 输出格式

一个整数，表示可选的住宿方案的总数。

## 说明

对于 $30\%$ 的数据，有 $n \le 100$；

对于 $50\%$ 的数据，有 $n \le 1,000$；

对于 $100\%$ 的数据，有 $2 \le n \le 200,000, 0 < k \le 50, 0 \le p \le 100$， 最低消费的范围与 $p$ 的范围相同。

# 题解

最暴力的做法是三重枚举，每次枚举第一个客栈的位置、第二个客栈的位置、中间的咖啡馆的位置。但是这样显然会爆，于是考虑优化。我们想到，如果我们能够确定第二个客栈的位置，那么剩下的两个信息都只与这个位置之前的位置的信息有关，所以我们考虑枚举第二个客栈的位置进行递推。

我们维护 $\rm{last}[i]$ 表示 $i$ 这种颜色上一次出现的位置，$\rm{count}[i]$ 表示 $i$ 这种颜色已经出现的次数，$\rm{now}$ 表示距离当前位置最近的可以选择的咖啡店的位置。每到一个新位置以后，我们要先更新 $\rm{now}$；如果当前这种颜色上一次出现的位置比 $\rm{now}$ 更靠前，那么当前这种颜色已经找到的方案数就可以被更新为这种颜色已经出现的次数；如果不比 $\rm{now}$ 更靠前，也就是说这次这种颜色对答案的贡献和上一次这种颜色对答案的贡献是一样的，那么就没有必要更新。然后维护 $\rm{last}[i]$ 与 $\rm{count}[i]$， 更新答案。

# 代码

```cpp
#include <algorithm>
#include <cstdio>
#include <cstring>

const int MAXK = 50 + 1;

int lastAppearance[MAXK], countOfColors[MAXK], countOfSolutions[MAXK], lastAvailable, ans;
int n, k, p;

int main() {
    scanf("%d %d %d", &n, &k, &p);
    int color, cost;
    for (int i = 1; i <= n; ++i) {
        scanf("%d %d", &color, &cost);
        if (cost <= p) {
            lastAvailable = i;
        }
        if (lastAvailable >= lastAppearance[color]) {
            countOfSolutions[color] = countOfColors[color];
        }
        lastAppearance[color] = i;
        ans += countOfSolutions[color];
        countOfColors[color]++;
    }
    printf("%d\n", ans);
    return 0;
}
```

Finita la comedia.