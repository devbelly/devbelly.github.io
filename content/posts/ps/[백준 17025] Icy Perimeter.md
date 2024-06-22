---
title: "[백준 17025] Icy Perimeter"
date: 2020-12-28T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/17025](https://www.acmicpc.net/problem/17025)

<br>

## 알고리즘

DFS

<br>

## 풀이

아이스크림의 형태가 주어질 때, 가장 큰 조각와 둘레의 길이를 구하는 문제입니다. 단, 가장 큰 조각이 여러 개 일때는 가장 작은 둘레를 출력해야합니다.

기초 dfs 문제입니다. 정답 pair의 갱신을 쉽게하기 위해 $cmp$함수를 작성하여 풀었습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

typedef pair<int, int> pii;
const int dy[4] = { 1, -1, 0, 0 };
const int dx[4] = { 0, 0, 1, -1 };

pii ans;
int N;
int board[1002][1002];
bool visited[1002][1002];

void cmp(pii& f, pii s) {
    if (f.first < s.first) {
        f.first = s.first;
        f.second = s.second;
    } else if (f.first == s.first) {
        f.second = min(f.second, s.second);
    }
}

pii dfs(int y, int x) {
    visited[y][x] = true;
    int wid = 1;
    int peri = 0;
    rep(i, 4) {
        int ny = y + dy[i];
        int nx = x + dx[i];
        if (visited[ny][nx])
            continue;
        if (board[ny][nx] == 1) {
            auto [w, p] = dfs(ny, nx);
            wid += w;
            peri += p;
        } else {
            peri += 1;
        }
    }
    return { wid, peri };
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
    REP(i, N) REP(j, N) {
        char x;
        cin >> x;
        if (x == '#')
            board[i][j] = 1;
    }

    REP(i, N) rep(j, N) {
        if (!visited[i][j] && board[i][j] == 1) {
            cmp(ans, dfs(i, j));
        }
    }
    cout << ans.first << ' ' << ans.second << '\n';
    return 0;
}
```
