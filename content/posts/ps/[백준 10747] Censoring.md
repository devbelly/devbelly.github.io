---
title: "[백준 10747] Censoring"
date: 2020-07-28T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/10747](https://www.acmicpc.net/problem/10747)

<br>

## 알고리즘

KMP

<br>

## 풀이

KMP를 통해서 제거할 문자열의 위치를 찾아내는 접근까지는 쉽습니다. 검열할 문자를 없앤 후, 우리는 이전의 상태로 돌아가야 하는데, 이를 위해 $matchCnt$배열을 사용합니다.

<p align=center>
	$matchCnt [i]$ = 문자열 S의 $i$까지 봤을 때, 문자열 T와 일치하는 길이
</p>

벡터에 지금까지 확인한 S글자들을 담으며, 제거할 문자열을 찾았다면 벡터에서 T의 길이만큼 제거합니다. 그 후 이전의 상태, 즉 제거한 문자열의 첫 문자가 들어오기 직전의 상태로 돌아가야 합니다. KMP에서 $j$의 의미는 지금까지 매칭된 길이와도 동일하므로(구현에 따라 차이가 날 수 있음, 이 코드에선 매칭된 길이 -1입니다) $j$값을 $matchCnt$를 통해 복구하도록 합시다.

### 코드

```c++
#include <bits/stdc++.h>
#include <unordered_set>
#define rep(i,n) for(int i=0;i<n;++i)
#define REP(i,n) for(int i=1;i<=n;++i)
#define FAST cin.tie(NULL);cout.tie(NULL); ios::sync_with_stdio(false)
using namespace std;

string A, B;
int al, bl;

int fail[1000001];
int matchCnt[1000001];
int main() {
    FAST;
    cin >> B >> A;
    al = A.size();
    bl = B.size();

    for (int i = 1, j = 0;i < bl;++i) {
        while (j && B[i] != B[j]) j = fail[j - 1];
        if (B[i] == B[j]) fail[i] = ++j;
    }

    vector<char> censored;
    for (int i = 0, j = 0;i < al;++i) {
        censored.emplace_back(A[i]);
        while (j && A[i] != B[j]) j = fail[j - 1];
        if (A[i] == B[j]) {
            if (j == bl - 1) {
                rep(k, bl) censored.pop_back();
                j = matchCnt[censored.size()];
            }
            else ++j;
        }
        matchCnt[censored.size()] = j;
    }
    for (auto x : censored)
        cout << x;

    return 0;
}
```
