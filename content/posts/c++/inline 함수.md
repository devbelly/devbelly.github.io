---
title: "inline 함수"
date: 2020-10-01T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - c++
---

### 일반적인 함수의 작동

컴파일 작업의 최종 산출물은 기계어 명령으로 이루어진 실행 프로그램입니다. 프로그램 실행 시 운영체제는 이 명령들을 컴퓨터의 메모리에 로드하고 각 명령들은 자신의 주소를 갖게 됩니다. 프로그램에서 함수의 호출이 이루어진다면 함수 호출 명령 다음에 있는 명령의 주소를 메모리에 저장하고 스택에 매개변수를 복사합니다. 그 후 호출한 함수가 시작되는 위치로 이동하게 됩니다. 호출된 함수는 실행되고 자신의 리턴 값을 레지스터에 복사하고 저장해두었던 주소의 명령으로 돌아오게 됩니다.

### Inline 함수의 필요성 및 특징

이렇게 자신이 돌아올 위치를 기억하고 점프를 수행했다가 돌아오는 행동은 함수를 사용하는데 시간이 많이 걸립니다. Inline 함수는 컴파일러가 함수호출 부분을 그에 대응되는 함수 코드로 대체함으로써 이러한 문제를 해결합니다. 하지만 해당 함수의 복사본을 프로그램에 호출한 횟수만큼 붙여 넣기 때문에 메모리 사용 측면에서의 단점이 있습니다. 또한 컴파일러는 사용자가 inline을 붙였다고 다 처리해주지 않습니다. 사이즈가 크거나 재귀 호출일 때 거절하는 경우도 있습니다.

함수 코드 자체의 수행시간이 길 경우, Inline 함수의 사용으로 보는 이득은 굉장히 미미합니다. 그러므로 함수 수행의 시간이 굉장히 짧은 경우 사용하는 것이 바람직합니다. Inline 함수를 사용하는 방법은 다음과 같습니다.

- 함수의 선언 앞에 inline 키워드를 사용한다.
- 함수의 정의 앞에 inline 키워드를 사용한다.

요즘은 선언을 따로 두지 않고 한 번에 적습니다.

```c++
#include <iostream>
using namespace std;

inline int Add(int a, int b) { return a + b; }

int main() {
    int a = 3, b = 5;
    cout << "Inline Add :" << Add(a, b) << '\n';
    return 0;
}
```

<br>

### #define vs inline

#define은 소스코드를 전처리기로 보냅니다. 이후 소스코드 전부를 확인하여 단어를 대체합니다. 대체가 끝나면 소스코드가 컴파일 과정으로 넘겨지게 됩니다. 이처럼 C에선 선행처리기 #define구문으로 해결 가능합니다. Inline 함수와 마찬가지로 해당 코드를 지정한 텍스트로 대체하는 형식으로 작동합니다. 하지만 매개변수를 넘기는 inline 함수와는 달리 매개변수를 넘기지 않은 #define은 여러 문제점이 있습니다.

```c++
#include <iostream>
using namespace std;

#define SQ(x) x*x

int main() {
    int x = 4;
    cout << SQ(5.0) << '\n'; // return 25
    cout << SQ(2 + 3) << '\n'; // return 11
    return 0;
}
```

두 줄에 걸쳐 사용한 SQ은 둘 다 25를 리턴할 것을 기대하지만 실행결과는 다릅니다. 위 코드는 아래 코드와 동일합니다.

```c++
#include <iostream>
using namespace std;

#define SQ(x) x*x

int main() {
    int x = 4;
    cout << 5.0 * 5.0 << '\n';
    cout << 2 + 3 * 2 + 3<<'\n';
    return 0;
}
```

소괄호를 사용함으로써 위와 같은 우선순위 문제를 해결할 수 있습니다 .하지만 아직 문제가 더 남아 있습니다.

```c++
#include <iostream>
using namespace std;

#define SQ(x) ((x)*(x))

int main() {
    int x = 4;
    int y = 4;
    cout << SQ(++y) << '\n';//return 36
    cout << SQ(x++) << '\n';//return 16
    return 0;
}
```

두 상황 다 25를 기대하기 쉽지만, 실제론 36과 16이 됩니다. Inline 함수는 매개변수를 전달함으로써 여러 상황에 대해 유연하게 대처합니다.
