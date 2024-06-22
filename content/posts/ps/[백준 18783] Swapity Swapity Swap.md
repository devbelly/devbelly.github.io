---
title: "[백준 18783] Swapity Swapity Swap"
date: 2020-11-09T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/18783](https://www.acmicpc.net/problem/18783)

<br>

## 알고리즘

Sparse table

<br>

## 풀이

소들이 $M$ step을 $K$번 반복했을 때, 최종적인 소의 배치를 구하는 문제입니다.

$M$과 $K$의 범위가 크므로 시간복잡도 $O(MK)$ 로는 해결할 수 없습니다. 대신 $K$을 비트로 표현하여 켜진 비트에 대한 다음의 상태를 관리할 수 있는 sparse table을 통해 $K$를 $logK$로 줄일 수 있습니다. 모든 배열의 $K$이후의 상태를 물어보기 때문에 sparse배열을 두개만 만들어 메모리를 절약할 수 있습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 1e5;

int N, M, K, prv, cur = 1;
int arr[MAXN];
int sparse[2][MAXN];
int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> M >> K;
    iota(arr, arr + N, 0);
    while (M--) {
        int s, t;
        cin >> s >> t;
        --s;
        reverse(arr + s, arr + t);
    }
    rep(i, N) {
        sparse[0][i] = arr[i];
    }
    --K;

    while (K) {
        if (K & 1) {
            rep(i, N) {
                arr[i] = sparse[prv][arr[i]];
            }
        }
        rep(i, N) {
            sparse[cur][i] = sparse[prv][sparse[prv][i]];
        }
        swap(cur, prv);
        K >>= 1;
    }
    rep(i, N) {
        cout << arr[i] + 1 << '\n';
    }
    return 0;
}
```
