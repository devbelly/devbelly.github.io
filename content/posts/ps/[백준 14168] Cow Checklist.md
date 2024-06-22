---
title: "[백준 14168] Cow Checklist"
date: 2021-02-11T11:10:28+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---


## 문제

[https://www.acmicpc.net/problem/14168](https://www.acmicpc.net/problem/14168)

<br>

## 알고리즘

DP

<br>

## 풀이

H종류의 소와 G종류의 소가 주어집니다. 순차적으로 (연속될 필요는 없음) 방문했을 때 최소의 에너지 구하기. 시작은 H종류의 1번소에서 시작해 H종류의 h번소에서 끝나야합니다.

경찰차와 비슷한 문제입니다. $cache$를 다음과 같이 선언합시다.

$cache[i][j][k]$ = H종류의 $i$번째 소, G종류의 $j$번째 소까지 방문이 끝났을 때, 필요한 에너지 최솟값. $k$는 $i, j$ 소중에서 현재 위치의 소를 알기 위함입니다.

$k$값이 0이면 H종류의 소에 있음을 나타내고 $k$값이 1이면 G종류의 소에 위치함을 나타냅니다. $cache[i][j][k]$가 갱신할수 있는 상태는 $cache[i+1][j][k]$와 $cache[i][j+1][k]$이고 $k$값에 따라 적절하게 거리를 구해주면 됩니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

#define fi first
#define se second
typedef pair<int, int> pii;
const int MAXN = 1e3 + 5;

int H, G;
int cache[MAXN][MAXN][2];

inline int dist(int a, int b, int c, int d) {
    return (a - c) * (a - c) + (b - d) * (b - d);
}

pii h[MAXN], g[MAXN];
int main() {
#ifndef ONLINE_JUDGE
    freopen("8.in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> H >> G;
    rep(i, MAXN) rep(j, MAXN) rep(k, 2) cache[i][j][k] = INT_MAX;
    REP(i, H) {
        cin >> h[i].first >> h[i].second;
    }
    REP(i, G) {
        cin >> g[i].first >> g[i].second;
    }
    cache[1][0][0] = 0;
    for (int i = 1; i <= H; ++i) {
        for (int j = 0; j <= G; ++j) {
            for (int k = 0; k < 2; ++k) {
                if (cache[i][j][k] == INT_MAX)
                    continue;
                int dist1 = dist(h[i].fi, h[i].se, h[i + 1].fi, h[i + 1].se);
                int dist2 = dist(h[i].fi, h[i].se, g[j + 1].fi, g[j + 1].se);
                int dist3 = dist(g[j].fi, g[j].se, h[i + 1].fi, h[i + 1].se);
                int dist4 = dist(g[j].fi, g[j].se, g[j + 1].fi, g[j + 1].se);
                if (k == 0) {
                    cache[i + 1][j][0] = min(cache[i + 1][j][0], cache[i][j][0] + dist1);
                    cache[i][j + 1][1] = min(cache[i][j + 1][1], cache[i][j][0] + dist2);
                } else {
                    cache[i + 1][j][0] = min(cache[i + 1][j][0], cache[i][j][1] + dist3);
                    cache[i][j + 1][1] = min(cache[i][j + 1][1], cache[i][j][1] + dist4);
                }
            }
        }
    }
    cout << cache[H][G][0];
    return 0;
}
```
