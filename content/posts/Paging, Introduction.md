---
title: Paging, Introduction
date: 2023-12-28T14:02:01+09:00
draft: false
comments: true
toc: true
tags:
  - os
---
가상 메모리 관리의 초기 형태인 `base & bound`와 세그멘테이션은 가변 크기로 메모리를 관리했습니다. 하지만 external fragmentation 문제와 비어있는 공간 관리가 까다로웠습니다. 그래서 이번 시간에는 고정 크기로 메모리를 관리하는 기법인 Paging에 대해 알아보겠습니다. 

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/d154ce7a-9718-4133-a55b-cc939b98103f">

Paging은 프로세스의 address space를 가변 크기의 논리적 세그먼트(코드, 힙, 스택)로 나누는 대신, **각각을 page라고 부르는 고정된 크기의 단위로 나눕니다.** 이러한 단위를 page라고 하며 물리 메모리의 frame이라 부르는 고정된 크기의 슬롯 배열에 배치합니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/e06bfa51-0cbd-44eb-9700-bfa156dd781c">

각 address space의 vitual page가 물리 메모리의 어디에 배치되었는지 기록하기 위해 운영 체제는 페이지 테이블을 가집니다. 대부분의 페이지 테이블은 per-process 입니다 (예외, Inverted page table). 만약 다른 프로세스가 실행된다면 virtual page는 다른 물리 페이지에 매핑되므로 운영체제는 이를 위해 다른 페이지 테이블을 사용해야합니다.

<img width="482" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/72449fdf-3818-4e79-b57f-acbb932a77c2">

실제로 페이지 테이블을 다루는 과정을 살펴보겠습니다. address space의 크기가 64바이트라면 virtual address는 6비트로 표현가능합니다. 이 가상 주소가 어떤 물리 메모리의 프레임에 속하는지 알기 위해선 가상  주소를 두 부분으로 나눠야 합니다. 두 부분을 각각 VPN, offset이라 합니다.

페이지의 크기가 예시에선 16바이트이므로 4개의 페이지를 선택할 수 있어야합니다. 그래서 최상위 2비트를 VPN으로 사용하고 남은 비트를 offset으로 사용합니다. 만약 가상 주소가 21이라면 VPN과 offset은 다음과 같을 것입니다.

<img width="264" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/657c4fda-4925-4886-9a64-a1feb0be0e47">

이제 주소변환정보인 페이지 테이블을 확인할 차례입니다. `VPN 1 → PF 7` 이므로 물리 메모리의 7번 프레임과1번 페이지가 매핑됩니다. MMU의 도움을 받아 변환해볼까요?

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/cae71ca6-c52d-4719-b20f-c1afa1200198">

실제 물리 메모리의 주소인 117을 얻어 해당 주소에 접근하는 모습입니다. Page와 Frame의 크기는 동일하고 offset은 페이지 내에서 어떤 바이트를 원하는지 알려주기 때문에 변환 후에도 VA가 갖고 있던 offset을 그대로 사용해 원하는 정보를 가져옵니다.
## Where Are Page Tables Stored?

페이지 테이블은 `base & bound`와 세그멘테이션에서 사용하는 변환정보보다 저장해야할 크기가 훨씬 큽니다. 32비트 머신을 가정해보겠습니다. 32비트 프로세서의 범용 레지스터 크기는 32비트이므로 프로그램 카운터의 크기 또한 4바이트입니다. 주소표현을 위해 4바이트 사용하므로 address space의 크기는 $2^{32}$byte입니다. 일반적으로 페이지 크기는 4KB이므로 `20비트 VPN + 12비트 offset`으로 나눕니다. 

운영체제는 **각 프로세스마다** $2^{20}$개의 변환을 처리해야하므로 Page Table Entry(PTE)의 크기가 약 4바이트라고 가정한다면, 각 페이지 테이블마다 4MB의 메모리가 필요합니다. 실행중인 프로세스 수가 조금만 증가하더라도 필요한 메모리 양이 급격히 늘어나므로 MMU에 이를 직접 저장하진 않습니다.

즉, 물리 메모리의 어딘가에 이를 저장해야합니다. 위 그림에서 0번 frame에 page table을 저장한 것을 확인할 수 있습니다. 

## What’s Actually In The Page Table?

페이지 테이블은 VPN에 알맞는 PFN을 찾아주는 역할을 합니다. 따라서 이를 처리할 어떠한 데이터 구조도 가능하지만 일단은 배열로 구현한 형태를 통해 페이지 테이블을 이해하고 나중에 더 고급 구조를 살펴보겠습니다. 

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/934b3d23-32c3-4b5e-88a8-f98037047fd3">

PTE에는 변환정보 외에도 추가적인 비트를 통해 다른 정보들을 나타냅니다.

### Valid bit

접근하는 주소공간이 유효한지 확인하고(1) 유효하지 않다면(0) 합법적이지 않는 접근으로 판단해 Segmentation Fault가 발생합니다. sparsed address space를 위해 중요한 요소입니다.

### Protection bit

해당 페이지에 읽기, 쓰기가 가능한지 검사합니다.
### Present bit

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/d37bbf9d-1732-4053-b58e-576d4e409709">

이 페이지가 물리 메모리에 있는지 디스크에 있는지를 나타냅니다. swapping은 운영체제가 잘 사용하지 않는 페이지를 디스크로 이동해 물리 메모리보다 큰 address space를 다룰 수 있도록 합니다. 

### Dirty bit, Reference bit

Dirty bit는 페이지를 메모리에 로드한 후 수정되었는지를 확인하고 Reference bit는 페이지에 엑세스 되었는지 확인하기 위해 사용됩니다. Reference bit는 swapped out 될 페이지를 결정할 때 사용됩니다. 만약 물리 메모리의 공간이 부족하여 페이지를 디스크로 내쫓아야할 때 인기 있는 페이지보다 자주 접근되지 않는 페이지를 내쫓는 것이 더 합리적이기 때문입니다.

## Paging, Also Too Slow

페이지 테이블이 너무 커서 메모리에 뒀지만 이 방법 또한 그리 좋진 않습니다. 

```perl
movl 21, %eax
```

이 명령어 수행 시, 물리 메모리에 몇 번 접근할까요? MMU에서 일어나는 일을 코드를 통해 살펴보겠습니다.

<img width="544" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c598f6d8-7aa7-46e2-a393-fd1b5a851c8f">

위 명령어를 가져온 상황에서 21(VA)를 117(PA)로 변경해야합니다. VA의 페이지가 어떤 프레임에 속하는지 파악해야하므로 VPN을 구하고 물리 메모리에 존재하는 페이지 테이블의 주소(PTBR)를 통해 원하는 PTE의 주소를 얻습니다. MMU에 모든 페이지 테이블 내용을 저장할 순 없으므로 대신 물리 메모리에 저장된 페이지 테이블 시작 주소인`PageTableBaseRegister`를 MMU에서 관리합니다. 즉, MMU는 다음과 같은 구조입니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/42ece15a-5d66-4cad-839f-777c51387e92">

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/3768f1c9-046b-4fbb-84ac-4b25711439bf">

PTBR과 VPN을 통해 배열처럼 direct indexing을 할 수 있습니다. PTE를 MMU로 읽어오면 하드웨어 수준에서 원하는 내용이 무엇인지 식별할 수 있게 됩니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/9133dcc6-587f-4bbd-92f5-2d5c6e55fc91">

PTE에서 PFN을 얻어 어떤 프레임에 속한지 확인했으니 PFN과 offset을 OR 연산을 통해 물리 메모리의 주소를 구할 수 있습니다. 그리고 해당 물리 메모리에 접근하면 원하는 값을 얻어 `eax` 레지스터에  값을 넣는 `movl` 연산을 수행할 수 있습니다.

위 과정을 통해 해결해야할 문제 두가지를 알 수 있습니다.
- VA를 tranlsation하는 과정에서 물리 메모리에 자주 접근한다.
- 페이지 테이블의 크기가 너무 크다.

## Summary

이번 시간에는 메모리 가상화 기법인 페이징에 대해 알아봤습니다. 이전 방법인 세그멘테이션보다 많은 장점이 있었습니다. 첫째, 메모리를 고정 크기로 나누어 외부 단편화 문제가 발생하지 않는다. 둘째, sparsed address space를 효율적으로 다룰 수 있다. 

하지만 효율적으로 구현하지 않으면 더 느려질 뿐만 아니라 메모리 낭비도 심해집니다. 다음 시간에 이러한 해결책에 대해 살펴보겠습니다.

## 참고

32비트 운영체제
- https://eine.tistory.com/entry/64%EB%B9%84%ED%8A%B8-32%EB%B9%84%ED%8A%B8-CPU%EC%99%80-%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C-%EC%97%90-%EB%8C%80%ED%95%98%EC%97%ACi