---
title: "[백준 18320] Loan Repayment"
date: 2021-04-28T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/18320](https://www.acmicpc.net/problem/18320)

<br>

## 알고리즘

이분탐색

<br>

## 풀이

갚을 금액, 기한, 하루에 최소 갚을 금액이 주어졌을 때, 조건을 충족하는 $X$를 구하는 문제입니다.

문제를 풀기 위한 첫 번째 단계는 $X$를 결정해야하는 것입니다. 결정문제는 이분탐색으로 바꿀 수 있습니다. 이 문제가 어려운 이유는 $X$를 결정했을 때, 그대로 시뮬레이션하여 $K$일 이내에 돈을 갚을 수 있는지 계산하게 되면 시간초과가 발생하기 때문입니다.

이유는 $N$제한이 $10^{12}$이기 때문입니다. $X$를 결정하는 시간복잡도는 $O(logN)$에 해당하는데 그대로 시뮬레이션 하게 되면 $X$마다 선형시간이 요구되기 때문입니다.

한가지 할 수 있는 관찰 중 하나는 John이 갚아나가는 금액은 비오름차순의 형태를 띤다는 것입니다. 이를 이용해 하루에 갚아나가야 하는 금액이 $A$라면 남은 돈에서 $A, A, A$... 처럼 하나씩 빼지말고 계산을 통해 $A$를 몇개 빼야할지 계산하면 됩니다.

이를 계산하기 위해선 하루에 갚아야 하는 금액 $A$에서 $X$를 곱하면 $A$를 내야하는 경계선 금액(남은 돈)이 나오게 됩니다. 이 값을 $leftMost$라는 변수를 통해 나타냈습니다. 위 문단에서 '몇개'를 계산하는 것은 (rest - leftMost + givePerday) / givePerday; 통해 가능합니다. givePerday를 더하는 이유는 올림을 계산하기 위함입니다. 마지막으로 최적화를 위해 $solve()$함수 내에서 $day$가 $K$를 넘거나 남은 금액이 0 이하가 되면 while문을 종료하도록 했습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

#define int long long
int N, K, M;

bool solve(int m) {
    int rest = N;
    int day = 0;
    while (1) {
        int givePerday = rest / m;
        if (givePerday <= M) {
            int ret = day + (rest + M-1) / M;
            return ret<=K;
        }
        int leftMost = givePerday * m;
        int dayCnt = (rest - leftMost + givePerday) / givePerday;

        day += dayCnt;
        if (day > K) return false;
        int give = dayCnt * givePerday;
        rest -= give;
        if (rest <= 0) {
            break;
        }
    }

    return day <= K;
}

signed main() {
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> K >> M;

     int lo = 1;
     int hi = N;
     int best = -1;
     while (lo <= hi) {
         int mid = (lo + hi) / 2;
         if (solve(mid)) {
             best = mid;
             lo = mid + 1;
         } else {
             hi = mid - 1;
         }
     }
     cout << best;
    return 0;
}
```
