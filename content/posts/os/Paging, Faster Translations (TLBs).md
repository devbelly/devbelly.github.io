---
title: Paging, Faster Translations (TLBs)
date: 2024-01-01T10:27:10+09:00
draft: false
comments: true
toc: true
tags:
  - os
---
지난 시간에 페이지 테이블이 물리 메모리에 저장되므로 프로그램이 생성하는 각 가상 주소에 접근할 때 추가적인 메모리 조회가 필요한 단점에 대해 살펴봤습니다. 이번 시간에는 TLB를 통해 물리 메모리에 자주 접근하는 문제를 해결해보겠습니다. 

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c375ade1-6761-4c8f-8a5b-1e467a5733af">

TLB는 `Translation-Lookaside Buffer`의 약어로 MMU 내에 저장됩니다. 각 가상 메모리 참조마다 MMU는 먼저 TLB를 확인하여 원하는 정보가 있는지 확인하고, 저장되어 있다면 페이지 테이블에 접근하지 않더라도 PFN을 얻을 수 있습니다. 

## TLB Basic Algorithm

<img width="550" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/e2357d5b-5dd5-4b82-a2f7-1e16ae3e31a0">

선형 페이지 테이블 + `hardware-managed TLB`를 기반으로 어떤 일이 일어나는지 살펴보겠습니다. VPN을 얻은 후 TLB가 VPN 변환 정보를 갖고 있는지 확인합니다. 만약 보유하고 있다면 **TLB hit**입니다. 메모리에 접근할 필요 없이 TLB에서 PFN을 가져와 접근할 물리 메모리의 주소를 획득합니다. 

만약 TLB에 원하는 정보가 없다면 **TLB miss**인 상황입니다. 어쩔 수 없이 하드웨어가 페이지 테이블에 접근하여 변환 정보를 찾습니다 (Line 11-12). `PTE.Valid` 비트를 확인하여 유효한 접근이라 판단되면 TLB를 업데이트하고 (Line 18) 하드웨어는 명령을 재시도합니다. 그러면 TLB에서 변환 정보를 찾고 빠른 속도로 메모리 참조가 가능합니다.

## Example, Accessing An Array

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/3cab6edd-3d08-448a-83a0-13433440f60f">

TLB 작동을 예시를 통해 살펴보겠습니다. 그림에서는 16바이트 페이지가 있는 8비트 address space를 가정합니다. 따라서 `4비트 VPN + 4비트 offset`으로 나뉩니다. TLB의 원리를 알아보기 위해 명령어나 다른 변수들에 대한 메모리 접근은 고려하지 않고 배열 $a$에 발생하는 메모리 접근만 살펴보도록 하겠습니다.

처음에 배열의 시작 원소인 $a[0]$이 위치하는 100에 접근하기 위해 VPN( = 06 )을 구하고 TLB에 VPN에 해당하는 엔트리가 있는지 확인합니다. 처음에 TLB는 비어있으므로 **TLB miss**가 발생합니다. 

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/63185dae-1623-4d7c-974f-3e3f26d0cafe">

이제 $a[1]$에 접근해보겠습니다. VA 104에서 VPN을 추출하면 6입니다. TLB에서 이와 대응하는 PFN이 있는지 확인해보니 8이라고 존재합니다. **TLB hit**인 상황입니다. 배열의 두 번째 요소는 첫 번째 요소와 인접하기 때문에 같은 페이지에 존재했고 그 결과 **TLB hit**이 되었습니다. 페이지 테이블에 직접 접근하지 않으므로 Translation을 빠르게 진행할 수 있습니다.

마찬가지로 $a[9]$까지 반복하면 TLB의 상황은 다음과 같습니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/72cf3901-cb2c-4b54-98b4-c1f73c11a502">

TLB miss은 파란색으로 TLB hit은 빨간색으로 표시했습니다. `TLB hit rate`를 계산해보면 70%로 배열에 처음 접근했음에도 꽤나 괜찮은 수치입니다. 이렇듯, 배열의 요소들이 같은 페이지에 밀접해 hit rate가 올라간 것을 `spatial locality`가 있다고 합니다. 만약 페이지의 크기가 16바이트 → 32바이트로 커진다면 hit rate가 더 증가할 것입니다. 

현재 TLB에 내용이 찬 상태에서 반복문을 다시 처음부터 돈다면 모든 메모리 접근에 대해 TLB hit이 발생합니다. 현재 접근한 메모리에 잠시 후 접근해 캐시의 이점을 살리는 특성을 `temporal locality`라고 합니다.  

## Who Handles The TLB Miss?

TLB Miss가 발생하면 하드웨어에서 처리하거나 소프트웨어에서 처리할 수 있습니다. TLB Basic Algorithm에서 살펴본 방식이 하드웨어가 직접 TLB Miss를 처리하는 예시입니다. 자신이 `PTBR`을 기억하고 있어 페이지 테이블에 접근해 PTE를 가져와 TLB정보를 업데이트 합니다. 

<img width="562" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/5f6899b8-eb71-4dae-8070-dac0947c0ab6">

좀 더 현대적인 아키텍처에선 소프트웨어로 TLB Miss를 처리할 수 있습니다. TLB Miss 발생 시 하드웨어가 예외를 발생시킵니다. 이로 인해 Trap이 발생하며 Trap Handler는 자신이 Trap에 빠진 이유를 확인합니다. 그래서 TLB Miss를 처리하기 위한 운영 체제 내의 코드가 실행되고 `privileged operation`을 사용해 TLB를 업데이트하고 `return-from-trap`을 수행합니다. 

이전 글에서 살펴본 `return-from-trap`은 procedure call 다음 명령으로 가서 프로그램이 계속 수행되지만 TLB Miss에서 `return-from-trap`이 수행되면 트랩을 시작한 명령으로 다시 돌아가야합니다. 그래야지 TLB hit이 되어 올바른 translation을 수행합니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/4fbfabc3-2463-4db1-a6d2-67b6401551de">

또한 하드웨어 대신 소프트웨어에서 TLB Miss를 처리할 때 TLB Miss가 계속 발생하지 않도록 주의해야합니다. trap이 발생하여 이를 처리할 trap handler를 찾아야하는데 이 과정에서 또 TLB Miss가 발생한다면 무한루프에 빠지게 됩니다. 

첫 번째 해결책은 TLB Miss handler를 물리 메모리에 유지하는 방법입니다. 이를 `unmapped`되었다고 하며 물리 메모리에 직접 뒀기 때문에 주소 변환의 대상이 되지 않습니다. 두 번째 해결책은 TLB의 일부 엔트리를 TLB Miss handler를 위해 영구적으로 할당하는 방법입니다. 항상 연결되어있다는 의미에서 `wired translation`이라고 하며 이 경우 무조건 TLB hit이 발생합니다. 

하드웨어 기반에 비해서 `software-managed TLB`는 다음과 같은 장점이 있습니다.
- 유연성 : 운영체제는 페이지 테이블 구현을 위해 어떠한 데이터 구조를 사용할 수 있다.
- 간결성 : TLB Miss가 발생했을 때, 하드웨어가 해야할 일이 간단하다. (예외만 발생)

## TLB Contents, What’s In There?

TLB는 `fully associative cache`로 변환 정보가 TLB 어디든 존재하고 전체 TLB를 병렬로 빠른 시간 내에 찾습니다. 그럼 TLB Entry는 `VPN → PFN` 의 정보 말고도 어떤 정보들을 갖는지 살펴봅시다.

## TLB Issue, Context Switches

<img width="248" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/67b80013-8e01-400a-bc50-bcd50bc9e59c">

TLB Entry에서 가장 중요한 정보는 `VPN → PFN` 입니다.  address space는 프로세스마다 가지므로 해당 정보는 컨텍스트 스위칭 시 현재 프로세스에게는 필요없는 정보입니다. 만약 Process 1이 `VPN(10) → PFN(100)` 정보를 TLB에 저장하고 컨텍스트 스위칭 후 Process 2가 `VPN(10) → PFN(170)` 정보를 기록했다면 다시 Process 1으로 컨텍스트 스위칭 됐을 때 어떤 정보를 활용해 PFN을 찾아야 하는지 알 수 없습니다.

비효율적인 해결책은 컨텍스트 스위칭이 될 때 TLB 정보를 비우는 방법입니다. 비울때는 valid bit를 0으로 설정하면 됩니다. 하지만 프로세스가 실행될때마다 TLB Miss가 발생하는 안좋은 상황이 따릅니다. 이 오버헤드를 줄이기 위해 컨텍스트 스위칭 간 TLB를 공유하기 위해 address space identifier bit를 추가적으로 사용할 수 있습니다. PID와 비슷하지만 TLB에 넣기 위해 32비트의 PID보다 작은 8비트를 사용합니다. 

<img width="304" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/3bb09dee-5764-424f-b48f-dfcc1fe71ec6">

ASID 비트가 있다면 컨텍스트 스위칭이 되더라도 TLB 정보를 활용할 수 있습니다. 

> **TLB.Valid vs PTE.Valid**
> 
> PTE에서 Valid bit가 0이라면 해당 페이지가 프로세스에 의해 할당되지 않았고, 제대로 동작하는 프로그램에서는 해당 페이지에 접근해선 안됩니다.
> 
> 하지만 TLB의 Valid bit는 단순히 해당 정보가 유효한지 아닌지를 판단할 때 사용됩니다. 만약 Valid bit가 0 이라면 해당 TLB는 비어 있습니다.

## A Real TLB Entry

<img width="528" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/74ddd35b-f81a-461c-acab-19454743379d">

`software-managed`의 MIPS 계열 TLB를 살펴보겠습니다. 4KB 페이지와 32비트 주소 공간을 지원하므로 `VPN(20) + offset(12)`를 예상했으나 실제 VPN은 19비트입니다. 왜냐하면 address space의 절반은 user space, 나머지 절반은 kernel space로 사용하기 때문입니다. 

VPN은 24비트의 PFN으로 변환되므로 최대 64GB($2^{24}$ X 4KB)의 물리 메모리를 지원할 수 있습니다. 이외에도 G는 전역비트로 프로세스와 관계 없이 전역으로 공유되는 페이지에 사용됩니다. C는 coherence bit로 하드웨어에 의해 페이지가 어떻게 캐시되었는지를 나타내고 Dirty나 Valid 비트로 페이지가 기록되었거나 유효한지를 나타낼 수 있습니다. 

## Summary

이번 시간에는 페이징의 첫 번째 문제였던 변환 속도를 TLB로 어떻게 해결하는지 살펴봤습니다. 다음 시간에는 페이지 테이블의 크기를 줄이기 위한 방법에 대해 알아보겠습니다.