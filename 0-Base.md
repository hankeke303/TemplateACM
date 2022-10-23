# 一切的开始

## 代码头

* hkk 版

```cpp
#include <bits/stdc++.h>
using namespace std;

#define fec(i, x, y) (int i = head[x], y = g[i].to; i; i = g[i].ne, y = g[i].to)
#define dbg(...) fprintf(stderr, __VA_ARGS__)
#define fi first
#define se second

using ll = long long; using ull = unsigned long long; using pii = pair<int, int>;

template <typename A, typename B> bool smax(A &a, const B &b) { return a < b ? a = b, 1 : 0; }
template <typename A, typename B> bool smin(A &a, const B &b) { return b < a ? a = b, 1 : 0; }
```

## 图论建图模板

### 带权图

```cpp
struct Edge {int to, ne, w;} g[M]; int head[N], tot;
void addedge(int x, int y, int z) { g[++tot].to = y; g[tot].w = z; g[tot].ne = head[x]; head[x] = tot; }
void adde(int x, int y, int z) { addedge(x, y, z); addedge(y, x, z); }
```

### 无权图

```cpp
struct Edge {int to, ne;} g[M]; int head[N], tot;
void addedge(int x, int y) { g[++tot].to = y; g[tot].ne = head[x]; head[x] = tot; }
void adde(int x, int y) { addedge(x, y); addedge(y, x); }
```

### 网络流

```cpp
struct Edge {int to, ne, f;} g[M * 2]; int head[N], tot = 1;
void addedge(int x, int y, int z) { g[++tot].to = y; g[tot].f = z; g[tot].ne = head[x]; head[x] = tot; }
void adde(int x, int y, int z) { addedge(x, y, z); addedge(y, x, 0); }
```

## 取模及组合数

### 取模

```cpp
int smod(int x) { return x >= P ? x - P : x; }
void sadd(int &x, const int &y) { x += y; x >= P ? x -= P : x; }
int fpow(int x, int y) {
    int ans = 1;
    for (; y; y >>= 1, x = (ll)x * x % P) if (y & 1) ans = (ll)ans * x % P;
    return ans;
}
```

### 组合数

```cpp
int fac[N], inv[N], ifac[N];
void ycl(const int &n = ::n) {
    fac[0] = 1; for (int i = 1; i <= n; ++i) fac[i] = (ll)fac[i - 1] * i % P;
    inv[1] = 1; for (int i = 2; i <= n; ++i) inv[i] = (ll)(P - P / i) * inv[P % i] % P;
    ifac[0] = 1; for (int i = 1; i <= n; ++i) ifac[i] = (ll)ifac[i - 1] * inv[i] % P;
}
int C(int x, int y) {
    if (x < y) return 0;
    return (ll)fac[x] * ifac[y] % P * ifac[x - y] % P;
}
```

## 快速读写

### 普通的快速度入

```cpp
template<typename I> inline void read(I &x) {
	int f = 0, c;
	while (!isdigit(c = getchar())) c == '-' ? f = 1 : 0;
	x = c & 15;
	while (isdigit(c = getchar())) x = (x << 1) + (x << 3) + (c & 15);
	f ? x = -x : 0;
}
```

### fread/fwrite

```cpp
namespace io {
	const int SIZE = (1 << 21) + 1;
	char ibuf[SIZE], *iS, *iT, obuf[SIZE], *oS = obuf, *oT = obuf + SIZE - 1, c, qu[55];
	int f, qr;
    #define gc() (iS == iT ? (iT = (iS = ibuf) + fread(ibuf, 1, SIZE, stdin), iS == iT ? EOF : *(iS++)) : *(iS++))
	void flush() { fwrite(obuf, 1, oS - obuf, stdout), oS = obuf; }
	void putc(char c) { *(oS++) = c; if (oS == oT) flush(); }
	template <typename I> void read(I& x) {
		for (f = 1, c = gc(); c < '0' || c > '9'; c = gc()) c == '-' ? f = -1 : 0;
		for (x = 0; c >= '0' && c <= '9'; c = gc()) x = (x << 1) + (x << 3) + (c & 15);
		~f ? 0 : x = -x;
	}
	template <typename I> void write(I x) {
		if (x == 0) putc('0');
		if (x < 0) x = -x, putc('-');
		for (; x; x /= 10) qu[++qr] = x % 10 + '0';
		for (; qr;) putc(qu[qr--]);
	}
	struct Flusher_ { ~Flusher_() { flush(); } } io_flusher;
}
```

## 对拍

```bash
#!/usr/bin/env bash
g++ -o r main.cpp -O2 -std=c++17
g++ -o std std.cpp -O2 -std=c++17
while true; do
    python gen.py > in
    ./std < in > stdout
    ./r < in > out
    if test $? -ne 0; then
        exit 0
    fi
    if diff stdout out; then
        printf "AC\n"
    else
        printf "GG\n"
        exit 0
    fi
done
```

+ 快速编译运行 （配合无插件 VSC）

```bash
#!/bin/bash
g++ $1.cpp -o $1 -O2 -std=c++14 -Wall -Dzerol -g
if $? -eq 0; then
	./$1
fi
```
