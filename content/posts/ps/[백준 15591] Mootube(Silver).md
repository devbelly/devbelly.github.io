---
title: "[백준 15591] Mootube(Gold)"
date: 2021-01-09T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/15591](https://www.acmicpc.net/problem/15591)

<br>

## 알고리즘

DFS

<br>

## 풀이

$N$개의 정점이 트리를 이루고 각 엣지마다 가중치가 있을 때, 가중치가 $k$이상으로 이어져 있는 노드의 수를 구하는 문제입니다.

$N$과 $Q$의 수가 최대 5000이므로 각 쿼리마다 $dfs$를 수행해도 시간초과가 나지 않습니다. $dfs(here, parent, weight)$를 호출하여 인접한 노드가 가중치 $k$이상으로 이어져 있는지 확인하면 됩니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

typedef pair<int, int> pii;

int N, Q;
vector<vector<pii>> adj;

int dfs(int here, int prv, int k) {
    int ret = 0;
    for (auto [there, edge] : adj[here]) {
        if (edge >= k && there != prv) {
            ret += 1;
            ret += dfs(there, here, k);
        }
    }
    return ret;
}

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> Q;
    adj.resize(N);
    rep(i, N - 1) {
        int u, v, w;
        cin >> u >> v >> w;
        --u, --v;
        adj[u].emplace_back(v, w);
        adj[v].emplace_back(u, w);
    }

    while (Q--) {
        int u, k;
        cin >> k >> u;
        --u;
        cout << dfs(u, -1, k) << '\n';
    }

    return 0;
}
```
