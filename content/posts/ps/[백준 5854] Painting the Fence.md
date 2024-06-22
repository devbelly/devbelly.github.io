---
title: "[백준 5854] Painting the Fence"
date: 2021-07-29T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[www.acmicpc.net/problem/5854](www.acmicpc.net/problem/5854)

<br>

## 알고리즘

Sorting

<br>

## 풀이

$K$번 겹치는 선분의 길이를 구하는 문제입니다.

세그먼트트리 문제에서 직사각형 관련 문제를 풀 때 비슷하게 풀어본 적이 있는 것 같습니다. 주어진 선분을 모두 구한 후 각 선분마다 선분의 왼쪽 점과 오른쪽 점을 구분합니다. 벡터에 넣어 정렬한 후, 만일 현재점이 선분의 왼쪽을 구성하는 점이라면 $psum$값을 1증가시키고 선분의 오른쪽을 구성하는 점이라면 $psum$값을 1감소시킵니다. $psum$은 현재 겹쳐진 횟수를 의미하며 해당 값이 $K$이상일때만 $ans$에 기록하여 문제를 풀었습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

typedef pair<int, int> pii; // second 0 : left 1:right

int N, K, mv, cur, ans, psum;
char dir;

vector<pii> vt;

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
        cin >> mv >> dir;
        if (dir == 'L') {
            vt.emplace_back(cur, 1);
            cur -= mv;
            vt.emplace_back(cur, 0);
        } else {
            vt.emplace_back(cur, 0);
            cur += mv;
            vt.emplace_back(cur, 1);
        }
    }
    sort(vt.begin(), vt.end());
    for (auto [pos, typ] : vt) {
        ans += (pos - cur) * (psum >= K);
        psum += (!typ ? 1 : -1);
        cur = pos;
    }
    cout << ans;
    return 0;
}
```
