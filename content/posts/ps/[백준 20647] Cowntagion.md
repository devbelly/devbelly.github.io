---
title: "[백준 20647] Cowntagion"
date: 2021-01-13T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/20647](https://www.acmicpc.net/problem/20647)

<br>

## 알고리즘

DFS

<br>

## 풀이

$N$개의 목장이 트리 형태로 주어집니다. 1번 목장에 감염된 소가 있을 때, 모든 목장의 소가 감염되기 까지 최소일수를 구하는 문제입니다.

그리디 문제인데 자세한 관찰을 하지 못해 많이 틀렸습니다. 제 잘못된 관찰은 1번 목장에서 거리가 2이하인 목장들은 1번 목장에서 감염 후 이동해야한다고 생각했습니다. 이렇게 해야 복제시간에서 이득을 본다고 생각했기 때문입니다. (거리가 1인 노드를 무시한 채 거리가 2인 노드들만 고려해서 계산했음. [복제(1번노드)-이동-이동] 과 [복제(1번노드)-이동-복제(2번노드)-복제] 로 비교를 했지만 이는 거리가 1인 노드들을 처리하며 갈 수 있음을 무시한 계산이므로 틀렸음)

올바른 해는 자식노드 이상만큼 복제한 후 하나 씩 소를 이동시키는 것을 노드마다 반복하면 됩니다. DFS를 사용하지 않는 풀이도 있으니 참고하시면 좋을 것 같습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 1e5;

int N, ans;
vector<vector<int>> adj;

void dfs(int here, int prv) {
    int cnt = 0;
    for (auto there : adj[here]) {
        if (there ^ prv) {
            cnt += 1;
            dfs(there, here);
        }
    }
    ans += cnt;
    ans += ceil(log2(cnt + 1));
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
    adj.resize(N);

    rep(i, N - 1) {
        int u, v;
        cin >> u >> v;
        --u, --v;
        adj[u].emplace_back(v);
        adj[v].emplace_back(u);
    }
    dfs(0, -1);
    cout << ans;
    return 0;
}
```
