---
title: Free Space Management
date: 2023-12-27T19:00:51+09:00
draft: false
comments: true
toc: true
tags:
  - os
---
메모리의 빈 공간 관리는 세그멘테이션처럼 가변 크기로 메모리를 할당하는 방법을 사용할 때 더욱 까다롭습니다. 이번 시간에는 C언어에서 제공하는 메모리 할당 라이브러리를 통해 어떻게 메모리를 관리하는지 살펴보도록 하겠습니다. 

## Low-level Mechanism

메모리 관리에 대해 살펴보기 전에, 대부분의 allocator에서 사용되는 메커니즘에 대해 알아보겠습니다. 
- Splitting and Coalescing (분할과 병합)
- Tracking The Size of Allocated Region
- Embedding A Free List

### Splitting and Coalescing

<img width="350" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/ae44af28-6bac-4266-a5f6-df76c72ef306">

다음과 같이 30바이트의 힙이 있고 현재 10-19바이트가 사용중이라 가정하겠습니다. External Fragmentation이 발생한 상황으로 비어있는 공간의 총 합은 20바이트지만 10바이트가 넘는 메모리 요청이 온다면 제대로 처리하지 못합니다.

<img width="346" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c24daaca-6559-4a31-bb8b-ba80cc0b3036">

이를 free list로 표현한 모습입니다. 10바이트가 넘는 요청 대신 10바이트 보다 적은 요청이 온다면 어떻게 될까요? 단일 바이트를 요청해보겠습니다. 이 경우 allocator는 splitting을 수행합니다. splitting은 요청을 만족할 수 있는 청크를 찾아서 둘로 나눕니다. 단일 바이트 요청을 만족할 수 있는 청크는 첫 번째, 두 번째 모두 가능하고 만약 두 번째 청크를 선택했다면 splitting 결과는 다음과 같습니다.

<img width="348" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/73011e6c-7c6b-46dc-9f37-8997835c1073">

두 번째 청크를 1과 9 덩어리로 나눈 후 첫번째 덩어리는 `malloc`을 호출한 클라이언트에게 주고 나머지는 free list에 추가한 것입니다. 분할은 특정 빈 청크의 크기보다 작은 크기의 요청이 올 때 allocator에서 흔히 사용하는 방법입니다. 다시 첫 번째 그림으로 돌아와서 사용자가 `used` 공간을 반환하면 어떻게 될까요? 간단하게 해당 공간 만큼 free list에 추가할 수 있습니다.

<img width="426" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/16737bc9-d018-4683-89c5-9d9328d1d547">

head의 맨 앞 부분에 해제한 공간을 추가하고 head를 맨 앞으로 변경했습니다. 하지만  20바이트 크기의 메모리 공간을 요청한다면 free list를 순회하더라도 적절한 공간을 찾지 못하고 실패하게 됩니다. 이 문제를 해결하기 위해 반환되는 공간이 비어있는 청크 옆에 존재하는 공간이라면 하나로 합치는 과정을 수행합니다. 이를 Coalescing, 병합이라 합니다.

<img width="254" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/d1b3e49a-e847-42f3-a1cb-5f32c70b49e1">

### Tracking The Size of Allocated Region

`malloc()`과 `free()`를 통해 힙 영역의 공간을 할당하거나 해제할 수 있습니다. 함수 정의를 살펴보면 `void *malloc(size_t size)`입니다. 할당 받고자 하는 크기만큼 파라미터를 넘기면 할당 받은 힙의 시작 주소를 리턴받게 되지요. `void free(void *ptr)`은 `malloc()`을 통해 반환 받은 주소”만” 파라미터로 넘기면 자동으로 할당받은 크기의 메모리를 해제하게 됩니다. 이를 통해 포인터만 전달 받더라도 `free()`는 해당 메모리 청크의 크기를 파악할 수 있다는 것을 알 수 있습니다.

<img width="504" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c4d589f8-0636-47f0-a1cf-96d57a615076">

어떻게 시작 주소의 위치만으로 크기를 파악할 수 있을까요? 이는 allocator가 할당한 메모리 청크 바로 앞에 헤더 블록을 활용하기 때문에 가능합니다. `malloc(20)`을 호출했다면 사용자는 `ptr`를 리턴 받게 됩니다. 하지만 그 앞에 헤더 블록을 통해 `free(ptr)`을 하더라도 얼만큼의 메모리를 할당했는지 추적할 수 있습니다. 위 그림에서는 헤더 블록의 간단한 요소를 보여주기 위해 `size`와 `magic` 을 사용했습니다. 여기서 magic은 integrity check시 사용되는 요소입니다.

### Embedding a Free List

free list는 실제로 어떤 식으로 존재하는지 살펴보겠습니다. 

<img width="486" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/66ee8789-d37d-4119-ba11-23c4e745d8e4">

이번 예제에서는 4KB 공간을 `mmap()` syscall로 할당받고 그 안에 free list를 관리하기 위한 노드를 넣은 모습입니다. 4096Byte를 할당받았지만 노드의 크기로 8바이트를 사용하므로 실제 할당가능한 크기는 4088임을 알 수 있습니다.

<img width="478" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/7daf9a0f-a122-4e26-8182-5ab25e7bc12c">

이제 100바이트를 요청해보겠습니다. 요청을 처리할 수 있는 청크는 하나이고 splitting을 통해 100+8(헤더)바이트와 나머지 공간으로 나뉘게 됩니다. `malloc()`을 사용한 클라이언트에는 `ptr`을 반환합니다. 남아있는 청크의 크기는 `4088-108 = 3980` 임을 알 수 있습니다. 

<img width="740" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/fd8a925c-1f18-44f7-927c-011d72b4f57a">

이제 100바이트씩 세 번 할당한 경우를 살펴보겠습니다. 4088바이트에서 108 x 3 바이트를 빼면 비어있는 청크의 크기가 3764바이트 임을 알 수 있습니다. 이 때 할당된 메모리 중 중간 영역을 해제하면 free list의 헤드가 가장 최근에 `free()`를 호출한 16500로 변경되고 해당 node의 next 값은 이전 헤드의 위치로 업데이트 됩니다.

## Basic Strategies

이제 free list에서 청크를 선택하는 전략에 대해 살펴보겠습니다. allocator는 빠르게 작동하면서도 파편화를 최소화 해야합니다. 하지만 할당 및 해제는 임의로 수행되므로 모든 상황을 커버할 수 있는Best Strategy를 찾기는 어렵습니다.
### Best Fit

free list의 청크 중 요청한 크기과 가장 비슷한 청크를 선택하는 전략입니다. 하지만 naive한 구현에서는 exhaustive search를 해야하고 만약 요청한 크기와 비어있는 청크간 차이가 조금이라도 있다면 수행을 반복할수록 external fragmentation이 점점 심해집니다.

### Worst Fit

Best Fit의 정반대 전략입니다. 선택한 청크 중 남아있는 공간의 크기가 가장 크게끔 청크를 선택하는 전략입니다. 나중에 다른 요청이 오더라도 비어있는 공간의 크기가 크다면 메모리를 적절하게 할당할 확률이 높다 라는 아이디어입니다. 하지만 여전히 Best Fit에서 발생하는 문제점이 동일하게 발생합니다. 

### First Fit

이전 두 방법들은 적절한 청크를 찾기 위해 exhaustive search를 해야하므로 효율성이 떨어집니다. 따라서 First Fit은 free list의 앞에서 부터 청크를 검사하다가 할당이 가능한 빈 청크를 찾는다면 search를 멈추고 해당 청크를 통해 메모리를 할당하는 전략입니다. 

하지만 이 방법은 free list의 앞 부분에 external fragmentation이 심해집니다. 따라서 이 문제를 완화하기 위해 free list를 관리할 때 address-based ordering을 사용합니다. allocator은 Coalescing을 할 때 주변 노드를 살펴보는데 만약 address-based ordering이 되어있다면 Coalescing할 기회가 더욱 많아지므로 fragmentation이 줄어듭니다.

### Next Fit

First Fit에서 external fragmentation이 앞 부분에 몰려있는 문제를 해결하기 위해 free list를 검색할 때 항상 맨 처음부터 검색하는 대신 마지막으로 검색한 위치에서부터 비어있는 청크를 검색합니다. 이 아이디어를 통해 external fragmentation을 전체적으로 분산 시킬 수 있습니다. 

## Other Approaches

위에서 설명한 기본적인 방법을 넘어서서, 메모리 할당을 개선하기 위해 다양한 알고리즘이 제안되었습니다.

### Segregated Lists

응용 프로그램이 특정 크기의 메모리 요청을 자주 수행한다면 `sepearate list`를 따로 두어 해당 크기를 관리하는 방법입니다. 이러한 접근 방식의 이점은 무엇일까요? 특정 크기의 요청을 처리하기 위해 메모리의 일부를 할당함으로써 단편화를 훨씬 덜 신경쓰고 할당 및 해제 요청을 더욱 빠르게 수행할 수 있습니다.

이렇게 자주 사용되는 메모리 크기를 할당하는 방법 중 slab 할당자 라는 것이 있습니다. 이 할당자는 커널이 부팅될 때 자주 요청되는 메모리 크기를 객체 캐시로 할당합니다. 그래서 해당 크기만큼 요청이 발생하면 바로 메모리를 할당할 수 있습니다. 

### Buddy Allocation

<img width="404" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/5d6d28f2-9866-4085-b658-cc1873965017">

allocator에 있어서 중요한 요소는 coalescing입니다. buddy allocator은 free memory가 $2^N$ 크기의 공간을 가정하며 작동합니다. 만약 메모리 요청이 발생하면 자유 공간을 재귀적으로 두 배로 나누어 요청을 수용할 충분히 큰 블록을 찾습니다. 

예를 들어 64KB공간에서 7KB 메모리를 요청했다면 가장 왼쪽의 8KB를 사용자에게 반환합니다. 고정된 크기($2^N$)을 사용하므로 할당한 공간 중 사용하지 않는 internal fragmentation 문제가 발생합니다. 

사용한 블록을 해제할 때 `Buddy 8KB`가 free 상태인지 확인합니다. 만약 그렇다면 두 블록을 16KB로 병합합니다. 마찬가지로 `Buddy 16KB`가 free 인지 확인하고 그렇다면 병합합니다. 이 과정을 재귀적으로 `Buddy`가 사용중일 때까지 수행합니다. 

## Summary

이번 글에서는 메모리 할당자가 어떻게 메모리를 할당하고 해제하는지 살펴봤습니다. 다음 글에서는 가변 크기의 할당방법이 아닌 고정 크기 할당 방법인 paging에 대해 살펴보겠습니다.