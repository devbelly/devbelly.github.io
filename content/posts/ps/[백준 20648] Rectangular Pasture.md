---
title: "[백준 20648] Rectangular Pasture"
date: 2021-03-15T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/20648](https://www.acmicpc.net/problem/20648)

<br>

## 알고리즘

DP, 좌표압축

<br>

## 풀이

$N$개의 소들의 좌표가 주어질 때, 직사각형 울타리로 가둘 수 있는 소들의 집합의 수를 구하는 문제입니다.

두마리 이상의 소들을 울타리 안에 넣을 때가 문제가 됩니다. 만약 임의의 소 두마리를 고른 후 그 소들을 포함한 집합의 수를 셀 때, 답에 영향을 주는 소들의 위치는 다음 그림에서 노란색 부분과 같습니다. 화살표는 두 소의 위치를 나타낸 것입니다.

![img](https://blog.kakaocdn.net/dn/cws3j7/btqZ4An3e7M/ZblCC4HEmLXTDDOfYMMHL1/img.png)

아래 노란색 사각형에서 소가 몇마리인지 세고 위에 노란색 사각형에서 소가 몇마리인지 센 후, 두 수를 곱하게 되면 선택한 두 소를 포함한 집합의 수를 구할 수 있습니다. 이 과정을 수행하기 위해 소들의 좌표를 압축한 후 $board$배열을 통해 원하는 사각형에 포함된 수를 $O(1)$에 구하도록 합시다. $board[i][j]$는 (1,1)부터 (i,j)까지 좌표에 포함된 소의 마릿 수를 의미합니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

typedef pair<int, int> pii;
typedef long long ll;
int N;

pii xy[2500 + 5];
ll board[2500 + 5][2500 + 5], ans;

ll sq(int a, int b, int c, int d) {
    return board[c][d] - board[a - 1][d] - board[c][b - 1] + board[a - 1][b - 1];
}

ll solve(int i, int j) {
    auto [a, b] = xy[i];
    auto [c, d] = xy[j];
    int minx = min(a, c);
    int maxx = max(a, c);
    int miny = min(b, d);
    int maxy = max(b, d);

    int lo = sq(minx, 0, maxx, miny);
    int hi = sq(minx, maxy, maxx, N);
    return lo * hi;
}

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
        cin >> xy[i].first >> xy[i].second;
    }
    sort(xy, xy + N, [](pii a, pii b) { return a.first < b.first; });
    rep(i, N) {
        xy[i].first = i + 1;
    }
    sort(xy, xy + N, [](pii a, pii b) { return a.second < b.second; });
    rep(i, N) {
        xy[i].second = i + 1;
    }
    rep(i, N) {
        auto [x, y] = xy[i];
        board[x][y] = 1;
    }
    REP(i, N) REP(j, N) {
        board[i][j] = board[i - 1][j] + board[i][j - 1] - board[i - 1][j - 1] + board[i][j];
    }
    rep(i, N) {
        for (int j = i + 1; j < N; ++j) {
            ans += solve(i, j);
        }
    }
    cout << ans + N + 1;
    return 0;
}
```
