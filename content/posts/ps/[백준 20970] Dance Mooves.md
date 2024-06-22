---
title: "[백준 20970] Dance Mooves"
date: 2021-04-30T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/20970](https://www.acmicpc.net/problem/20970)

<br>

## 알고리즘

Graph

<br>

## 풀이

$N$마리의 소가 $K$번 자리를 바꾸는 것을 계속 반복할 때, 각 소가 차지했던 위치의 수를 구하는 문제입니다.

$K$번 자리를 바꾸는 것을 한 사이클이라고 하겠습니다. 하나의 사이클을 반복하면 아래와 같은 결과를 얻을 수 있습니다.

![img](https://blog.kakaocdn.net/dn/c1g9Js/btq4YJ7IRDM/Kg4Ejy0DI8kUNuEsu4N9gK/img.png)

1번째 있는 소는 3, 3, 2 번째를 들른 후 한 사이클이 끝나면 4번째에 있는 곳에 위치하게 됩니다. 아래 2,3,4,5 번째 소들도 마찬가지로 그림을 해석하면 됩니다. 이 사이클을 계속 반복했을 때 원래 자리로 돌아오도록 시뮬레이션하면 답을 구할 수 있습니다. 아래 사진은 1번째소를 시뮬레이션한 결과입니다.

![img](https://blog.kakaocdn.net/dn/b4zPAA/btq4WKGRmVK/I8ycrD0Y2puk3oseeju19K/img.png)

파란색 원들이 각각 시작점과 끝점입니다. 그 사이에 들렀던 노란색원들의 중복값을 제거하면 1번소에 대한 답은 {1,2,3,4} 임을 알 수 있습니다.

하지만 모든 소들에 대해 이렇게 시뮬레이션 하면 시간초과를 받게 됩니다. 하나의 소마다 최대로 탐색해야하는 노란원은 $2*K$개인데(위 그림은 편의를 위해 중복된 위치를 표시하였습니다), 이를 $N$마리의 소에 대해 반복하게 되면 시간초과를 받게 됩니다.

하지만 우리는 1번째소의 시뮬레이션을 통해서 얻을 수 있는 관찰은 각 사이클이 끝난 지점에 위치하며 시뮬레이션에 포함되는 2번 4번 소 또한 또한 1번소와 동일한 답을 갖는다는 것입니다. 같은 사이클내에 있기 때문이죠. 즉 이들은 시뮬레이션 할 필요가 없음을 확인하며 답을 구하면 됩니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 1e5 + 5;

int N, K;
int arr[MAXN];
int ans[MAXN];
vector<int> AgotoB[MAXN];

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> K;
    iota(arr + 1, arr + 1 + N, 1);
    rep(i, K) {
        int u, v;
        cin >> u >> v;
        AgotoB[arr[u]].emplace_back(v);
        AgotoB[arr[v]].emplace_back(u);
        swap(arr[u], arr[v]);
    }
    REP(i, N) {
        if (AgotoB[i].empty()) {
            AgotoB[i].emplace_back(i);
        }
    }
    memset(arr, 0, sizeof(arr));
    REP(i, N) {
        set<int> st;
        int here = i;
        int prv = 0;
        if (arr[here]) {
            ans[here] = ans[arr[here]];
            continue;
        }
        st.emplace(here);
        while (1) {
            arr[prv] = i;
            if (arr[here]) {
                break;
            }
            st.insert(AgotoB[here].begin(), AgotoB[here].end());

            prv = here;
            here = AgotoB[here].back();
        }
        ans[i] = st.size();
    }
    REP(i, N) {
        cout << ans[i] << ' ';
    }
    return 0;
}
```
