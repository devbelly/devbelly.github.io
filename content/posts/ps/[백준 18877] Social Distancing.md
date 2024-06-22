---
title: "[백준 18877] Social Distancing"
date: 2020-10-23T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/18877](https://www.acmicpc.net/problem/18877)

<br>

## 알고리즘

이분 탐색

<br>

## 풀이

COWVID-19의 영향으로 사회적 거리두기를 실천하는 소들의 수가 주어집니다. 목초지의 범위가 주어지고 해당 범위안에 소를 배치하되, 가장 가까운 소의 거리가 최대가 되도록 하는 문제입니다.

소들을 임의의 거리 $d$로 배치할 수 있다면, $d$이하의 거리로 소를 배치할 수 있음이 자명합니다. 이분탐색으로 범위를 좁혀나갈 수 있다는 뜻과 동치입니다.

$solve$함수는 거리$d$로 $N$마리의 소를 배치할 수 있는가를 리턴하는 함수입니다. $cur$은 가장 최근에 소를 배치한 지역을 의미합니다. 맨 처음에는 당연히 소를 배치하지 않았으므로 가장 작은 수로 초기화를 합니다. 다음의 $cur$이 될 수 있는 후보들은 $cur+d$ 또는 범위의 시작인 $f$ 입니다.

$solve$함수는 $cnt$값이 $N$이상이 되면 종료하므로 하나의 $solve$함수는 $O(N+M)$안에 검사가 가능합니다.

전체 시간복잡도는 $O((N+M)*R)$ 입니다. (단, R은 전체 범위를 나타냄 )

## 코드

```c++
#include <bits/stdc++.h>
#define rep(i,n) for(int i=0;i<n;++i)
#define REP(i,n) for(int i=1;i<=n;++i)
#define FAST cin.tie(NULL);cout.tie(NULL); ios::sync_with_stdio(false)
using namespace std;

typedef long long ll;
typedef pair<ll, ll> pll;

int N, M;
vector<pll> vt;

bool solve(ll d) {
    int cur = INT_MIN;
    int cnt = 0;
    rep(i, M) {
        auto [f, s] = vt[i];
        while (cur <= s - d) {
            cur = max(cur + d, f);
            cnt += 1;
        }
    }
    return cnt >= N;
}

int main() {
    FAST;
    cin >> N >> M;
    vt.resize(M);
    rep(i, M) {
        auto& [f, s] = vt[i];
        cin >> f >> s;
    }
    sort(vt.begin(), vt.end());
    ll lo = 1;
    ll hi = 1e18;
    ll best = 0;
    while (lo <= hi) {
        ll mid = (lo + hi) / 2;
        if (solve(mid)) {
            best = mid;
            lo = mid + 1;
        }
        else hi = mid - 1;
    }
    cout << best;

    return 0;
}
```
