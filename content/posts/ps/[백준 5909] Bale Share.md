---
title: "[백준 5909] Bale Share"
date: 2021-08-31T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[www.acmicpc.net/problem/5909](www.acmicpc.net/problem/5909)

<br>

## 알고리즘

DP

<br>

## 풀이

주어진 건초더미를 세 개 공평하게 나누는 문제입니다.

[이 문제](https://www.acmicpc.net/problem/19645)와 매우 매우 유사합니다. $cache[k][i][j]$를 k번째 건초더미까지 분배했을때 i가 B_1, j가 B_2라고 해석하여 가능하다면 1, 불가능하다면 0을 저장하여 풀면 됩니다. 나머지 세 번째 곳간에 저장될 양은 $psum-i-j$를 통해 알아 낼 수 있습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 20 + 5;

int N, sum, ans = INT_MAX, prv, cur = 1, arr[MAXN];
bool cache[2][2001][2001];

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N;
    rep(i, N) {
        cin >> arr[i];
        sum += arr[i];
    }
    cache[0][0][0] = 1;
    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < 2000; ++j) {
            for (int k = 0; k < 2000; ++k) {
                if (cache[prv][j][k]) {
                    cache[cur][j][k] = cache[prv][j][k];
                    cache[cur][j + arr[i]][k] = cache[prv][j][k];
                    cache[cur][j][k + arr[i]] = cache[prv][j][k];
                }
            }
        }
        swap(prv, cur);
    }

    for (int i = 0; i <= 2000; ++i) {
        for (int j = i; j >= 0; --j) {
            if (j >= sum - i - j && cache[prv][i][j]) {
                ans = min(ans, i);
            }
        }
    }
    cout << ans;
    return 0;
}
```
