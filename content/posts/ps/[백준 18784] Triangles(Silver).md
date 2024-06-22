---
title: "[백준 18784] Triangles(Silver)"
date: 2020-11-15T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/18784](https://www.acmicpc.net/problem/18784)

<br>

## 알고리즘

정렬

<br>

## 풀이

$N$개의 좌표가 주어질 때, 3개를 골라 만들 수 있는 삼각형의 넓이의 총 합 \* 2을 구하는 문제입니다. 단, 만든 삼각형은 X, Y축에 평행하는 변이 존재해야합니다.

각 축에 평행해야하는 조건을 만족하려면 한 점에서 직교해야합니다. $(x1, y1)$ 에서 직교하는 삼각형을 찾기 위해 x좌표가 $x1$인 모든 좌표들과 y좌표가 $y1$인 모든 좌표들을 구해야합니다. 각각의 집합을 A, B라고 하겠습니다. $\sum (|A_i-y1|)$을 통해 y축에 평행한 모든 길이의 합을 구할 수 있고 x축에 평행한 변도 마찬가지로 풀면 됩니다.

남은 문제는 길이의 합을 구하는 것입니다. $x1$을 공유하는 $y$좌표 $a, b, c ,d$ 가 있다고 가정하겠습니다. a와 떨어진 모든 거리들의 합을 $psum$이라고 한다면 b와 떨어진 모든 거리들의 합을 $psum$을 이용하여 $O(1)$에 구할 수 있습니다. 현재 가리키는 인덱스가 j(즉 현재는 b를 가리키므로 1) 일 때$b-a$의 거리를 고려해야하는 갯수는 j개 증가하지만 고려하지 않아도 되는 갯수는 (len-j)개 입니다. 즉 $psum$ += $(2*j-len)(b-a)$를 통해 다음 $psum$을 구할 수 있습니다.

마지막으로 모듈러를 관리하기 위해 $modint$ 구조체를 사용했습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

typedef long long ll;
typedef pair<int, int> pii;
const int mod = 1e9 + 7;
const int MAXN = 1e5;

struct mi {
    int v;
    mi(ll _v) : v(_v % mod) {
        v += (v < 0) * mod;
    }
    mi() : v(0) { }
    mi operator+(mi b) {
        return mi(v + b.v);
    }
    mi operator-(mi b) {
        return mi(v - b.v);
    }
    mi operator*(mi b) {
        return mi((ll)v * b.v);
    }
};

int N;

vector<vector<pii>> xyi;
vector<pii> xy;
vector<mi> ans[MAXN + 1];

void solve() {
    for (int i = 0; i <= 20000; ++i) {
        int len = xyi[i].size();
        if (len) {
            mi psum = 0;
            for (int j = 0; j < len; ++j) {
                psum = psum + xyi[i][j].first - xyi[i][0].first;
            }
            for (int j = 0; j < len; ++j) {
                if (j)
                    psum = psum + (2 * j - len) * (xyi[i][j].first - xyi[i][j - 1].first);
                ans[xyi[i][j].second].emplace_back(psum);
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

    cin >> N;
    xyi.resize(20001);
    rep(i, N) {
        int x, y;
        cin >> x >> y;
        xy.emplace_back(x, y);
        xyi[x + 10000].emplace_back(y, i);
    }
    solve();
    xyi.clear();
    xyi.resize(20001);
    rep(i, N) {
        auto [x, y] = xy[i];
        xyi[y + 10000].emplace_back(x, i);
    }
    solve();
    mi ret = 0;
    for (int i = 0; i < N; ++i) {
        ret = ret + ans[i][0] * ans[i][1];
    }
    cout << ret.v;
    return 0;
}
```
