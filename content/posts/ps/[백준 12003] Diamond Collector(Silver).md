---
title: "[백준 12003] Diamond Collector(Sliver)"
date: 2021-03-16T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/12003](https://www.acmicpc.net/problem/12003)

<br>

## 알고리즘

Union Find, 오프라인 쿼리

<br>

## 풀이

두 개의 진열장에 다이아몬드를 진열하려 합니다. 같은 진열장 내에 있는 다이아몬드들은 그 크기의 차이가 $K$이하일 때만 진열이 가능합니다. 이러한 조건에서 진열할 수 있는 다이아몬드의 최대 갯수를 구하는 문제입니다.

$N$제한이 5만이므로 $O(NlogN)$에 해당하는 알고리즘을 떠올려야합니다. 주어지는 다이아몬드의 크기를 정렬한 후 앞에서부터 크기의 차이가 $K$이하가 되도록 골라봅시다. $i$번째 다이아몬드부터 $i+j$번째 다이아몬드를 하나의 진열장에 넣었다면 $i+j+1$번째 다이아몬드부터 마지막 다이아몬드까지 어느 연속된 부분을 나머지 하나의 진열장에 넣어야합니다. 만약 각 다이아몬드를 선택했을 때 몇개를 담을 수 있는지 미리 처리를 해놓고 세그먼트 트리의 구간 최댓값을 이용한다면 $O(logN)$으로 다른 진열장에 넣은 갯수의 최댓값을 구할 수 있습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
#define left (i << 1)
#define right (i << 1 | 1)
using namespace std;

const int MAXN = 5e4 + 5;

int N, K, ans;
int arr[MAXN], tree[MAXN * 2];

int query(int l, int r) {
    int ans = 0;
    l += N;
    r += N;
    for (; l <= r; l >>= 1, r >>= 1) {
        if (l & 1) {
            ans = max(ans, tree[l++]);
        }
        if (!(r & 1)) {
            ans = max(ans, tree[r--]);
        }
    }
    return ans;
}

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> K;
    rep(i, N) {
        cin >> arr[i];
    }
    sort(arr, arr + N);
    rep(i, N) {
        int cnt = upper_bound(arr, arr + N, arr[i] + K) - arr - i;
        tree[N + i] = cnt;
    }
    for (int i = N - 1; i >= 1; --i) {
        tree[i] = max(tree[left], tree[right]);
    }
    rep(i, N) {
        int cnt = tree[i + N];
        ans = max(ans, cnt + query(i + cnt, N));
    }
    cout << ans;
    return 0;
}
```
