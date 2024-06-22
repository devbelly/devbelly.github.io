---
title: "[백준 14450] Hoof, Paper, Scissors(Gold)"
date: 2021-02-18T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/14450](https://www.acmicpc.net/problem/14450)

<br>

## 알고리즘

DP

<br>

## 풀이

소와 존이 가위바위보를 진행합니다. 소는 가위바위보 전문가라 존이 무엇을 낼지 미리 알고 있지만, 게을러서 연속적인 모션을 취합니다. 최대 $K$번 내기 시작하는 것을 바꿀 수 있을 때, 얻을 수 있는 최대 점수를 구하는 문제입니다.

$i$번째의 결과가 바로 뒤의 $i+1$번의 결과에 영향을 미친다는 점에서 다이나믹 프로그래밍을 떠올릴 수 있습니다. 우리는 현재 가위바위보 단계에서 저장해야할 정보는 다음 세가지 입니다. 현재 몇 번째 가위바위보를 진행하고 있는지, $K$를 몇번 사용했는지, 현재 단계에서 소가 내고 있는 제스쳐는 무엇인지 입니다. 이를 위해 총 3차원의 배열을 사용하며 $cache[i][j][k]$가 영향을 미치는 다음단계는 아래 세가지 입니다.

<p align=center>
	1. $cache[i+1][j][k]$ = 제스쳐를 바꾸지 않고 그대로 진행 <br>
2. $cache[i+1][j+1][k+1]$ = 제스쳐를 바꿈(1) <br>
3. $cache[i+1][j+2][k+1]$ = 제스쳐를 바꿈(2)
</p>

$j$에서 0은 Hoop , 1은 Scissors, 2는 paper을 의미합니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 1e5 + 5;

int N, K, ans, prv, cur = 1;
int cache[2][3][21];

inline int versus(int a, int b) { // 작으면 이긴다
    if (a == 0 && b == 2)
        return 0;
    if (a == 2 && b == 0)
        return 1;

    return a < b;
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
    memset(cache, -1, sizeof(cache));
    char x;
    cin >> x;
    int v = (x == 'P' ? 2 : x == 'S' ? 1 : 0);
    rep(i, 3) {
        cache[0][i][0] = versus(i, v);
    }
    for (int i = 0; i < N - 1; ++i) {
        char x;
        cin >> x;
        int v = (x == 'P' ? 2 : x == 'S' ? 1 : 0);
        for (int j = 0; j < 3; ++j) {
            for (int k = 0; k <= K; ++k) {
                if (~cache[prv][j][k]) {
                    cache[cur][j][k] = max(cache[cur][j][k], cache[prv][j][k] + versus(j, v));

                    if (k < K) {
                        REP(nxt, 2) {
                            cache[cur][(j + nxt) % 3][k + 1] =
                                max(cache[cur][(j + nxt) % 3][k + 1], cache[prv][j][k] + versus((j + nxt) % 3, v));
                        }
                    }
                }
            }
        }
        swap(prv, cur);
    }
    rep(i, 3) rep(j, K + 1) {
        ans = max(ans, cache[prv][i][j]);
    }
    cout << ans;
    return 0;
}
```
