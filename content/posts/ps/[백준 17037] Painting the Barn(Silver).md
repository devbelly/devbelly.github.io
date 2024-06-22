---
title: "[백준 17037] Painting the Barn(Silver)"
date: 2020-12-05T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/17037](https://www.acmicpc.net/problem/17037)

<br>

## 알고리즘

DP

<br>

## 풀이

$N$개의 사각형이 주어졌을 때, $K$번 겹치는 사각형들의 넓이의 합을 구하는 문제입니다.

일차적으로 생각할 수 있는 풀이는 주어지는 사각형의 범위가 작으므로 2차원 배열을 만든 후, 배열을 색칠하여 푸는 풀이를 생각할 수 있습니다. 하지만 $N$제한이 10만이고, 사각형을 색칠하는데에는 길이의 제곱에 해당하는 시간이 필요합니다. 최악의 시간복잡도는 $10^5 * 1000*1000$ 이므로 시간초과입니다.

사각형이 주어질 때 마다 색칠하는 것이 아니라, 네 모서리에 체크만 한 후 최종적으로 배열을 확인할 때, 모서리를 옮기며 풀면 최종적으로 $O(N)$에 해결가능합니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

int N, K;
int cache[1005][1005];

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> K;
    rep(i, N) {
        int a, b, c, d;
        cin >> a >> b >> c >> d;
        cache[a][b]++;
        cache[a][d]--;
        cache[c][b]--;
        cache[c][d]++;
    }
    int ans = 0;
    rep(i, 1001) rep(j, 1001) {
        cache[i + 1][j] += cache[i][j];
        cache[i][j + 1] += cache[i][j];
        cache[i + 1][j + 1] -= cache[i][j];
        if (cache[i][j] == K)
            ans += 1;
    }
    cout << ans;
    return 0;
}
```
