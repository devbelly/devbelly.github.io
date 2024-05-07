---
title: Paging, Smaller Tables
date: 2024-01-05T09:49:40+09:00
draft: false
comments: true
toc: true
tags:
  - os
---
오늘은 페이징의 두 번째 문제인 페이지 테이블의 크기에 대해 이야기해 보겠습니다. VPN을 인덱스로 원하는 PTE를 얻기 위해 선형 페이지 테이블 구조인 배열을 사용했습니다. $O(1)$에 원하는 엔트리에 접근하는 장점이 있지만 페이지 테이블의 크기가 굉장히 커지는 단점 또한 존재합니다.

## Simple Solution, Bigger Pages

32비트 주소 공간에서 페이지 크기가 4KB라면 `20bit VPN` + `12bit offset`으로 나뉩니다. 여기서 페이지 크기를 16KB로 확장한다면 `18bit VPN` + `14bit offset`이므로 페이지 테이블의 엔트리 갯수가 $2^{18}$개로 감소합니다. 하지만 이 방식은 할당된 페이지의 일부만 사용하는 internal fragmentation 문제로 이어져 본질적인 해결책이 아닙니다.

## Hybrid Approach, Paging And Segments

두 번째 방법은 프로세스의 전체 address space를 단일 페이지 테이블로 구성하는 대신, 세그먼트 당 하나씩 페이지 테이블을 갖게하는 방법입니다. 원래 세그멘테이션은 주소 공간을 여러 세그먼트로 나눈 뒤, 각 세그먼트마다 `base & bound`를 갖는 방식입니다. 하지만 이 방법에서 `base`는 세그먼트 자체를 가리키는 주소 대신 해당 세그먼트의 페이지 테이블의 물리적 주소를 보관하는데 사용합니다. `bound` 는 유효한 페이지 수를 나타냅니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/cf4aed9b-1f2a-4fb2-a8d3-876ac13e62a8">

VA를 보면 전부 VPN으로 사용하는 대신 앞에 두 비트를 통해 세그먼트를 구분하고 있습니다. 10 세그먼트는 사용하지 않고 나머지 3개의 세그먼트는 페이지 테이블의 시작 주소를 가리킵니다. 

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/baf01fc2-8752-42c2-9d3d-8cfdaabd3d8b">

MMU는 다음과 같은 구조입니다. Paging and Segments 방식과는 무관하게 TLB가 있어 만약 TLB Hit이라면 바로 PFN을 얻을 수 있지만 TLB Miss라면 하드웨어는 Segment Number(SN)을 사용해 사용할 `base & bound`를 결정합니다. 그 다음 하드웨어는 `base`에 있는 물리 주소를 사용해 PTE의 주소를 얻을 수 있습니다.

기존 페이징에서 사용했던 `PTBR` 대신 `Seg Table`을 사용하는 것 외엔 큰 차이는 없습니다. `bound` 레지스터를 통해 세그먼트의 끝을 넘어가는 메모리 엑세스는 예외를 발생시키거나 10 세그먼트처럼 사용하지 않는 부분을 not valid라고 표시해 메모리를 절약할 수 있습니다.

하지만 sparsed address space를 완벽하게 지원하기 어렵고, 페이지 테이블의 크기가 가변적이므로 세그멘테이션에서 발생했던 external fragmentation 문제가 다시 발생합니다.

## Multi-level Page Tables

address space를 세그멘트로 나누는 대신 페이지 테이블을 잘라서 공간을 절약하는 방법이 있습니다. 이러한 방법을 `multi-level page table`이라 하며 현대 시스템에서 자주 사용됩니다.

기본적인 아이디어는 페이지 테이블을 **페이지 크기로 자른 후** 해당 페이지에 속하는 PTE가 하나라도 없다면 새롭게 자른 페이지를 invalid라고 표시하는 방법입니다. 페이지 테이블의 페이지가 유효한지 확인하기 위해 추가적으로 `Page Directory`를 사용합니다.

<img width="604" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/5f296cd6-e666-4f3a-907f-d2085cfd3625">

왼쪽에 보이는 그림은 기존 페이지 테이블 구조입니다. Direct Indexing을 위해 중간에 필요하지 않은 공간도 사용하므로 물리 메모리 낭비가 심합니다. 오른쪽에 보이는 그림은 `Multi-level Page Table`구조로 페이지 테이블을 페이지 크기로 자른 후 페이지 테이블의 페이지가 어디에 존재하는지 알려주거나 해당 페이지 테이블의 페이지가 유효하지 않다는 것을 표시하고 있습니다.

장점은 무엇일까요?`Multi-level Page Table`은 사용한 address space에 비례해 페이지 테이블 공간을 할당하므로 sparsed address space를 완벽하게 지원합니다. 또한 세그멘테이션에서는 **가변적인 페이지 테이블의 크기** 때문에 비어있는 공간 관리가 어려웠지만 고정된 크기로 할당하는 방식에서는 페이지 테이블을 확장할 때 간단하게 가능합니다. 

하지만 TLB Miss가 발생하면 올바른 주소 변환을 위해 메모리에 두 번 접근을 해야합니다(페이지 디렉토리 1번 + 페이지 테이블 1번). 

## Detailed Multi-Level Example

address space가 16KB이고 페이지 크기가 64바이트인 예시를 살펴봅시다. 이 경우 VPN으로 8비트를 사용하고 offset은 6비트를 사용합니다. 만약 선형 페이지 테이블 구조를 사용한다면 256개의 엔트리가 필요합니다. 이를 개선하기 위해 `two-level page table`을 사용해봅시다.

<img width="524" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/304a0196-e13f-4c46-a01e-f49e3c1f48c2">

우선 선형 페이지 테이블을 페이지 크기로 잘라야합니다. PTE 크기가 4바이트라면 1개의 페이지에는 16개의 PTE가 속합니다. 페이지 디렉토리 엔트리의 갯수는 `256 / 16`이므로 16개입니다. 

<img width="410" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c0a932fc-9386-4438-b8c6-cb493b7beea6">

즉, 페이지 디렉토리를 인덱싱하기 위해 4비트를 사용합니다. `PDIndex`로 (조각난) 페이지 테이블을 찾은 후 테이블 내에서 인덱싱하기 위해 4비트를 사용합니다. MMU가 `Page Directory Base Register`만 기억하고 있다면 `PDEAddr = PDBR + (PDIndex * sizeof(PDE))`로 PDE를 얻을 수 있습니다. 

만약 PDE가 invalid로 표시되어 있다면 해당 접근이 유효하지 않으므로 예외가 발생합니다. 하지만 valid하다면 `PTEAddr = (PDE.PFN << SHIFT) + (PTIndex * sizeof(PTE))`을 통해 PTE를 구하고 최종적으로 PFN을 구할 수 있습니다.

실제로 `VPN 254 + offset 0`을 참조해보겠습니다. `1111 1110 000000` 에서 PDIndex는 15입니다. PDE에서 페이지 테이블의 시작 주소가 PFN 101에 있다는 정보를 얻었습니다. PTIndex는 14이므로 페이지 테이블의 14번째 인덱스에 접근하면 PFN 55를 최종적으로 얻을 수 있습니다. 

## More Than Two Levels

`multi-level page table`을 간단히 설명하기 위해 하나의 페이지 디렉터리와 페이지 테이블 조각을 가정했습니다. 하지만 주소 공간의 크기가 커진다면 그만큼 PTE 갯수도 늘어나므로 페이지 디렉터리 또한 커집니다. 

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/47eeaa84-8a5a-487f-b556-4b882a3cb941">

30비트 address space와 512바이트 페이지를 가정해보겠습니다. 엔트리의 크기가 4바이트라면 하나의 페이지에 들어갈 수 있는 PTE의 갯수는 $2^{9-2}$이므로 $2^7$개 입니다. VPN에서 뒤에 7비트는 Page Table Index로 사용한다면 앞에 14비트는 Page Directory Index로 사용합니다. 이렇게 되면 디렉터리 자체도 매우 커집니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/99bf0977-37b4-4951-b536-3bedd0288c54">

따라서 페이지 디렉터리 또한 페이지 크기로 자른 후 추가적인 페이지 디렉터리를 사용합니다. 이제 `upper-level page directory`를 인덱싱할 때 VPN의 상위 7비트를, `second level page directory`를 인덱싱할 때는 그 다음 7비트를 사용하고 원하는 페이지 테이블 조각의 주소를 얻었다면 마지막 하위 7비트를 페이지 테이블 내에서 인덱싱할때 사용합니다.

## Inverted Page Tables

페이지 테이블 공간을 절약하는 다른 방법은 `Inverted Page Table`이 있습니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/6b41b55c-88d3-4b30-9298-a1600bf46043">

왼쪽 그림은 각 프로세스마다 페이지 테이블이 있고 현재 페이지가 어떤 물리 프레임과 대응하는지 기록합니다. 오른쪽 `Inverted Page Table`은 시스템에 물리 메모리와 동일한 크기로 단 하나 존재합니다. 물리 프레임을 어떤 프로세스의 페이지(`pid + Page no`)가 사용하는지 기록합니다. 어차피 물리 메모리를 사용할 수 있는 페이지는 한정적이기 때문입니다. 

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c3b48c8b-a1fb-488f-9c06-c5911251f0d0">

공간을 극단적으로 절약하는 장점은 확인했지만 단점은 무엇일까요? 페이지 테이블에서는 연속적인 주소 공간을 사용해 Direct Indexing을 사용할 수 있어 `O(1)`에 검색이 가능했습니다. 하지만 `Inverted Page Table`은 원하는 정보를 찾기 위해 모든 정보를 다 확인해야하므로 `O(N)`이 소요됩니다. 따라서 검색에 용이한 해시 테이블을 함께 사용해 단점을 해결합니다. 아래 그림에서 `Inverted Page Table`이 어떻게 동작하는지 확인할 수 있습니다.

<img width="422" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/814d6441-2699-4fd7-bffb-9897f5393e0c">

## Summary

이번 시간에는 페이지 테이블의 크기를 줄일 수 있는 여러 방법을 살펴봤습니다. 하지만 트레이드 오프 관계에 놓여있으므로 물리 메모리의 상황을 고려해 적절한 페이지 테이블을 사용하는 것이 중요합니다. 





