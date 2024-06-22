---
title: "[백준 18878] Cereal"
date: 2020-10-26T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/18878](https://www.acmicpc.net/problem/18878)

<br>

## 알고리즘

sorting

<br>

## 풀이

$i$번째 소부터 자신의 기호식품의 순위에 따라 가져 갈 때, 각 $i$번째 소마다 몇 개의 시리얼을 나누어 줄 수 있는지 묻는 문제입니다.

문제를 나이브하게 접근하면 처음부터 기호식품을 확인하며 나누어 주면 됩니다. 즉 1번째 소 뒤에 있는 $N-1$마리의 소들에 대해서 나누어주고, 2번째 소 뒤에 있는 $N-2$마리의 소들에 대해 나누어주는 식입니다. 이와 같이 시뮬레이션하면 요구하는 시간 복잡도는 $O(N^2)$ 이므로 시간 초과를 받게 됩니다.

뒤에 서있는 소부터 계산해보도록 합시다. 만약 $k$번째 소 이후에 몇마리가 가져갈 수 있는지를 다 계산을 하였다면

$k-1$번째 소는 이전의 상황과는 관계없이 무조건 자신이 가장 좋아하는 시리얼을 가져올 수 있습니다. 이 상황에 대해 피해를 입는 소는 $k-1$소가 가장 좋아하는 시리얼을 골랐었던 소입니다.

$cereal$ 배열을 통해 어떤 소가 어떤 시리얼을 가지고 있는지를 저장하고 만약 시리얼을 빼앗긴 소 $c$가 자신의 첫 번째 기호식품을 빼앗긴 것이라면 위와 같은 과정을 반복하고 자신의 두 번째 기호식품을 빼앗긴 것이라면 그 소는 영원히 먹을 식품이 사라지게 됩니다.

위 과정에서 각 단계마다 다시 시리얼을 배분하는 것의 시간복잡도가 $O(N)$이라고 생각하면 안 됩니다. 최대 깊이는 $O(N)$ 이 될 수 있지만, 최종 시간 복잡도는 $O(N)$ 이 됩니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i,n) for(int i=0;i<n;++i)
#define REP(i,n) for(int i=1;i<=n;++i)
#define FAST cin.tie(NULL);cout.tie(NULL); ios::sync_with_stdio(false)
using namespace std;

typedef pair<int, int> pii;

const int MAXN = 1e5 + 1;
int N, M;

int ans[MAXN], f[MAXN], s[MAXN], cereal[MAXN];
int main() {
    FAST;
    cin >> N >> M;
    rep(i, N) {
        int F, S;
        cin >> F >> S;
        f[i] = F;
        s[i] = S;
    }
    int cnt = 0;
    for (int i = N - 1;i >= 0;--i) {
        int j = i;
        int best = f[j];
        while (1) {
            if (cereal[best] == 0) {
                ++cnt;
                cereal[best] = j;
                break;
            }
            else if (cereal[best] < j) {
                break;
            }
            else {
                int OUT = cereal[best];
                cereal[best] = j;
                if (s[OUT] == best) break;
                j = OUT;
                best = s[j];
            }
        }
        ans[i] = cnt;
    }
    rep(i, N) {
        cout << ans[i] << '\n';
    }
    return 0;
}
```
