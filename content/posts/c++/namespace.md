---
title: "namespace"
date: 2020-10-31T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - c++
---

### 선언 영역

말 그대로 선언을 할 수 있는 영역입니다. 예를 들어 전역 변수는 모든 함수의 바깥에서 선언할 수 있습니다. 그 변수의 선언 영역은 그것이 선언된 파일입니다. 또한 어떤 변수를 함수 안에 선언한다면 그 변수의 선언 영역은 그것이 선언된 블록입니다.

### namespace

새로운 종류의 선언 영역을 정의함으로써 이름이 명명된 namepsace를 만들 수 있다는 기능이 c++에 추가되었습니다.

하나의 namespace에 속한 이름은 동일한 이름의 다른 namespace에 선언된 이름과 충돌하지 않습니다. namespace는 전역 위치 또는 다른 namespace에 놓일 수 있습니다. 이는 namespace가 상수 참조를 하지 않는다면 기본적으로 외부 링크를 갖게 됩니다. (const는 static 하므로 extern 한 namespace와는 반대됩니다)

**✔사용범위 결정 연산자 ::**  
namespace에 접근을 할 때는 사용범위 결정 연산자인 ::을 통해 접근합니다.

```c++
namespace Jack{
    double pail;
    void fetch();
    int pal;
}

namespace Jill{
    double bucket(double n);//{someting}
    double fetch;
    int pal;
}

int main(){
    Jack::pal=3;
    Jill::pal=4;
    return 0;
}
```

좋은 변수명을 namespace로 구분하여 사용할 수 있다는 장점이 있지만 매번 ::을 통해 namespace에 접근하는 것은 굉장히 번거롭습니다. 이에 대한 해결책으로 C++은 using선언과 using지시자를 제공합니다.

**✔ using 선언**  
using선언은 제한된 이름 앞에 키워드 using을 붙이는 것입니다. 제한된 이름(qualified name)이란 위의 코드에서 jack::pal의 경우처럼 namespace가 지정된 이름을 제한하는 경우를 의미합니다. using선언을 사용하면 그것이 나타나는 선언 영역에 하나의 특별한 이름을 추가합니다. 예를 들어 main()에 있는 Jill::fetch의 using선언은 main()에 의해 정의되는 선언 영역에 fetch를 추가합니다. 이 선언을 한 후에는 Jill:fetch 대신에 fetch를 사용할 수 있습니다.

```c++
namespace Jill{
    double bucket(double n){//somthing};
    double fetch;
}

char fetch;
int main(){
    using Jill::fetch;
    double fetch; // 불가능 이미 지역fetch가존재
    cin>>fetch;  //값을 jill:fetch에 저장
    cin>>::fetch // 전역 fetch를 사용!
}

```

**✔ using 지시자**  
이와 반대로 using 지시사는 모든 이름을 사용할 수 있게 만듭니다. 만약 using지시자를 전역 위치에 놓으면 그 이름 공간에 속한 이름들을 전역적으로 사용할 수 있습니다.

**✔ using 선언 vs using 지시자**  
using 지시자를 사용하는 것은 using 선언을 여러 번 하는 것과는 명백히 다릅니다. 오히려 :: 연산자를 통해 여러번 적용하는 것에 가까운 행동입니다. using 선언을 사용할 때, 이는 using 선언이 놓이는 위치에 그 이름을 선언하는 것과 같습니다. 즉 선언할 블록에 이미 선언하려는 이름이 선언되어있으면 using 선언을 사용할 수없습니다. 반면에 using 지시자는 using지시자와 그 namespace 자체를 둘 다 포함하는 최소한의 선언 영역에 그 이름들을 선언하는 것과 같습니다. 이는 using 지시자만 포함하는 블록보다 상위 개념이므로 만약 동일한 이름이 using 지시자를 사용하는 블록 안에 선언되어있으면 using 지시자를 사용한 이름은 지역 이름에게 범위가 가려지게 됩니다.

```c++
namespace Jill
{
    double bucket(double n); //{someting}
    double fetch;
    int pal;
} // namespace Jill

int main()
{
    using namespace Jill;     // 해당 이름 공간의 모든 이름을 불러옵니다.
    double water = bucket(2); // 가능
    double fetch;             // 가능 Jill::fetch를 가리게 됨.
    cin >> fetch;             // 지역 fetch에 입력을 받음
    cin >> ::fetch;           // 전역 fetch에 입력을 받음
    cin >> Jill::fetch;       // Jill::fetch에 입력을 받음
    return 0;
}
```
