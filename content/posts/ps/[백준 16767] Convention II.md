---
title: "[백준 16767] Convention II"
date: 2020-11-21T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/16767](https://www.acmicpc.net/problem/16767)

<br>

## 알고리즘

priority queue

<br>

## 풀이

소들의 도착시간과 풀을 먹는 시간이 주어질 때, 가장 오래 기다리는 소의 시간을 출력하는 문제입니다. 한 소가 풀을 먹을 때 다른 소들은 기다려야하며, 대기하는 소들이 여럿일 때는 연장자부터 풀을 먹습니다.

우선순위 큐를 사용하는 문제입니다. info 구조체를 만들어 풀도록 합시다. 각각 나이, 도착시간, 풀을 먹는 시간에 대한 정보를 담고 있습니다. 구조체를 우선순위 큐, lower bound에서 사용하기 위해 각각 비교함수들을 작성했습니다. 우선순위 큐에 있는 소들을 처리할 때, 또다른 소가 도착할 수 있음을 유의합시다. 마지막 소까지 시뮬레이션이 끝나고 큐를 모두 비워내기 위해 for문의 범위를 $i<N$ 에서 $i\leq N$으로 수정하였습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 1e5;

struct info {
    int l, s, t;
};

struct cmp {
    bool operator()(info a, info b) {
        return a.l > b.l;
    }
};

bool operator<(info const a, info const b) {
    if (a.s == b.s)
        return a.l < b.l;
    return a.s < b.s;
};

priority_queue<info, vector<info>, cmp> pq;

int N;
info cow[MAXN];
info k;
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
        cow[i].l = i;
        cin >> cow[i].s >> cow[i].t;
    }
    sort(cow, cow + N);

    int ans = -1;

    for (int i = 0; i <= N;) {
        if (pq.empty()) {
            k.s = cow[i].s + cow[i].t;
            i += 1;
        } else {
            auto [L, S, T] = pq.top();
            pq.pop();
            ans = max(ans, k.s - S);
            k.s += T;
        }

        int nxt = lower_bound(cow + i, cow + N, k) - cow;

        for (; i < nxt; ++i) {
            pq.emplace(cow[i]);
        }
    }

    cout << ans;
    return 0;
}
```
