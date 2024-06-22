---
title: "[백준 12012] Closing the Farm(Gold)"
date: 2021-03-08T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/12012](https://www.acmicpc.net/problem/12012)

<br>

## 알고리즘

Union Find, 오프라인 쿼리

<br>

## 풀이

헛간을 하나씩 닫아나갈 때 마다, 열린 헛간들의 연결성을 확인하는 문제입니다. 헛간을 닫을 때, 인접한 경로 또한 사라집니다.

우리가 아는 자료구조중 유니온 파인드는 조건에 따라 같은 노드들을 하나로 합치는 연산을 수행할 뿐, 합쳐진 노드들을 분리해내지는 않습니다. 연결성을 끊는 것은 쿼리를 거꾸로 보면 연결하는 것과 연관이 있습니다. 즉 쿼리를 순서대로 보지 않고 거꾸로 보며 유니온 파인드로 문제를 해결해 봅시다.

처음에 모든 헛간이 문을 닫은 상태에서 쿼리의 마지막부터 헛간을 다시 열어 나갑니다. 열려있는 헛간들의 연결성을 확인하는 것은 쿼리에 해당하는 헛간의 집합크기를 보면 됩니다. 마지막 쿼리의 헛간의 부모집합의 크기가 1이라면 모든 헛간이 연결되어있고, 그 다음 쿼리의 헛간의 부모집합의 크기가 2라면 모든 헛간이 연결되어있다는 의미입니다.

$par$배열은 안에 있는 값이 음수하면 해당 인덱스는 부모 노드임을 가리키고, 절댓값은 해당 그룹의 크기를 나타내도록 했습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 2e5 + 5;

int N, M;
int par[MAXN], query[MAXN], ans[MAXN], visited[MAXN];
vector<int> adj[MAXN];

int find(int x) {
    return par[x] < 0 ? x : (par[x] = find(par[x]));
}

void _union(int a, int b) {
    int pa = find(a);
    int pb = find(b);
    if (pa == pb)
        return;
    par[pb] += par[pa];
    par[pa] = pb;
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
    memset(par, -1, sizeof(par));
    rep(i, M) {
        int u, v;
        cin >> u >> v;
        --u, --v;
        adj[u].emplace_back(v);
        adj[v].emplace_back(u);
    }
    rep(i, N) {
        cin >> query[i];
        query[i]--;
    }
    for (int i = N - 1; ~i; --i) {
        int here = query[i];

        for (auto there : adj[here]) {
            if (visited[there])
                _union(here, there);
        }
        if (-par[find(here)] == N - i) {
            ans[i] = 1;
        }
        visited[here] = true;
    }
    rep(i, N) {
        cout << (ans[i] ? "YES\n" : "NO\n");
    }
    return 0;
}
```
