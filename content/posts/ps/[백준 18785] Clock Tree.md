---
title: "[백준 18785] Clock Tree"
date: 2021-05-16T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/18785](https://www.acmicpc.net/problem/18785)

<br>

## 알고리즘

DFS

<br>

## 풀이

Bessie가 트리위에 주어진 맵을 이동해가며 시계를 조정할 때, 모든 시계가 가르키는 시간이 12시가 되도록 할수 있는 방의 시작갯수를 구하는 문제입니다.

임의의 출발지에서 모든 노드가 12가 될 수 있는지를 확인하는 것이 목표입니다. $N$제한이 2500이므로 $O(N^2)$에 근접하는 알고리즘을 떠올려봅시다.

첫 번째로는 리프노드의 시간을 12시로 맞추기 위해서는 무조건 리프노드와 리프노드의 부모를 계속해서 왔다갔다 해야지만 리프노드를 원하는 숫자 12로 맞출 수 있습니다. 12로 설정된 리프노드는 우리의 관심사 밖이므로 제거되고 리프노드의 부모였던 노드는 새로운 리프노드가 됩니다.

위 시뮬레이션을 계속 진행하면 결국 출발점을 제외한 모든 노드를 제거할 수 있습니다. 다시 설명하자면 노드를 제거했다는 것은 값이 12로 설정되었다는 의미입니다. 출발노드에 적힌 숫자는 0과 1 일 때만 우리가 원하는 출발노드임이 될 수 있습니다. 0이 적혀있다면 주변의 아무 12로된 노드와 왔다갔다한 후 다시 시작노드로 돌아오면 되고, 1이 적혀있다면 역시 주변의 아무 12로된 노드와 왔다갔다한 후, 주변노드에서 시뮬레이션을 종료하면 되기 때문입니다.

노드마다 위 DFS를 수행하면 우리는 $O(N^2)$에 해당하는 알고리즘을 통해 문제가 해결가능합니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 2500 + 5;

int N, ans;
int arr[MAXN], dfsn[MAXN];
vector<int> adj[MAXN];

void dfs(int here, int p) {
    for (auto there : adj[here]) {
        if (there ^ p) {
            dfsn[there]++;
            dfs(there, here);
            dfsn[here] += (12 - dfsn[there]);
            dfsn[here] %= 12;
        }
    }
    dfsn[p]++;
}

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N;
    REP(i, N) {
        cin >> arr[i];
    }
    rep(i, N - 1) {
        int u, v;
        cin >> u >> v;
        adj[u].emplace_back(v);
        adj[v].emplace_back(u);
    }
    REP(i, N) {
        copy(arr + 1, arr + 1 + N, dfsn + 1);
        dfs(i, 0);
        ans += (dfsn[i] % 12 == 0 || dfsn[i] % 12 == 1);
    }
    cout << ans;
    return 0;
}
```
