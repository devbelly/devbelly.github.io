---
title: "[백준 20649] Stuck in a Rut"
date: 2021-05-14T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/20649](https://www.acmicpc.net/problem/20649)

<br>

## 알고리즘

DP

<br>

## 풀이

x, y 좌표가 모두 다른 소들의 좌표가 주어졌을 때, 각 소들의 잘못 점수(?)를 구하는 문제입니다.

문제를 관찰하면 동쪽으로 이동하는 소들 $A$는 북쪽으로 이동하는 소들 $B$중에서 $A$보다 오른쪽에 있으면서 아래 있는 소들에게 막히는지 안 막히는지 확인해야 하고 반대로 $B$는 $B$보다 왼쪽에 있으면서 위에 있는 소들에게 막히는지 안 막히는지 확인해야 합니다.

그래프가 DAG의 형태를 띠는 것에서 DP를 착안합시다. 이차원 포문을 통해 모든 소들의 쌍을 검사하며 각 소들의 관계를 따져보도록 합시다. 하지만 소들을 정렬하지 않은 채 무작정 이차원 포문을 통해 각 소들의 관계를 따지게 되면 현재 소가 잘못 점수가 완벽히 갱신되지 않았음에도 자신의 잘못된 정보를 자신을 가로막는 소$b$에게 업데이트하는 경우가 발생할 수 있습니다.

만일 $A$를 y좌표를 기준으로 정렬하고 $B$를 x좌표를 기준으로 정렬한다면 이러한 문제점을 해결할 수 있습니다. $a$(동쪽으로 향하는 임의의 소)가 자신의 잘못 점수를 계산할 때는 다음과 같은 두 가지 요소에 의해 값이 증가합니다.

- 북쪽으로 향하는 소
- 북쪽으로 향하는 소가 가로막는 동쪽으로 향하는 소

위 2번에 해당하는 간단을 사고를 통해 알 수 있듯이, 2에 해당하는 소들은 $a$보다 y값이 작은 소들입니다. 즉 $a$를 계산하는 시점에서는 위 2번에 해당하는 동쪽으로 향하는 소들의 잘못 점수는 모두 계산이 된 상황입니다. $b$를 계산할 때도 이러한 논리는 마찬가지로 적용됩니다.

즉 y값이 작은 동쪽으로 향하는 소, x값이 작은 북쪽으로 향하는 소들부터 짝지어 나가며 서로가 막고 막히는지 계산하면 답을 구할 수 있습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 1e4 + 5;

struct cow {
    int x, y, idx;
};

vector<cow> nc, ec;

int N, cache[MAXN], stopped[MAXN];

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
        char d;
        cow c;
        cin >> d >> c.x >> c.y;
        c.idx = i;
        if (d == 'E') {
            ec.emplace_back(c);
        } else {
            nc.emplace_back(c);
        }
    }
    sort(ec.begin(), ec.end(), [](cow a, cow b) { return a.y < b.y; });
    sort(nc.begin(), nc.end(), [](cow a, cow b) { return a.x < b.x; });

    for (auto a : ec) {
        for (auto b : nc) {
            if (!stopped[a.idx] && !stopped[b.idx] && a.x < b.x && a.y > b.y) {
                cow c = a;
                cow d = b;
                if (c.x + c.y == d.x + d.y)
                    continue;
                if (c.x + c.y > d.x + d.y) {
                    swap(c, d);
                }
                stopped[c.idx] = true;
                cache[d.idx] += 1 + cache[c.idx];
            }
        }
    }
    rep(i, N) {
        cout << cache[i] << '\n';
    }
    return 0;
}
```
