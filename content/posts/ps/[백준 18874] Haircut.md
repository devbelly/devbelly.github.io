---
title: "[백준 18874] Haircut"
date: 2021-02-27T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/18874](https://www.acmicpc.net/problem/18874)

<br>

## 알고리즘

segment tree

<br>

## 풀이

$j$보다 긴 머리카락들이 $j$가 될 때, badhair의 수를 구하는 문제입니다.

관찰이 조금 필요한 문제입니다. 우선 머리카락 길이가 변하는 것을 생각하지 않고 counting inversion을 할 때 브루트포스로 하면 $N^2$이 필요하고 문제에서 $N$제한은 10만이므로 $NlogN$에 inversion을 셀 수 있는 세그먼트 트리를 이용해야합니다.

쿼리($j$)마다 머리카락 길이를 업데이트를 하려면 아무리 빨리한다 하더라도 $O(N)$보다 빠를 순 없고 inversion을 세면 결국 시간초과입니다.

문제의 핵심은 $j$보다 긴 머리카락들이 $j$가 될 때, badhair을 발생시키는 머리카락들은 각자의 길이가 $j$미만이면서 각자의 위치 왼쪽에 있는 머리카락들보다 짧은 머리카락입니다.

즉 머리카락의 길이마다(각자의 길이가 $j$미만) inversion(각자의 위치 왼쪽)을 세주면 정답을 구할 수 있습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 1e5 + 5;
#define int long long

int N;
int ans[MAXN], BIT[MAXN], psum;

void update(int idx, int val) {
    ++idx;
    while (idx <= MAXN) {
        BIT[idx] += val;
        idx += idx & -idx;
    }
}

int query(int idx) {
    ++idx;
    int ret = 0;
    while (idx > 0) {
        ret += BIT[idx];
        idx -= idx & -idx;
    }
    return ret;
}

signed main() {
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
        ans[v] += i - query(v);
        update(v, 1);
    }
    rep(i, N) {
        cout << psum << '\n';
        psum += ans[i];
    }

    return 0;
}
```
