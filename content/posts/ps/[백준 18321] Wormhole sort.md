---
title: "[백준 18321] Wormhole sort"
date: 2020-11-05T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/18321](https://www.acmicpc.net/problem/18321)

<br>

## 알고리즘

union find

<br>

## 풀이

$N$개의 지역과 $N$마리의 소가 있습니다. 초기 배치 상태가 주어질 때, $i$번 지역에는 $i$번 소가 있도록 해야 합니다. 소들이 움직이기 위해 wormhole이 주어질 때, 사용하는 wormhole 넓이의 최솟값이 최대가 하도록 요구하는 문제입니다.

최소의 최대화, 최대의 최소화는 익숙한 이분탐색 주제입니다. 이분 탐색 + DFS로도 풀 수 있으나 좀 더 빠른 Union find로 풀이법을 소개해드리겠습니다.

초기에는 모든 지역이 이어져 있지 않다고 가정합시다. 그리디하게 넓이가 큰 간선부터 양 지역을 연결하는 것은 최선의 전략입니다. 정답이 $k$인데 $k+x$넓이의 간선(x는 양수)을 사용하는 것은 정답에 아무런 영향을 끼치지 않기 때문입니다. 즉 가장 큰 간선부터 지역을 이어나가며 모든 지역이 연결되었다면 마지막에 사용한 간선이 최소이므로 해당 간선의 넓이가 정답입니다. 이때, 지역 간의 연결성을 확인하기 위해 union find를 사용합니다. 시간 복잡도는 $O(MlogM + N+M)$입니다. 코드의 41번째 줄은 while문이기는 하나 이미 연결된 지역에 대해서 다시 루프를 도는 것이 아닙니다. 즉 최대 수행은 $O(N)$입니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 1e5 + 1;

struct edges {
    int w, u, v;
};

int N, M;
int cow[MAXN], p[MAXN];
edges eg[MAXN];

int find(int x) {
    return p[x] < 0 ? x : (p[x] = find(p[x]));
}

int main() {
#ifndef ONLINE_JUDGE
    freopen("9.in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> M;
    memset(p, -1, sizeof(p));
    REP(i, N) {
        cin >> cow[i];
    }
    rep(i, M) {
        cin >> eg[i].u >> eg[i].v >> eg[i].w;
    }
    sort(eg, eg + M, [&](edges& f, edges& s) { return f.w > s.w; });
    int ans = -1;
    for (int m = 0,j=1; m < M; ++m) {
        int sorted = 0;
        while(j<=N&&find(j)==find(cow[j])) ++j;
        if(j==N+1) break;

        if (sorted == N)
            break;
        auto [W, U, V] = eg[m];
        V = find(V);
        U = find(U);
        if (V ^ U) {
            p[V] = U;
            ans = W;
        }
    }

    cout << ans;

    return 0;
}
```
