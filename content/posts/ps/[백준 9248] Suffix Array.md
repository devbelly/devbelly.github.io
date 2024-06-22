---
title: "[백준 9248] Suffix Array"
date: 2020-07-17T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/9248](https://www.acmicpc.net/problem/9248)

<br>

## 알고리즘

Suffix array

<br>

## 풀이

문자열에서 가장 어려운 알고리즘인 Suffix Array입니다. 이 글에선 제가 해당 알고리즘을 공부할 때 어려웠던 부분 및 이해하기 위한 선행지식을 간단하게 정리하려고 합니다.

### Counting sort & Radix sort

Counting sort는 non-comparison sort으로 $O(N)$에 정렬을 해주는 알고리즘입니다. 주로 정렬할 데이터에 대한 사전 지식이 있을 때 사용합니다. 사전 지식이란 $k$이하의 정수를 배열을 정렬하라는 문제에서 $k$이하의 정수라는 조건을 의미합니다. 방법은 stable 한 방법과 unstable한 방법이 있는데, 정렬할 데이터가 단일 데이터(정수 1개, 문자 1개) 일 때는 unstable한 방식도 문제가 안되지만, 여러 개로 묶여있는 데이터(pair, tuple 등)에선 문제가 발생하게 되므로 우리는 stable한 Counting sort를 사용합니다. 방법은 다음과 같습니다.

1. 주어진 배열을 순회하며 $cnt$ 배열에 각각의 값이 몇 번 등장했는지 Count 합니다.
2. $cnt$ 배열의 누적합을 구합니다. 이 과정에서 $cnt[i]$의 의미는 $i$의 개수에서 $i$이하의 개수로 바뀌게 됩니다.
3. 주어진 배열을 뒤에서부터 순회하며 $cnt[arr[i]]$ 값을 확인합니다. 이때 $cnt$의 최종 의미는 정렬된 배열의 idx를 의미합니다. 해당 idx에 값을 배치한 후에는 참조한 $cnt$을 감소시켜줍니다.

Suffix Array를 처음 공부할 때, 가장 어려운 부분은 여러 배열들의 의미입니다. 각 배열의 의미는 다음과 같습니다.

<p align=center>
	$g[i]$ = $i$에서 시작하는 접미사의 그룹 번호, 그룹번호가 작을수록 사전순으로 빠르다는 뜻입니다.
</p>
<p align=center>
	$sfx[i]$ = Suffix들을 사전순으로 정렬 했을때, $i$번째 Suffix의 원래 인덱스 위치.
</p>

중요한 점은 $g[i]$와 $sfx[i]$가 g[sfx[i]]=i 관계라는 것입니다. 고등학교 때 배운 역함수와 비슷한 역배열 개념이죠. 나머지 부분은 코드를 본 후 설명드리겠습니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i,n) for(int i=0;i<n;++i)
#define REP(i,n) for(int i=1;i<=n;++i)
#define FAST cin.tie(NULL);cout.tie(NULL); ios::sync_with_stdio(false)
using namespace std;

vector<int> g, ng, sfx, cnt, lcp, ordered;

void getsfx(string & s) {
    int n = s.size();
    int mx = max(257, n + 1);

    g.resize(n + 1);
    ng.resize(n + 1);
    sfx.resize(n);
    ordered.resize(n);

    for (int i = 0;i < n;++i) g[i] = s[i];

    for (int t = 1;t < n;t <<= 1) {
        cnt.clear(); cnt.resize(mx);
        for (int i = 0;i < n;++i) cnt[g[min(i + t, n)]]++;
        for (int i = 1;i < mx;++i) cnt[i] += cnt[i - 1];
        for (int i = n - 1;i >= 0;--i) ordered[--cnt[g[min(i + t, n)]]] = i;

        cnt.clear(); cnt.resize(mx);
        for (int i = 0;i < n;++i) cnt[g[i]]++;
        for (int i = 1;i < mx;++i) cnt[i] += cnt[i - 1];
        for (int i = n - 1;i >= 0;--i) sfx[--cnt[g[ordered[i]]]] = ordered[i];

        auto cmp = [&](int i, int j) {
            if (g[i] == g[j]) return g[i + t] < g[j + t];
            return g[i] < g[j];
        };

        ng[sfx[0]] = 1;
        for (int i = 1;i < n;++i) {
            if (cmp(sfx[i - 1], sfx[i])) ng[sfx[i]] = ng[sfx[i - 1]] + 1;
            else ng[sfx[i]] = ng[sfx[i - 1]];
        }
        g = ng;
    }

    lcp.resize(n);
    for (int i = 0;i < n;++i)
        g[sfx[i]] = i;

    for (int i = 0, k = 0;i < n;++i, k = max(k - 1, 0)) {
        if (g[i] == n - 1) continue;
        for (int j = sfx[g[i] + 1];s[i + k] == s[j + k];++k);
        lcp[g[i]] = k;
    }
}


int main() {
    FAST;
    string input;
    cin >> input;
    getsfx(input);

    for (auto x : sfx) cout << x + 1 << ' ';
    cout << '\n';
    cout << 'x' << ' ';
    for (int i = 0;i < lcp.size() - 1;++i) cout << lcp[i] << ' ';

    return 0;
}
```

<br>

#### 1. 22번째 줄 g[min(i+t,n)]의 의미?

g[i]의 의미는 i에서 시작하는 suffix의 그룹번호라고 설명을 드렸는데 i+t를 하게 되면 당연히 인덱스 초과가 날 수 있습니다. 인덱스를 벗어나게 되면 g[i+t] 대신 g[n]을 취하게 되는데 이 값은 처음 g배열을 resize(n+1)을 했으므로 0이 됩니다. 그룹번호가 0이라는 것은? 비교우선순위에서 최상위를 차지하는 값이죠. 인덱스가 벗어날 때는 비교 길이보다 문자열이 짧을 때인데 비교우선순위가 높다는 것은 직관적이죠. 예를 들어 "an" 와"a*"를 비교할 때 (*는 공백) 뒤에 문자열의 순위가 더 빠르죠

#### 2. 24번째 줄 ordered의 의미?

위 코드는 (g[i],g[i+t]) 와 g[j],g[j+t])를 Radix sort하는 상황입니다. g[i+t]와 g[j+t]를 정렬한 후g[i] 와 g[j]를 정렬하게 되는데 임시로 정렬된 결과를 돕기 위함입니다. 일차적으로 정렬된 배열을 tempsfx라고 하겠습니다. Counting sort를 제대로 이해했다면 24번째 줄이 다음과 같이 된다는 것을 이해할 수 있습니다.

```c++
						tempsfx[--cnt[g[min(i+t,n)]]=g[i];
```

여기서 역배열을 양쪽 변에 사용한 것이 24번째 줄이라고 이해하시면 편합니다. h[g[i]]=i 라고 한다면

h[tempsfx[i]]=ordered[i] 이고 이를 통해 아래 29번째 줄에서 일차적으로 정렬된 tempsfx에 접근하기 위하여 h[tempsfx[i]]=ordered[i] 의 양변에 g를 씌워주게 되는 것이죠. 즉 tempsfx[i]=g[ordered[i]]입니다.

#### 3. getLCP

위 내용들을 통해 Suffix Array를 구하고, 이를 통해 LCP 배열을 $O(N)$에 구할 수 있습니다. LCP를 구할 때 가장 중요한 성질은 아래와 같습니다.

접미사 X,Y의 Commom Prefix길이를 z라고 할 때, 두 접미사 X,Y의 맨 앞글자를 하나씩 제거한 x,y 또한 접미사이며 이들의 commom Prefix의 길이는 최소 z-1 이다.

가장 긴 Suffix부터 가장 짧은 suffix순으로 LCP를 갱신하도록 합시다. 가장 긴 Suffix에 접근하기 위해 g[i] 배열을 갱신한 후, 차례대로 LCP를 구해나가면 됩니다.
