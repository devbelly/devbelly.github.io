---
title: "Test"
date: 2023-09-21T09:07:24+09:00
draft: true
comments: true
toc: true
tags:
  - untagged
---
안녕하세요. 이번 시간에는 주소 변환에 대해 살펴보겠습니다. 

CPU virtualization를 위해서 OS는 효율성과 제어 두가지 측면을 고려해야 했습니다. 때문에 `Limited Direct Execution`을 개발했고 프로세스는 CPU 하드웨어에서 `Direct`하게 실행되지만 특정 시점에 OS가 개입하여 프로세스가 할 수 있는 동작에 `limited`를 두었습니다.  

마찬가지로 Memory Virtualization은 `hardware based address translation`, 줄여서 `address translation` 을 통해 구현가능합니다.  실행 파일 안에 있는 주소들은 모드 `Virtual Address`입니다. 따라서 CPU에서 다루는 주소도 `VA`이고 실제 명령어 수행을 위해 물리 메모리에 접근할 때는 `Physical Address`로 변환되어야 합니다. 이때 변환을 하드웨어를 통해 이루어집니다.

이렇게 Memory Virtualization을 통해서 프로세스는 자신이 메모리를 독점하고 있다는`beautiful illustion`을 얻게 됩니다. 개발자는 환상에 대한 혜택을 누리게 되지요. 예를 들면 배열을 선언할 때 `배열이 크다면 다른 프로세스의 메모리 영역을 침범할까?`와 같은 고려를 하지 않기 때문입니다. 내가 선언한 변수나 자료구조는 내가 작성한 코드에서만 접근한다고 가정합니다.


이번 글에서는 아래 과정을 어떻게 해결해나가는지 살펴보겠습니다.

- 효율성
- 제어
- 유연성

하드웨어를 거치면 효율적이긴 하다(사진)
Of course, the hardware alone cannot virtualize memory, as it just pro- vides the low-level mechanism for doing so efficiently. 

하드웨어만으론 VM을 제공할 수 없고 OS가 메모리들에 대해서 관리를 해주어야 최종적으로 VM을 제공할 수 있다.



## Assumption

Memory Virtualization 어떻게 변화했는지 살펴보기 위해 아래와 같은 가정들을 하겠습니다.

- 사용자의 address spsace는 물리 메모리에 `contiguously` 하게 배치되어있다.
- address spacce의 크기가 물리 메모리 보다 작다
- 모든 address space의 크기는 일치한다.

스케줄링때와 마찬가지로 이 비현실적인 가정들을 없애나가며 최종적인 Memory virtualization에 대해 이해해보겠습니다.

## An Example

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/48011b94-9750-4e47-8116-c955a51a7021">
변수 `x`를 선언한 후 3을 증가시키는 코드입니다. 위 코드를 어셈블리로 살펴볼까요?

<img width="542" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/e51e6ef3-e2b6-456a-ab1b-633cb01f23ea">
변수 `x`는 `ebx` 레지스터에 저장되어 있다고 가정하겠습니다. 처음에는 `ebx`에서 `0x0`만큼 떨어진 메모리에 접근하여 값을 읽어 `eax` 레지스터에 저장합니다. `eax` 레지스터에 3을 더한 후 다시 `ebx`에서 `0x0`만큼 떨어진 메모리에 연산한 값을 저장하고 있습니다.  Address Space 상에서 코드와 데이터들은 어떤 식으로 저장되어있는지 보겠습니다.

<img width="262" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/6ae5f85a-2f66-4b5d-bdd7-d303ab88eebc">
프로세스는 자신이 메모리를 독점하고 있다고 생각하므로 0KB부터 필요한 데이터들을 저장합니다. 프로그램 코드는 0KB 근처에 저장하고 변수 `x`는 스택 근처인 15KB에 저장합니다. 만약 프로그램이 실행 된다면 다음과 같은 순서일 것입니다. 

- 128번지에 있는 명령어를 Fetch 한다
- 물리 메모리(15KB)에 접근하여 LOAD 명령어를 Execute 한다.
- 132번지에 있는 명령어를 Fetch 한다
- `eax`레지스터에 있는 값을 3 증가시킨다 (이때는 물리 메모리에 대한 접근이 없다.)
- 135번지에 있는 명령어를 Fetch 한다.
- 물리 메모리(15KB)에 접근하여 STORE 명령어를 Execute 한다.

프로세스는 가상 주소를 다루지만 결국에는 물리 메모리에 접근하는 것을 알 수 있습니다. 일반적으로 명령어나 데이터에 대한 물리 메모리로 바인딩 하는 과정은 크게 아래 3단계 중 한 군데에서 이루어 집니다.


<img width="336" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/87503051-9785-4904-b21b-250d05d1bd6c">

## Compile-Time Binding

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/75e6fd70-4e57-41f3-af2a-79ffa1ee84c8">
컴파일러가 컴파일 타임에 주소 바인딩을 하는 것을 의미합니다. 컴파일러가 symbolic address를 absolute address로 변환하는 특징이 있습니다. absolute address는 런타임과 상관 없이 물리 주소에서 사용하는 고정된 값의 주소를 의미하는데 위 사진에서 `Disk Image`에있는 `0x1018`와 `Memory Content`에 있는 `0x1018`이 같은 모습을 통해 absolute address를 사용함을 알 수 있습니다.

이 방식은 현재 메인 메모리에 대한 컨텐츠를 **미리** 파악하고 어디서 프로그램을 실행할지 결정해야하므로 multi-processing system과는 어울리지 않는 방식입니다. 만약 

## Load-Time Binding

<img width="631" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/8c29561c-5ab9-47e7-9fe0-8c039575f20c">
컴파일러가 생성한 `relocatable code`를 메모리에 로딩할 때 물리 메모리로 변환하는 방식을 의미합니다. 위 그림과 다르게 `Disk Image`에서 사용하는 주소는 물리 메모리에 매핑되는 주소가 아닌 `(.BS+0x18)` 처럼 상대 주소로 매핑 되어있습니다. 

물리 메모리 어디에서나 로딩하여 실행할 수 있으므로 멀티 프로그래밍이 가능해집니다. 하지만 로딩 시 프로그램에서 사용하는 주소들에 대한 변환이 **모두** 이루어진 후에 실행이 가능하기 때문에 로딩할 때 시간이 많이 걸리는 문제가 발생합니다.
## Execution-Time Binding

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c40e9f3c-b93a-48a9-9f12-ca9b3d1e0320">

로딩할 때 주소를 전부 변환하는 대신 하드웨어인 MMU의 도움을 받아 실행 시간에 주소를 변환해주는 방식을 의미합니다. `Memory Content`에서 사용하는 주소도 Virtual Address이고 실제 명령어를 가져와 실행할 때 MMU를 통해 Physcial Address에 접근하게 됩니다. 

## Base & Bound

런타임에 동적으로 주소를 재배치하는 것도 시간이 지남에 따라 달라졌지만 초기 형태인 `base & bound` 방식부터 살펴보겠습니다. 

Address space를 하나의 덩어리로 보고 해당 메모리들의 기준 값을 `base`, 메모리들의 범위 또는 크기를 `bound`라 하고 이를 통해 물리 메모리의 어디든 배치할 수 있습니다. 프로그램이 컴파일되어 실행될 때 물리 메모리에 접근 한다면 아래 공식을 통해 물리 메모리가 계산 됩니다.

<img width="442" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/284c661e-1fe0-431a-b418-fb27663ebe29">

즉, 초기 MMU의 상태는 간단한 형태로 구현할 수 있습니다.

<img width="308" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/7089b75d-a5cb-4d00-b2bc-512dd4c5d16c">

base와 bound 레지스터와 physical address 를 계산을 위한 덧셈 유닛, out of bound를 감지하기 위한 비교 유닛만으로도 구현할 수 있습니다. 이제 예시를 통해 어떻게 변환하는지 확인해볼까요? 초기 가정대로 address space는 굉장히 작게 4KB로 설정하기 위해 bound 레지스터를 4KB로, base 레지스터는 16KB로 가정하겠습니다.

 
<img width="480" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/f9b47ed4-ef84-49c3-ab4f-b7d457e67a01">

위 공식처럼 virtual address에 base값을 더하는 간단한 연산을 진행하여 변환했습니다. bound 값에 대한 검사는 base값을 더하기 전, base 값을 더한 후 둘 다 가능하지만 교재에서는 base값을 더하기 전 virtual address에서 검사를 진행했습니다.





여기서 문제가 발생한다. 어떻게 프로세스가 모르는 채로 우리는 relocate를 할 수 있을까?
어떻게 VM은 0부터 시작하시만 PA는 다른 곳에서 시작할 수 있을까?



Limited Direct Execution
- https://www.chegg.com/flashcards/chapter-6-mechanism-limited-direct-execution-945601e8-979b-4afe-b4ee-6ea0b775e505/deck
https://medium.com/@danny4410.eecs04/%E4%BD%9C%E6%A5%AD%E7%B3%BB%E7%B5%B1%E7%AD%86%E8%A8%98-memory-5be6f0e66de4