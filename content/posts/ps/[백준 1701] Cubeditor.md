---
title: "[백준 1701] Cubeditor"
date: 2020-07-05T12:27:59+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/1701](https://www.acmicpc.net/problem/1701)

<br>

## 알고리즘

KMP

<br>

## 풀이

실패 함수의 의미를 알고 있어야 풀 수 있는 문제였습니다. $fail[x]=y$는 문자열 $S$의 처음 $x$+1 글자에서 일치하는 접두사/접미사 중 최대 길이 $y$를 의미합니다.

실패 함수를 기록할 때, 위 정의에 따라 두 번 이상 등장하는 부분 문자열(접두사, 접미사)이라는 조건 만족하게 됩니다. 그리고 원래 문자열의 모든 접두사만 고려하는 실패 함수는 중간에서 등장하는 부분 문자열을 고려하지 못하므로 KMP를 반복하며 원래 문자열의 맨 앞을 하나씩 제거하는 방식을 사용합니다.

원래의 실패함수내에서 $j$의 의미는 현재까지 매칭한 개수를 의미합니다. 그러나 원래 문자열의 맨 앞을 제거하기 위해 사용한 $k$로 인하여 의미가 변질되므로 $ans$에 기록할 때 $j−k$를 통하여 원래 의미를 갖도록 합시다. KMP를 문자열의 길이만큼 반복하므로 시간복잡도는 $O(N^2)$ 입니다.

### 코드

```c++
#include <bits/stdc++.h>
#define rep(i,n) for(int i=0;i<n;++i)
#define REP(i,n) for(int i=1;i<=n;++i)
#define FAST cin.tie(NULL);cout.tie(NULL); ios::sync_with_stdio(false)
using namespace std;

int len,ans;
int fail[5001];
string S;

int main() {
    FAST;
    cin >> S;
    len = S.size();

    for (int k = 0;k < len;++k) {
        rep(l, len) fail[l] = k;
        for (int i = 1+k, j = k;i < len;++i) {
            while (j > k && S[i] != S[j])
                j = fail[j - 1];
            if (S[i] == S[j]) {
                fail[i] = ++j;
                ans = max(ans, j - k);
            }
        }
    }
    cout << ans;
    return 0;
}
```
