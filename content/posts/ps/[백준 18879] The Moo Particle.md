---
title: "[백준 18879] The Moo Particle"
date: 2020-11-01T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/18879](https://www.acmicpc.net/problem/18879)

<br>

## 알고리즘

sorting

<br>

## 풀이

2차원 공간에 여러 particle들이 주어졌을 때, 상호작용 후 최소한의 particle의 수를 묻고 있습니다. 상호작용의 조건은 $x1\leq x2,\,  y1\leq y2$ 을 만족해야합니다.

만약 두 좌표가 있을 때, 상호작용해서 없애야하는 좌표는 상대적으로 오른쪽 위에있는 좌표입니다. 왼쪽 아래 좌표는 후에 다른 좌표들을 더욱 없앨 수 있기 때문입니다. 주어진 좌표들을 정렬한 후, y좌표를 담을 벡터를 만듭니다. 이는 지금까지 살아있는 y좌표들의 그룹입니다.

좌표를 정렬했으므로 좌표벡터의 $i$번쨰 원소는 이전의 모든 좌표들과 상호작용할 첫번째 조건이 충족되었습니다($x1\leq x2$). y좌표만 모아놓은 벡터에서 upper_bound를 통해 상호작용할 수 있는 좌표들의 갯수를 파악하고 적절하게 처리해주면 됩니다. 시간복잡도는 $O(NlogN)$ 입니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

typedef pair<int, int> pii;
vector<pii> ord;
vector<int> Y;

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    int N;
    cin >> N;
    rep(i, N) {
        int x, y;
        cin >> x >> y;
        ord.emplace_back(x, y);
    }

    sort(ord.begin(), ord.end());

    rep(i, N) {
        auto [x, y] = ord[i];
        auto iter = upper_bound(Y.begin(), Y.end(), y);
        if (iter == Y.begin()) {
            Y.insert(iter, y);
        } else {
            Y.erase(Y.begin() + 1, iter);
        }
    }
    cout << Y.size();

    return 0;
}
```
