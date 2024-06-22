---
title: "[백준 15586] Mootube(Gold)"
date: 2021-01-09T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/15586](https://www.acmicpc.net/problem/15586)

<br>

## 알고리즘

union find

<br>

## 풀이

$N$개의 정점으로 이루어진 트리 위에서 쿼리마다 가중치가 $k$이상인 인접한 노드들의 수를 구하는 문제입니다.

[이 문제](https://www.acmicpc.net/problem/15591)에서는 정점의 수와 쿼리가 5000이어서 각 쿼리마다 dfs를 돌려 수행이 가능했지만 이 문제는 둘 다 10만이므로 각 쿼리마다 dfs를 수행하면 시간 초과가 발생합니다.

경로의 최솟값이 $k$이상인 정점의 갯수는 가중치가 $k$이상인 간선들을 이어 만든 포레스트에서 자신이 속한 포레스트의 크기를 찾는 것과 동일합니다. 자신의 속한 포레스트의 크기를 구하기 위해선 유니온 파인드를 사용해야 하는데, 유니온 한 정점끼리 일관성을 갖기 위해서는 쿼리를 $k$를 기준으로 내림차순 정렬을 해야 합니다. 정렬을 하게 되면 $k$보다 작은 쿼리를 할 때, $k$이상의 간선들은 이미 유니온 된 상태이므로 현재 가중치만을 추가적으로 유니온 하게 되면 모든 정답을 고려하기 때문입니다. 쿼리마다 유니온 파인드를 하고 while문은 최대 $N-1$번만 돌기 때문에 시간 복잡도는 $O(NlogN+QlogQ+N+Q)$입니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 100000;
typedef tuple<int, int, int> tp;
typedef pair<int, int> pii;

int N, M;
int p[MAXN], ans[MAXN];
vector<tp> edges;
vector<tp> query;

int find(int x) {
    return p[x] < 0 ? x : (p[x] = find(p[x]));
}

void _union(int u, int v) {
    u = find(u);
    v = find(v);
    if (u == v)
        return;
    if (p[u] < p[v])
        swap(u, v);
    p[v] += p[u];
    p[u] = v;
}

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> M;
    memset(p, -1, sizeof(p));
    rep(i, N - 1) {
        int u, v, c;
        cin >> u >> v >> c;
        --u, --v;
        edges.emplace_back(u, v, c);
    }

    sort(edges.begin(), edges.end(), [](tp a, tp b) { return get<2>(a) > get<2>(b); });

    rep(i, M) {
        int k, v;
        cin >> k >> v;
        --v;
        query.emplace_back(k, v, i);
    }
    int i = 0;
    sort(query.begin(), query.end(), [](tp a, tp b) { return get<0>(a) > get<0>(b); });

    for (auto [k, u, idx] : query) {
        while (i < N - 1) {
            auto [U, V, C] = edges[i];
            if (C >= k) {
                _union(U, V);
                ++i;
            } else
                break;
        }
        ans[idx] = -p[find(u)] - 1;
    }

    rep(i, M) {
        cout << ans[i] << '\n';
    }
    return 0;
}
```
