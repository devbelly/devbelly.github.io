---
title: Segmentation
date: 2023-12-24T15:18:13+09:00
draft: false
comments: true
toc: true
tags:
  - os
---

이전 시간에는 `base & bound` 방식에 대해 살펴봤습니다. `base`를 기준으로 물리 메모리에 프로세스를 한 번에 배치하는 방식인 `base & bound`는 `internal fragmentation` 문제가 발생합니다. address space가 작을 때는 크게 상관없지만 만약 address space의 크기가 4GB라면 엄청난 낭비가 됩니다.

<img width="418" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/a0bb4a37-a7b5-4ecd-90f2-06f9a790bd8d">

이를 해결하기 위해 `base & bound`의 일반화 형태인 Segmentation이 등장했습니다. `base & bound`에서는 MMU에 하나의 base와 하나의 bound를 가지는 형태였지만 Segmentation에서는 각 세그먼트마다 base와 bound를 갖도록 합니다. 

<img width="258" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/f41d4cfa-4499-4ce8-b372-00f18d96f8b0">

물리 메모리에 각 세그먼트를 배치해 볼까요? 0KB부터 16KB는 OS가 사용중이므로 `kernel space`임을 알 수 있습니다. 00, 01, 11 세그먼트만 배치하고 10 세그먼트는 배치되지 않아 기존 `base & bound`의 문제였던 `internal fragmentation`가 어느정도 해소됨을 확인할 수 있습니다. 

<img width="232" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/5380d951-14ec-4409-9532-b440f53be4db">

각 세그먼트가 물리 메모리의 어디에 배치되었는지 추적하기 위해 segment table을 기록해야합니다. 이제 MMU는 위 테이블 정보를 통해 VA → PA 변환을 계산할 수 있습니다. 예를 들어 명령어 검색을 위해 가상 주소 100에 대한 참조가 이뤄진다면 `32KB + 100` 연산을 통해 물리 메모리 주소인 32868을 구합니다. 이후에 100이 올바른 접근인지 확인하기 위해 `2KB`와 비교를 하고 적절한 범위 내에 있으므로 32868에 대한 참조를 수행합니다. 

또 다른 예시로 가상 주소 4200에 대한 참조를 살펴보겠습니다. 4200은 `4KB ~ 8KB` 사이에 존재하는 주소이므로 Heap에 대한 참조임을 알 수 있습니다. offset을 구하기 위해 4200 - 4KB를 수행하면 104를 얻습니다. segment table 정보에 따라 Heap의 Base인 34KB와 offset 104를 더하면 원하는 물리 메모리 주소인 34920을 얻을 수 있습니다.

만약 힙의 끝을 벗어나는 접근을 한다면 OS는 비정상적인 상황임을 감지해 Trap을 발생시키고 Segmentation fault임을 확인할 것입니다. 원래는 Segmentation을 지원하는 OS에서만 사용했지만 현재는 Segmentation을 사용하지 않더라도 Segmentation fault 용어는 여전히 사용됩니다.

## Which Segment Are We Referring To?

<img width="380" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/8cbd2672-b58e-4ec3-9006-ed54a643d2c2">

가상 주소가 어떤 세그먼트에 속하는지 확인하기 위해 사용하는 방법 중 하나는 상위 일정 비트를 세그먼트 비트로 사용하는 것입니다. 처음 그림에서는 4개의 세그먼트를 사용했으므로 상위 두 개 비트를 세그먼트 비트로 사용해야 합니다.

또 다른 방법도 있을까요? 네, 있습니다. 만약 세그먼트를 많이 나눈다면 각 세그먼트의 최대 크기가 제한되는 문제가 발생하게 됩니다. 때문에 상위 비트를 통해 구분하는 대신, 주소가 어디서 생성되었는지 추적하는 방법을 통해 세그먼트를 구분할 수도 있습니다. 예를 들어 주소가 PC에서 생성되었다면 코드 세그먼트에 속하고, SP에서 생성되었다면 스택 세그먼트에 속한 것을 암시적으로 추적할 수 있습니다.

## What about Stack ? 

지금까지 세그먼트의 기본 구조에 대해 살펴봤습니다. 실제 물리 메모리에 배치된 그림을 유심히 살펴봤다면 눈치챘겠지만 코드와 힙 영역과는 달리 스택은 역방향으로 자라나고 있습니다. 그래서 MMU를 통해 VA → PA로 변환할 때 스택 세그먼트는 계산식이 조금 다릅니다. 이를 위해 MMU에는 추가적인 정보를 저장해야합니다.

<img width="430" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/17699e60-8e22-4efd-b86f-3173dc74a89d">

Grow Positive가 1이라면 순방향으로, 0이라면 역방향으로 자라나는 것을 표시합니다. VA가 15KB인 곳에 접근해볼까요? 이진수로 표시하면 `11 1100 0000 0000`이므로 11 세그먼트에 속하게 됩니다. 11 세그먼트는 `12KB ~ 16KB`이고 시작이 16KB이므로 오프셋은 -1KB입니다. 즉, 물리 메모리는 28KB - 1KB = 27KB에 위치하게 됩니다.
## Support for Sharing

<img width="526" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/2f0e5924-f6a1-43d4-be83-cb94fcbd32af">

만약 동일한 프로세스가 하나 이상 있다면 적어도 사용하는 코드 세그먼트는 동일합니다. 세그먼트 테이블의 도움을 받는다면 동일한 코드 세그먼트를 물리 메모리에 두 군데 배치하는 대신 공유하도록 할 수도 있습니다. 위 사진에서처럼 Protection bit를 추가적으로 사용해 만약 `Read-Execute` 비트라면 여러 프로세스가 해당 세그먼트를 공유하도록 허용함으로써(변경될 위험이 없으므로) 메모리를 절약하고 각 프로세스에게는 여전히 `beautiful illusion`을 제공할 수 있습니다.

## OS Support

Segmentation을 통해 물리 메모리의 절약이 어떻게 이뤄지는지 살펴봤습니다. `base & bound`보다 물리 메모리를 더 잘 활용하고 large and sparse virtual address 구조를 가진 프로세스에 적합합니다. 이를 위해 운영체제가 해야하는 역할은 무엇일까요?

- context switching 시, 주소변환정보인 segment table이 적절하게 관리되어야 한다.
- 세그먼트가 성장하거나 줄어들 때 운영체제가 적절히 관리해야한다.
- free space를 적절하게 관리해야한다.

<img width="364" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/fe56d86b-9e1e-45b6-8d07-e131520ed757">

`malloc()`은 정해진 힙 크기 내에서 메모리의 동적 할당을 위해 사용하는 함수입니다. 원문에서는 다음과 같이 표현하고 있습니다.

> a program may call malloc() to allocate an object. In some cases, the existing heap will be able to service the request, and thus malloc() will find free space for the object and return a pointer to it to the caller.

하지만 힙 세그먼트 자체가 커져야 할때는 UNIX syscall 중 하나인 `sbrk`를 사용할 수도 있습니다. 만약 호출에 성공한다면 새로운 break 위치를 리턴하게 됩니다. 

<img width="538" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/cf1e6ba2-eb8f-4096-b4dc-9b928a5b9de1">

마지막으로 free space를 적절하게 관리 해야하는 이유에 대해 살펴보겠습니다. 각 프로세스마다 여러 세그먼트를 사용하고 각 세그먼트의 크기도 제각각입니다. 세그먼트를 사용 및 해제하는 과정을 반복하다보면 왼쪽 그림처럼 빈 공간들이 발생하게 됩니다. 전체 크기를 합한다면 새로운 세그먼트를 배치할 수 있지만 파편화 되어 새로운 세그먼트를 배치하지 못하는 상황이지요. 이러한 문제를 `External fragmentation`이라 합니다. 

이 문제를 해결하기 위한 방법은 세그먼트를 예쁘게 재배치 하는 것입니다. OS는 실행 중인 프로세스를 중지하고 데이터를 메모리의 연속된 영역으로 복사하고 세그먼트 테이블을 업데이트 함으로써 오른쪽과 같이 연속된 영역을 얻을 수 있습니다. 새로운 세그먼트를 배치할 수 있다는 장점이 있지만 memory-intensive하고 상당한 양의 processor time을 소모합니다. 또한 기존에 존재하는 세그먼트의 크기를 확장하는데에도 어려움이 있다는 단점이 있음에 유의해야합니다.

## Summary

Segmentation은 기존 여러 문제들을 해결하는데 도움이 되지만 여전히 `External fragmentation` 문제가 존재하고 아직 완벽하게 `sparsed address space`를 지원하지는 못합니다. 