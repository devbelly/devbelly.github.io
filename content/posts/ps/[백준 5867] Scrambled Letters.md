---
title: "[백준 5867] Scrambled Letters"
date: 2021-09-14T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[www.acmicpc.net/problem/5867](www.acmicpc.net/problem/5867)

<br>

## 알고리즘

Sorting

<br>

## 풀이

뒤섞인 문장의 가능한 높은 등수와 낮은 등수를 찾는 문제입니다.

가장 높은 등수는 다른 문장들이 거꾸로 정렬되어있을 때, 자신은 정렬된 문장을 통해 순서를 찾는 것이고 가장 낮은 등수는 다른 문장들이 올바르게 정렬되어있을 때, 자신은 거꾸로 정렬된 문장을 통해 순서를 찾는 것입니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

typedef pair<int, int> pii;
const int MAXN = 50000;
int N;
vector<string> fi, se, arr;
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
        string s;
        cin >> s;
        arr.emplace_back(s);
        sort(s.begin(), s.end());
        fi.emplace_back(s);

        sort(s.rbegin(), s.rend());
        se.emplace_back(s);
    }
    sort(fi.begin(), fi.end());
    sort(se.begin(), se.end());
    rep(i, N) {
        string s = arr[i];
        sort(s.begin(), s.end());
        cout << lower_bound(se.begin(), se.end(), s) - se.begin() + 1 << ' ';
        sort(s.rbegin(), s.rend());
        cout << upper_bound(fi.begin(), fi.end(), s) - fi.begin() << '\n';
    }
    return 0;
}
```
