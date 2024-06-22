---
title: "[백준 17038] The Great Revegetation"
date: 2020-12-08T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/17038](https://www.acmicpc.net/problem/17038)

<br>

## 알고리즘

2-SAT

<br>

## 풀이

소들마다 두 개의 목초지에서 식사를 하고, 소들의 식성은 두가지로 나뉩니다. 한 식성은 두 목초지의 풀이 같아야만 하고, 다른 식성은 두 목초지의 풀이 달라야만 식사를 합니다. 이 때, 가능한 목초지의 경우의 수를 구하는 문제입니다.

두 종류의 풀이 주어지는 점, 이를 동시에 만족하도록 하는 점에서 2-SAT임을 파악하여 풀도록 합시다. 한 종류의 풀을 True, 다른 종류의 풀을 False로 설정하고 SCC를 사용합니다. 입력에서 10개의 목초지가 주어지고 5마리의 소의 입력이 다음과 같다고 생각해봅시다.

<p align=center>
	S 1 2 <br>
	S 3 4  <br>
	S 5 6  <br>
	S 7 8  <br>
	S 9 10

</p>

이 경우는 32가지가 정답인 것을 알 수 있습니다. 1 2 목초지를 어떤 것을 선택하든 나머지 목초지들과는 독립적이기 때문입니다. SCC을 사용한 이후 SC/2 만큼 2의 제곱을 한다면 정답임을 관찰하면 풀 수 있습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

#define T(x) (x << 1)
#define F(x) (x << 1 | 1)

int N, M, VC, SC;

vector<vector<int>> adj;
vector<int> discovered, sccid;
stack<int> st;

int scc(int here) {
    int ret = discovered[here] = VC++;
    st.emplace(here);

    for (auto there : adj[here]) {
        if (discovered[there] == -1)
            ret = min(ret, scc(there));
        else if (sccid[there] == -1)
            ret = min(ret, discovered[there]);
    }

    if (ret == discovered[here]) {
        while (1) {
            int t = st.top();
            st.pop();
            sccid[t] = SC;
            if (t == here)
                break;
        }
        SC++;
    }
    return ret;
}

void OR(int a, int b) {
    adj[a ^ 1].emplace_back(b);
    adj[b ^ 1].emplace_back(a);
}

void AOA(int a, int b, int c, int d) {
    OR(a, c);
    OR(a, d);
    OR(b, c);
    OR(b, d);
}

int main() {
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> N >> M;
    adj.resize(N << 1);
    rep(i, M) {
        char w;
        int a, b;
        cin >> w >> a >> b;
        --a, --b;
        if (w == 'S') {
            AOA(T(a), T(b), F(a), F(b));
        } else {
            AOA(T(a), F(b), F(a), T(b));
        }
    }
    discovered = sccid = vector<int>(N << 1, -1);
    rep(i, N << 1) if (discovered[i] == -1) scc(i);
    bool flag = true;
    rep(i, N) {
        if (sccid[T(i)] == sccid[F(i)]) {
            flag = false;
            break;
        }
    }

    if (flag) {
        cout << 1;
        rep(i, SC / 2) {
            cout << 0;
        }
    } else
        cout << 0;
    return 0;
}
```
