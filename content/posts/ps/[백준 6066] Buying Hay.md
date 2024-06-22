---
title: "[백준 6066] Buying Hay"
date: 2021-07-05T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[www.acmicpc.net/problem/6066](www.acmicpc.net/problem/6066)

<br>

## 알고리즘

Knapsack

<br>

## 풀이

갯수제한이 없는 01 Knapsack 문제입니다.

<p align=center>
	$cache[i][j]$ = $i$번째 물건까지 택하고 현재 얻은 hay양이 $j$ 일때 지불해야하는 최소 금액
</p>

으로 가정하면 풀 수 있습니다. $cache[i][j]$가 갱신하는 다음 상태는 현재 상태에서 hay를 구매했을때 상태인 $cache[i][j+hay]$ 와 현재 물건을 택하지 않고 다음 물건을 택하는 $cache[i+1][j]$가 있습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

int N, H;
int sup[105][2];
int cache[101][55100 + 5];
int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> H;
    rep(i, N) rep(j, 2) {
        cin >> sup[i][j];
    }
    int ans = INT_MAX;
    memset(cache, 0x3f, sizeof(cache));
    cache[0][0] = 0;
    for (int i = 0; i < N; ++i) {
        int hay = sup[i][0];
        int cost = sup[i][1];
        for (int j = 0; j + hay <= 55000; ++j) {

            cache[i][j + hay] = min(cache[i][j + hay], cache[i][j] + cost);
            cache[i + 1][j] = min(cache[i + 1][j], cache[i][j]);
            if (j >= H) {
                ans = min(ans, cache[i][j]);
            }
        }
    }
    cout << ans;

    return 0;
}
```
