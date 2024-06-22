---
title: "[백준 1887] Cow Pizza"
date: 2021-07-11T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/1887](https://www.acmicpc.net/problem/1887)

<br>

## 알고리즘

bitmask

<br>

## 풀이

모든 subset중 특정 subset을 포함하는 여부를 확인하는 문제입니다.

$T$제한이 20이므로 비트마스크의 느낌이 팍팍납니다. 현재 subset이 어떠한 subset을 갖고 있는지는 &연산을 통해 바로 확인이 가능합니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

int T, N, cnt, num, ans;
int cst[52];

bool check(int sub) {
    rep(i, N) {
        if ((cst[i] & sub) == cst[i]) return false;
    }
    return true;
}

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> T >> N;
    rep(i, N) {
        cin >> cnt;
        while (cnt--) {
            cin >> num;
            num--;
            cst[i] |= (1 << num);
        }
    }

    for (int i = 0; i < (1 << T); ++i) {
        if (check(i)) ++ans;
    }
    cout << ans;

    return 0;
}
```
