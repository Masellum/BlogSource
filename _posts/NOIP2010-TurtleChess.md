---
title: "[NOIP2010] 乌龟棋"
date: 2018-08-24 19:59:56
tags:
  - 动态规划
categories: 题解
summary: ' '
---

<!-- more -->

# 题目

[题目链接](https://www.luogu.org/problemnew/show/P1541)

## 题目背景

小明过生日的时候，爸爸送给他一副乌龟棋当作礼物。

## 题目描述

乌龟棋的棋盘是一行 $N$ 个格子，每个格子上一个分数（非负整数）。棋盘第 $1$ 格是唯一的起点，第 $N$ 格是终点，游戏要求玩家控制一个乌龟棋子从起点出发走到终点。

乌龟棋中 $M$ 张爬行卡片，分成 $4$ 种不同的类型（$M$ 张卡片中不一定包含所有 $4$ 种类型的卡片，见样例），每种类型的卡片上分别标有 $1,2,3,4$ 四个数字之一，表示使用这种卡片后，乌龟棋子将向前爬行相应的格子数。游戏中，玩家每次需要从所有的爬行卡片中选择一张之前没有使用过的爬行卡片，控制乌龟棋子前进相应的格子数，每张卡片只能使用一次。

游戏中，乌龟棋子自动获得起点格子的分数，并且在后续的爬行中每到达一个格子，就得到该格子相应的分数。玩家最终游戏得分就是乌龟棋子从起点到终点过程中到过的所有格子的分数总和。

很明显，用不同的爬行卡片使用顺序会使得最终游戏的得分不同，小明想要找到一种卡片使用顺序使得最终游戏得分最多。

现在，告诉你棋盘上每个格子的分数和所有的爬行卡片，你能告诉小明，他最多能得到多少分吗？

## 输入输出格式

### 输入格式

每行中两个数之间用一个空格隔开。

第 $1$ 行 $2$ 个正整数 $N,M$，分别表示棋盘格子数和爬行卡片数。

第 $2$ 行 $N$ 个非负整数，$a_1, a_2, \ldots , a_N$，其中 $a_i$ 表示棋盘第 $i$ 个格子上的分数。

第 $3$ 行 $M$ 个整数，$b_1, b_2, \dots , b_M$，表示 $M$ 张爬行卡片上的数字。

输入数据保证到达终点时刚好用光 $M$ 张爬行卡片。

### 输出格式

$1$ 个整数，表示小明最多能得到的分数。

### 说明

对于 $30\%$ 的数据有 $1\le N \le 30,1 \le M \le 12$。

对于 $50\%$ 的数据有 $1 \le N \le 120,1 \le M \le 50$，且 $4$ 种爬行卡片，每种卡片的张数不会超过 $20$。

对于 $100\%$ 的数据有 $1 \le N \le 350,1 \le M \le 120$，且 $4$ 种爬行卡片，每种卡片的张数不会超过 $40$；$0 \le a_i \le 100,1 \le i \le N,1 \le b_i \le 4,1 \le i \le M$。

# 题解

因为是在一个线性表上做游戏，所以考虑DP。直接的想法是做关于位置的DP。设
$$
f(i, a, b, c, d)
$$
表示第1~4种卡牌分别用了 $a, b, c, d$ 张，目前到达的位置是 $i$ 时的最大分数。一个显然的优化是 $i = 1 \times a + 2 \times b + 3 \times c + 4 \times d$，于是可以优化掉一维。这里由于总牌数固定，实际上可以由三种牌的数量算出第四种牌的数量，因此可以再优化掉一维空间；但是因为不写这个优化也能过这道题我就没写（。

考虑如何转移。当棋子在某个位置的时候，它一定是在之前的某个位置使用了一张卡片转移过来的。因此可以列出转移方程：
$$
f(a, b, c, d) = \max\{f(a - 1, b, c, d) , f(a, b - 1, c, d), f(a, b, c - 1, d), f(a, b, c, d - 1\} + s(a, b, c, d)
$$
，其中 $s(a, b, c, d) = \rm{score}[1 + 1 \times a + 2 \times b + 3 \times c + 4 \times d]$，$\rm{score}[i]$ 代表第 $i$ 个格子的分数。之所以加一是因为开始时的位置就是第一个格子。写代码时需要注意的是特判以下式中的各个量不能小于零。

# 代码

```cpp
#include <algorithm>
#include <cstdio>
#include <climits>

const int MAXN = 350 + 5, MAXM = 120 + 5;

using std::max;

int a[MAXN], b[MAXM], n, m, count[5], f[41][41][41][41];

int main() {
    scanf("%d %d", &n, &m);
    for (int i = 1; i <= n; ++i) {
        scanf("%d", &a[i]);
    }
    for (int i = 1; i <= m; ++i) {
        scanf("%d", &b[i]);
        count[b[i]]++;
    }
    f[0][0][0][0] = a[1];
    for (int i = 0; i <= count[1]; ++i) {
        for (int j = 0; j <= count[2]; ++j) {
            for (int k = 0; k <= count[3]; ++k) {
                for (int l = 0; l <= count[4]; ++l) {
                    int rest = a[1 + i + 2 * j + 3 * k + 4 * l];
                    if (i)
                        f[i][j][k][l] =
                            max(f[i][j][k][l], f[i - 1][j][k][l] + rest);
                    if (j)
                        f[i][j][k][l] =
                            max(f[i][j][k][l], f[i][j - 1][k][l] + rest);
                    if (k)
                        f[i][j][k][l] =
                            max(f[i][j][k][l], f[i][j][k - 1][l] + rest);
                    if (l)
                        f[i][j][k][l] =
                            max(f[i][j][k][l], f[i][j][k][l - 1] + rest);
                }
            }
        }
    }
    printf("%d", f[count[1]][count[2]][count[3]][count[4]]);
    return 0;
}
```

Finita la comedia.