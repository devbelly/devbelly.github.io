---
title: "[백준 15590] Rental Service"
date: 2021-01-26T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/15590](https://www.acmicpc.net/problem/15590)

<br>

## 알고리즘

greedy

<br>

## 풀이

$N$마리의 소, $M$명의 우유 구매자, $R$명의 대여 희망자가 주어질 때, 얻을 수 있는 최대 수익을 구하는 문제입니다.

소를 우유 생산량에 따라 정렬해놨을 때, $i$번 소까지는 대여를 하고, 그 이후의 소들은 젖을 짜 우유를 판매하는 것이 최적이므로 우리는 어느 소까지만 대여를 해줄지 구해야합니다.

일단 모든 소를 대여해준 후 가장 가치가 높은 소(우유 생산량이 가장 많은 소)부터 우유를 짜서 가장 비싸게 사는 사람에게 판매합니다. 대여 희망자에 대한 누적합을 구해놓았다면 $i$번째 사람까지 대여를 했을 때 벌 수 있는 돈을 $O(1)$에 구할 수 있습니다. 이때 정답이 더 크게 갱신이 된다면 계속 진행을 하고 그렇지 않다면 break를 통해 즉시 종료해주면 됩니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 1e5 + 5;
typedef pair<int, int> pii;
typedef long long ll;

#define fi first
#define se second

int N, M, R;
int cow[MAXN], rent[MAXN];

pii buy[MAXN];
ll psum[MAXN];

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> M >> R;
    REP(i, N) cin >> cow[i];
    REP(i, M) cin >> buy[i].fi >> buy[i].se;
    REP(i, R) cin >> rent[i];

    sort(cow + 1, cow + N + 1, [](int a, int b) { return a > b; });
    sort(buy + 1, buy + M + 1, [](pii a, pii b) { return a.se > b.se; });
    sort(rent + 1, rent + R + 1, [](int a, int b) { return a > b; });
    REP(i, R) psum[i] = psum[i - 1] + rent[i];

    ll ans = psum[min(R, N)], ps = 0;
    int cur = 1;
    REP(i, N) {
        int MX = cow[i];
        while (cur <= M && buy[cur].fi <= MX) {
            ps += (ll)buy[cur].fi * buy[cur].se;
            MX -= buy[cur++].fi;
        }
        ps += (ll)MX * buy[cur].se;
        buy[cur].fi -= MX;
        ll cand = psum[min(R, N - i)] + ps;
        if (cand > ans)
            ans = cand;
        else
            break;
    }
    cout << ans;
    return 0;
}
```
