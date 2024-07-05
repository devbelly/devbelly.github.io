---
title: "[백준 9686] The Exam"
date: 2024-07-05T10:00:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/9686](https://www.acmicpc.net/problem/9686)

<br>

## 알고리즘

Greedy

<br>

## 풀이

인접한 원소의 차이가 최소 K이상이어야 하는 문제입니다.

1부터 $N$까지의 원소중에서 가장 문제가 되는 원소는 중간에 있는 값입니다. 만약 $N$이 7이라면 4를 처리하는 것이 가장 까다롭습니다. 따라서 배열의 맨 처음이나 끝에 보내는 것이 좋습니다. 
그 이후에는 중간을 기준으로 왼쪽과 오른쪽 원소들을 나누어 차례대로 배치하면 됩니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

// 시간복잡도를 고려했나요?
// 풀이과정을 설명할 수 있나요? 또는 막연하게 접근하고 있는건 아닌가요?
// 풀이가 맞는지 다른 예제 케이스로 검증했나요?

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    int N, K;
    cin >> N >> K;

    if (N / 2 < K) {
        cout << "NIE\n";
        return 0;
    }
    vector<int> f, s;
    REP(i, N / 2) {
        f.emplace_back(i);
    }
    for (int i = N / 2 + 1; i <= N; ++i) {
        s.emplace_back(i);
    }
    auto si = s.begin();
    auto fi = f.begin();
    rep(i, N) {
        if (i % 2 == 0) {
            cout << *(si++);
        } else {
            cout << *(fi++);
        }
        cout << ' ';
    }
    return 0;
}
```
