---
title: "[백준 10652] Piggy Back"
date: 2021-05-27T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/10652](https://www.acmicpc.net/problem/10652)

<br>

## 알고리즘

Dijkstra, BFS

<br>

## 풀이

Bessie와 Elsie가 돌아다닐때 드는 에너지와 Piggy Back 상태에서 드는 에너지가 주어졌을 때, $N$까지 도달하기 위한 최소에너지는 구하는 문제입니다.

3인통화 문제와 굉장히 유사합니다. 도착지도 하나의 소처럼 취급을 하여 세 마리의 소에서 각각 다익스트라를 사용하면 문제가 해결됩니다. 다만, 각 소마다 그래프를 이동할 때 드는 비용은 동일하므로 다익스트라 대신 BFS를 사용하면 더 빠르게 문제를 풀 수 있습니다.

<br>

## 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

typedef pair<int, int> pii;
const int MAXN = 4e4 + 5;

int N, M;
vector<int> adj[MAXN];
int dist[3][MAXN];
int cost[3];
void dijkstra(int i, int _here) {
    dist[i][_here] = 0;
    priority_queue<pii, vector<pii>, greater<pii>> pq;
    pq.emplace(0, _here);

    while (!pq.empty()) {
        auto [cd, here] = pq.top();
        pq.pop();
        if (dist[i][here] < cd) continue;
        for (auto there : adj[here]) {
            if (dist[i][there] > cd + cost[i]) {
                dist[i][there] = cd + cost[i];
                pq.emplace(dist[i][there], there);
            }
        }
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

    cin >> cost[0] >> cost[1] >> cost[2] >> N >> M;
    memset(dist, 0x7f, sizeof(dist));

    rep(i, M) {
        int u, v;
        cin >> u >> v;
        adj[u].emplace_back(v);
        adj[v].emplace_back(u);
    }
    dijkstra(0, 1);
    dijkstra(1, 2);
    dijkstra(2, N);

    long long ans = LLONG_MAX;
    REP(i, N) {
        long long temp = 0;
        rep(j, 3) {
            temp += dist[j][i];
        }
        ans = min(ans, temp);
    }
    cout << ans;

    return 0;
}
```
