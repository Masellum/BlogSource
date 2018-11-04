---
title: '[NOIP2012] 开车旅行'
date: 2018-11-04 17:16:02
categories: 题解
tags:
  - 倍增
  - 双向链表
summary: ' '
---

# 题目

[题目链接](https://loj.ac/problem/2604)

## 题目描述

小 A 和小 B 决定利用假期外出旅行，他们将想去的城市从 $1$ 到 $N$ 编号，且编号较小的城市在编号较大的城市的西边，已知各个城市的海拔高度互不相同，记城市 $i$ 的海拔高度为 $Hi$，城市 $i$ 和城市 $j$ 之间的距离 $d_{i, j}$ 恰好是这两个城市海拔高度之差的绝对值，即 $d_{i, j} = |H_i - H_j|$。

旅行过程中，小 A 和小 B 轮流开车，第一天小 A 开车，之后每天轮换一次。他们计划选择一个城市 $S$ 作为起点，一直向东行驶，并且最多行驶 $X$ 公里就结束旅行。小 A 和小 B 的驾驶风格不同，小 B 总是沿着前进方向选择一个最近的城市作为目的地，而小 A 总是沿着前进方向选择第二近的城市作为目的地（注意：本题中如果当前城市到两个城市的距离相同，则认为离海拔低的那个城市更近）。如果其中任何一人无法按照自己的原则选择目的城市，或者到达目的地会使行驶的总距离超出 $X$ 公里，他们就会结束旅行。

在启程之前，小 A 想知道两个问题：

1. 对于一个给定的 $X = X_0$，从哪一个城市出发，小 A 开车行驶的路程总数与小 B 行驶的路程总数的比值最小（如果小 B 的行驶路程为 $0$，此时的比值可视为无穷大，且两个无穷大视为相等）。如果从多个城市出发，小 A 开车行驶的路程总数与小 B 行驶的路程总数的比值都最小，则输出海拔最高的那个城市。
2. 对任意给定的 $X = X_i$ 和出发城市 $S_i$，小 A 开车行驶的路程总数以及小 B 行驶的路程总数。

## 输入格式

第一行包含一个整数 $N$，表示城市的数目。

第二行有 $N$ 个整数，每两个整数之间用一个空格隔开，依次表示城市 $1$ 到城市 $N$ 的海拔高度，即 $H_1,H_2,\ldots,H_n$，且每个 $H_i$ 都是不同的。

第三行包含一个整数 $X_0$。

第四行为一个整数 $M$，表示给定 $M$ 组 $S_i$ 和 $X_i$。

接下来的 $M$ 行，每行包含 $2$ 个整数 $S_i$ 和 $X_i$，表示从城市 $S_i$ 出发，最多行驶 $X_i$ 公里。

## 输出格式

第一行包含一个整数 $S_0$，表示对于给定的 $X_0$，从编号为 $S_0$ 的城市出发，小 A 开车行驶的路程总数与小 B 行驶的路程总数的比值最小。

接下来的 $M$ 行，每行包含 $2$ 个整数，之间用一个空格隔开，依次表示在给定的 $S_i$ 和 $X_i$ 下小 A 行驶的里程总数和小 B 行驶的里程总数。

## 数据范围与提示

对于 $30\%$ 的数据，有 $1 \leq N \leq 20$，$1 \leq M \leq 20$；

对于 $40\%$ 的数据，有 $1 \leq N \leq 100$，$1 \leq M \leq 100$；

对于 $50\%$ 的数据，有 $1 \leq N \leq 100$，$1 \leq M \leq 1\,000$；

对于 $70\%$ 的数据，有 $1 \leq N \leq 1\,000$，$1 \leq M \leq 10\,000$；

对于 $100\%$ 的数据，有 $1 \leq N \leq 100\,000$，$1 \leq M \leq 10\,000$，$|H_i| \leq 10^9$，$0 \leq X_i \leq 10^9\,\,\forall i \geq 0$，$1 \leq S_i \leq N\,\,\forall i \geq 1$，数据保证 $H_i$ 各不相同。

# 题解

直接去想做法的话没有什么思路，我们先去考虑暴力。对于第一问，枚举每个城市作为起点，在向后走的过程中枚举寻找最近点或次近点，然后走过去，复杂度 $O(n^3)$；对于第二问，做法类似，复杂度 $O(mn^2)$。思考发现对于第一问和第二问，枚举起点这个过程不太能优化，于是我们考虑怎样优化。首先我们想到，如果我们能快速地求出每个城市的最近点和次近点，就可以优化掉一个 $O(n)$。但是这样复杂度仍然不可接受，于是我们再去考虑知道每个城市的最近点和次近点时怎样求从某个城市出发在某个距离内最远能走到的城市。考虑到复杂度原因，对单个城市求解这个问题的复杂度必须低于 $O(n)$；于是想到 $O(\log n)$ 的二分或倍增。二分似乎也可做，但是倍增大概会更好写&调试一些……？（强行找理由）于是我们考虑倍增求解这个问题。

重新考虑第一个优化。首先发现，这个问题中所有信息都不会被动态修改，因此我们可以预处理信息。将所有城市按照海拔高度排序后，排在第 $i$ 位的城市的最近点和次近点一定是排在第 $i - 2, i - 1, i + 1, i + 2$ 位的这四个城市中的两个。但直接排序后这样计算是不对的，原因在于排序后我们丢失了原来的城市间的位置关系，于是无法判断这四个相邻城市是否符合“方向向东”这个条件。怎样正确处理位置关系呢？我们考虑，某个城市不会成为它东边的城市的最近点或次远点，所以我们忽略掉（或删掉）一个城市不会对这个城市东边的城市的信息更新造成影响。而编号为 $1$ 的城市西边没有城市，所以我们如果能从编号为 $1$（也就是最西边）的城市一直向右处理，每次处理完后标记这个城市会被忽略（也就是删掉这个城市），就可以正确处理信息。想到快速处理对某个元素的删除操作，很容易联想到链表。而这个问题中城市间的相对关系不会改变，因此可以用双向链表简单地（假的）对每个城市 $O(1)$ 地处理信息。具体做法是：先对所有城市按海拔高度排序，并按排序后的顺序建立双向链表。然后从**原编号**为 $1$ 的城市向**原编号**为 $n$ 的城市依次求解，每求解完一个城市就将它从链表中删除，就不会再对之后的城市的求解产生影响。这样，我们便可以 $O(n)$ 地预处理出每个城市的最近点和次近点。

对于第二个优化，我们可以用类似 ST 表的方式实现。

# 代码

有一些小细节：

1. 这份代码的链表用了指针实现。一开始我是用数组模拟指针写链表，不过写着写着发现用数组的话就会出现 `node[node[node[index[x]].prev].prev].h` 这种恶心的东西……我实在是忍受不了，于是改用指针（虽然也没好太多……），结果陷入了 RE 地狱……原因是因为对某些指针赋值的时候没有判断是否为 `NULL`，于是在代码里加了很多这种判断，就出现了一堆 `if` 和 `else` 互相嵌套导致代码复杂度激增（。
2. 题目中提到，两个城市与当前城市的高度差相同时，判海拔低的城市为较近。大部分题解都不需要特别判断。但我的写法好像要特殊判断这种情况……于是代码复杂度再次++。
3. 类似 ST 表的处理方式具体实现方式在代码中有注释。

```cpp
#include <algorithm>
#include <climits>
#include <cstdio>
#include <iostream>
#include <utility>

const int MAXN = 100000 + 5;

int n, h[MAXN];
int nextA[MAXN], nextB[MAXN], f[MAXN][18];
long long fA[MAXN][18], fB[MAXN][18];
// nextA[i] 表示编号为 i 的城市的东边的次近城市的编号，nextB[i] 类似。
// f[i][j] 表示编号为 i 的城市向东走 j 轮（一轮的意思是小 A 和小 B 都开了一段，也就是实际上前进了两个城市）所能到达的城市的编号。
// fA[i][j] 表示从编号为 i 的城市向东走了几轮，其中小 A 开了 j 段时前进的距离，注意这里同样计入了小 B 开的距离，没有忽略；fB[i][j] 表示的意义类似。

struct Node {
    Node *prev, *next;
    int pos, h;

    bool operator<(const Node &that) const { return this->h < that.h; }
} node[MAXN], *index[MAXN];
// node[i].pos 表示这个链表上的第 i 个节点对应的原城市的编号，index[i]
// 是一个指向 (表示 (编号为 i 的城市) 的链表节点) 的指针；
// 为什么用指针？因为数组模拟链表的写法会有一堆中括号套来套去实在是太丑了。

std::pair<int, int> getDistance(int x, int start) {
    // 以一个二元组的形式返回以 start 为起点，向前走不超过 x 的距离时小 A 和小 B 各自最远开的距离
    int s = start;
    long long da = 0, db = 0;
    for (int j = 17; j >= 0; --j) {
        if (f[s][j] && da + db + fA[s][j] + fB[s][j] <= x) {
            da += fA[s][j], db += fB[s][j];
            s = f[s][j];
        }
    }
    if (nextA[s] != 0 && da + db + fA[s][0] <= x) {
        da += fA[s][0];
    } // 小 A 可能可以在很多轮后恰好还能多开一段，对这种情况进行处理。
    return std::make_pair(da, db);
}

int main() {
    std::scanf("%d", &n);
    for (int i = 1; i <= n; ++i) {
        node[i].pos = i;
        std::scanf("%d", &node[i].h);
    }
    std::sort(node + 1, node + n + 1);
    for (int i = 1; i <= n; ++i) {
        index[node[i].pos] = &node[i];
    }
    for (int i = 1; i <= n; ++i) {
        node[i].prev = &node[i - 1];
        node[i].next = &node[i + 1];
    }
    node[1].prev = node[n].next = NULL;
    // 从下一行到第 120 行为止，都是在链表上处理信息。
    for (int i = 1; i <= n; ++i) {
        if (index[i]->prev == NULL) {
            if (index[i]->next != NULL) {
                nextB[i] = index[i]->next->pos;
                if (index[i]->next->next != NULL) {
                    nextA[i] = index[i]->next->next->pos;
                }
            }
        } else if (index[i]->next == NULL) {
            if (index[i]->prev != NULL) {
                nextB[i] = index[i]->prev->pos;
                if (index[i]->prev->prev != NULL) {
                    nextA[i] = index[i]->prev->prev->pos;
                }
            }
        } else {
            if (abs(index[i]->next->h - index[i]->h) >
                abs(index[i]->prev->h - index[i]->h)) {
                nextB[i] = index[i]->prev->pos;
                nextA[i] = index[i]->next->pos;
                if (index[i]->prev->prev != NULL) {
                    if (abs(index[i]->next->h - index[i]->h) >
                        abs(index[i]->prev->prev->h - index[i]->h)) {
                        nextA[i] = index[i]->prev->prev->pos;
                    } else if (abs(index[i]->next->h - index[i]->h) ==
                        abs(index[i]->prev->prev->h - index[i]->h)) {
                            if (index[i]->next->h > index[i]->prev->prev->h) {
                                nextA[i] = index[i]->prev->prev->pos;
                            }
                    }
                }
            } else if (abs(index[i]->next->h - index[i]->h) <
                       abs(index[i]->prev->h - index[i]->h)) {
                nextB[i] = index[i]->next->pos;
                nextA[i] = index[i]->prev->pos;
                if (index[i]->next->next != NULL) {
                    if (abs(index[i]->prev->h - index[i]->h) >
                        abs(index[i]->next->next->h - index[i]->h)) {
                        nextA[i] = index[i]->next->next->pos;
                    } else if (abs(index[i]->prev->h - index[i]->h) == 
                        abs(index[i]->next->next->h - index[i]->h)) {
                            if (index[i]->prev->h > index[i]->next->next->h) {
                                nextA[i] = index[i]->next->next->pos;
                            }
                        }
                }
            } else {
                if (index[i]->next->h > index[i]->prev->h) {
                    nextB[i] = index[i]->prev->pos;
                    nextA[i] = index[i]->next->pos;
                } else {
                    nextA[i] = index[i]->prev->pos;
                    nextB[i] = index[i]->next->pos;
                }
            }
        }
        if (index[i]->prev != NULL) {
            index[i]->prev->next = index[i]->next;
        }
        if (index[i]->next != NULL) {
            index[i]->next->prev = index[i]->prev;
        }
    }
    for (int i = 1; i <= n; ++i) {
        if (nextA[i] != 0) {
            fA[i][0] = abs(index[nextA[i]]->h - index[i]->h);
            if (nextB[nextA[i]] != 0) {
                fB[i][0] = abs(index[nextB[nextA[i]]]->h - index[nextA[i]]->h);
            }
        }
        f[i][0] = nextB[nextA[i]];
    }
    // 对倍增数组赋初值
    for (int j = 1; j <= 17; ++j) {
        for (int i = 1; i <= n; ++i) {
            f[i][j] = f[f[i][j - 1]][j - 1];
            fA[i][j] = fA[i][j - 1] + fA[f[i][j - 1]][j - 1];
            fB[i][j] = fB[i][j - 1] + fB[f[i][j - 1]][j - 1];
        }
    }
    // ……十分显然的倍增预处理
    int x, s, m;
    std::scanf("%d", &x);
    double min = __DBL_MAX__;
    for (int i = 1; i <= n; ++i) {
        std::pair<int, int> p = getDistance(x, i);
        if (p.second != 0 && 1.0 * p.first / p.second < min) {
            min = 1.0 * p.first / p.second;
            s = i;
        }
    }
    std::printf("%d\n", s);
    std::scanf("%d", &m);
    for (int i = 1; i <= m; ++i) {
        std::scanf("%d %d", &s, &x);
        std::pair<int, int> p = getDistance(x, s);
        printf("%d %d\n", p.first, p.second);
    }
    return 0;
}
```



Finita la comedia.