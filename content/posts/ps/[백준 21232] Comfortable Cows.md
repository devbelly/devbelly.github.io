---
title: "[백준 21232] Comfortable Cows"
date: 2021-03-16T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/21232](https://www.acmicpc.net/problem/21232)

<br>

## 알고리즘

queue

<br>

## 풀이

$N$마리의 소가 차례대로 목장에 추가가 됩니다. 소의 주변에 3마리의 소가 있으면 소는 Comfortable한 상태로 됩니다. 이 상태에선 우유생산을 제대로 하지 못하므로 새로운 소를 추가해서 이를 방지하려 합니다. 이때 새로이 추가해야하는 소의 마릿수를 정해진 소를 추가할때마다 구하는 문제입니다.

$adj[i][j]$는 $(i, j)$에 살고 있는 소의 인접한 소의 마릿수를 나타내는 배열입니다. 정해진 소를 추가할때마다 영향을 미치는 좌표는 해당 소의 상하좌우에 있는 좌표입니다. 영향을 받는 좌표들이 있다면 해당 좌표들의 $adj$값이 3인지 확인해주고 3이라면 큐에 넣어줍니다. 큐는 comfortable한 상태들의 소를 모아놓은 큐입니다.

유의해야할 점은 $(i, j)$를 추가할 때, 자신의 좌표도 이미 $adj$값이 3인지 확인을 해야합니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int dy[4] = { 1, -1, 0, 0 };
const int dx[4] = { 0, 0, 1, -1 };
typedef pair<int, int> pii;

queue<pii> q;
int N, ans;
int adj[4000][4000];
bool visited[4000][4000], addcow[4000][4000];
int main() {
#ifndef ONLINE_JUDGE
    freopen("10.in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N;
    rep(i, N) {
        int y, x;
        cin >> y >> x;
        x += 2000;
        y += 2000;

        visited[y][x] = true;
        if (adj[y][x] == 3) {
            q.emplace(y, x);
        }
        rep(j, 4) {
            int ny = y + dy[j];
            int nx = x + dx[j];
            if (!addcow[y][x])
                adj[ny][nx] += 1;

            if (visited[ny][nx] || addcow[ny][nx]) {
                if (adj[ny][nx] == 3) {
                    q.emplace(ny, nx);
                }
            }
        }
        if (addcow[y][x]) {
            addcow[y][x] = false;
            ans -= 1;
        }

        while (!q.empty()) {
            auto [y, x] = q.front();
            q.pop();
            rep(j, 4) {
                int ny = y + dy[j];
                int nx = x + dx[j];
                if (!visited[ny][nx] && !addcow[ny][nx]) {
                    ans += 1;
                    addcow[ny][nx] = 1;
                    if (adj[ny][nx] == 3) {
                        q.emplace(ny, nx);
                    }
                    rep(k, 4) {
                        int nny = ny + dy[k];
                        int nnx = nx + dx[k];
                        adj[nny][nnx] += 1;

                        if (adj[nny][nnx] == 3 && (visited[nny][nnx] || addcow[nny][nnx])) {
                            q.emplace(nny, nnx);
                        }
                    }
                }
            }
        }

        cout << ans << '\n';
    }
    return 0;
}
```
