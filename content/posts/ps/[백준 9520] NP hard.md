---
title: "[백준 9520] NP hard"
date: 2020-09-15T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/9520](https://www.acmicpc.net/problem/9520)

<br>

## 알고리즘

DP

<br>

## 풀이

$k$번째 도시를 방문할 때, 그보다 낮은 번호의 도시들은 이전에 다 방문하거나 그 이후에 방문해야 합니다. 모든 도시를 방문할 때 필요한 최소거리를 묻고 있습니다.

1번 도시부터 방문한다고 해봅시다. 순차적으로 2번도시를 처리하게 된다면 다음과 같은 행동을 통해 위 조건을 만족할 수 있습니다.

1. 1번 도시의 오른쪽에 2번도시를 배치한다
2. 1번 도시의 왼쪽에 2번 도시를 배치한다.

위 행동 다음 3번 도시를 배치할 때 또한 지금까지 배치한 도시들의 맨 왼쪽, 또는 맨 오른쪽에 도시를 배치한다면 낮은 번호의 도시를 이후에 방문하거나 이전에 방문하는 것을 처리할 수 있습니다. 즉 부분 문제가 성립하므로 다이나믹 프로그래밍으로 문제를 해결할 수 있습니다.

$solve(a, b)$에서 a, b는 최대 $N$번이 호출될 수 있으므로 시간 복잡도는 $O(N^2)$입니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i,n) for(int i=0;i<n;++i)
#define REP(i,n) for(int i=1;i<=n;++i)
#define FAST cin.tie(NULL);cout.tie(NULL); ios::sync_with_stdio(false)
using namespace std;

int N;
int dist[1500][1500],cache[1500][1500];

int solve(int ql, int qr) {
    if (max(ql, qr) == N-1) return 0;

    int& ret = cache[ql][qr];
    if (~ret) return ret;
    int nxt = max(ql, qr) + 1;

    ret = min(dist[ql][nxt] + solve(nxt,qr),dist[qr][nxt] + solve(ql,nxt));
    return ret;
}

int main() {
    FAST;
    memset(cache, -1, sizeof(cache));

    cin >> N;
    rep(i, N) rep(j, N) cin >> dist[i][j];

    cout<<solve(0, 0);
    return 0;
}
```
