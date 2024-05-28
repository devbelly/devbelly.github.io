---
title: "[백준 9869] Milk Scheduling"
date: 2021-05-23T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/9869](https://www.acmicpc.net/problem/9869)

<br>

## 알고리즘

Greedy

<br>

## 풀이

소마다 우유생산량과 수명이 주어질 때, 최대 우유생산량을 구하는 문제입니다.

자주 보이는 그리디 테크닉입니다. 시간을 0부터 접근하는 대신, 가장 마지막 시간부터 0까지 역순으로 소들을 고려합니다. T에 해당하는 시간에 젖을 짤 수 있는 소들을 PQ에 넣습니다. 이렇게 하면 현재 젖을 짤 수 있는 소들이 PQ에 담기게 됩니다. 이 가운데 우유 생산량이 높은 소부터 젖을 짜나가면 됩니다.

<br>

## 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

typedef pair<int, int> pii;
const int MAXN = 1e5;
int N, T, ans;
bool used[MAXN + 5];
int main() {
#ifndef ONLINE_JUDGE
    freopen("3.in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N;
    vector<pii> vt;
    rep(i, N) {
        int gal, dead;
        cin >> gal >> dead;
        T = max(T, dead);
        vt.emplace_back(gal, dead);
    }
    priority_queue<pii> pq;
    sort(vt.begin(), vt.end(), [](const pii& a, const pii& b) {
        return a.second > b.second;
    });
    int i = 0;
    while (T) {
        for (; i < N;) {
            if (vt[i].second == T) pq.emplace(vt[i++]);
            else
                break;
        }
        if (!pq.empty()) {
            ans += pq.top().first;
            pq.pop();
        }
        --T;
    }

    cout << ans;
    return 0;
}
```
