---
title: '[NOIP2013] 货车运输'
tags:
  - LCA
  - 生成树
date: 2018-08-03 19:11:51
categories: 题解
summary: ' '
---

<!-- more -->

# 题目

[题目链接](https://www.luogu.org/problemnew/show/P1967)

## 题目描述

A 国有 $n$ 座城市，编号从 $1$ 到 $n$ ，城市之间有 $m$ 条双向道路。每一条道路对车辆都有重量限制，简称限重。现在有 $q$ 辆货车在运输货物，司机们想知道每辆车在不超过车辆限重的情况下，最多能运多重的货物。

## 输入输出格式

### 输入格式

第一行有两个用一个空格隔开的整数 $n, m$ ，表示 A 国有 $n$ 座城市和 $m$ 条道路。

接下来 $m$ 行每行 $3$ 个整数 $x, y, z$ ，每两个整数之间用一个空格隔开，表示从 $x$ 号城市到 $y$ 号城市有一条限重为 $z$ 的道路。注意： **$x$ 不等于 $y$ ，两座城市之间可能有多条道路**。

接下来一行有一个整数 $q$，表示有 $q$ 辆货车需要运货。

接下来 $q$ 行，每行两个整数 $x, y$ ，之间用一个空格隔开，表示一辆货车需要从 $x$ 城市运输货物到 $y$ 城市，注意：**$x$ 不等于 $y$**。

### 输出格式

共有 $q$ 行，每行一个整数，表示对于每一辆货车，它的最大载重是多少。如果货车不能到达目的地，输出 `-1` 。

### 说明

对于 $30\%$ 的数据， $0 < n < 1,000,0 < m < 10,000,0 < q< 1,000$；

对于 $60\%$ 的数据， $0 < n < 1,000,0 < m < 50,000,0 < q< 1,000$；

对于 $100\%$ 的数据，$0 < n < 10,000,0 < m < 50,000,0 < q< 30,000,0 \le z \le 100,000$。

# 题解

简化题意为在无向图上求给定两点间路径上边权的最小值的最大值。

显然当这条路径在无向图的最大生成树上时路径上各边边权的最小值最大，否则一定能在最大生成树上找到一条路径使最小值比当前最小值更大。

求最大生成树的算法与求最小生成树的算法基本相同，只是把每次选择最小值换成选择最大值。

能满足所求的路径即为最大生成树上两点间的最短路径。对于每个询问，在最大生成树上求两点的 LCA，然后从两个点依次向上走到 LCA，求路径上的最小边权即为这个询问的答案。

需要注意的是图不保证联通，因此求 LCA 的时候应判断两点是否在最大生成森林（……可以这么叫吗）中的同一棵树上，不是的话输出 `-1` 。

# 代码

（写得又丑又长（向最大生成树中加边那个构造函数非常毒瘤（是因为我写到后来发现不对但是懒得重构了（逃

```cpp
#include <cmath>
#include <ctime>
#include <cstdio>
#include <climits>
#include <cstdlib>
#include <algorithm>

const int MAXN = 10000 + 5, MAXM = 50000 + 5;

int n, m, q, ans;

struct DisjointSet {
    int fa[MAXN], rank[MAXN];

    int find(int x) {
        return x == fa[x] ? x : (fa[x] = find(fa[x]));
    }

    void merge(int x, int y) {
        x = find(x);
        y = find(y);
        if (x == y)
            return;
        if (rank[x] > rank[y])
            std::swap(x, y);
        if (rank[x] < rank[y]) {
            fa[x] = y;
        } else {
            if (rand() % 2 > 1) {
                fa[x] = y;
                rank[y]++;
            } else {
                fa[y] = x;
                rank[x]++;
            }
        }
    }

    DisjointSet(int n) {
        for (int i = 1; i <= n; ++i) {
            fa[i] = i;
        }
    }
};

struct Edge;

Edge *head[MAXN], *hmst[MAXN];
int cnt;

struct Edge {
    int from, to, weight;
    Edge *next;

    Edge() {}
    Edge(int f, int t, int w) : from(f), to(t), weight(w), next(head[f]) {}
    Edge(int f, int t, int w, int flag) : from(f), to(t), weight(w), next(hmst[f]) {}
} edge[MAXM];

void addEdge(int f, int t, int w) {
    head[f] = new Edge(f, t, w);
    head[t] = new Edge(t, f, w);
    edge[++cnt] = *head[f];
}
void addMSTEdge(int f, int t, int w) {
    hmst[f] = new Edge(f, t, w, 0);
    hmst[t] = new Edge(t, f, w, 0);
}

void kruskal() {
    std::sort(edge + 1, edge + m + 1, [](const Edge &a, const Edge &b) -> bool {
       return a.weight > b.weight;
    });
    cnt = 0;
    DisjointSet ds(n);
    for (int i = 1; i <= m; ++i) {
        if (cnt == 2 * n - 2)
            return;
        if (ds.find(edge[i].from) != ds.find(edge[i].to))
        {
            ds.merge(edge[i].from, edge[i].to);
            addMSTEdge(edge[i].from, edge[i].to, edge[i].weight);
            cnt++;
        }
    }
}

bool vis[MAXN];
int f[MAXN][15], dep[MAXN], log_2[MAXN], faw[MAXN];

void initlog() {
    for (int i = 1; i <= n; ++i) {
        log_2[i] = (log_2[i - 1]) + (1 << log_2[i - 1] == i);
    }
}

void dfs(int x, int fa) {
    vis[x] = true;
    f[x][0] = fa;
    dep[x] = dep[fa] + 1;
    for (int j = 1; (1 << j) <= dep[x]; ++j) {
        f[x][j] = f[f[x][j - 1]][j - 1];
    }
    for (Edge *e = hmst[x]; e; e = e->next) {
        if (!vis[e->to]) {
            dfs(e->to, x);
            faw[e->to] = e->weight;
        }
    }
}

int lca(int x, int y) {
    if (dep[x] < dep[y])
        std::swap(x, y);
    while (dep[x] > dep[y]) {
        x = f[x][log_2[dep[x] - dep[y]] - 1];
    }
    if (x == y)
        return x;
    for (int j = log_2[dep[x]]; j >= 0; --j) {
        if (f[x][j] == f[y][j])
            continue;
        x = f[x][j];
        y = f[y][j];
    }
    return f[x][0];
}

int main() {
    scanf("%d %d", &n, &m);
    int x, y, z;
    for (int i = 1; i <= m; ++i) {
        scanf("%d %d %d", &x, &y, &z);
        addEdge(x, y, z);
    }
    kruskal();
    initlog();
    for (int i = 1; i <= n; ++i) {
        if (!vis[i])
            dfs(i, 0);
    }
    scanf("%d", &q);
    for (int i = 1; i <= q; ++i) {
        ans = INT_MAX;
        scanf("%d %d", &x, &y);
        z = lca(x, y);
        if (z != 0) {
            while (x != z) {
                ans = std::min(ans, faw[x]);
                x = f[x][0];
            }
            while (y != z) {
                ans = std::min(ans, faw[y]);
                y = f[y][0];
            }
            printf("%d\n", ans);
        } else {
            puts("-1");
        }
    }
    return 0;
}
```

Finita la comedia.
