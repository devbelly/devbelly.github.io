---
title: "[백준 6206] Milk Patterns"
date: 2020-07-16T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/6206](https://www.acmicpc.net/problem/6206)

<br>

## 알고리즘

라빈 카프

<br>

## 풀이

[https://www.acmicpc.net/problem/3033](https://www.acmicpc.net/problem/3033) 3033번 문제에선 두 번 이상 등장하는 패턴 중, 가장 긴 문자열의 길이를 물었지만, 이 문제는 두 번이 아닌 $K$번 등장하는 패턴을 묻고 있습니다.

등장 횟수를 카운트하기 위하여 $set$ 대신 $map$을 사용한 것을 제외하곤 3033번 문제와 동일하게 풀 수 있습니다.

각 우유 샘플이 100만 이하의 정수를 가질 수 있으므로, $BASE$는 100만 이상의 값을 주도록 합시다.

### 코드

```c++
#include <bits/stdc++.h>
#include <unordered_set>
#define rep(i,n) for(int i=0;i<n;++i)
#define REP(i,n) for(int i=1;i<=n;++i)
#define FAST cin.tie(NULL);cout.tie(NULL); ios::sync_with_stdio(false)
using namespace std;

typedef long long ll;

const ll MOD[2] = { 1000000000 + 7,1000000000 + 9 };
const ll BASE = 1000001;

int arr[20000];
ll hs[2], b[2];
int N, K;

unordered_map<ll, int> um;

bool ok(int m) {
    um.clear();
    rep(j, 2) hs[j] = arr[0], b[j] = 1;
    for (int i = 1;i < m;++i) rep(j, 2) {
        hs[j] = (hs[j] * BASE + arr[i]) % MOD[j];
        b[j] = b[j] * BASE % MOD[j];
    }
    um[hs[0] << 32 | hs[1]] = 1;
    for (int i = m;i < N;++i) {
        rep(j, 2) {
            hs[j] = (hs[j] - arr[i - m] * b[j] % MOD[j] + MOD[j]) % MOD[j];
            hs[j] = (hs[j] * BASE + arr[i]) % MOD[j];
        }
        if (++um[hs[0] << 32 | hs[1]] >= K) return true;
    }
    return false;
}

int main() {
    FAST;
    cin >> N >> K;
    rep(i, N) cin >> arr[i];

    int lo = 0;
    int hi = N;
    int best;
    while (lo <= hi) {
        int mid = (lo + hi) >> 1;
        if (ok(mid)) {
            best = mid;
            lo = mid + 1;
        }
        else hi = mid - 1;
    }
    cout << best;
    return 0;
}
```
