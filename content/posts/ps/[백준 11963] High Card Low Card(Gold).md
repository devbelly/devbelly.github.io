---
title: "[백준 11963] High Card Low Card(Gold)"
date: 2021-02-27T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/11963](https://www.acmicpc.net/problem/11963)

<br>

## 알고리즘

greedy

<br>

## 풀이

2$N$개의 카드를 나누어 가져 게임을 진행할 때, 얻을 수 있는 최대 점수를 구하는 문제입니다. $N$/2 이전 라운드는 큰 수를 내는 사람이 이기고, 이후 라운드는 작은 수를 내는 사람이 이깁니다.

큰 수 게임에서 bessie가 이길 수 있는 카드들이 있을 때, 최대한 작은 수를 내야 나중의 라운드를 준비할 때 유리합니다. 이를 위해 큰 수 게임을 진행할 때, elsie카드들 중 큰 수부터 처리해 나갑니다. 작은 수 게임도 마찬가지 이유로 elsie카드들 중 작은 수부터 처리해 나갑니다.

이 때 큰 수게임 중 카드를 낼 때, 이길 수 있는 카드들 중에서 작은 카드를 내게 되면 작은 수 게임에서 불리해질 수 도 있습니다. 이를 위해 작은 수 게임을 진행한 후 큰 수 게임을 진행하면 됩니다. 작은 수 게임을 진행하면서도 elsie카드들이 정렬이 되어있으므로 낼 수 있는 카드들 중에서 가장 작은 카드들을 내도 상관이 없어집니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

int N, q, ans;
int arr[100001];

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N;
    rep(i, N) {
        int v;
        cin >> v;
        arr[v] = (i < N / 2 ? 1 : 2);
    }

    for (int i = 1; i <= 2 * N; ++i) {
        if (arr[i] == 0)
            ++q;
        else if (arr[i] == 2 && q) {
            ++ans;
            --q;
        }
    }
    q = 0;
    for (int i = 2 * N; i >= 1; --i) {
        if (arr[i] == 0)
            ++q;
        else if (arr[i] == 1 && q) {
            ++ans;
            --q;
        }
    }
    cout << ans;
    return 0;
}
```
