---
title: "[백준 5872] Clumsy Cows"
date: 2021-07-14T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[www.acmicpc.net/problem/5872](www.acmicpc.net/problem/5872)

<br>

## 알고리즘

Greedy

<br>

## 풀이

올바른 괄호문자열을 만들기 위해 몇 개의 문자를 추가해야하는지에 대한 문제와 거의 비슷합니다.

괄호 하나를 뒤집는 것은 두 개의 문자를 추가하는 것과 일치합니다. 왼쪽부터 문자열을 검사하며 만일 닫는 괄호가 더 많아지는 순간에는 바로 해당 괄호를 여는 괄호로 바꿉니다. 문자열 끝까지 검사한 후 괄호쌍이 맞지 않다면 $sum$만큼의 닫는 괄호를 추가해주면 해결가능합니다. 이는 $sum/2$ 횟수만큼 괄호를 뒤집는 것과 일치합니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

string s;
int sum, ans;
int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> s;
    int len = s.length();
    rep(i, len) {
        char x = s[i];
        if (x == '(') {
            sum += 1;
        } else {
            sum -= 1;
        }
        if (sum < 0) {
            ans += 1;
            sum = 1;
        }
    }
    cout << ans + sum / 2;
    return 0;
}
```
