---
title: "[백준 18267] Milk Visits"
date: 2020-11-17T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/18267](https://www.acmicpc.net/problem/18267)

<br>

## 알고리즘

union find

<br>

## 풀이

트리에서 각 노드마다 소의 종류가 정해져 있을 때, 주어진 쿼리에서 그 사이 경로에 해당 소가 있는지 찾는 문제입니다.

트리에서 인접한 노드가 서로 같은 종류의 소라면 유니온 파인드를 통해 묶어줍시다. 이후 쿼리에서 두 노드의 부모노드가 다르다면 경로상에 두 종류의 소가 있음을 알 수 있습니다. 만약 부모노드가 같다면 그 부모노드의 소가 어떤 종류인지 찾아 비교해주면 됩니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 1e5;

int N, M;
int p[MAXN];
char w[MAXN];

int find(int x) {
    return p[x] < 0 ? x : (p[x] = find(p[x]));
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
    rep(i, N) {
        cin >> w[i];
    }
    rep(i, N - 1) {
        int u, v;
        cin >> u >> v;
        --u, --v;
        int pu = find(u);
        int pv = find(v);
        if (w[pu] ^ w[pv])
            continue;
        p[pu] = pv;
    }

    while (M--) {
        int u, v;
        char x;
        cin >> u >> v >> x;
        --u, --v;
        int pu = find(u);
        int pv = find(v);
        if (pu ^ pv)
            cout << 1;
        else if (w[pu] == x)
            cout << 1;
        else
            cout << 0;
    }

    return 0;
}
```
