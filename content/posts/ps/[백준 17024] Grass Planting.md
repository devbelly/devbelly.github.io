---
title: "[백준 17024] Grass Planting"
date: 2020-12-27T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/17024](https://www.acmicpc.net/problem/17024)

<br>

## 알고리즘

Greedy

<br>

## 풀이

인접한 지역과 거의 인접한 지역에 다른 종류의 풀을 심어야 할 때, 풀 종류의 최솟값을 묻는 문제입니다.

주어진 예제말고도 그림을 그려 특징을 파악하며 풀면 됩니다. 한 노드 A에 인접한 노드 B, C, D가 있다고 가정해봅시다. 이 노드들은 A와는 무조건 다른 종류의 풀을 사용해야만 합니다. 하지만 A를 제외한 B의 인접 노드에 풀을 심을 때는 A와 다른 종류의 풀을 심어야 하지만(A는 두칸이 떨어져 있어 거의 인접한 지역이므로) C, D에 심었던 풀을 심을 수 있음을 관찰하면 됩니다. 일반화하면 노드 중 최대 indegree값 + 1 이 정답입니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

int N;
int cnt[100000];

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N;
    rep(i, N - 1) {
        int a, b;
        cin >> a >> b;
        --a, --b;
        cnt[a]++;
        cnt[b]++;
    }
    cout << *max_element(cnt, cnt + N) + 1;

    return 0;
}
```
