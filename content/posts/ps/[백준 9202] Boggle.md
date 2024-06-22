---
title: "[백준 9202] Boggle"
date: 2020-08-17T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/9202](https://www.acmicpc.net/problem/9202)

<br>

## 알고리즘

트라이

<br>

## 풀이

격자판에서 주어진 문자열을 조건에 따라 찾아야 하는 문제입니다. 최대 점수, 찾은 단어의 개수, 가장 긴 단어를 구하기를 원하고 있습니다. 단어의 개수를 찾는 과정에서 최대 점수와 가장 긴 단어 또한 해결 가능하므로 단어를 찾는 것에 집중합시다.

격자판 위에서 단어를 탐색해야 하므로 DFS를 통해 일차적인 접근이 가능합니다. DFS연산량만 어림짐작해보면

(주어지는 보글의 수) X (4x4 격자판에서 DFS시작 가능한 위치의 개수) X (DFS의 깊이에 해당하는 단어의 길이) X

(각 알파벳마다 가능한 선택지의 수) 이고, 이는 대략 3천만에 가까운 숫자입니다. 즉 단어를 선형 시간 안에 찾지 않게 되면 시간 초과가 발생할 수 있으므로 트라이를 문제 해결을 위해 선택할 수 있습니다.

메모리 제한은 512MB입니다. 주어지는 단어의 수는 30만이고, 단어의 최대길이는 8, 알파벳 대문자의 갯수는 26개입니다. 즉 대략 249MB를 사용하므로 메모리 제한 안에 들어오게 됩니다. DFS과정에서 단어를 발견했다면 트라이의 위치만 보고 바로 단어에 접근하기 위해 map 자료구조와 이전에 발견한 단어인지 아닌지를 알기 위해 set를 추가적으로 사용했습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i,n) for(int i=0;i<n;++i)
#define REP(i,n) for(int i=1;i<=n;++i)
#define FAST cin.tie(NULL);cout.tie(NULL); ios::sync_with_stdio(false)
using namespace std;

const int MAXC = 26;
const int MAXN = 300000;
const int dy[8] = { -1,-1,-1,0,0,1,1,1 };
const int dx[8] = { -1,0,1,-1,1,-1,0,1 };
const int score_board[9] = { 0,0,0,1,1,2,3,5,11 };

int cnt, w, b;
int ans, score;
int trie[MAXN][MAXC];
map<int, string> term;

char board[4][4];
bool visited[4][4];
string S;

set<int> SET;

void insert(string& s) {
    int p = 0;
    for (auto j : s) {
        if (!trie[p][j - 'A'])
            trie[p][j - 'A'] = ++cnt;
        p = trie[p][j - 'A'];
    }
    term[p] = s;
}

bool inr(int y, int x) {
    return 0 <= y && y < 4 && 0 <= x && x < 4;
}

void dfs(int y, int x, int p, int cur) {
    visited[y][x] = true;

    if (term.find(p) != term.end() && SET.find(p) == SET.end()) {
        SET.insert(p);
        ++ans;

        score += score_board[cur];
        if (term[p].length() > S.length()) {
            S = term[p];
        }
        else if (term[p].length() == S.length()) {
            if (term[p] < S) S = term[p];
        }
    }

    for (int i = 0;i < 8;++i) {
        int ny = y + dy[i];
        int nx = x + dx[i];
        if (!inr(ny, nx)) {
            continue;
        }
        int nxt = board[ny][nx] - 'A';

        if (visited[ny][nx]) {
            continue;
        }

        if (trie[p][nxt]) {
            dfs(ny, nx, trie[p][nxt], cur + 1);
        }
    }
    visited[y][x] = false;
}


int main() {
    FAST;
    cin >> w;
    rep(i, w) {
        cin >> S;
        insert(S);
    }
    cin >> b;
    while (b--) {
        S = "";
        SET.clear();
        score = ans = 0;
        rep(i, 4) rep(j, 4) cin >> board[i][j];

        rep(i, 4) rep(j, 4) {
            if (trie[0][board[i][j] - 'A']) {
                dfs(i, j, trie[0][board[i][j] - 'A'], 1);
            }
        }
        cout << score << ' ' << S << ' ' << ans << '\n';
    }

    return 0;
}
```
