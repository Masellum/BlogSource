---
title: '数论学习笔记'
date: 2018-09-07 09:39:48
tags:
  - 数论
  - 数学
  - 学习笔记
categories: 学习笔记
summary: ' '
---

<!-- more -->

# 基本定义

对于两个自然数 $a, b$，如果 $a = kb$ 且 $b \neq 0$，那么我们说 $b$ 整除 $a$ 或者说 $a$ 被 $b$ 整除，记作 $b \mid a$；否则 $b$ 不整除 $a$，记作 $b \nmid a$。如果 $b \mid a$，那么我们说 $b$ 是 $a$ 的因数，$a$ 是 $b$ 的倍数。如果一个数同时是两个数的因数，那么这个数是这两个数的公因数。如果两个数除了 $1$ 之外没有别的公因数，那么这两个数互质，记作 $a \ \bot \ b$。

# 欧几里得算法

欧几里得算法又称辗转相除法，用来求两个数的最大公因数。依据是 $\gcd(a, b) = \gcd(b, a\  \% \ b)$。

证明：

$\gcd(a, b) = \gcd(b, a\  \% \ b) \iff \gcd(a, b) = \gcd(b, a - b)$。使用反证法证明。设 $r = \gcd(a, b)$，则 $r$ 也是 $b, a - b$ 的因数。如果 $r \neq \gcd(b, a - b)$，那么一定有一个数 $s$ 满足 $s = \gcd(b, a - b)$。此时 $s \mid b$ 且 $s \mid a - b$，故 $s \mid a$，故 $s$ 也是 $a, b$ 的公因数。但 $s \gt r = \gcd(a, b)$，产生矛盾，故不可能有符合条件的 $s$，故 $r = \gcd(b, a - b)$。

证毕。

代码：

```cpp
int gcd(int a, int b) {
    return b == 0 ? a : gcd(b, a % b);
}
```

时间复杂度为对数级别。

关于最大公约数还有一个性质是 $\gcd(a, b) \times \operatorname{lcm}(a, b) = a \times b$。

证明：

根据算术基本定理，$a, b$ 均可表示成如下形式：


$$
\begin{aligned}
a = p_1^{a_1}p_2^{a_2} \ldots p_n^{a_n}\\
b = p_1^{b_1}p_2^{b_2} \ldots p_m^{b_m}\\
\end{aligned}
$$
其中 $p_i$ 为质数。我们将 $a$ 和 $b$ “对齐”，也就是将他们的质因数逐个对应，只在一个数里出现过的令其指数为 $0$。显然
$$
\begin{aligned}
\gcd(a, b) = \prod_{i = 1}^{\max(n, m)}p_i^{\min(a_i, b_i)}\\
\operatorname{lcm}(a, b) = \prod_{i = 1}^{\max(n, m)}p_i^{\max(a_i, b_i)}
\end{aligned}
$$
故显然 $\gcd(a, b) \times \operatorname{lcm}(a, b) = a \times b$。

# 扩展欧几里得算法

扩展欧几里得算法用来求解形如 $ax + by = \gcd(a, b)$ 的不定方程。
$$
\begin{aligned}
ax + by &= \gcd(a, b) \\[2ex]
&= \gcd(b, a \ \% \ b) \\[2ex]
\end{aligned}
$$
因此我们可以列出新的方程
$$
\begin{aligned}
bx' + (a \ \% \ b)y' &= \gcd(b, a \ \% \ b) \\[2ex]
&= bx' + (a - \lfloor \frac{a}{b} \rfloor \times b) y' \\[2ex]
&= ay' + b \cdot (x' - \lfloor \frac{a}{b} \rfloor \times y')
\end{aligned}
$$


故 $x = y', y = x' - \lfloor \frac{a}{b} \rfloor \times y'$。所以我们可以在进行欧几里得算法的同时递归得出每一组 $a, b$ 所对应的 $x, y$ 。递归终止条件是当 $b = 0$ 时 $x = 1, y = 0$。其实 $y $应该是可以随便取的，不过大家都取 $0 $（可能是为了方便计算）。这样便可得到不定方程的一组解。已经求出一组 $x, y$ 以后，$ax + by = a(x - b) + b(y + a)$，所以我们可以由一组解计算出其他解。

代码：

```cpp
int exgcd(int a, int b, int &x, int &y) {
    if (b == 0) {
        x = 1, y = 0;
        return a;
    }
    int res = exgcd(b, a % b, y, x);
    y -= x * a / b;
    return res;
}
```

扩展欧几里得算法还可以用来计算 $ax \equiv 1 \pmod b$ 的同余方程。因为这种方程可以化为 $ax + by = 1$，当 $\gcd(a, b) = 1$ 即 $a, b$ 互质的时候此方程有解，其他情况无解。

# 质数

质数的定义不再重复。

## 算术基本定理

任意一个正整数都可以表示成如下形式：
$$
n  = p_1^{a_1}p_2^{a_2} \ldots p_n^{a_n}
$$
，其中 $p$ 是质数。

## 质因数分解

分解 $n$ 的质因数的方法是，枚举所有小于等于 $\sqrt{n}$ 的数，如果这个数是 $n$ 的因数，那么就把这个数从 $n$ 中除去并记录它的指数增加了一；如果除完后这个数仍能整除 $n$，那么继续除到无法整除为止。枚举完这些数以后如果 $n$ 被除后剩余的商仍不等于一，那么这个剩余的数就是 $n$ 的唯一一个大于 $\sqrt{n}$ 的质因数。这个算法的正确性十分显然，不再证明。

根据乘法原理，如果 $n  = p_1^{a_1}p_2^{a_2} \ldots p_n^{a_n}$，那么 $n$ 的因数个数为 $\prod_{i=1}^n (a_i+1)$。

## 质数筛法

这里只记录线性筛相关知识，暴力筛和埃氏筛不再记录。

线性筛的基本思想是，每个合数只会被它的最小质因数筛去。

代码：

```cpp
bool isPrime[MAXM];
int primes[7300], cnt, sum[MAXM];

void sieve(int m) {
    std::fill(isPrime + 1, isPrime + m + 1, true);
    isPrime[0] = isPrime[1] = false;
    for (int i = 2; i <= m; ++i) {
        if (isPrime[i]) {
            primes[++cnt] = i;
        }
        for (int j = 1; j <= cnt && i * primes[j] <= m; ++j) {
            isPrime[i * primes[j]] = false;
            if (i % primes[j] == 0) { // 线性之处
                break;
            }
        }
    }
}
```

线性筛还可以用来预处理出一系列数论函数的值。

## 逆元

因为除法在取模意义下不封闭，因此如果我们想在取模意义下做“除法”，就要用到逆元。

如果 $a \cdot a^{-1} \equiv 1 \pmod p$，那么我们就将 $a^{-1}$ 记作 $a$ 在 模 $p$ 意义下的逆元。

求逆元的方法有若干种。

### 费马小定理

如果 $p$ 是质数，那么 $a^{p - 1} \equiv 1 \pmod p$。所以 $a \cdot a^{p-2} \equiv 1 \pmod p$，即 $a$ 在模 $p$ 意义下的逆元是 $a^{p - 2}$。可以用快速幂在 $O(\log n)$ 的时间复杂度内求出 $a$ 的逆元。

### 线性求逆元

考虑取模的定义式：
$$
p \bmod a = p - \lfloor \frac{p}{a} \rfloor \times a
$$
将这个式子放在模 $p$ 同余系下：
$$
p \bmod a \equiv -\lfloor \frac{p}{a} \rfloor \times a \pmod p
$$
两边同时乘以 $(p \bmod a)^{-1} \cdot a^{-1}$：
$$
a^{-1} \equiv -\lfloor \frac{p}{a} \rfloor \times (p \bmod a)^{-1} \pmod p
$$
因为 $(p \bmod a)^{-1} \lt a$，因此逆元可以线性递推得出。初始条件为 $1^{-1} = 1$。写代码的时候需注意负数取模。

代码：

```cpp
int inv[MAXN];

void get_inv(int n, int p) {
    inv[1] = 1;
    for (int i = 2; i <= n; ++i) {
        inv[i] = 1LL * (p - p / i) * inv[p % i] % p;
    }
}
```

# 积性函数与线性筛

## 积性函数

设 $f$ 是一个定义域为 $\mathrm{N}^+$ 的函数，如果对于任意两个互质的正整数 $a, b$ 都满足 $f(a \cdot b) = f(a) \cdot f(b)$，那么称 $f$ 为积性函数。如果对于任意两个正整数（不要求互质）都满足 $f(a \cdot b) = f(a) \cdot f(b)$，那么称 $f$ 为完全积性函数。

### 积性函数的性质

1. 对于任意积性函数 $f$ 都有 $f(1) = 1$。证明略。

2. 对于一个大于 $1$ 的正整数 $n$，设$ n = \prod p_i^{a_i}$，其中 $p_i$ 为互不相同的素数，那么对于积性函数 $f$ 有
   $$
   f(n) = f(\prod p_i^{a_i}) = \prod f(p_i^{a_i})
   $$
   对于完全积性函数 $f$ 还有
   $$
   f(n) = \prod f(p_i)^{a_i}
   $$
   证明略。 

### 常见的积性函数

#### 欧拉函数

定义欧拉函数 $\varphi(n)$ 为小于 $n$ 的正整数中与 $n$ 互质的数的个数。

根据容斥原理可以推出：

若
$$
n = \prod_{i = 1}^{m} p_i^{a_i}
$$
其中 $p_i$ 为质数，那么
$$
\varphi(n) = n \prod_{i = 1}^{m} (1 - \frac{1}{p_i})
$$
（但是这个式子我暂时还不会证明（逃

##### 欧拉函数的性质

1. 欧拉函数是积性函数，但是不是完全积性函数。

   证明：设 $a, b$ 为两个互质的正整数，则
   $$
   \begin{aligned}
   \varphi(a) &= a \prod (1 - \frac{1}{p_{a_i}}) \\
   \varphi(b) &= b \prod (1 - \frac{1}{p_{b_i}}) \\
   \varphi(a) \varphi(b) &= a \prod p_{a_i} \cdot b \prod p_{b_i} \\
   &= ab \prod (1 - \frac{1}{p_{a_i}})(1 - \frac{1}{p_{b_i}})
   \end{aligned}
   $$
   因为 $a, b$ 互质，所以 $p_{a_1}, p_{b_2}$ 互不相同，所以 $\varphi(a)\varphi(b) = \varphi(ab)$。显然若 $a, b$ 不互质则 $\varphi(a)\varphi(b) = \varphi(ab)$，因此欧拉函数不是完全积性函数。

2. 设 $p$ 为质数， $k$ 为正整数，那么 $\varphi(p^k) = p^k - p^{k - 1}$。

   证明：我们考虑枚举小于 $p^k$ 的正整数中与 $p^k$ 不互质的数。显然这样的数都是 $p$ 的倍数。它们是：
   $$
   1 \times p, 2 \times p, \cdots, (p^{k-1} - 1) \times p
   $$
   所以 $\varphi (p^k)$ 的值为小于 $p^k$ 的正整数的个数减去这些数中与 $p^k$ 互质的数的个数。即
   $$
   (p^k - 1) - (p ^{k - 1} - 1) = p^k - p^{k - 1}
   $$
   也可以从定义式直接证明。

3. 欧拉定理：设 $a, n$ 为两个互质的正整数，那么 $a^{\varphi(n)} \equiv 1 \pmod n$。

   证明：设 $x_1, x_2, \cdots, x_{\varphi(n)}$ 是小于 $n$ 的数中与 $n$ 互质的数，显然这些数有 $\varphi(n)$ 个。令 $m_i = a \cdot x_i$。

   引理 1 ：$m_i \bmod n$ 的值互不相同。证明：设 $m_s \gt m_r$。若 $m_s \equiv m_r \pmod n$，则有 $m_s - m_r = a(x_s - x_r) = kn$，即 $n \mid a(x_s - x_r)$。但 $a \ \bot \ n$，而 $x_s - x_r \le n$，故 $n \nmid a(x_s - x_r)$，故不可能有两个 $m_i$ 模 $n$ 同余。所以 $\varphi(n)$ 个数有 $\varphi(n)$ 个不同的余数。

   引理 2：$(m_i \bmod n) \ \bot \ n$。证明：设 $r$ 为 $m_i \bmod n$ 与 $n$ 的公因数，则 $ax_i = m_i = pn + qr$，$ax_i$ 与 $n$ 不互质。但 $a \ \bot \ n$， $x_i \ \bot \  n$，所以 $ax_i \ \bot \  n$，故 $m_i \bmod n$ 与 $n$ 不可能有大于 $1$ 的公因数。

   由此可得 $m_i$ 模 $n$ 的余数是小于 $n$ 的数中与 $n$ 互质的数，即 $\{m_i \bmod n\} = \{x_i\}$。

   故
   $$
   \begin{aligned}
   &\hspace{6ex}\prod m_i \equiv \prod x_i &\pmod n \\
   &\implies a^{\varphi(n)} \prod x_i \equiv \prod x_i &\pmod n \\
   &\implies (a^{\varphi(n)} - 1) \prod x_i \equiv 0 &\pmod n \\
   \end{aligned}
   $$
   因为 $x_i$ 与 $n$ 互质，所以
   $$
   a^{\varphi(n)} - 1 \equiv 0 \pmod n
   \implies a^{\varphi(n)} \equiv 1 \pmod n
   $$
   证毕。

   费马小定理是当 $n$ 是质数的时候的欧拉定理的一种特殊情况。

4. 对于正整数 $n$，$\sum_{d \mid n} \varphi(d) = n$。

   证明：当 $n = 1$ 时，原式显然成立。

   当 $n = p^k$，其中 $p$ 是质数时，
   $$
   \begin{aligned}
    \sum_{d \mid n} \varphi(d) &= \sum_{i = 0}^{k} \varphi(p^i) \\
    &= 1 + \sum_{i = 1}^k \varphi(p^i) \\
    &= 1 + \sum_{i = 1}^k (-p^{i - 1} + p^i) \\
    &= p^i
    \end{aligned}
   $$
   当 $n = p_1^{a_1}p_2^{a_2}...p_k^{a_k}$ 时，利用积性函数的性质可以证明。

   证毕。

#### 莫比乌斯函数

// 待续

#### 其他常见积性函数

// 待续

### 线性筛

利用积性函数的性质，我们可以在线性筛质数的同时筛出积性函数的值。通用过程是考虑将所有数分为三类：

1. 质数
2. 最小质因子的指数是 $1$
3. 最小质因子的指数大于 $1$

据说还有不需要分三种情况而直接按照最小质因子的指数计算的方法，但是我没学过也还不会这里就不讲（逃

#### 欧拉函数

计算 $\varphi(n)$ 的值，按照上面的三类考虑：

1. $n$ 是质数时，$\varphi(n) = n - 1$
2. $n$ 的最小质因子 $q$ 的指数是 $1$ 时，$\varphi(n) = \varphi(m \cdot q) = \varphi(m) \cdot \varphi(q)$
3. $n$ 的最小质因子 $q$ 的指数大于 $1$ 时，$\varphi(n) = \varphi(m \cdot q) = q \cdot \varphi(m)$。证明：$\varphi(m \cdot q) = m \cdot q \prod (1- \frac{1}{p_i}) = q \cdot m \cdot \prod (1- \frac{1}{p_i}) = q \cdot \varphi(m)$。

代码：

```cpp
bool isPrime[MAXM];
int primes[7300], cnt, phi[MAXM];

void sieve() {
    std::fill(isPrime + 1, isPrime + m + 1, true);
    isPrime[0] = isPrime[1] = false;
    phi[1] = 1;
    for (int i = 2; i <= m; ++i) {
        if (isPrime[i]) {
            primes[++cnt] = i;
            phi[i] = i - 1;
        }
        for (int j = 1; j <= cnt && i * primes[j] <= m; ++j) {
            isPrime[i * primes[j]] = false;
            if (i % primes[j] == 0) {
                phi[i * primes[j]] = primes[j] * phi[i];
                break;
            }
            phi[i * primes[j]] = phi[i] * phi[primes[j]];
        }
    }
}
```



// 待补充：莫比乌斯反演



Finita la comedia.