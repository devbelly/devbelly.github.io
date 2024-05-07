---
title: Introduction to Operating Systems
date: 2023-08-17T22:46:12+09:00
draft: false
toc: true
comments: true
tags:
  - os
---

컴퓨터에서는 일반적으로 여러 프로그램들이 실행됩니다. 한정적인 물리적 리소스를 통해 많은 프로그램들을 실행하려면 어떻게 할까요? 운영체제는 이 문제를 해결하기 위해 가상화라는 기술을 사용합니다. 이를 통해 운영체제는 여러 프로그램들을 실행하고 메모리를 공유하며 장치에 쉽게 접근할 수 있도록 해줍니다. 

## Virtualizing The CPU

<img width="100%" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/f4af59a1-8424-43e9-a96f-4cebe4cfeb62">

`argc` 값을 확인하여 두번째 파라미터에 입력된 값을 1초 간격으로 출력하는 프로그램입니다. `"usage: cpu <string>\n"` 부분을 통해 오브젝트 파일의 이름은 `cpu`임을 추측할 수 있습니다. 이 프로그램을 백그라운드에서 동시에 4개를 실행해보겠습니다.

<img alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/edbad6fa-b78b-409b-9e49-4be195c6cc0f">

`&` 를 통해 백그라운드에서 총 4개의 프로세스가 생성된 것을 확인할 수 있습니다. 프로세스를 식별하기 위한 pid도 확인할 수 있습니다.

컴퓨터의 CPU는 한정적인 자원입니다. 서버는 수십, 수천개를 갖고 있지만 일반적인 노트북은 그렇게 갖고 있지 못합니다. 그럼에도 CPU보다 많은 수의 프로그램을 실행할 수 있는데 그 이유는 CPU의 가상화에 있습니다. 물리적으로는 하나의 CPU지만 현상적으로는 무한대처럼 보이게 합니다.

<img alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/44dd9b12-89a8-4878-89c8-d66b4a927699">

## Virtualizing The Memroy

메모리는 프로그램이 수행되는 동안 CPU에 의해 처리되는 데이터를 저장하거나 CPU가 언제든 그 데이터에 엑세스 할 수 있게 하는 역할을 합니다. 그리고 프로그램의 각 명령어도 메모리에 있으므로 명령어를 가져올때마다 메모리에 엑세스하게 됩니다.

<img  alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/462043e6-965e-4367-bab5-854dec4edf32">

프로그램을 살펴보면 4바이트의 메모리를 할당받아 할당받은 주소를 출력한 후 1초 간격으로 해당 주소로 접근 해 1씩 증가하는 프로그램입니다. 아래는 위 프로그램을 그림으로 표현한 것입니다. 

<img  alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/7ba8fe64-86ae-4476-ba9a-bf72f24b79a8">

이 프로그램을 앰퍼샌드(`&`)를 통해 동시에 두 개를 실행해보겠습니다. 두 개를 실행했으므로 두 프로그램이 협력해서 동일한 물리적 메모리에 접근하여 숫자를 증가시킬까요?

<img alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/ea0f8021-797d-4165-9b61-fe0db022700b">

실행결과는 예상과 다른 것을 확인할 수 있습니다. 서로 다른 프로세스가 동일한 주소(`0x200000`)를 가리키고 있지만 서로 독립적인 값을 업데이트 하고 있습니다. 즉, 프로그램이 실행되면 프로세스가 생성되고 프로세스가 사용하는 메모리는 물리적인 메모리가 아닌 가상화된 메모리임을 알 수 있습니다. 각 프로세스마다 갖는 가상화된 메모리를 address Space라고 합니다. 

## Concurrency

![image](https://github.com/devbelly/image-issue/assets/67682840/995bdd24-d2ea-48ab-ad98-a7466e1f2cb2)

다음은 동시성 문제를 확인할 수 있는 코드를 살펴보겠습니다.  

`worker` 함수를 살펴보면 전역변수로 설정된 `loops` 만큼 `counter`를 증가시키고 있습니다. `main`함수에서 이러한 역할을 하는 쓰레드를 두 개 생성을 하여 쓰레드의 실행이 종료될 때 까지 `join`으로 기다리고 있습니다. 쓰레드는 `Pthread_Create`를 통해서 생성되고 두번째와 네번째 파라미터를 NULL로 설정함으로 스택의 크기가 자동으로 설정됩니다. 

프로그램에서 사용하는 `counter`와 `loops`은 전역변수이므로 메모리의 한 군데에 배치됩니다.  한 가지 다른점은 `counter`은 0으로 명시적으로 초기화를 해주지만 `loops`는 초기화 되어있지 않습니다. `loops`도 하드디스크에 있는 프로그램이 실행되기 위해 메모리에 올라올때 자동으로 0으로 초기화 되긴 합니다. 이 차이는 실행파일에 전역변수를 포함시키냐 안시키냐를 구분짓는 기준이 됩니다.

![image](https://github.com/devbelly/image-issue/assets/67682840/2e1c129e-d092-49e6-ad3b-edaa9bde95a4)

초기값이 있는 `counter`는 데이터 영역에 속하게 되어 실행파일에 포함이 됩니다. 다음은 실행파일에 포함되는 요소입니다.

- 초기값이 있는 전역변수
- 초기값이 있는 static variable
- 프로그램 코드(text 영역을 가리킵니다)
- BSS 영역의 크기

반면 `loops`는 초기값이 없어 실행파일에 속하지 않습니다. 이러한 변수들을 저장하는 공간을 bss영역이라 합니다.  이 영역은 프로그램이 메모리에 로드될 때 `null`로 초기화 됩니다. 예제에서 bss에 속하는 `loops`는 `int` 이므로 0이 됩니다.

![image](https://github.com/devbelly/image-issue/assets/67682840/41e5bb32-e984-4524-8ca1-e07b5b24869e)

앞에서 `Pthread_create` 부분에 두번째와 네번째 파라미터에 `Null`을 설정하면 자동으로 스택의 크기가 할당된다고 했습니다. 이를 통해 쓰레드마다 각자 스택을 갖는다는 것을 추측해볼 수 있습니다.  프로세스는 text, data, bss, heap 영역을 공유하지만 프로세스 안에 쓰레드는 각자의 stack을 갖게 됩니다. 이를 Execution stack이라 합니다.

스택의 크기가 정해진 후 `worker`함수를 실행하기 위해 `activation record`를 저장합니다. `activation record`란 실행 도중 사용할 지역변수와 파라미터 등을 저장하는 데이터 구조라고 생각하면 됩니다.

<img width="540" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/6b6d5fe3-3ee2-4daf-9e4c-aae6f8bf34b6">

프로그램 컴파일 후 아규먼트를 100000으로 지정했습니다. 예상되는 실행결과는 각 쓰레드가 협력해서 `counter`를 증가시키므로 200000 될 것 같지만 실제로는 그에 못미치는 숫자뿐 아니라 매 실행마다 결과값이 달라지는 비결정적인 모습도 확인할 수 있습니다.

<img width="457" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/2b6bcab8-d4f5-437f-b50b-7bf6483d51a9">

원인은 `++` 오퍼레이션에 숨어있습니다. 고급 언어에서는 한번의 연산처럼 보이지만 기계어 수준까지 내려가면 3단계로 나누어 연산하게 됩니다.

1. 메모리에서 cpu의 레지스터로 `counter`를 읽어옵니다. `counter`는 `volatile`로 선언되어있어 메모리에서 읽어옴이 보장됩니다.
2. ALU에서 읽어온 변수를 1 증가시킵니다.
3. 작업한 결과를 다시 메모리에 반영합니다.

프로그램에서는 두 개의 쓰레드가 위 1,2,3 연산을 통해 `counter`를 증가시키게 됩니다. 문제 상황은 쓰레드 `A`가 1, 2 연산을 통해 `counter`를 증가시켰지만 아직 3을 하지 않아 메모리에 반영을 하지 않았다고 해보겠습니다. 이 상황에서 쓰레드 B가 1, 2 연산을 수행하면 아직 메모리에는 증가된 값이 반영되지 않아 마찬가지로 `counter: 0`을 읽어온 후 레지스터에 `counter: 1`을 기록하게 됩니다. 다시 컨텍스트 스위칭으로 쓰레드 `A`가 레지스터 값을 메모리에 반영하면 메모리에 `counter: 1`이 기록되고 쓰레드`B` 도 자신의 레지스터 값을 메모리에 반영하면 `counter: 1`을 기록합니다. 결과적으로 각 쓰레드가 `counter`를 1씩 증가시키는 작업을 했지만 `counter`의 값은 2가 아닌 1이 됩니다.

즉 `++` 오퍼레이션은 atomic 하지 않아 race condition이 발생한 상황입니다. 나중에 이 상황에 대해 자세히 다루도록 하겠습니다.