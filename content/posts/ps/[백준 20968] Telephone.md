---
title: "[백준 20968] Telephone"
date: 2021-05-11T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/20968](https://www.acmicpc.net/problem/20968)

<br>

## 알고리즘

queue

<br>

## 풀이

소의 품종마다 서로 연락할 수 있는 관계가 주어졌을 때, 첫 번째 소가 맨 마지막 소에게 메시지를 전달하기 위한 최솟값을 구하는 문제입니다.

일반적으로 PQ를 이용한 다익스트라의 시간복잡도는 $O(ElogV)$입니다. 문제에서 $N$제한이 50000이므로 무작정 다익스트라를 사용하게 되면 시간 초과를 받게 됩니다.

![img](https://blog.kakaocdn.net/dn/ccXZ10/btq4HGXm0Ed/1QCHfgBckPHHXSqykOKlG1/img.png)

빨간색 원이 현재 PQ에서 꺼낸 소라고 하겠습니다. 녹색은 빨간색 품종이 연락할 수 있는 품종을 의미합니다. 만일 위와 같이 녹색품종들이 모두 빨간색에서 메시지를 받을 수 있다면 굳이 첫 번째를 제외한 2, 3, 4번째들을 검사할 필요는 없습니다. 즉 현재 소에서 연락 가능한 품종들마다 가장 가까이 있는 소들만 PQ에 넣어서 다익스트라를 사용하면 됩니다.

하지만 이 풀이또한 특이 케이스에서 문제가 발생합니다. "굳이" 가장 가까이 있는 소가 아닌 뒤에 있는 소를 검사해야 하는 경우가 발생합니다. 만일 위 사진에서 마지막 소가 녹색 품종이고 녹색 품종끼리는 서로 메시지를 주고받지 않는다면 위 풀이는 답을 찾지 못하게 됩니다. 즉 매번 현재 소가 마지막에 있는 소의 품종과 메시지를 주고받을 수 있는지를 추가적으로 확인해주어야 합니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

const int MAXN = 5e4 + 5;
const int MAXK = 50 + 5;
typedef pair<int, int> pii;

int N, K;

int cow[MAXN], dist[MAXN];
vector<int> breedPosition[MAXK];
bool canReach[MAXK][MAXK];

int main() {
#ifndef ONLINE_JUDGE
    freopen("13.in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> K;
    rep(i, N) {
        cin >> cow[i];
        cow[i]--;
        breedPosition[cow[i]].emplace_back(i);
    }
    rep(i, K) rep(j, K) {
        char x;
        cin >> x;
        canReach[i][j] = (x == '1');
    }

    memset(dist, 0x3f, sizeof(dist));
    priority_queue<pii, vector<pii>, greater<pii>> pq;
    dist[0] = 0;
    pq.emplace(0, 0);
    while (!pq.empty()) {
        auto [cd, here] = pq.top();
        pq.pop();

        if (canReach[cow[here]][cow[N - 1]]) {
            dist[N - 1] = min(dist[N - 1], N - 1 - here + dist[here]);
        }
        if (dist[here] < cd) {
            continue;
        }

        rep(thereB, K) {
            if (canReach[cow[here]][thereB]) {
                int closeRight = upper_bound(breedPosition[thereB].begin(), breedPosition[thereB].end(), here)
                                 - breedPosition[thereB].begin();

                if (closeRight != breedPosition[thereB].size()) {
                    int idx = breedPosition[thereB][closeRight];

                    if (dist[idx] > dist[here] + idx - here) {
                        dist[idx] = dist[here] + idx - here;
                        pq.emplace(dist[idx], idx);
                    }
                }
                if (here == 0)
                    continue;
                int closeLeft = lower_bound(breedPosition[thereB].begin(), breedPosition[thereB].end(), here)
                                - breedPosition[thereB].begin();
                if (closeLeft != 0) {
                    closeLeft--;
                    int idx = breedPosition[thereB][closeLeft];
                    if (idx > here)
                        continue;
                    if (dist[idx] > dist[here] + here - idx) {
                        dist[idx] = dist[here] + here - idx;
                        pq.emplace(dist[idx], idx);
                    }
                }
            }
        }
    }
    cout << (dist[N - 1] == 0x3f3f3f3f ? -1 : dist[N - 1]);
    return 0;
}
```
