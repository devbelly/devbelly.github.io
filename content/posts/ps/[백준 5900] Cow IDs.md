---
title: "[백준 5900] Cow IDs"
date: 2021-08-03T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[www.acmicpc.net/problem/5900](www.acmicpc.net/problem/5900)

<br>

## 알고리즘

DP

<br>

## 풀이

1비트를 k개 사용했을 때, n번째 비트를 구하는 문제입니다.

일단 n번째 비트의 자릿수 먼저 구해야 합니다. 만약 자릿수가 5이고 사용한 k가 3일 때 5자리로 나타낼 수 있는 가짓수는 4C2입니다. 왜냐하면 맨 처음 1은 무조건 나와야 하기 때문입니다. 그렇다면 자릿수를 구하기 위해서는 2C2 + 3C2 + 4C2.. 와같이 세 자리에서 k 3개일 때 경우의 수 + 네 자리에서 k 3개 일 때 경우의 수 + 다섯 자리에서 k 3개일 때의 경우의 수처럼 앞에서부터의 누적합을 구해야 nth가 몇 번째 자릿수인지 구할 수 있습니다.

하지만 굳이 누적합을 구하지 않더라도 이미 누적합이 구해져 있습니다. 5C3은 2C2 + 3C2 + 4C2과 동일하기 때문입니다. 이는 조합의 성질을 고려해보면 파악할 수 있습니다. 5C3=4C2 + 4C3, 4C3 = 3C2 + 3C3 이기 때문입니다. 이를 이용하여 바로 현재 자릿수를 파악할 수 있습니다.

자릿수를 결정한 후, $solve$함수 내에서 $piv$과 $kth$를 비교하여 $piv$가 더 크다면 현재 0을 사용해야 함을 의미하고 아니라면 1을 사용한 후 다시 재귀 호출하여 풀면 됩니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

typedef long long ll;

const int MAXN = 1e7;

ll cache[5000][11];
int N, K;

ll binary(int n, int l) {
    if (n < l) return 0;
    if (n == l || l == 0) return 1;
    ll& ret = cache[n][l];
    if (ret != -1) return ret;
    ret = binary(n - 1, l - 1) + binary(n - 1, l);
    if (ret > MAXN) ret = MAXN;
    return ret;
}

void solve(int n, int k, int kth) {
    if (!n) return;
    ll piv = binary(n - 1, k);
    if (kth <= piv) {
        cout << 0;
        solve(n - 1, k, kth);
    } else {
        cout << 1;
        solve(n - 1, k - 1, kth - piv);
    }
}

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    memset(cache, -1, sizeof(cache));
    cin >> N >> K;

    if (K == 1) {
        cout << 1;
        rep(i, N - 1) {
            cout << 0;
        }
    } else {
        for (int i = K; i < 5000; ++i) {
            if (binary(i, K) >= N) {
                solve(i, K, N);
                break;
            }
        }
    }
    return 0;
}
```
