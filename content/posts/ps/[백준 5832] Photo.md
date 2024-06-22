---
title: "[백준 5832] Photo"
date: 2021-07-21T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[www.acmicpc.net/problem/5832](www.acmicpc.net/problem/5832)

<br>

## 알고리즘

Greedy

<br>

## 풀이

겹치는 선분이 있을 때, 해당 구간을 잘라 그리디하게 푸는 문제입니다.

$K$ 쌍을 입력받고 선분의 끝점 값을 기준으로 정렬합니다. 현재 보고 있는 선분의 구간이 [u,v] 일 때, 선분들 중 시작점에 있는 값이 v보다 작다면 겹치는 구간이 발생하는 선분입니다. 해당 선분들은 [u,v] 구간을 자를 때 같이 해결이 되므로 $visited$배열을 통해 체크하며 답을 찾으면 쉽게 풀 수 있습니다. $K$쌍이 주어질 때 u>v 일 수도 있으니 u>v라면 swap을 통해 시작점<끝점을 유지해주도록 합시다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

typedef pair<int, int> pii;

int N, K, ans = 1;
bool visited[1000];
vector<pii> vt;
int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> K;
    rep(i, K) {
        int u, v;
        cin >> u >> v;
        if (u > v) swap(u, v);
        vt.emplace_back(u, v);
    }
    sort(vt.begin(), vt.end(), [](const pii& a, const pii& b) { return a.second < b.second; });
    rep(i, K) {
        if (!visited[i]) {
            ++ans;
            auto [u, v] = vt[i];
            for (int j = i + 1; j < K; ++j) {
                auto [s, t] = vt[j];
                if (s < v) {
                    visited[j] = true;
                }
            }
        }
    }
    cout << ans;
    return 0;
}
```
