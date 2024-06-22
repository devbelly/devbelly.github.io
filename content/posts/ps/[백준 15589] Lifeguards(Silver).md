---
title: "[백준 15589] Lifeguards(Silver)"
date: 2021-02-05T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/15589](https://www.acmicpc.net/problem/15589)

<br>

## 알고리즘

sweeping

<br>

## 풀이

$N$명의 라이프가드가 있습니다. 한 명을 해고했을 때, 가장 긴 시간을 커버하도록 해야 합니다.

다른 사람과 근무시간이 겹치지 않는 시간을 고유 시간이라고 정의하겠습니다. 해고하지 말아야할 사람은 고유시간이 큰 사람들입니다. 자신의 가치가 높다는 뜻입니다. 반대로 해고해야할 사람은 고유시간이 가장 작은 사람입니다. 사람마다의 고유시간을 구할 수 있다면 [이 문제](https://www.acmicpc.net/problem/2170)에서 푼 것과 같이 $N$명의 총 근무시간에서 고유시간이 가장 짧은 사람을 해고하면 될 일입니다.

첫 번째 단계로 모든 사람의 총 근무시간을 구해봅시다.

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

typedef pair<int, int> pii;

int N, ans, cand = INT_MAX;
vector<pii> vt;

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
        int x, y;
        cin >> x >> y;
        vt.emplace_back(x, y);
    }
    sort(vt.begin(), vt.end());

    for (int i = 0, l = INT_MIN, r; i < N; ++i) {
        auto [s, e] = vt[i];
        l = max(s, l);
        ans += max(e - l, 0);
        l = max(e, l);
    }
    cout << ans;
    return 0;
}
```

32번째 줄 변수 $l$은 계산을 완료한 좌표를 의미합니다. 루프를 돌아 30번째 줄에서는 세 가지의 상태가 있습니다.

1.  $l<s$

2.  $s<l<e$

3.  $e<l$

첫 번째 경우에서는 계산을 시작할 부분으로 s를 취해주면 되고 두 번째 경우에는 l, 세 번째 경우는 현재 선분이 완전히 포함되는 상태이므로 $ans$에 답을 더해선 안됩니다. 위 두 가지 상태는 $max(s,l)$, 세 번째 상태는 $max(e-l,0)$을 통해 해결 가능합니다.

두 번째 단계에 해당하는 고유 시간을 구해보도록 합시다. 자신의 고유 시간은 당연하게도 다른 사람에 의해 결정이 됩니다. 상대적으로 다음과 같은 세 가지의 상태가 가능합니다. 좌표를 입력받고 x좌표를 기준으로 정렬했다고 가정하겠습니다.

<br>

![사진1](https://user-images.githubusercontent.com/67682840/147637416-4c184c86-2590-47f0-91fa-938b04a1dc68.png)

첫 번째 상태입니다. 자신의 다음 사람의 시작 지점에서 계산을 시작해야 하는 부분을 빼면 됩니다.

<br>

![tkwls2](https://user-images.githubusercontent.com/67682840/147637442-f4a7c69e-7081-47be-8a30-9beacb95aa8f.png)

두 번째 상태입니다. 다른 사람과 겹치지 않으므로 자신의 끝 지점에서 계산을 시작하는 부분을 빼면 됩니다. 위 두 가지 상태는 $min(e,next s)$에서 계산을 시작해야 하는 부분$l$을 빼면 자신의 고유 시간을 구할 수 있습니다.

<br>

![tkwls3](https://user-images.githubusercontent.com/67682840/147637456-d062f3a0-feb5-4782-9748-ee398d734b92.png)

세 번째 상태입니다. $min(e,next s)$를 구한 후 계산을 시작하는 지점을 빼면 고유 시간을 구하는 데에 무리가 있습니다. 왜냐하면 앞에 부분은 구할 수 있어도 뒤에 있는 자신의 고유시간을 구하지 못하기 때문입니다.

하지만 세 번째 상태 또한 동일하게 처리할 것입니다. 이 경우는 다음 사람(완전히 포함되는 사람)이 잘리는 것이 확정되기 때문입니다. 다음 사람의 고유 시간을 계산할 때 0이 되므로 자신의 고유시간을 잘못 구해도 상관이 없습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < n; ++i)
#define REP(i, n) for (int i = 1; i <= n; ++i)
using namespace std;

typedef pair<int, int> pii;

int N, ans, cand = INT_MAX;
vector<pii> vt;

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
        int x, y;
        cin >> x >> y;
        vt.emplace_back(x, y);
    }
    sort(vt.begin(), vt.end());

    for (int i = 0, l = INT_MIN, r; i < N; ++i) {
        auto [s, e] = vt[i];
        l = max(s, l);
        ans += max(e - l, 0);
        r = i == N - 1 ? e : min(vt[i + 1].first, e);
        cand = min(cand, max(r - l, 0));
        l = max(e, l);
    }
    cout << ans - cand;
    return 0;
}
```
