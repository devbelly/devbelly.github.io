---
title: "[백준 15587] Cow at Large(Gold)"
date: 2021-02-08T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/15587](https://www.acmicpc.net/problem/15587)

<br>

## 알고리즘

BFS

<br>

## 풀이

Bessie가 트리 형태의 목장에서 탈출하려 합니다. 이때 이 탈출을 제지해야하는 최소의 농부 수를 구하는 문제입니다. 탈출구는 리프노드입니다.

만일 한 농부가 $u$노드까지의 거리가 베시보다 가까우면서 다른 농부들보다도 가깝다면 다른 농부들을 카운트할 필요는 없습니다. 이를 위해 다양한 source로부터(여기서는 리프노드, 농부들의 시작점) bfs를 수행할 것입니다. $dist$의 상태에는 3가지가 있습니다. 아직 방문하지 않은 상태인 -1, 농부가 방문한 상태인 0, bessie가 도착한 int_max의 상태입니다.

아래 코드에서 가장 눈여겨 볼 점은 bessie에 해당하는 노드를 queue에 넣을 때, 농부들보다 뒤에 넣는다는 점입니다. 이렇게 하면 $u$노드에 동시에 도착하는 것은 물론 터널 사이에서 만나는 것 또한 고려할 수 있습니다. 시간복잡도는 $O(N)$입니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 1e5 + 5;

int N, K, ans;
int dist[MAXN];
queue<int> q;
vector<vector<int>> adj;

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> K;
    adj.resize(N);
    memset(dist, -1, sizeof(dist));
    --K;
    rep(i, N - 1) {
        int u, v;
        cin >> u >> v;
        --u, --v;
        adj[u].emplace_back(v);
        adj[v].emplace_back(u);
    }

    rep(i, N) {
        if (adj[i].size() == 1) {
            q.emplace(i);
            dist[i] = 0;
        }
    }

    q.emplace(K);
    dist[K] = INT_MAX;
    while (!q.empty()) {
        int here = q.front();
        q.pop();

        for (auto there : adj[here]) {
            if (dist[there] == -1) {
                if (dist[here] == INT_MAX) {
                    dist[there] = INT_MAX;
                } else {
                    dist[there] = 0;
                }
                q.emplace(there);
            } else if (dist[here] == INT_MAX && dist[there] != INT_MAX) {
                ans += 1;
            }
        }
    }
    cout << ans;
    return 0;
}
```
