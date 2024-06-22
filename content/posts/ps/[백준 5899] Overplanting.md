---
title: "[백준 5899] Overplanting"
date: 2021-09-14T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[www.acmicpc.net/problem/5899](www.acmicpc.net/problem/5899)

<br>

## 알고리즘

sweeping

<br>

## 풀이

사각형의 총 넓이를 구하는 문제입니다.

대표적인 스위핑문제입니다. 골드문제여서 $N^2$에 해결할 수 있는 문제입니다. 사각형을 두 개의 선분, 오른쪽 선분과 왼쪽 선분으로 나눕니다. 이후 x좌표를 기준으로 정렬합니다. 왼쪽 선분에 해당하면 set에 넣고 오른쪽 선분에 해당하면 set에서 해당 선분을 제외합니다. set은 지금까지 본 선분들 중에서 아직 넓이를 계산 해야하는 선분들 입니다. 현재 선분과 지금까지 가지고 있는 선분들을 통해 겹치는 y좌표들을 제거하면 중복된 사각형을 제외한 총 넓이를 구할 수 있습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

#define int long long

const int MAXN = 1e3;

struct p {
    int x, y, y2, t;
    bool operator<(const p& a) const {
        return y < a.y;
    }
};

int N, ans;
p arr[MAXN << 1];
set<p> st;
signed main() {
#ifndef ONLINE_JUDGE
    freopen("10.in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N;
    rep(i, N) {
        int a, b, c, d;
        cin >> a >> b >> c >> d;
        arr[i * 2].x = a;
        arr[i * 2].y = d;
        arr[i * 2].y2 = b;
        arr[i * 2].t = 1;
        arr[i * 2 + 1].x = c;
        arr[i * 2 + 1].y = d;
        arr[i * 2 + 1].y2 = b;
        arr[i * 2 + 1].t = -1;
    }
    int X = INT_MIN;
    sort(arr, arr + 2 * N, [](const p& a, const p& b) { return a.x < b.x; });
    rep(i, 2 * N) {
        auto [a, b, c, d] = arr[i];
        int prv = INT_MIN;
        for (auto [a1, b1, c1, d1] : st) {
            prv = max(prv, b1);
            int len = max((long long)0, c1 - prv);
            ans += len * (a - X);
            prv = max(prv, c1);
        }
        if (d == 1) {
            st.emplace(arr[i]);
        } else {
            arr[i].t = 1;
            st.erase(arr[i]);
        }
        X = a;
    }

    cout << ans;
    return 0;
}
```
