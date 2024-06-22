---
title: "[백준 19606] Escape Room"
date: 2021-03-15T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/19606](https://www.acmicpc.net/problem/19606)

<br>

## 알고리즘

BFS

<br>

## 풀이

격자판마다 양수가 쓰여있습니다. 양수를 V라고 가정할 때, r X c = V 에 해당하는 (r, c)로 이동이 가능합니다. (1, 1)에서 시작하여 (M, N)으로 이동 가능한지 판별하는 문제입니다.

(1,1)에서 BFS를 수행하려면 조금 까다롭습니다. 가능한 (r, c)를 조사를 해야하는데 이 과정이 시간이 걸립니다. 역으로 (M, N)에서 (1,1)로 이동하면 비교적 간단해집니다. V값을 담고 있는 좌표들을 조사하는 방법으로 수행하면 되기 때문입니다. V마다 좌표를 담기 위해 2차원 배열을 사용했고 BFS를 수행해 나가면 됩니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

typedef pair<int, int> pii;

int N, M;
vector<pii> vt[1000001];
bool visited[1005][1005];

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> M;
    rep(i, N) rep(j, M) {
        int x;
        cin >> x;
        vt[x].emplace_back(i + 1, j + 1);
    }
    queue<int> q;
    q.emplace(N * M);
    visited[N][M] = true;
    while (!q.empty()) {
        int v = q.front();
        q.pop();
        for (auto [x, y] : vt[v]) {
            if (x == 1 && y == 1) {
                cout << "yes";
                exit(0);
            }
            if (visited[x][y])
                continue;
            visited[x][y] = true;
            q.emplace(x * y);
        }
    }
    cout << "no";
    return 0;
}
```
