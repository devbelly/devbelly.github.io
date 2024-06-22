---
title: "[백준 7045] Tree Cutting"
date: 2021-07-02T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/7045](https://www.acmicpc.net/problem/7045)

<br>

## 알고리즘

DFS

<br>

## 풀이

각 정점을 루트로 했을 때, 서브트리 크기의 최댓값이 절반 이하가 되는 정점을 찾는 문제입니다.

문제에서 주어진 $N$ 제한은 1만이므로 각 정점마다 DFS를 돌리면 $O(N^2)$ 이므로 시간초과를 받게 됩니다. 한 정점에서 DFS를 시작하여 임의의 정점에 도달했을 때, 해당 정점의 서브트리의 크기는 재귀적으로 찾을 수 있습니다. 추가적으로 고려해야할 것은 현재까지 탐색한 트리의 크기입니다. 이 정보는 현재까지 탐색했던 트리를 제외한 서브트리 크기의 총합을 안다면 $N-sum[i]$를 통해 구할 수 있습니다. $sum[i]$는 서브트리크기의 총합을 의미합니다.

DFS를 수행한 후 각 정점마다 $MAX[i]$ 값과 $N-sum[i]$값 중 최댓값을 N/2와 비교하면 정답을 얻을 수 있습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 1e5 + 5;

int N;
bool visited[MAXN];
int MAX[MAXN], SUM[MAXN];
vector<int> adj[MAXN];
vector<int> ans;

void solve(int here) {

    visited[here] = true;
    SUM[here] = 1;
    for (auto there : adj[here]) {
        if (visited[there]) continue;
        solve(there);
        MAX[here] = max(MAX[here], SUM[there]);
        SUM[here] += SUM[there];
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

    cin >> N;
    rep(i, N - 1) {
        int u, v;
        cin >> u >> v;
        adj[u].emplace_back(v);
        adj[v].emplace_back(u);
    }
    solve(1);
    bool ex = false;
    REP(i, N) {
        if (max(MAX[i], N - SUM[i]) <= N / 2) {
            ex = true;
            cout << i << '\n';
        }
    }
    if (!ex) {
        cout << "NONE";
    }
    return 0;
}
```
