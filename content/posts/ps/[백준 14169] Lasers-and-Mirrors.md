---
title: "[백준 14169] Lasers-and-Mirrors"
date: 2021-02-14T11:10:28+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---


## 문제

[https://www.acmicpc.net/problem/14169](https://www.acmicpc.net/problem/14169)

<br>

## 알고리즘

BFS

<br>

## 풀이

레이저를 시작점에서 농장까지 비추도록 거울을 설치해야 합니다. 이때 필요한 최소의 거울 수를 묻고 있습니다.

좌표가 작다면 [이 문제](https://www.acmicpc.net/problem/2151)처럼 2차원 배열을 만든 후 BFS를 수행하면 됩니다. 하지만 이 문제는 좌표의 범위가 10억 이하이므로 배열을 선언할 수 없을 뿐더러 거울을 설치할 수 있는 장소의 개수인 $N$이 최대 10만이므로 좌표압축을 해도 2차원 배열안에는 넣을 수 없습니다.

핵심은 주어진 좌표를 BFS를 사용할 수 있도록 적절하게 자료구조에 넣는 것 입니다. 이를 위해 $adj$를 사용했습니다. 인접한 좌표끼리 연결할 때(인접하는 것은 x좌표가 동일한 좌표끼리 묶고 y좌표가 동일한 좌표끼리 묶는 것 입니다), 바로 근처에 있는 좌표만 담아주도록 합시다(예를 들어 1,1 1,2 1,3 이 주어진다면 1,1과 1,3은 연결할 필요가 없음).

<p align=center>
	$adj[i]$ = $i$번째 좌표와 인접한 벡터. first에는 $j$번째 좌표, second에는 방향
</p>

을 담았습니다. 방향은 위, 아래로 향하면 1, 왼쪽, 오른쪽을 향하면 0으로 설정했습니다. 만일 방향이 같다면 거울을 설치하지 않은 것이므로 cost는 0이지만 방향이 다르다면 cost는 1입니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 1e5 + 5;
typedef tuple<int, int, int> tp;
typedef pair<int, int> pii;

int N, a, b, c, d;
vector<tp> xy;
vector<pii> adj[MAXN];
int visited[MAXN];

int main() {
#ifndef ONLINE_JUDGE
    freopen("2.in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    memset(visited, 0x3f, sizeof(visited));
    cin >> N >> a >> b >> c >> d;
    xy.emplace_back(a, b, N);
    xy.emplace_back(c, d, N + 1);
    rep(i, N) {
        int x, y;
        cin >> x >> y;
        xy.emplace_back(x, y, i);
    }

    sort(xy.begin(), xy.end());
    rep(i, N + 1) {
        auto [a, b, idx] = xy[i];
        auto [c, d, IDX] = xy[i + 1];
        if (a == c) {
            adj[idx].emplace_back(IDX, 1);
            adj[IDX].emplace_back(idx, 1);
        }
    }
    sort(xy.begin(), xy.end(), [](tp a, tp b) {
        auto [q, w, e] = a;
        auto [z, x, c] = b;

        if (w == x)
            return q < z;
        return w < x;
    });
    rep(i, N + 1) {
        auto [a, b, idx] = xy[i];
        auto [c, d, IDX] = xy[i + 1];
        if (b == d) {
            adj[idx].emplace_back(IDX, 0);
            adj[IDX].emplace_back(idx, 0);
        }
    }
    queue<pii> q;
    q.emplace(N, 0);
    q.emplace(N, 1);

    visited[N] = 0;
    while (!q.empty()) {
        auto [here, dir] = q.front();
        q.pop();

        for (auto [there, tdir] : adj[here]) {
            if (visited[there] < visited[here] + 1)
                continue;

            visited[there] = min(visited[there], visited[here] + (dir ^ tdir));
            int nd = (tdir == dir ? dir : 1 - dir);
            q.emplace(there, nd);
        }
    }
    cout << (visited[N + 1] == 1061109567 ? -1 : visited[N + 1]);
    return 0;
}
```
