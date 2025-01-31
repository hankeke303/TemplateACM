# 数学

## 矩阵运算

```cpp
struct Mat {
    static const LL M = 2;
    LL v[M][M];
    Mat() { memset(v, 0, sizeof v); }
    void eye() { FOR (i, 0, M) v[i][i] = 1; }
    LL* operator [] (LL x) { return v[x]; }
    const LL* operator [] (LL x) const { return v[x]; }
    Mat operator * (const Mat& B) {
        const Mat& A = *this;
        Mat ret;
        FOR (k, 0, M)
            FOR (i, 0, M) if (A[i][k])
                FOR (j, 0, M)     
                    ret[i][j] = (ret[i][j] + A[i][k] * B[k][j]) % MOD;
        return ret;
    }
    Mat pow(LL n) const {
        Mat A = *this, ret; ret.eye();
        for (; n; n >>= 1, A = A * A)
            if (n & 1) ret = ret * A;
        return ret;
    }
    Mat operator + (const Mat& B) {
        const Mat& A = *this;
        Mat ret;
        FOR (i, 0, M)
            FOR (j, 0, M)
                 ret[i][j] = (A[i][j] + B[i][j]) % MOD;
        return ret;
    }
    void prt() const {
        FOR (i, 0, M)
            FOR (j, 0, M)
                 printf("%lld%c", (*this)[i][j], j == M - 1 ? '\n' : ' ');
    }
};
```

## 筛

* 线性筛

```cpp
const LL p_max = 1E6 + 100;
LL pr[p_max], p_sz;
void get_prime() {
    static bool vis[p_max];
    FOR (i, 2, p_max) {
        if (!vis[i]) pr[p_sz++] = i;
        FOR (j, 0, p_sz) {
            if (pr[j] * i >= p_max) break;
            vis[pr[j] * i] = 1;
            if (i % pr[j] == 0) break;
        }
    }
}
```

* 线性筛+欧拉函数

```cpp
const LL p_max = 1E5 + 100;
LL phi[p_max];
void get_phi() {
    phi[1] = 1;
    static bool vis[p_max];
    static LL prime[p_max], p_sz, d;
    FOR (i, 2, p_max) {
        if (!vis[i]) {
            prime[p_sz++] = i;
            phi[i] = i - 1;
        }
        for (LL j = 0; j < p_sz && (d = i * prime[j]) < p_max; ++j) {
            vis[d] = 1;
            if (i % prime[j] == 0) {
                phi[d] = phi[i] * prime[j];
                break;
            }
            else phi[d] = phi[i] * (prime[j] - 1);
        }
    }
}
```


* 线性筛+莫比乌斯函数

```cpp
const LL p_max = 1E5 + 100;
LL mu[p_max];
void get_mu() {
    mu[1] = 1;
    static bool vis[p_max];
    static LL prime[p_max], p_sz, d;
    FOR (i, 2, p_max) {
        if (!vis[i]) {
            prime[p_sz++] = i;
            mu[i] = -1;
        }
        for (LL j = 0; j < p_sz && (d = i * prime[j]) < p_max; ++j) {
            vis[d] = 1;
            if (i % prime[j] == 0) {
                mu[d] = 0;
                break;
            }
            else mu[d] = -mu[i];
        }
    }
}
```

## 亚线性筛

### min_25

```cpp
namespace min25 {
    const int M = 1E6 + 100;
    LL B, N;

    // g(x)
    inline LL pg(LL x) { return 1; }
    inline LL ph(LL x) { return x % MOD; }
    // Sum[g(i),{x,2,x}]
    inline LL psg(LL x) { return x % MOD - 1; }
    inline LL psh(LL x) {
        static LL inv2 = (MOD + 1) / 2;
        x %= MOD;
        return x * (x + 1) % MOD * inv2 % MOD - 1;
    }
    // f(pp=p^k)
    inline LL fpk(LL p, LL e, LL pp) { return (pp - pp / p) % MOD; }
    // f(p) = fgh(g(p), h(p))
    inline LL fgh(LL g, LL h) { return h - g; }

    LL pr[M], pc, sg[M], sh[M];
    void get_prime(LL n) {
        static bool vis[M]; pc = 0;
        FOR (i, 2, n + 1) {
            if (!vis[i]) {
                pr[pc++] = i;
                sg[pc] = (sg[pc - 1] + pg(i)) % MOD;
                sh[pc] = (sh[pc - 1] + ph(i)) % MOD;
            }
            FOR (j, 0, pc) {
                if (pr[j] * i > n) break;
                vis[pr[j] * i] = 1;
                if (i % pr[j] == 0) break;
            }
        }
    }

    LL w[M];
    LL id1[M], id2[M], h[M], g[M];
    inline LL id(LL x) { return x <= B ? id1[x] : id2[N / x]; }

    LL go(LL x, LL k) {
        if (x <= 1 || (k >= 0 && pr[k] > x)) return 0;
        LL t = id(x);
        LL ans = fgh((g[t] - sg[k + 1]), (h[t] - sh[k + 1]));
        FOR (i, k + 1, pc) {
            LL p = pr[i];
            if (p * p > x) break;
            ans -= fgh(pg(p), ph(p));
            for (LL pp = p, e = 1; pp <= x; ++e, pp = pp * p)
                ans += fpk(p, e, pp) * (1 + go(x / pp, i)) % MOD;
        }
        return ans % MOD;
    }

    LL solve(LL _N) {
        N = _N;
        B = sqrt(N + 0.5);
        get_prime(B);
        int sz = 0;
        for (LL l = 1, v, r; l <= N; l = r + 1) {
            v = N / l; r = N / v;
            w[sz] = v; g[sz] = psg(v); h[sz] = psh(v);
            if (v <= B) id1[v] = sz; else id2[r] = sz;
            sz++;
        }
        FOR (k, 0, pc) {
            LL p = pr[k];
            FOR (i, 0, sz) {
                LL v = w[i]; if (p * p > v) break;
                LL t = id(v / p);
                g[i] = (g[i] - (g[t] - sg[k]) * pg(p)) % MOD;
                h[i] = (h[i] - (h[t] - sh[k]) * ph(p)) % MOD;
            }
        }
        return (go(N, -1) % MOD + MOD + 1) % MOD;
    }
    pair<LL, LL> sump(LL l, LL r) {
        return {h[id(r)] - h[id(l - 1)],
                g[id(r)] - g[id(l - 1)]};
    }
}
```

### 杜教筛

求 $S(n)=\sum_{i=1}^n f(i)$，其中 $f$ 是一个积性函数。

构造一个积性函数 $g$，那么由 $(f*g)(n)=\sum_{d|n}f(d)g(\frac{n}{d})$，得到 $f(n)=(f*g)(n)-\sum_{d|n,d<n}f(d)g(\frac{n}{d})$。

\begin{eqnarray}
g(1)S(n)&=&\sum_{i=1}^n (f*g)(i)-\sum_{i= 1}^{n}\sum_{d|i,d<i}f(d)g(\frac{n}{d}) \\
&\overset{t=\frac{i}{d}}{=}& \sum_{i=1}^n (f*g)(i)-\sum_{t=2}^{n} g(t) S(\lfloor \frac{n}{t} \rfloor)
\end{eqnarray}


当然，要能够由此计算 $S(n)$，会对 $f,g$ 提出一些要求：

+ $f*g$ 要能够快速求前缀和。
+ $g$  要能够快速求分段和（前缀和）。
+ 对于正常的积性函数 $g(1)=1$，所以不会有什么问题。

在预处理 $S(n)$ 前 $n^{\frac{2}{3}}$ 项的情况下复杂度是 $O(n^{\frac{2}{3}})$。

```cpp
namespace dujiao {
    const int M = 5E6;
    LL f[M] = {0, 1};
    void init() {
        static bool vis[M];
        static LL pr[M], p_sz, d;
        FOR (i, 2, M) {
            if (!vis[i]) { pr[p_sz++] = i; f[i] = -1; }
            FOR (j, 0, p_sz) {
                if ((d = pr[j] * i) >= M) break;
                vis[d] = 1;
                if (i % pr[j] == 0) {
                    f[d] = 0;
                    break;
                } else f[d] = -f[i];
            }
        }
        FOR (i, 2, M) f[i] += f[i - 1];
    }
    inline LL s_fg(LL n) { return 1; }
    inline LL s_g(LL n) { return n; }

    LL N, rd[M];
    bool vis[M];
    LL go(LL n) {
        if (n < M) return f[n];
        LL id = N / n;
        if (vis[id]) return rd[id];
        vis[id] = true;
        LL& ret = rd[id] = s_fg(n);
        for (LL l = 2, v, r; l <= n; l = r + 1) {
            v = n / l; r = n / v;
            ret -= (s_g(r) - s_g(l - 1)) * go(v);
        }
        return ret;
    }
    LL solve(LL n) {
        N = n;
        memset(vis, 0, sizeof vis);
        return go(n);
    }
}
```



## 素数测试

+ 前置： 快速乘、快速幂
+ int 范围内只需检查 2, 7, 61
+ long long 范围 2, 325, 9375, 28178, 450775, 9780504, 1795265022
+ 3E15内 2, 2570940, 880937, 610386380, 4130785767
+ 4E13内 2, 2570940, 211991001, 3749873356
+ http://miller-rabin.appspot.com/

```cpp
bool checkQ(LL a, LL n) {
    if (n == 2) return 1;
    if (n == 1 || !(n & 1)) return 0;
    LL d = n - 1;
    while (!(d & 1)) d >>= 1;
    LL t = bin(a, d, n);  // 不一定需要快速乘
    while (d != n - 1 && t != 1 && t != n - 1) {
        t = mul(t, t, n);
        d <<= 1;
    }
    return t == n - 1 || d & 1;
}

bool primeQ(LL n) {
    static vector<LL> t = {2, 325, 9375, 28178, 450775, 9780504, 1795265022};
    if (n <= 1) return false;
    for (LL k: t) if (!checkQ(k, n)) return false;
    return true;
}
```

## Pollard-Rho

```cpp
mt19937 mt(time(0));
LL pollard_rho(LL n, LL c) {
    LL x = uniform_int_distribution<LL>(1, n - 1)(mt), y = x;
    auto f = [&](LL v) { LL t = mul(v, v, n) + c; return t < n ? t : t - n; };
    while (1) {
        x = f(x); y = f(f(y));
        if (x == y) return n;
        LL d = gcd(abs(x - y), n);
        if (d != 1) return d;
    }
}

LL fac[100], fcnt;
void get_fac(LL n, LL cc = 19260817) {
    if (n == 4) { fac[fcnt++] = 2; fac[fcnt++] = 2; return; }
    if (primeQ(n)) { fac[fcnt++] = n; return; }
    LL p = n;
    while (p == n) p = pollard_rho(n, --cc);
    get_fac(p); get_fac(n / p);
}

void go_fac(LL n) { fcnt = 0; if (n > 1) get_fac(n); }
```

## BM 线性递推

```cpp
namespace BerlekampMassey {
    using V = vector<LL>;
    inline void up(LL& a, LL b) { (a += b) %= MOD; }
    V mul(const V&a, const V& b, const V& m, int k) {
        V r; r.resize(2 * k - 1);
        FOR (i, 0, k) FOR (j, 0, k) up(r[i + j], a[i] * b[j]);
        FORD (i, k - 2, -1) {
            FOR (j, 0, k) up(r[i + j], r[i + k] * m[j]);
            r.pop_back();
        }
        return r;
    }

    V pow(LL n, const V& m) {
        int k = (int) m.size() - 1; assert (m[k] == -1 || m[k] == MOD - 1);
        V r(k), x(k); r[0] = x[1] = 1;
        for (; n; n >>= 1, x = mul(x, x, m, k))
            if (n & 1) r = mul(x, r, m, k);
        return r;
    }

    LL go(const V& a, const V& x, LL n) {
        // a: (-1, a1, a2, ..., ak).reverse
        // x: x1, x2, ..., xk
        // x[n] = sum[a[i]*x[n-i],{i,1,k}]
        int k = (int) a.size() - 1;
        if (n <= k) return x[n - 1];
        if (a.size() == 2) return x[0] * bin(a[0], n - 1, MOD) % MOD;
        V r = pow(n - 1, a);
        LL ans = 0;
        FOR (i, 0, k) up(ans, r[i] * x[i]);
        return (ans + MOD) % MOD;
    }

    V BM(const V& x) {
        V C{-1}, B{-1};
        LL L = 0, m = 1, b = 1;
        FOR (n, 0, x.size()) {
            LL d = 0;
            FOR (i, 0, L + 1) up(d, C[i] * x[n - i]);
            if (d == 0) { ++m; continue; }
            V T = C;
            LL c = MOD - d * get_inv(b, MOD) % MOD;
            FOR (_, C.size(), B.size() + m) C.push_back(0);
            FOR (i, 0, B.size()) up(C[i + m], c * B[i]);
            if (2 * L > n) { ++m; continue; }
            L = n + 1 - L; B.swap(T); b = d; m = 1;
        }
        reverse(C.begin(), C.end());
        return C;
    }
}
```

## 扩展欧几里得

* 求 $ax+by=gcd(a,b)$ 的一组解
* 如果 $a$ 和 $b$ 互素，那么 $x$ 是 $a$ 在模 $b$ 下的逆元
* 注意 $x$ 和 $y$ 可能是负数

```cpp
LL ex_gcd(LL a, LL b, LL &x, LL &y) {
    if (b == 0) { x = 1; y = 0; return a; }
    LL ret = ex_gcd(b, a % b, y, x);
    y -= a / b * x;
    return ret;
}
```

+ 卡常欧几里得

```cpp
inline int ctz(LL x) { return __builtin_ctzll(x); }
LL gcd(LL a, LL b) {
    if (!a) return b; if (!b) return a;
    int t = ctz(a | b);
    a >>= ctz(a);
    do {
        b >>= ctz(b);
        if (a > b) swap(a, b);
        b -= a;
    } while (b);
    return a << t;
}
```

## 类欧几里得

* $m = \lfloor \frac{an+b}{c} \rfloor$.
* $f(a,b,c,n)=\sum_{i=0}^n\lfloor\frac{ai+b}{c}\rfloor$: 当 $a \ge c$ or $b \ge c$ 时，$f(a,b,c,n)=(\frac{a}{c})n(n+1)/2+(\frac{b}{c})(n+1)+f(a \bmod c,b \bmod c,c,n)$；否则 $f(a,b,c,n)=nm-f(c,c-b-1,a,m-1)$。
* $g(a,b,c,n)=\sum_{i=0}^n i \lfloor\frac{ai+b}{c}\rfloor$: 当 $a \ge c$ or $b \ge c$ 时，$g(a,b,c,n)=(\frac{a}{c})n(n+1)(2n+1)/6+(\frac{b}{c})n(n+1)/2+g(a \bmod c,b \bmod c,c,n)$；否则 $g(a,b,c,n)=\frac{1}{2} (n(n+1)m-f(c,c-b-1,a,m-1)-h(c,c-b-1,a,m-1))$。
* $h(a,b,c,n)=\sum_{i=0}^n\lfloor \frac{ai+b}{c} \rfloor^2$: 当 $a \ge c$ or $b \ge c$ 时，$h(a,b,c,n)=(\frac{a}{c})^2 n(n+1)(2n+1)/6 +(\frac{b}{c})^2 (n+1)+(\frac{a}{c})(\frac{b}{c})n(n+1)+h(a \bmod c, b \bmod c,c,n)+2(\frac{a}{c})g(a \bmod c,b \bmod c,c,n)+2(\frac{b}{c})f(a \bmod c,b \bmod c,c,n)$；否则 $h(a,b,c,n)=nm(m+1)-2g(c,c-b-1,a,m-1)-2f(c,c-b-1,a,m-1)-f(a,b,c,n)$。


## 逆元

* 如果 $p$ 不是素数，使用拓展欧几里得

* 前置模板：快速幂 / 扩展欧几里得

```cpp
inline LL get_inv(LL x, LL p) { return bin(x, p - 2, p); }
LL get_inv(LL a, LL M) {
    static LL x, y;
    assert(exgcd(a, M, x, y) == 1);
    return (x % M + M) % M;
}
```

* 预处理 1~n 的逆元

```cpp
LL inv[N];
void inv_init(LL n, LL p) {
    inv[1] = 1;
    FOR (i, 2, n)
        inv[i] = (p - p / i) * inv[p % i] % p;
}
```

* 预处理阶乘及其逆元

```cpp
LL invf[M], fac[M] = {1};
void fac_inv_init(LL n, LL p) {
    FOR (i, 1, n)
        fac[i] = i * fac[i - 1] % p;
    invf[n - 1] = bin(fac[n - 1], p - 2, p);
    FORD (i, n - 2, -1)
        invf[i] = invf[i + 1] * (i + 1) % p;
}
```

## 组合数

+ 如果数较小，模较大时使用逆元
+ 前置模板：逆元-预处理阶乘及其逆元

```cpp
inline LL C(LL n, LL m) { // n >= m >= 0
    return n < m || m < 0 ? 0 : fac[n] * invf[m] % MOD * invf[n - m] % MOD;
}
```

+ 如果模数较小，数字较大，使用 Lucas 定理
+ 前置模板可选1：求组合数    （如果使用阶乘逆元，需`fac_inv_init(MOD, MOD);`）
+ 前置模板可选2：模数不固定下使用，无法单独使用。

```cpp
LL C(LL n, LL m) { // m >= n >= 0
    if (m - n < n) n = m - n;
    if (n < 0) return 0;
    LL ret = 1;
    FOR (i, 1, n + 1)
        ret = ret * (m - n + i) % MOD * bin(i, MOD - 2, MOD) % MOD;
    return ret;
}
```

```cpp
LL Lucas(LL n, LL m) { // m >= n >= 0
    return m ? C(n % MOD, m % MOD) * Lucas(n / MOD, m / MOD) % MOD : 1;
}
```

* 组合数预处理

```cpp
LL C[M][M];
void init_C(int n) {
    FOR (i, 0, n) {
        C[i][0] = C[i][i] = 1;
        FOR (j, 1, i)
            C[i][j] = (C[i - 1][j] + C[i - 1][j - 1]) % MOD;
    }
}
```

## 斯特灵数

### 第一类斯特灵数

+ 绝对值是 $n$ 个元素划分为 $k$ 个环排列的方案数。
+ $s(n,k)=s(n-1,k-1)+(n-1)s(n-1,k)$

### 第二类斯特灵数

+ $n$  个元素划分为 $k$ 个等价类的方案数
+ $S(n, k)=S(n-1,k-1)+kS(n-1, k)$

```cpp
S[0][0] = 1;
FOR (i, 1, N)
    FOR (j, 1, i + 1) S[i][j] = (S[i - 1][j - 1] + j * S[i - 1][j]) % MOD;
```

## simpson 自适应积分

```cpp
LD simpson(LD l, LD r) {
    LD c = (l + r) / 2;
    return (f(l) + 4 * f(c) + f(r)) * (r - l) / 6;
}

LD asr(LD l, LD r, LD eps, LD S) {
    LD m = (l + r) / 2;
    LD L = simpson(l, m), R = simpson(m, r);
    if (fabs(L + R - S) < 15 * eps) return L + R + (L + R - S) / 15;
    return asr(l, m, eps / 2, L) + asr(m, r, eps / 2, R);
}

LD asr(LD l, LD r, LD eps) { return asr(l, r, eps, simpson(l, r)); }
```

+ FWT

```cpp
template<typename T>
void fwt(LL a[], int n, T f) {
    for (int d = 1; d < n; d *= 2)
        for (int i = 0, t = d * 2; i < n; i += t)
            FOR (j, 0, d)
                 f(a[i + j], a[i + j + d]);
}

auto f = [](LL& a, LL& b) { // xor
        LL x = a, y = b;
        a = (x + y) % MOD;
        b = (x - y + MOD) % MOD;
};
```

## 快速乘

```cpp
LL mul(LL a, LL b, LL m) {
    LL ret = 0;
    while (b) {
        if (b & 1) {
            ret += a;
            if (ret >= m) ret -= m;
        }
        a += a;
        if (a >= m) a -= m;
        b >>= 1;
    }
    return ret;
}
```

+ O(1)

```cpp
LL mul(LL u, LL v, LL p) {
    return (u * v - LL((long double) u * v / p) * p + p) % p;
}
LL mul(LL u, LL v, LL p) { // 卡常
    LL t = u * v - LL((long double) u * v / p) * p;
    return t < 0 ? t + p : t;
}
```

## 快速幂

* 如果模数是素数，则可在函数体内加上`n %= MOD - 1;`（费马小定理）。

```cpp
LL bin(LL x, LL n, LL MOD) {
    LL ret = MOD != 1;
    for (x %= MOD; n; n >>= 1, x = x * x % MOD)
        if (n & 1) ret = ret * x % MOD;
    return ret;
}
```

* 防爆 LL
* 前置模板：快速乘

```cpp
LL bin(LL x, LL n, LL MOD) {
    LL ret = MOD != 1;
    for (x %= MOD; n; n >>= 1, x = mul(x, x, MOD))
        if (n & 1) ret = mul(ret, x, MOD);
    return ret;
}
```

## 高斯消元

### 简略版（只要唯一解，不需要求解自由变量的个数）

```cpp
const double EPS = 1E-9;
int n;
vector<vector<double> > a(n, vector<double>(n));

double det = 1;
for (int i = 0; i < n; ++i) {
  int k = i;
  for (int j = i + 1; j < n; ++j)
    if (abs(a[j][i]) > abs(a[k][i])) k = j;
  if (abs(a[k][i]) < EPS) {
    det = 0;
    break;
  }
  swap(a[i], a[k]);
  if (i != k) det = -det;
  det *= a[i][i];
  for (int j = i + 1; j < n; ++j) a[i][j] /= a[i][i];
  for (int j = 0; j < n; ++j)
    if (j != i && abs(a[j][i]) > EPS)
      for (int k = i + 1; k < n; ++k) a[j][k] -= a[i][k] * a[j][i];
}

cout << det;
```

### 完整版

* n - 方程个数，m - 变量个数， a 是 n \* \(m + 1\) 的增广矩阵，free 是否为自由变量
* 返回自由变量个数，-1 无解

* 浮点数版本

```cpp
typedef double LD;
const LD eps = 1E-10;
const int maxn = 2000 + 10;

int n, m;
LD a[maxn][maxn], x[maxn];
bool free_x[maxn];

inline int sgn(LD x) { return (x > eps) - (x < -eps); }

int gauss(LD a[maxn][maxn], int n, int m) {
    memset(free_x, 1, sizeof free_x); memset(x, 0, sizeof x);
    int r = 0, c = 0;
    while (r < n && c < m) {
        int m_r = r;
        FOR (i, r + 1, n)
            if (fabs(a[i][c]) > fabs(a[m_r][c])) m_r = i;
        if (m_r != r)
            FOR (j, c, m + 1)
                 swap(a[r][j], a[m_r][j]);
        if (!sgn(a[r][c])) {
            a[r][c] = 0;
            ++c;
            continue;
        }
        FOR (i, r + 1, n)
            if (a[i][c]) {
                LD t = a[i][c] / a[r][c];
                FOR (j, c, m + 1) a[i][j] -= a[r][j] * t;
            }
        ++r; ++c;
    }
    FOR (i, r, n)
        if (sgn(a[i][m])) return -1;
    if (r < m) {
        FORD (i, r - 1, -1) {
            int f_cnt = 0, k = -1;
            FOR (j, 0, m)
                if (sgn(a[i][j]) && free_x[j]) {
                    ++f_cnt;
                    k = j;
                }
            if(f_cnt > 0) continue;
            LD s = a[i][m];
            FOR (j, 0, m)
                if (j != k) s -= a[i][j] * x[j];
            x[k] = s / a[i][k];
            free_x[k] = 0;
        }
        return m - r;
    }
    FORD (i, m - 1, -1) {
        LD s = a[i][m];
        FOR (j, i + 1, m)
            s -= a[i][j] * x[j];
        x[i] = s / a[i][i];
    }
    return 0;
}
```

+ 数据

```
3 4
1 1 -2 2
2 -3 5 1
4 -1 1 5
5 0 -1 7
// many

3 4
1 1 -2 2
2 -3 5 1
4 -1 -1 5
5 0 -1 0 2
// no

3 4
1 1 -2 2
2 -3 5 1
4 -1 1 5
5 0 1 0 7
// one
```

## 质因数分解

hkk 注：感觉像这样枚举 sqrt 以内的所有质数很慢……hkk 一般是在筛素数的时候记录下每个数的最小质因子，然后不断地除下去，复杂度 $O(\log)$。

* 前置模板：素数筛

* 带指数

```cpp
LL factor[30], f_sz, factor_exp[30];
void get_factor(LL x) {
    f_sz = 0;
    LL t = sqrt(x + 0.5);
    for (LL i = 0; pr[i] <= t; ++i)
        if (x % pr[i] == 0) {
            factor_exp[f_sz] = 0;
            while (x % pr[i] == 0) {
                x /= pr[i];
                ++factor_exp[f_sz];
            }
            factor[f_sz++] = pr[i];
        }
    if (x > 1) {
        factor_exp[f_sz] = 1;
        factor[f_sz++] = x;
    }
}
```

* 不带指数

```cpp
LL factor[30], f_sz;
void get_factor(LL x) {
    f_sz = 0;
    LL t = sqrt(x + 0.5);
    for (LL i = 0; pr[i] <= t; ++i)
        if (x % pr[i] == 0) {
            factor[f_sz++] = pr[i];
            while (x % pr[i] == 0) x /= pr[i];
        }
    if (x > 1) factor[f_sz++] = x;
}
```

## 原根

* 定义：设m是正整数，a是整数，若a模m的阶等于φ(m)，则称a为模m的一个原根。（其中φ(m)表示m的欧拉函数）
* 性质：假设一个数 $g$ 是 $P$ 的原根，那么 $g^i \bmod P$ 的结果两两不同，且有 $1<g<P$，$0<i<P$，归根到底就是 $g^{P-1} = 1 \pmod P$ 当且仅当指数为 $P-1$ 的时候成立(这里 $P$ 是素数)。简单来说，$g^i \bmod p ≠ g^j \bmod p$（$p$为素数），其中 $i\neq j$且 $i, j$ 介于 $1$ 至 $p-1$ 之间，则 $g$ 为 $p$ 的原根。
* 前置模板：素数筛，快速幂，分解质因数
* 要求 p 为质数

```cpp
LL find_smallest_primitive_root(LL p) {
    get_factor(p - 1);
    FOR (i, 2, p) {
        bool flag = true;
        FOR (j, 0, f_sz)
            if (bin(i, (p - 1) / factor[j], p) == 1) {
                flag = false;
                break;
            }
        if (flag) return i;
    }
    assert(0); return -1;
}
```

## 公式

### 一些数论公式

- 当 $x\geq\phi(p)$ 时有 $a^x\equiv a^{x \; mod \; \phi(p) + \phi(p)}\pmod p$
- $\mu^2(n)=\sum_{d^2|n} \mu(d)$
- $\sum_{d|n} \varphi(d)=n$
- $\sum_{d|n} 2^{\omega(d)}=\sigma_0(n^2)$，其中 $\omega$ 是不同素因子个数
- $\sum_{d|n} \mu^2(d)=2^{\omega(d)}$

### 一些数论函数求和的例子

+ $\sum_{i=1}^n i[gcd(i, n)=1] = \frac {n \varphi(n) + [n=1]}{2}$
+ $\sum_{i=1}^n \sum_{j=1}^m [gcd(i,j)=x]=\sum_d \mu(d) \lfloor \frac n {dx} \rfloor  \lfloor \frac m {dx} \rfloor$
+ $\sum_{i=1}^n \sum_{j=1}^m gcd(i, j) = \sum_{i=1}^n \sum_{j=1}^m \sum_{d|gcd(i,j)} \varphi(d) = \sum_{d} \varphi(d) \lfloor \frac nd \rfloor \lfloor \frac md \rfloor$
+ $S(n)=\sum_{i=1}^n \mu(i)=1-\sum_{i=1}^n \sum_{d|i,d < i}\mu(d) \overset{t=\frac id}{=} 1-\sum_{t=2}^nS(\lfloor \frac nt \rfloor)$
  + 利用 $[n=1] = \sum_{d|n} \mu(d)$
+ $S(n)=\sum_{i=1}^n \varphi(i)=\sum_{i=1}^n i-\sum_{i=1}^n \sum_{d|i,d<i} \varphi(i)\overset{t=\frac id}{=} \frac {i(i+1)}{2} - \sum_{t=2}^n S(\frac n t)$
  + 利用 $n = \sum_{d|n} \varphi(d)$
+ $\sum_{i=1}^n \mu^2(i) = \sum_{i=1}^n \sum_{d^2|n} \mu(d)=\sum_{d=1}^{\lfloor \sqrt n \rfloor}\mu(d) \lfloor \frac n {d^2} \rfloor$ 
+ $\sum_{i=1}^n \sum_{j=1}^n gcd^2(i, j)= \sum_{d} d^2 \sum_{t} \mu(t) \lfloor \frac n{dt} \rfloor ^2 \\
  \overset{x=dt}{=} \sum_{x} \lfloor \frac nx \rfloor ^ 2 \sum_{d|x} d^2 \mu(\frac xd)$
+ $\sum_{i=1}^n \varphi(i)=\frac 12 \sum_{i=1}^n \sum_{j=1}^n [i \perp j] - 1=\frac 12 \sum_{i=1}^n \mu(i) \cdot\lfloor \frac n i \rfloor ^2-1$

### 斐波那契数列性质

- $F_{a+b}=F_{a-1} \cdot F_b+F_a \cdot F_{b+1}$
- $F_1+F_3+\dots +F_{2n-1} = F_{2n},F_2 + F_4 + \dots + F_{2n} = F_{2n + 1} - 1$
- $\sum_{i=1}^n F_i = F_{n+2} - 1$
- $\sum_{i=1}^n F_i^2 = F_n \cdot F_{n+1}$
- $F_n^2=(-1)^{n-1} + F_{n-1} \cdot F_{n+1}$
- $gcd(F_a, F_b)=F_{gcd(a, b)}$
- 模 $n$ 周期（皮萨诺周期）
  - $\pi(p^k) = p^{k-1} \pi(p)$
  - $\pi(nm) = lcm(\pi(n), \pi(m)), \forall n \perp m$
  - $\pi(2)=3, \pi(5)=20$
  - $\forall p \equiv \pm 1\pmod {10}, \pi(p)|p-1$
  - $\forall p \equiv \pm 2\pmod {5}, \pi(p)|2p+2$

### 常见生成函数

+ $(1+ax)^n=\sum_{k=0}^n \binom {n}{k} a^kx^k$
+ $\dfrac{1-x^{r+1}}{1-x}=\sum_{k=0}^nx^k$
+ $\dfrac1{1-ax}=\sum_{k=0}^{\infty}a^kx^k$
+ $\dfrac 1{(1-x)^2}=\sum_{k=0}^{\infty}(k+1)x^k$
+ $\dfrac1{(1-x)^n}=\sum_{k=0}^{\infty} \binom{n+k-1}{k}x^k$
+ $e^x=\sum_{k=0}^{\infty}\dfrac{x^k}{k!}$
+ $\ln(1+x)=\sum_{k=0}^{\infty}\dfrac{(-1)^{k+1}}{k}x^k$

### 佩尔方程

若一个丢番图方程具有以下的形式：$x^2 - ny^2= 1$。且 $n$ 为正整数，则称此二元二次不定方程为**佩尔方程**。

若 $n$ 是完全平方数，则这个方程式只有平凡解 $(\pm 1,0)$（实际上对任意的 $n$，$(\pm 1,0)$ 都是解）。对于其余情况，拉格朗日证明了佩尔方程总有非平凡解。而这些解可由 $\sqrt{n}$ 的连分数求出。

$x = [a_0; a_1, a_2, a_3]=x = a_0 + \cfrac{1}{a_1 + \cfrac{1}{a_2 + \cfrac{1}{a_3 + \cfrac{1}{\ddots\,}}}}$

设 $\tfrac{p_i}{q_i}$ 是 $\sqrt{n}$ 的连分数表示：$[a_{0}; a_{1}, a_{2}, a_{3}, \,\ldots ]$ 的渐近分数列，由连分数理论知存在 $i$ 使得 $(p_i,q_i)$ 为佩尔方程的解。取其中最小的 $i$，将对应的 $(p_i,q_i)$ 称为佩尔方程的基本解，或最小解，记作 $(x_1,y_1)$，则所有的解 $(x_i,y_i)$ 可表示成如下形式：$x_{i}+y_{i}{\sqrt  n}=(x_{1}+y_{1}{\sqrt  n})^{i}$。或者由以下的递回关系式得到：

$\displaystyle x_{i+1} = x_1 x_i + n y_1 y_i$, $\displaystyle y_{{i+1}}=x_{1}y_{i}+y_{1}x_{i}$。

**但是：**佩尔方程千万不要去推（虽然推起来很有趣，但结果不一定好看，会是两个式子）。记住佩尔方程结果的形式通常是 $a_n=ka_{n−1}−a_{n−2}$（$a_{n−2}$ 前的系数通常是 $−1$）。暴力 / 凑出两个基础解之后加上一个 $0$，容易解出 $k$ 并验证。

### Burnside & Polya

+ $|X/G|={\frac  {1}{|G|}}\sum _{{g\in G}}|X^{g}|$

注：$X^g$ 是 $g$ 下的不动点数量，也就是说有多少种东西用 $g$ 作用之后可以保持不变。

+ $|Y^X/G| = \frac{1}{|G|}\sum_{g \in G} m^{c(g)}$

注：用 $m$ 种颜色染色，然后对于某一种置换 $g$，有 $c(g)$ 个置换环，为了保证置换后颜色仍然相同，每个置换环必须染成同色。

### 皮克定理

$2S = 2a+b-2$

+ $S$ 多边形面积
+ $a$ 多边形内部点数
+ $b$ 多边形边上点数

### 莫比乌斯反演

+ $g(n) = \sum_{d|n} f(d) \Leftrightarrow f(n) = \sum_{d|n} \mu (d) g( \frac{n}{d})$
+ $f(n)=\sum_{n|d}g(d) \Leftrightarrow g(n)=\sum_{n|d} \mu(\frac{d}{n}) f(d)$

### 低阶等幂求和

+ $\sum_{i=1}^{n} i^{1} = \frac{n(n+1)}{2} = \frac{1}{2}n^2 +\frac{1}{2} n$
+ $\sum_{i=1}^{n} i^{2} = \frac{n(n+1)(2n+1)}{6} = \frac{1}{3}n^3 + \frac{1}{2}n^2 + \frac{1}{6}n$
+ $\sum_{i=1}^{n} i^{3} = \left[\frac{n(n+1)}{2}\right]^{2} = \frac{1}{4}n^4 + \frac{1}{2}n^3 + \frac{1}{4}n^2$
+ $\sum_{i=1}^{n} i^{4} = \frac{n(n+1)(2n+1)(3n^2+3n-1)}{30} = \frac{1}{5}n^5 + \frac{1}{2}n^4 + \frac{1}{3}n^3 - \frac{1}{30}n$
+ $\sum_{i=1}^{n} i^{5} = \frac{n^{2}(n+1)^{2}(2n^2+2n-1)}{12} = \frac{1}{6}n^6 + \frac{1}{2}n^5 + \frac{5}{12}n^4 - \frac{1}{12}n^2$

### 一些组合公式

+ 错排公式：$D_1=0,D_2=1,D_n=(n-1)(D_{n-1} + D_{n-2})=n!(\frac 1{2!}-\frac 1{3!}+\dots + (-1)^n\frac 1{n!})=\lfloor \frac{n!}e + 0.5 \rfloor$
+ 卡塔兰数（$n$ 对括号合法方案数，$n$ 个结点二叉树个数，$n\times n$ 方格中对角线下方的单调路径数，凸 $n+2$ 边形的三角形划分数，$n$ 个元素的合法出栈序列数）：$C_n=\frac 1{n+1}\binom {2n}n=\frac{(2n)!}{(n+1)!n!}$

## 二次剩余

- 定义：$X^2$ 在数论中，特别在同余理论里，一个整数 $X$ 对另一个整数 $p$ 的**二次剩余**（英语：Quadratic residue）指 $X$ 的平方除以 $p$ 得到的余数。当存在某个 $X$，式子 $x^2\equiv d\pmod p$ 成立时，称 $d$ 是模 $p$ 的二次剩余。

URAL 1132

```cpp
LL q1, q2, w;
struct P { // x + y * sqrt(w)
    LL x, y;
};

P pmul(const P& a, const P& b, LL p) {
    P res;
    res.x = (a.x * b.x + a.y * b.y % p * w) % p;
    res.y = (a.x * b.y + a.y * b.x) % p;
    return res;
}

P bin(P x, LL n, LL MOD) {
    P ret = {1, 0};
    for (; n; n >>= 1, x = pmul(x, x, MOD))
        if (n & 1) ret = pmul(ret, x, MOD);
    return ret;
}
LL Legendre(LL a, LL p) { return bin(a, (p - 1) >> 1, p); }

LL equation_solve(LL b, LL p) {
    if (p == 2) return 1;
    if ((Legendre(b, p) + 1) % p == 0)
        return -1;
    LL a;
    while (true) {
        a = rand() % p;
        w = ((a * a - b) % p + p) % p;
        if ((Legendre(w, p) + 1) % p == 0)
            break;
    }
    return bin({a, 1}, (p + 1) >> 1, p).x;
}

int main() {
    int T; cin >> T;
    while (T--) {
        LL a, p; cin >> a >> p;
        a = a % p;
        LL x = equation_solve(a, p);
        if (x == -1) {
            puts("No root");
        } else {
            LL y = p - x;
            if (x == y) cout << x << endl;
            else cout << min(x, y) << " " << max(x, y) << endl;
        }
    }
}
```

## （扩展）中国剩余定理

- hkk 版

```cpp
ll smod(ll x, ll P) { return x < 0 ? x + P : (x >= P ? x - P : x); }
ll fmul(ll x, ll y, ll P) {
	ll ans = 0;
	x = smod(x, P);
	for (; y; y >>= 1, x = smod(x + x, P))
		if (y & 1) ans = smod(ans + x, P);
	return ans;
}
ll exgcd(ll a, ll b, ll &x, ll &y) {
	if (!b) return x = 1, y = 0, a;
	ll t = exgcd(b, a % b, y, x);
	y -= a / b * x;
	return t;
}
ll exCRT() {
	ll P = p[1], ans = a[1];
	for (int i = 2; i <= n; ++i) {
		ll a1 = P, a2 = p[i], c = smod((a[i] - ans) % a2 + a2, a2);
		ll x, y, d = exgcd(a1, a2, x, y);
		if (a1 % d) return -1;
		x = fmul(x, c / d, a2 / d);
		ans += x * P;
		P *= a2 / d;
		ans = smod(ans % P + P, P);
	}
	return ans;
}
```

- 模板原版

+ 无解返回 -1
+ 前置模板：扩展欧几里得

```cpp
LL CRT(LL *m, LL *r, LL n) {
    if (!n) return 0;
    LL M = m[0], R = r[0], x, y, d;
    FOR (i, 1, n) {
        d = ex_gcd(M, m[i], x, y);
        if ((r[i] - R) % d) return -1;
        x = (r[i] - R) / d * x % (m[i] / d);
        // 防爆 LL
        // x = mul((r[i] - R) / d, x, m[i] / d);
        R += x * M;
        M = M / d * m[i];
        R %= M;
    }
    return R >= 0 ? R : R + M;
}

```

- 这儿还有一份 py 版，可以用高精度

```python
def exgcd(a, b):
    if not b:
        return a, 1, 0
    d, x, y = exgcd(b, a % b)
    return d, y, x - a // b * y

n = int(input())
m, a = map(int, input().split())
for i in range(1, n):
    m2, a2 = map(int, input().split())
    d, x, y = exgcd(m, m2)
    x *= (a2 - a) // d
    a += x * m
    m = m // d * m2
    a = ((a % m) + m) % m

print(a)
```

## 伯努利数和等幂求和

* 预处理逆元
* 预处理组合数
* $\sum_{i=0}^n i^k = \frac{1}{k+1} \sum_{i=0}^k \binom{k+1}{i} B_{k+1-i} (n+1)^i$.
* 也可以 $\sum_{i=0}^n i^k = \frac{1}{k+1} \sum_{i=0}^k \binom{k+1}{i} B^+_{k+1-i} n^i$。区别在于 $B^+_1 =1/2$。(心态崩了)

```cpp
namespace Bernoulli {
    const int M = 100;
    LL inv[M] = {-1, 1};
    void inv_init(LL n, LL p) {
        FOR (i, 2, n)
            inv[i] = (p - p / i) * inv[p % i] % p;
    }

    LL C[M][M];
    void init_C(int n) {
        FOR (i, 0, n) {
            C[i][0] = C[i][i] = 1;
            FOR (j, 1, i)
                C[i][j] = (C[i - 1][j] + C[i - 1][j - 1]) % MOD;
        }
    }

    LL B[M] = {1};
    void init() {
        inv_init(M, MOD);
        init_C(M);
        FOR (i, 1, M - 1) {
            LL& s = B[i] = 0;
            FOR (j, 0, i)
                s += C[i + 1][j] * B[j] % MOD;
            s = (s % MOD * -inv[i + 1] % MOD + MOD) % MOD;
        }
    }

    LL p[M] = {1};
    LL go(LL n, LL k) {
        n %= MOD;
        if (k == 0) return n;
        FOR (i, 1, k + 2)
            p[i] = p[i - 1] * (n + 1) % MOD;
        LL ret = 0;
        FOR (i, 1, k + 2)
            ret += C[k + 1][i] * B[k + 1 - i] % MOD * p[i] % MOD;
        ret = ret % MOD * inv[k + 1] % MOD;
        return ret;
    }
}
```

## 单纯形

+ 要求有基本解，也就是 x 为零向量可行
+ v 要初始化为 0，n 表示向量长度，m 表示约束个数

```cpp
// min{ b x } / max { c x }
// A x >= c   / A x <= b
// x >= 0
namespace lp {
    int n, m;
    double a[M][N], b[M], c[N], v;

    void pivot(int l, int e) {
        b[l] /= a[l][e];
        FOR (j, 0, n) if (j != e) a[l][j] /= a[l][e];
        a[l][e] = 1 / a[l][e];

        FOR (i, 0, m)
            if (i != l && fabs(a[i][e]) > 0) {
                b[i] -= a[i][e] * b[l];
                FOR (j, 0, n)
                    if (j != e) a[i][j] -= a[i][e] * a[l][j];
                a[i][e] = -a[i][e] * a[l][e];
            }
        v += c[e] * b[l];
        FOR (j, 0, n) if (j != e) c[j] -= c[e] * a[l][j];
        c[e] = -c[e] * a[l][e];
    }
    double simplex() {
        while (1) {
            v = 0;
            int e = -1, l = -1;
            FOR (i, 0, n) if (c[i] > eps) { e = i; break; }
            if (e == -1) return v;
            double t = INF;
            FOR (i, 0, m)
                if (a[i][e] > eps && t > b[i] / a[i][e]) {
                    t = b[i] / a[i][e];
                    l = i;
                }
            if (l == -1) return INF;
            pivot(l, e);
        }
    }
}
```

## 离散对数

### BSGS

+ 模数为素数

```cpp
LL BSGS(LL a, LL b, LL p) { // a^x = b (mod p)
    a %= p;
    if (!a && !b) return 1;
    if (!a) return -1;
    static map<LL, LL> mp; mp.clear();
    LL m = sqrt(p + 1.5);
    LL v = 1;
    FOR (i, 1, m + 1) {
        v = v * a % p;
        mp[v * b % p] = i;
    }
    LL vv = v;
    FOR (i, 1, m + 1) {
        auto it = mp.find(vv);
        if (it != mp.end()) return i * m - it->second;
        vv = vv * v % p;
    }
    return -1;
}
```

### exBSGS

+ 模数可以非素数

```cpp
LL exBSGS(LL a, LL b, LL p) { // a^x = b (mod p)
    a %= p; b %= p;
    if (a == 0) return b > 1 ? -1 : b == 0 && p != 1;
    LL c = 0, q = 1;
    while (1) {
        LL g = __gcd(a, p);
        if (g == 1) break;
        if (b == 1) return c;
        if (b % g) return -1;
        ++c; b /= g; p /= g; q = a / g * q % p;
    }
    static map<LL, LL> mp; mp.clear();
    LL m = sqrt(p + 1.5);
    LL v = 1;
    FOR (i, 1, m + 1) {
        v = v * a % p;
        mp[v * b % p] = i;
    }
    FOR (i, 1, m + 1) {
        q = q * v % p;
        auto it = mp.find(q);
        if (it != mp.end()) return i * m - it->second + c;
    }
    return -1;
}
```

## 数论分块

$f(i) = \lfloor \frac{n}{i} \rfloor=v$ 时 $i$ 的取值范围是 $[l,r]$。

```cpp
for (LL l = 1, v, r; l <= N; l = r + 1) {
    v = N / l; r = N / v;
}
```

## 博弈

+ Nim 游戏：每轮从若干堆石子中的一堆取走若干颗。先手必胜条件为石子数量异或和非零。
+ 阶梯 Nim 游戏：可以选择阶梯上某一堆中的若干颗向下推动一级，直到全部推下去。先手必胜条件是奇数阶梯的异或和非零（对于偶数阶梯的操作可以模仿）。
+ Anti-SG：无法操作者胜。先手必胜的条件是：
  + SG 不为 0 且某个单一游戏的 SG 大于 1 。
  + SG 为 0 且没有单一游戏的 SG 大于 1。
+ Every-SG：对所有单一游戏都要操作。先手必胜的条件是单一游戏中的最大 step 为奇数。
  + 对于终止状态 step 为 0
  + 对于 SG 为 0 的状态，step 是最大后继 step +1
  + 对于 SG 非 0 的状态，step 是最小后继 step +1
+ 树上删边：叶子 SG 为 0，非叶子结点为所有子结点的 SG 值加 1 后的异或和。

尝试：

+ 打表找规律
+ 寻找一类必胜态（如对称局面）
+ 直接博弈 dp

## 线性基

```cpp
constexpr int M = 64;
ull b[M];
bool ins(ull x) {
    for (int i = M - 1; ~i; --i) if ((x >> i) & 1) {
        if (b[i]) x ^= b[i];
        else { b[i] = x; return true; }
    }
    return false;
}
ull getmax() {
    ull ans = 0;
    for (int i = M - 1; ~i; --i) smax(ans, ans ^ b[i]);
    return ans;
}
```

## 多项式相关

### NTT

- hkk 版

```cpp
void ntt(int *a, int n, int f = 1) { // a 是要 ntt 的序列，n 是长度（2^l），f 是正向运算(1)还是逆向运算（-1）
	for (int i = 0, j = 0; i < n; ++i) {
		if (i < j) std::swap(a[i], a[j]);
		for (int l = n >> 1; (j ^= l) < l; l >>= 1) ;
	}
	for (int i = 1; i < n; i <<= 1) {
		int w = fpow(f > 0 ? G : Gi, (P - 1) / (i << 1));
		for (int j = 0; j < n; j += (i << 1))
			for (int k = 0, e = 1; k < i; ++k, e = (ll)e * w % P) {
				int x = a[j + k], y = (ll)e * a[i + j + k] % P;
				a[j + k] = smod(x + y), a[i + j + k] = smod(x - y + P);
			}
	}
	if (f < 0) for (int i = 0, p = fpow(n, P - 2); i < n; ++i) a[i] = (ll)a[i] * p % P;
}
```

- 模板原版

```cpp
LL wn[N << 2], rev[N << 2];
int NTT_init(int n_) {
    int step = 0; int n = 1;
    for ( ; n < n_; n <<= 1) ++step;
    FOR (i, 1, n)
        rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (step - 1));
    int g = bin(G, (MOD - 1) / n, MOD);
    wn[0] = 1;
    for (int i = 1; i <= n; ++i)
        wn[i] = wn[i - 1] * g % MOD;
    return n;
}

void NTT(LL a[], int n, int f) {
    FOR (i, 0, n) if (i < rev[i])
        std::swap(a[i], a[rev[i]]);
    for (int k = 1; k < n; k <<= 1) {
        for (int i = 0; i < n; i += (k << 1)) {
            int t = n / (k << 1);
            FOR (j, 0, k) {
                LL w = f == 1 ? wn[t * j] : wn[n - t * j];
                LL x = a[i + j];
                LL y = a[i + j + k] * w % MOD;
                a[i + j] = (x + y) % MOD;
                a[i + j + k] = (x - y + MOD) % MOD;
            }
        }
    }
    if (f == -1) {
        LL ninv = get_inv(n, MOD);
        FOR (i, 0, n)
            a[i] = a[i] * ninv % MOD;
    }
}
```

### FFT

+ n 需补成 2 的幂 （n 必须超过 a 和 b 的最高指数之和）

```cpp
struct Complex {
	double x, y;
	Complex(double x = 0, double y = 0) : x(x), y(y) { }
	Complex operator + (const Complex &a) const { return Complex(x + a.x, y + a.y); }
	Complex operator - (const Complex &a) const { return Complex(x - a.x, y - a.y); }
	Complex operator * (const Complex &a) const { return Complex(x * a.x - y * a.y, x * a.y + y * a.x); }
} a[N], b[N];

void fft(Complex *a, int n, int f = 1) { // a 是要 ntt 的序列，n 是长度（2^l），f 是正向运算(1)还是逆向运算（-1）
	for (int i = 0, j = 0; i < n; ++i) {
		if (i < j) std::swap(a[i], a[j]);
		for (int l = n >> 1; (j ^= l) < l; l >>= 1) ;
	}
	for (int i = 1; i < n; i <<= 1) {
		Complex w(cos(PI / i), f * sin(PI / i));
		for (int j = 0; j < n; j += (i << 1)) {
			Complex e(1, 0);
			for (int k = 0; k < i; ++k, e = e * w) {
				Complex x = a[j + k], y = e * a[i + j + k];
				a[j + k] = x + y, a[i + j + k] = x - y;
			}
		}
	}
	if (f < 0) for (int i = 0; i < n; ++i) a[i].x /= n;
}
```

### FWT

+ $C_k=\sum_{i \oplus j=k} A_i B_j$
+ FWT 完后需要先模一遍

```cpp
template<typename T>
void fwt(LL a[], int n, T f) {
    for (int d = 1; d < n; d *= 2)
        for (int i = 0, t = d * 2; i < n; i += t)
            FOR (j, 0, d)
                f(a[i + j], a[i + j + d]);
}

void AND(LL& a, LL& b) { a += b; }
void OR(LL& a, LL& b) { b += a; }
void XOR (LL& a, LL& b) {
    LL x = a, y = b;
    a = (x + y) % MOD;
    b = (x - y + MOD) % MOD;
}
void rAND(LL& a, LL& b) { a -= b; }
void rOR(LL& a, LL& b) { b -= a; }
void rXOR(LL& a, LL& b) {
    static LL INV2 = (MOD + 1) / 2;
    LL x = a, y = b;
    a = (x + y) * INV2 % MOD;
    b = (x - y + MOD) * INV2 % MOD;
}
```

+ FWT 子集卷积

```text
a[popcount(x)][x] = A[x]
b[popcount(x)][x] = B[x]
fwt(a[i]) fwt(b[i])
c[i + j][x] += a[i][x] * b[j][x]
rfwt(c[i])
ans[x] = c[popcount(x)][x]
```

### 多项式求逆

前置：NTT、取模模板

```cpp
void getinv(int *f) {
	g[0] = fpow(f[0], P - 2);
	for (int d = 2; d < (n << 1); d <<= 1) {
		int l = d << 1;
		for (int i = 0; i < d; ++i) A[i] = f[i];
		ntt(A, l, 1), ntt(g, l, 1);
		for (int i = 0; i < l; ++i) g[i] = g[i] * (2 - (ll)A[i] * g[i] % P + P) % P;
		ntt(g, l, -1);
		for (int i = d; i < l; ++i) g[i] = 0;
	}
}
```

### 分治 fft

```cpp
void cdqfft(int l, int r) { // 注意：l, r 从 0 开始编号，一般调用时取 [0, n - 1]
	if (l == r) return;
	int mid = (l + r) >> 1, L = 1;
	cdqfft(l, mid);
	while (L <= (mid - l + r - l)) L <<= 1;
	std::fill(A, A + L, 0), std::fill(B, B + L, 0);
	for (int i = 0; i < mid - l + 1; ++i) A[i] = f[i + l];
	for (int i = 1; i <= r - l; ++i) B[i] = g[i];
	ntt(A, L, 1), ntt(B, L, 1);
	for (int i = 0; i < L; ++i) A[i] = (ll)A[i] * B[i] % P;
	ntt(A, L, -1);
	for (int i = mid - l + 1; i < r - l + 1; ++i) sadd(f[i + l], A[i]);
	cdqfft(mid + 1, r);
}
```

### 多项式模板大全

来自某位大佬

```cpp
namespace Polynomial {
    using Poly = std::vector<int>;
    constexpr int P(998244353), G(3);
    inline void inc(int &x, int y) { (x += y) >= P ? x -= P : 0; }
    inline int mod(int64_t x) { return x % P; }
    inline int fpow(int x, int k = P - 2) {
        int r = 1;
        for (; k; k >>= 1, x = 1LL * x * x % P)
            if (k & 1) r = 1LL * r * x % P;
        return r;
    }
    template <int N>
    std::array<int, N> getOmega() {
        std::array<int, N> w;
        for (int i = N >> 1, x = fpow(G, (P - 1) / N); i; i >>= 1, x = 1LL * x * x % P) {
            w[i] = 1;
            for (int j = 1; j < i; j++) w[i + j] = 1LL * w[i + j - 1] * x % P;
        }
        return w;
    }
    auto w = getOmega<1 << 18>();
    Poly &operator*=(Poly &a, int b) {
        for (auto &x : a) x = 1LL * x * b % P;
        return a;
    }
    Poly operator*(Poly a, int b) { return a *= b; }
    Poly operator*(int a, Poly b) { return b * a; }
    Poly &operator/=(Poly &a, int b) { return a *= fpow(b); }
    Poly operator/(Poly a, int b) { return a /= b; }
    Poly &operator+=(Poly &a, Poly b) {
        a.resize(std::max(a.size(), b.size()));
        for (int i = 0; i < b.size(); i++) inc(a[i], b[i]);
        return a;
    }
    Poly operator+(Poly a, Poly b) { return a += b; }
    Poly &operator-=(Poly &a, Poly b) {
        a.resize(std::max(a.size(), b.size()));
        for (int i = 0; i < b.size(); i++) inc(a[i], P - b[i]);
        return a;
    }
    Poly operator-(Poly a, Poly b) { return a -= b; }
    Poly operator-(Poly a) {
        for (auto &x : a) x ? x = P - x : 0;
        return a;
    }
    Poly &operator>>=(Poly &a, int x) {
        if (x >= (int)a.size()) {
            a.clear();
        } else {
            a.erase(a.begin(), a.begin() + x);
        }
        return a;
    }
    Poly &operator<<=(Poly &a, int x) {
        a.insert(a.begin(), x, 0);
        return a;
    }
    Poly operator>>(Poly a, int x) { return a >>= x; }
    Poly operator<<(Poly a, int x) { return a <<= x; }
    inline Poly &dotEq(Poly &a, Poly b) {
        assert(a.size() == b.size());
        for (int i = 0; i < a.size(); i++) a[i] = 1LL * a[i] * b[i] % P;
        return a;
    }
    inline Poly dot(Poly a, Poly b) { return dotEq(a, b); }
    void norm(Poly &a) {
        if (!a.empty()) {
            a.resize(1 << std::__lg(a.size() * 2 - 1));
        }
    }
    void dft(int *a, int n) {
        assert((n & n - 1) == 0);
        for (int k = n >> 1; k; k >>= 1) {
            for (int i = 0; i < n; i += k << 1) {
                for (int j = 0; j < k; j++) {
                    int y = a[i + j + k];
                    a[i + j + k] = 1LL * (a[i + j] - y + P) * w[k + j] % P;
                    inc(a[i + j], y);
                }
            }
        }
    }
    void idft(int *a, int n) {
        assert((n & n - 1) == 0);
        for (int k = 1; k < n; k <<= 1) {
            for (int i = 0; i < n; i += k << 1) {
                for (int j = 0; j < k; j++) {
                    int x = a[i + j], y = 1LL * a[i + j + k] * w[k + j] % P;
                    a[i + j + k] = x - y < 0 ? x - y + P : x - y;
                    inc(a[i + j], y);
                }
            }
        }
        for (int i = 0, inv = P - (P - 1) / n; i < n; i++) a[i] = 1LL * a[i] * inv % P;
        std::reverse(a + 1, a + n);
    }
    void dft(Poly &a) { dft(a.data(), a.size()); }
    void idft(Poly &a) { idft(a.data(), a.size()); }
    // expand dft of length n to 2n
    void dftDoubling(int *x, int n) {
        std::copy_n(x, n, x + n);
        idft(x + n, n);
        for (int i = 0; i < n; i++) x[n + i] = 1LL * w[n + i] * x[n + i] % P;
        dft(x + n, n);
    }
    Poly operator*(Poly a, Poly b) {
        int len = a.size() + b.size() - 1;
        if (a.size() <= 8 || b.size() <= 8) {
            Poly c(len);
            for (size_t i = 0; i < a.size(); i++)
                for (size_t j = 0; j < b.size(); j++) c[i + j] = (c[i + j] + 1LL * a[i] * b[j]) % P;
            return c;
        }
        int n = 1 << std::__lg(len - 1) + 1;
        a.resize(n), b.resize(n);
        dft(a), dft(b);
        dotEq(a, b);
        idft(a);
        a.resize(len);
        return a;
    }
    // poly inverse
    // len(a) must be power of 2
    // $O(n \log n)$
    Poly invRec(Poly a) {
        int n = a.size();
        assert((n & n - 1) == 0);
        if (n == 1) return { fpow(a[0]) };
        int m = n >> 1;
        Poly b = invRec(Poly(a.begin(), a.begin() + m)), c = b;
        b.resize(n);
        dft(a), dft(b), dotEq(a, b), idft(a);
        for (int i = 0; i < m; i++) a[i] = 0;
        for (int i = m; i < n; i++) a[i] = P - a[i];
        dft(a), dotEq(a, b), idft(a);
        for (int i = 0; i < m; i++) a[i] = c[i];
        return a;
    }
    // arbitary length
    Poly inverse(Poly a) {
        int n = a.size();
        norm(a);
        a = invRec(a);
        a.resize(n);
        return a;
    }
    // get c where a = b * c + r, (len(a) = n, len(b) = m, len(c) = n - m + 1).
    // $O(n \log n)$
    Poly operator/(Poly a, Poly b) {
        int n = a.size(), m = b.size();
        if (n < m) return { 0 };
        int k = 1 << std::__lg(n - m << 1 | 1);
        std::reverse(a.begin(), a.end());
        std::reverse(b.begin(), b.end());
        a.resize(k), b.resize(k), b = invRec(b);
        a = a * b;
        a.resize(n - m + 1);
        std::reverse(a.begin(), a.end());
        return a;
    }
    // get {c, r}, len(c) = n - m + 1, len(r) = m - 1
    // $O(n \log n)$
    std::pair<Poly, Poly> operator%(Poly a, Poly b) {
        int m = b.size();
        Poly c = a / b;
        b = b * c;
        a.resize(m - 1);
        for (int i = 0; i < m - 1; i++) inc(a[i], P - b[i]);
        return { c, a };
    }
    // $\sqrt a$
    // $O(n \log n)$
    Poly sqrt(Poly a) {
        int raw = a.size();
        int d = 0;
        while (d < raw && !a[d]) d++;
        if (d == raw) return a;
        if (d & 1) return {};
        norm(a >>= d);
        int len = a.size();
        Poly b(len), binv(1), bsqr{ a[0] }, foo, bar;  // sqrt, sqrt_inv, sqrt_sqr
        // auto sq = SqrtMod::sqrtMod(a[0], P);
        // if (sq.empty()) return {};
        // b[0] = sq[0], binv[0] = fpow(b[0]);
        assert(a[0] == 1);
        b[0] = binv[0] = 1;
        auto shift = [](int x) { return (x & 1 ? x + P : x) >> 1; };  // quick div 2
        for (int m = 1, n = 2; n <= len; m <<= 1, n <<= 1) {
            foo.resize(n), bar = binv;
            for (int i = 0; i < m; i++) {
                foo[i + m] = a[i] + a[i + m] - bsqr[i];
                if (foo[i + m] >= P) foo[i + m] -= P;
                if (foo[i + m] < 0) foo[i + m] += P;
                foo[i] = 0;
            }
            binv.resize(n);
            dft(foo), dft(binv), dotEq(foo, binv), idft(foo);
            for (int i = m; i < n; i++) b[i] = shift(foo[i]);
            // inv
            if (n == len) break;
            for (int i = 0; i < n; i++) foo[i] = b[i];
            bar.resize(n), binv = bar;
            dft(foo), dft(bar), bsqr = dot(foo, foo), idft(bsqr);
            dotEq(foo, bar), idft(foo);
            for (int i = 0; i < m; i++) foo[i] = 0;
            for (int i = m; i < n; i++) foo[i] = P - foo[i];
            dft(foo), dotEq(foo, bar), idft(foo);
            for (int i = m; i < n; i++) binv[i] = foo[i];
        }
        b <<= d / 2;
        b.resize(raw);
        return b;
    }
    // $O(n)$
    Poly deriv(Poly a) {
        for (int i = 0; i + 1 < a.size(); i++) a[i] = (i + 1LL) * a[i + 1] % P;
        a.pop_back();
        return a;
    }
    std::vector<int> inv = { 1, 1 };
    void updateInv(int n) {
        if ((int)inv.size() <= n) {
            int p = inv.size();
            inv.resize(n + 1);
            for (int i = p; i <= n; i++) inv[i] = 1LL * (P - P / i) * inv[P % i] % P;
        }
    }
    // $O(n)$
    Poly integ(Poly a, int c = 0) {
        int n = a.size();
        updateInv(n);
        Poly b(n + 1);
        b[0] = c;
        for (int i = 0; i < n; i++) b[i + 1] = 1LL * inv[i + 1] * a[i] % P;
        return b;
    }
    // $O(n \log n)$
    Poly ln(Poly a) {
        int n = a.size();
        assert(a[0] == 1);
        a = inverse(a) * deriv(a);
        a.resize(n - 1);
        return integ(a);
    }
    // newton
    // $O(n \log n)$, slower than exp2
    Poly expNewton(Poly a) {
        int n = a.size();
        assert((n & n - 1) == 0);
        assert(a[0] == 0);
        if (n == 1) return { 1 };
        int m = n >> 1;
        Poly b = expNewton(Poly(a.begin(), a.begin() + m)), c;
        b.resize(n), c = ln(b);
        a.resize(n << 1), b.resize(n << 1), c.resize(n << 1);
        dft(a), dft(b), dft(c);
        for (int i = 0; i < n << 1; i++) a[i] = (1LL + P + a[i] - c[i]) * b[i] % P;
        idft(a);
        a.resize(n);
        return a;
    }
    // half-online conv
    // $O(n\log^2n)$
    // $b = e^a, b' = a'b$
    // $(n+1)b_{n+1} = \sum_{i=0}^n a'_ib_{n-i}$
    // $nb_n = \sum_{i=0}^{n-1} a'_ib_{n - 1 - i}$
    Poly exp2(Poly a) {
        if (a.empty()) return {};
        assert(a[0] == 0);
        int n = a.size();
        updateInv(n);
        for (int i = 0; i + 1 < n; i++) {
            a[i] = a[i + 1] * (i + 1LL) % P;
        }
        a.pop_back();
        Poly b(n);
        b[0] = 1;
        for (int m = 1; m < n; m++) {
            int k = m & -m, l = m - k, r = std::min(m + k, n);
            Poly p(a.begin(), a.begin() + (r - l - 1));
            Poly q(b.begin() + l, b.begin() + m);
            p.resize(k * 2), q.resize(k * 2);
            dft(p), dft(q);
            dotEq(p, q);
            idft(p);
            for (int i = m; i < r; i++) inc(b[i], p[i - l - 1]);
            b[m] = 1LL * b[m] * inv[m] % P;
        }
        return b;
    }
    // half-online conv
    // $O(\frac{n\log^2n}{\log\log n})$
    // $nb_n = \sum_{i=0}^{n-1} a'_ib_{n - 1 - i}$
    Poly exp(Poly a) {
        if (a.empty()) return {};
        assert(a[0] == 0);
        int n = a.size();
        updateInv(n);
        for (int i = 0; i + 1 < n; i++) {
            a[i] = a[i + 1] * (i + 1LL) % P;
        }
        a.pop_back();
        Poly b(n);
        b[0] = 1;
        std::vector<Poly> val_a[6], val_b(n);
        for (int m = 1; m < n; m++) {
            int k = 1, d = 0;
            while (!(m / k & 0xf)) k *= 16, d++;
            int l = m & ~(0xf * k), r = std::min(n, m + k);
            if (k == 1) {
                for (int i = m; i < r; i++) {
                    for (int j = l; j < m; j++) {
                        b[i] = (b[i] + 1LL * b[j] * a[i - j - 1]) % P;
                    }
                }
            } else {
                assert(d < 6);
                if (val_a[d].empty()) val_a[d].resize(n);
                val_b[m] = Poly(b.begin() + (m - k), b.begin() + m);
                val_b[m].resize(k * 2);
                dft(val_b[m]);
                Poly res(k * 2);
                for (; l < m; l += k) {
                    auto &p = val_a[d][m - l - k];
                    if (p.empty()) {
                        p = Poly(a.begin() + (m - l - k), a.begin() + (r - l - 1));
                        p.resize(2 * k);
                        dft(p);
                    }
                    auto &q = val_b[l + k];
                    for (int i = 0; i < k * 2; i++) res[i] = (res[i] + 1LL * p[i] * q[i]) % P;
                }
                idft(res);
                for (int i = m; i < r; i++) inc(b[i], res[i - m + k - 1]);
            }
            b[m] = 1LL * b[m] * inv[m] % P;
        }
        return b;
    }
    Poly power(Poly a, int k) {
        int n = a.size();
        long long d = 0;
        while (d < n && !a[d]) d++;
        if (d == n) return a;
        a >>= d;
        int b = fpow(a[0]);
        norm(a *= b);
        a = exp(ln(a) * k) * fpow(b, P - 1 - k % (P - 1));
        a.resize(n);
        d *= k;
        for (int i = n - 1; i >= d; i--) a[i] = a[i - d];
        d = std::min(d, 1LL * n);
        for (int i = d; i; a[--i] = 0)
            ;
        return a;
    }
    Poly power(Poly a, int k1, int k2) {  // k1 = k % (P - 1), k2 = k % P
        int n = a.size();
        long long d = 0;
        while (d < n && !a[d]) d++;
        if (d == n) return a;
        a >>= d;
        int b = fpow(a[0]);
        norm(a *= b);
        a = exp(ln(a) * k2) * fpow(b, P - 1 - k1 % (P - 1));
        a.resize(n);
        d *= k1;
        for (int i = n - 1; i >= d; i--) a[i] = a[i - d];
        d = std::min(d, 1LL * n);
        for (int i = d; i; a[--i] = 0)
            ;
        return a;
    }
    // [x^n](f / g)
    // $O(m \log m \log n)$
    int divAt(Poly f, Poly g, int64_t n) {
        int len = std::max(f.size(), g.size()), m = 1 << std::__lg(len * 2 - 1);
        f.resize(len), g.resize(len);
        for (; n; n >>= 1) {
            f.resize(m * 2), g.resize(m * 2);
            dft(f), dft(g);
            for (int i = 0; i < m * 2; i++) f[i] = 1LL * f[i] * g[i ^ 1] % P;
            for (int i = 0; i < m; i++) g[i] = 1LL * g[i * 2] * g[i * 2 + 1] % P;
            g.resize(m);
            idft(f), idft(g);
            for (int i = 0, j = n & 1; i < len; i++, j += 2) f[i] = f[j];
            f.resize(len), g.resize(len);
        }
        return f[0];
    }
    // [x^i](1/q) i = l...r
    Poly invRange(Poly q, int64_t l, int64_t r) {
        assert(l <= r);
        assert(r - l + 1 <= 5e5);
        int len = std::max<int>(q.size(), r - l + 1), m = 1 << std::__lg(len * 2 - 1);

        std::function<Poly(Poly &, int64_t)> cal = [&](Poly &a, int64_t n) -> Poly {
            if (n == 0) {
                Poly res(len);
                int c = 0;
                for (int i = 0; i < m; i++) inc(c, a[i]);
                res.back() = 1LL * m * fpow(c) % P;
                return res;
            }
            dftDoubling(a.data(), m);
            Poly b(m * 2);
            for (int i = 0; i < m; i++) b[i] = 1LL * a[i * 2] * a[i * 2 + 1] % P;
            auto c = cal(b, n >> 1);
            std::fill(b.begin(), b.end(), 0);
            for (int i = 0, o = n & 1 ^ 1; i < len; i++) b[i * 2 ^ o] = c[i];
            dft(b);
            for (int i = 0; i < m * 2; i++) b[i] = 1LL * b[i] * a[i ^ 1] % P;
            idft(b);
            return Poly(b.begin() + len, b.begin() + len * 2);
        };

        q.resize(m * 2);
        dft(q.data(), m);
        q = cal(q, r);
        return Poly(q.end() - (r - l + 1), q.end());
    }
    int divAt2(Poly f, Poly g, int64_t n) {
        g = invRange(g, n - ((int)f.size() - 1), n);
        g = f * g;
        return g[f.size() - 1];
    }
}
```

