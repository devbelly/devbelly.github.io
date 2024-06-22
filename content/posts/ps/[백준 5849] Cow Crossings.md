---
title: "[백준 5849] Cow Crossings"
date: 2021-07-22T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[www.acmicpc.net/problem/5849](www.acmicpc.net/problem/5849)

<br>

## 알고리즘

stack

<br>

## 풀이

inversion을 만들지 않는 소의 개수를 구하는 문제입니다.

$a_i$를 u, $b_i$를 v라고 하겠습니다. inversion을 만들지 않기 위해서는 u를 기준으로 정렬했을 때, v가 오름차순의 형태를 가져야만합니다. 오름차순을 유지하기 위해 스택을 사용해서 문제를 풀면 됩니다. 스택의 top이 현재 보고 있는 소의 v보다 크다면 이는 inversion을 만들기 때문에 계속 pop을 해주고, 지금까지 확인한 v값들보다 현재 v값이 작다면 현재 v는 inversion을 만들지 않으므로 스택에 push 해주면 됩니다. 아래 코드는 이 해설과는 다른 코드입니다. 풀이만 참고하세요.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

typedef pair<int, int> pii;

int N, ans, piv;
vector<pii> xy;
vector<int> X, Y;
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
        int u, v;
        cin >> u >> v;
        xy.emplace_back(u, v);
        X.emplace_back(u);
        Y.emplace_back(v);
    }
    sort(X.begin(), X.end());
    sort(Y.begin(), Y.end());
    sort(xy.begin(), xy.end(), [](pii a, pii b) { return min(a.first, a.second) < min(b.first, b.second); });
    rep(i, N) {
        auto [u, v] = xy[i];
        // cout << u << ' ' << v << '\n';
        // cout << piv << '\n';
        int U = lower_bound(X.begin(), X.end(), u) - X.begin();
        int V = lower_bound(Y.begin(), Y.end(), v) - Y.begin();
        if (U == V && piv <= U) {
            ++ans;
        } else {
            piv = max(piv, max(U, V));
        }
    }
    cout << ans;
    return 0;
}
```
