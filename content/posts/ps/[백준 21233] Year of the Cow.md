---
title: "[백준 21233] Year of the Cow"
date: 2021-03-17T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/21233](https://www.acmicpc.net/problem/21233)

<br>

## 알고리즘

greedy

<br>

## 풀이

12의 배수에 해당하는 연도에 포탈이 열립니다. 모든 선조들을 만나고 돌아올 때, 걸리는 가장 짧은 시간을 구하는 문제입니다. 포탈은 최대 $K$번 이용할 수 있습니다.

전형적인 그리디 문제입니다. 포탈을 사용해서 건너뛰어야하는 구간은 연도차이가 가장 큰 구간입니다. 각 선조들의 연도를 거쳐야만 하는 연도(12의 배수중 하나)로 바꿉니다. 예를 들어 35년 전에 존재하는 선조는 36년도 전에 열리는 포탈을 거쳐야만 (또는 더 이전의 포탈에서 시간이 흘러야만) 올 수 있습니다. 거쳐야하는 포탈을 골랐다면 각 포탈의 차이(A포탈의 시작과 B 포탈의 끝)를 담는 벡터를 만든 후, 정렬해서 가장 큰 것부터 $K-1$개를 제외해나가면 됩니다. $K-1$인 이유는 처음에 과거로 갈 때 포탈을 한번 사용하기 때문입니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

int N, K, ans;
vector<int> por, gap;
map<int, bool> visited;

int main() {
#ifndef ONLINE_JUDGE
    freopen("10.in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> K;
    rep(i, N) {
        int x;
        cin >> x;
        x = (x / 12 + 1) * 12;

        if (!visited[x]) {
            por.emplace_back(x);
            visited[x] = true;
        }
    }
    por.emplace_back(0);
    sort(por.rbegin(), por.rend());
    ans = *por.begin();

    for (int i = 0; i < (int)por.size() - 1; ++i) {
        gap.emplace_back(por[i] - 12 - por[i + 1]);
    }
    sort(gap.rbegin(), gap.rend());
    for (int i = 0; i < K - 1 && i < gap.size(); ++i) {
        ans -= gap[i];
    }
    cout << ans;

    return 0;
}
```
