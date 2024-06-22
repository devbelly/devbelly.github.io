---
title: "[백준 13276] Prefix와 Suffix"
date: 2020-07-17T11:10:28+09:00
draft: false
comments: true
toc: false
tags:
  - ps
---

## 문제

[https://www.acmicpc.net/problem/13276](https://www.acmicpc.net/problem/13276)

<br>

## 알고리즘

KMP

<br>

## 풀이

다양한 풀이가 있는 문제입니다. 라빈 카프를 이용한 해싱, 가장 효율적인 Suffix Array 등 이 있지만 구현이 가장 쉬운 KMP로 풀었습니다.

$S$에 대해 두 문자열$A$,$B$가 나타나는 위치를 KMP를 통해 찾은 후, 부분 문자열이 될 수 있는 것들을 담았습니다. $vector$에 직접 부분 문자열을 담게 되면 메모리 초과가 발생하니, $set$을 통해 부분 문자열을 담고, 자연스레 중복도 제거하도록 합시다. 시간 복잡도는 모든 인덱스에서 부분 문자열이 시작하고 끝날 수 있으므로 $O(N^2)$ + 해시 접근입니다.

### 코드

```c++
#include <bits/stdc++.h>
#include <unordered_set>
#define rep(i,n) for(int i=0;i<n;++i)
#define REP(i,n) for(int i=1;i<=n;++i)
#define FAST cin.tie(NULL);cout.tie(NULL); ios::sync_with_stdio(false)
using namespace std;

string S, A, B;
int slen, alen, blen;
int af[2000], bf[2000];
vector<int> aidx, bidx;
void getfail(string s, int* fail) {
    int len = s.size();
    for (int i = 1, j = 0;i < len;++i) {
        while (j && s[i] != s[j]) j = fail[j - 1];
        if (s[i] == s[j]) fail[i] = ++j;
    }
}

void KMP(string H, int* fail, vector<int>& vt) {
    int len = H.size();
    for (int i = 0, j = 0;i < slen;++i) {
        while (j && S[i] != H[j]) j = fail[j - 1];
        if (S[i] == H[j]) {
            if (j == len - 1) {
                vt.emplace_back(i - j);
                j = fail[j];
            }
            else ++j;
        }
    }
}

int main() {
    FAST;
    cin >> S >> A >> B;
    slen = S.size();
    alen = A.size();
    blen = B.size();

    getfail(A, af);
    getfail(B, bf);
    KMP(A, af, aidx);
    KMP(B, bf, bidx);

    unordered_set<string> us;
    for (auto& y : aidx) for (auto& x : bidx) {
        int len = x + blen - y;
        if (len < alen || len < blen) continue;
        us.emplace(S.substr(y, x + blen - y));
    }
    cout << us.size();

    return 0;
}
```
