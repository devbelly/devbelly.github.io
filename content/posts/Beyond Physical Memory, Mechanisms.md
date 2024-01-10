---
title: Beyond Physical Memory, Mechanisms
date: 2024-01-09T11:25:33+09:00
draft: false
comments: true
toc: true
tags:
  - os
---
지금까지는 물리 메모리에 address space를 담기 충분하다고 가정했습니다. 하지만 32비트 Windows에서 사용 가능한 address space는 $2^{32}$바이트입니다. 프로세스마다 4GB를 저장하려면 물리 메모리가 턱 없이 부족합니다. 

<img width="448" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/b0224f81-3d64-4f0c-9e87-f54c5001519e">

즉, 메인 메모리를 지원하기 위한 추가 계층이 필요합니다. 모든 페이지를 물리 메모리에 저장하는 대신 운영체제는 현재 큰 수요가 없는 주소 공간의 일부를 저장할 다른 공간이 필요합니다. 현대 시스템에서는 하드 디스크가 이 역할을 수행합니다.

왜 하나의 프로세스마다 큰 주소 공간을 지원하려고 할까요? 과거에는 `memory overlay`기술을 사용해 메모리를 관리했습니다. 이 기술은 프로그래머가 필요할 때마다 코드나 데이터 조각을 일일이 교체해야했습니다. 프로그래머가 로직 작성에 집중하기 어려웠기 때문에 큰 주소 공간을 지원함으로써 필요할 때마다 메모리 공간을 사용하는 방식으로 바뀌었습니다.

## Swap space

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/109f4447-06fd-4752-88f4-e7ba4ad38328">

디스크에 페이지를 저장하기 위한 공간을 `swap space`이라 합니다. 왜냐하면 메모리에서 페이지를 이곳으로 `swap out`하고 이곳에서 메모리로 페이지를 `swap in`하기 때문입니다. 따라서 운영체제는 `swap space`에서 페이지 단위로 읽고 쓰기 때문에 주어진 페이지의 디스크 주소를 기억해야합니다.

위 그림에서 세 개의 프로세스가 물리 메모리에 올라와 있습니다. 실행중인 프로세스의 모든 주소 공간이 물리 메모리에 올라와 있는 대신 일부만을 갖고 있으며 `Process 3`은 모든 페이지가 디스크로 `swap out`된 상태이므로 현재는 실행되지 않습니다. 

## The Present bit

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/6e8d858c-82a6-47be-92cc-3f62193a71da">

address space에서 물리 주소를 얻는 과정에 대해 생각해봅시다. 가상 주소에서 하드웨어는 VPN을 추출하고 TLB에 정보가 없다면 `PTBR`과 `VPN`을 사용해 PTE를 가져옵니다. 이제는 물리 메모리에 항상 원하는 페이지가 존재하지 않는 사실을 알기 때문에 PFN을 가져오기 전 원하는 페이지가 메모리에 있는지 확인해야합니다. 확인하는 방법은 `Present bit`를 확인하면 됩니다. PTE의 `Present bit`가 1이라면 물리 메모리에 존재하지만 0이라면 `swap space`에 존재합니다. 

## The Page Fault

`hardware-managed TLB` 이든 `software-managed TLB` 이든 상관없이 원하는 페이지가 메모리에 없다면 OS가 `page fault`를 처리합니다. OS는 페이지의 PFN을 사용해 디스크 주소를 찾고 페이지를 메모리로 가져오기 위해 디스크에 요청을 보냅니다. 

디스크 I/O가 완료되면 OS는 `PTE.Present`와 `PTE.PFN`을 업데이트 한 후 명령을 다시 시도합니다. 다시 시도할때도 TLB Miss가 발생하고 TLB를 업데이트합니다. 마지막으로 다시 시도할때는 TLB에 원하는 엔트리가 존재하기 때문에 올바르게 물리 주소로 변환할 수 있습니다. 구현에 따라 `page fault`가 발생할 때 TLB를 업데이트 하는 방식 또한 가능합니다.

이번 챕터에서는 물리 메모리 공간이 여유로워서 원하는 페이지를 디스크에서 쉽게 가져왔지만 만약에 물리 메모리에 공간이 없다면 어떤 페이지를 `swap space`로 이동시켜야 할까요? 나중에 페이지 교체 정책에 대해서도 살펴보겠습니다.

## Page Fault Control Flow

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/2d473b1b-7d61-436d-9031-a0a4c0e800ae">

그림과 함께 살펴보도록 하겠습니다. address space의 크기가 64바이트이고 프로세스에서 `LD A 10 0010`명령을 실행했습니다. 

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/3d713249-87c8-4ab3-b441-36abfee6a867">

VPN이 `10`인 것을 확인하고 MMU에 주소 변환을 맡깁니다. MMU에서 TLB Miss가 발생한다면 직접 페이지 테이블 주소에 접근해 PTE를 확인합니다. `PTE.Present`가 0이므로 디스크에서 원하는 페이지를 가져와야합니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/e85981e0-eee3-4356-a29c-d5824fca86db">

물리 메모리에서 비어있는 공간이 있는지 확인하고 핸들러는 페이지를 `swap space`에서 읽어 오기 위한 I/O 요청을 실행합니다. 이때 프로세스는 `Blocked`상태이고 굉장히 느린 I/O 요청이 완료되면 다시 `Ready`상태가 됩니다. 이제 물리 메모리에 존재하는 사실을 알기 때문에 OS는 PTE를 업데이트하고 명령을 재시도합니다. TLB Miss가 다시 발생하고 TLB Entry를 업데이트하면 마지막 재시도에서 TLB Hit이 되므로 결국 디스크에 있는 페이지 정보를 읽게 됩니다.

## When Replacements Really Occur

물리 메모리에 공간이 없다면 페이지 교체 알고리즘을 수행해 페이지를 내쫓는 것처럼 설명했습니다. 하지만 OS는 효율성을 위해 메모리의 일부를 적극적으로 비워 놓습니다. 운영체제는 `High Watermark`랑 `Low Watermark`라는 기준을 통해 언제 페이지를 추방하기 시작할지 결정합니다. 동작 방식은 다음과 같습니다. OS가 사용 가능한 페이지가 LW 미만인 것을 감지하면 메모리 해제를 담당하는 스레드를 실행합니다. 스레드는 HW 페이지만큼 사용 가능 할때까지 페이지를 추방합니다. 이러한 역할을 하는 스레드를 `page daemon` 또는 `swap daemon`이라 합니다.

또한 여러 페이지 교체를 한꺼번에 수행해서 성능 최적화를 할 수 있습니다. 여러 페이지를 클러스터링하면 한번에 스왑 파티션에 기록하기 때문에 디스크 I/O의 효율성을 높일 수 있기 때문입니다. 
## Summary

이번 시간에는 물리 메모리보다 더 많은 메모리에 접근하는 방법을 살펴봤습니다. 이를 위해서 PTE에 추가 정보가 필요하고 만약 페이지가 메모리에 없다면 운영체제의 도움을 받아서 원하는 페이지를 물리 메모리에 가져올 수 있었습니다. 

