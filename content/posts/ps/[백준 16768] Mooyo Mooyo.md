---
title: "[백준 16768] Mooyo Mooyo"
date: 2020-11-22T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/16768](https://www.acmicpc.net/problem/16768)

<br>

## 알고리즘

DFS

<br>

## 풀이

인접한 곳에 동일한 숫자가 있으면 연결이 됩니다. 연결된 크기가 $K$이상일 때 숫자들은 터지고 터지는 숫자들이 없을 때까지 시뮬레이션 하여 결과를 출력하면 됩니다.

[이 문제](https://www.acmicpc.net/problem/11559)와 거의 유사한 문제로 조건이 4개가 모이면 터지는 것에서 $K$개가 모이면 터지는 것 외엔 다른 것이 없습니다. 연결된 크기를 세기 위한 $dfs$함수와 터지고나서 숫자들을 떨어뜨리는 $gravity$함수만 잘 구현하면 쉽게 풀 수 있습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int dy[4] = { 1, -1, 0, 0 };
const int dx[4] = { 0, 0, 1, -1 };

int N, K;
bool visited[100][10];
char board[100][10];

bool inr(int y, int x) {
    return 0 <= y && y < N && 0 <= x && x < 10;
}

int dfs(int y, int x, bool flag) {
    visited[y][x] = true;
    int ret = 1;
    rep(i, 4) {
        int ny = y + dy[i];
        int nx = x + dx[i];
        if (inr(ny, nx) && !visited[ny][nx] && (board[ny][nx] == board[y][x])) {
            ret += dfs(ny, nx, flag);
        }
    }
    if (flag)
        board[y][x] = '0';
    return ret;
}

bool update() {
    memset(visited, 0, sizeof(visited));
    bool ret = false;
    rep(i, N) rep(j, 10) {
        if (board[i][j] != '0' && !visited[i][j]) {
            if (dfs(i, j, 0) >= K) {
                memset(visited, 0, sizeof(visited));
                ret = true;
                dfs(i, j, 1);
            }
        }
    }
    return ret;
}

void gravity() {
    rep(j, 10) for (int i = N - 1; i >= 0; --i) {
        if (board[i][j] != '0') {
            for (int ii = i; ii + 1 < N && board[ii + 1][j] == '0'; ++ii) {
                swap(board[ii][j], board[ii + 1][j]);
            }
        }
    }
}

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> K;
    rep(i, N) rep(j, 10) {
        cin >> board[i][j];
    }
    while (update()) {
        gravity();
    }

    rep(i, N) {
        rep(j, 10) {
            cout << board[i][j];
        }
        cout << '\n';
    }
    return 0;
}
```
