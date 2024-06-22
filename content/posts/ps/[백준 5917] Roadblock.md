---
title: "[백준 5917] Roadblock"
date: 2021-05-31T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[www.acmicpc.net/problem/5917](www.acmicpc.net/problem/5917)

<br>

## 알고리즘

Dijkstra

<br>

## 풀이

최단거리 중 하나의 간선의 값을 두배로 하였을 때의 최댓값을 구하는 문제입니다.

다익스트라임은 최단거리임을 보고 알 수 있습니다. 문제 제한은 $N$이 100 이하, $M$은 10000 이하입니다. 단순한 접근으로는 $M$의 간선들을 하나씩 두배로 만든 후 그때마다 새로이 다익스트라를 사용하는 것입니다.

하지만 $M$제한이 10000이므로 이 방법은 무리가 있습니다. 이 방법에는 불필요한 부분이 있습니다. 바로 최단거리에 포함되지 않는 간선들 또한 두 배로 바꾸는 것입니다. 이는 아무런 의미가 없는 행동입니다. $N$개로 이루어진 노드에서 최단거리는 최대 $N-1$개의 간선을 통해 이루어져 있습니다. 즉 최단거리에 포함되는 간선들을 찾고 그 간선들만 두배로 한 후 다익스트라를 해서 가장 큰 값을 찾으면 됩니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

typedef pair<int, int> pii;
const int MAXN = 105;

int N, M;
int adj[MAXN][MAXN];
int parent[MAXN];
int dist[MAXN];

vector<pii> vt;

void dijkstra() {
    memset(dist, 0x3f, sizeof(dist));
    priority_queue<pii, vector<pii>, greater<pii>> pq;
    dist[0] = 0;
    pq.emplace(0, 0);

    while (!pq.empty()) {
        auto [hereCost, here] = pq.top();
        pq.pop();
        if (dist[here] < hereCost) continue;
        rep(i, N) {
            if (adj[here][i] != 0x3f3f3f3f) {
                if (dist[i] > dist[here] + adj[here][i]) {
                    parent[i] = here;
                    dist[i] = dist[here] + adj[here][i];
                    pq.emplace(dist[i], i);
                }
            }
        }
    }
}
void _path(int s, int e) {
    if (s != e) {
        vt.emplace_back(parent[e], e);
        _path(s, parent[e]);
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

    memset(adj, 0x3f, sizeof(adj));
    cin >> N >> M;
    rep(i, M) {
        int u, v, c;
        cin >> u >> v >> c;
        --u, --v;
        adj[u][v] = c;
        adj[v][u] = c;
    }
    dijkstra();
    int MIN = dist[N - 1];
    int secMin = 0;
    _path(0, N - 1);
    for (auto [a, b] : vt) {
        int prvDist = adj[a][b];
        adj[a][b] = adj[b][a] = prvDist * 2;
        dijkstra();
        secMin = max(secMin, dist[N - 1]);
        adj[a][b] = adj[b][a] = prvDist;
    }

    cout << secMin - MIN;
    return 0;
}
```
