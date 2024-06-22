---
title: "[백준 17026] Mountain View"
date: 2021-01-04T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/17026](https://www.acmicpc.net/problem/17026)

<br>

## 알고리즘

스택, 정렬

<br>

## 풀이

직각이등변삼각형으로 된 산의 꼭짓점이 주어집니다. 산의 색깔이 같을 때, 구분되는 산의 갯수를 구하는 문제입니다.

산 자체를 관찰하면 조금은 헷갈릴 수 있는 문제입니다. 산을 산의 시작 좌표와 산의 끝 좌표로 치환하여 문제를 풀어 나가야 합니다. 즉 산을 선분으로 치환하면 문제는 $N$개의 선분이 주어질 때, 완전히 겹치는 선분을 제외하고 세는 문제로 바뀌게 됩니다.

바뀌게 된 문제는 시작좌표를 오름차순으로 정렬하되, 시작좌표가 같다면 끝좌표는 내림차순으로 정렬하여 풀면 됩니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

typedef pair<int, int> pii;
vector<pii> vt;

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    int N, ans = 0;
    cin >> N;
    rep(i, N) {
        int u, v;
        cin >> u >> v;
        vt.emplace_back(u - v, u + v);
    }
    sort(vt.begin(), vt.end(), [](pii a, pii b) {
        if (a.first == b.first)
            return a.second > b.second;
        return a.first < b.first;
    });
    int pivot = -1;
    for (auto [f, s] : vt) {
        if (s > pivot) {
            ans += 1;
            pivot = s;
        }
    }
    cout << ans;
    return 0;
}
```
