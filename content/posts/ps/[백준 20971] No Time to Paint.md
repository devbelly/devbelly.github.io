---
title: "[백준 20971] No Time to Paint"
date: 2021-04-30T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/20971](https://www.acmicpc.net/problem/20971)

<br>

## 알고리즘

queue

<br>

## 풀이

특정 구간을 칠하지 않은 상태로 할 때, 남은 구간들을 칠하기 위한 최솟값을 구하는 문제입니다. 문제의 가장 큰 특징은 페인트를 칠하는 순서가 정해져 있다는 것입니다. 이를 유의하며 풀도록 합시다.

쿼리를 빠르게 처리하기 위해 DP 아이디어를 떠올렸습니다. 특정 구간에 대한 DP를 사용하기 위해선 $cache[i][j]$와 같은 배열을 사용할 수 있는데, 구간의 길이가 100,000 이라는 점에서 해당 2차원 배열을 사용하기는 어렵습니다.

하지만 문제를 자세히 보면 특정 구간을 칠하기 위한 최솟값이 아닌 특정 구간을 제외한 나머지 구간을 칠하기 위한 최솟값을 묻고 있습니다. 이는 DP의 정의가 "앞에서부터 길이 $N$인 구간을 칠하기 위한 최솟값" 과 " 뒤에서부터 길이 $N$인 구간을 칠하기 위한 최솟값" 이라면 두 1차원 배열을 통해 문제에서 요구하는 값을 구할 수 있습니다.

이제 가장 중요한 $cache[i]$와 $cache[i+1]$ 의 관계에 대해 살펴봐야합니다. 주어지는 문자열을 "ABCB" 라고 가정하겠습니다. ABC에서 ABCB를 만들때는 최솟값이 변하지 않습니다. 왜냐하면 A... - ABBB - ABCB 와 같이 진행하면 되기 때문입니다. 일반화해서 말하자면 가장 최근에 등장한 색깔 'B'의 위치와 현재 위치 사이에 자신보다 옅은 색깔이 존재하지

않기 때문입니다. 즉 인덱스 2와 인덱스 4 사이에 자신보다 옅은 색깔이 존재하지 않습니다.

$cache[i]$에서 $cache[i+1]$를 갱신할 때, 현재의 색깔이 처음 등장하거나, 가장 최근에 등장했던 현재 색깔의 위치와 현재 위치 사이에 옅은 색깔이 존재한다면 이전 상태에서 +1을 해주고 아니라면 이전과 동일한 값입니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 1e5 + 5;

int N, Q;
int cache[2][MAXN];
string S;

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> Q;
    cin >> S;

    for (int d = 0; d < 2; ++d) {
        int color[26] {};
        memset(color, -1, sizeof(color));
        for (int i = 0; i < N; ++i) {
            int c = S[i] - 'A';
            cache[d][i + 1] =
                cache[d][i] + (color[c] == -1 || any_of(color, color + c, [&](int pos) { return color[c] < pos; }));
            color[c] = i;
        }
        reverse(S.begin(), S.end());
    }
    while (Q--) {
        int s, e;
        cin >> s >> e;
        cout << cache[0][s - 1] + cache[1][N - e] << '\n';
    }
    return 0;
}
```
