---
title: Address Translation, and Caching
date: 2024-01-23T11:39:38+09:00
draft: false
comments: true
toc: true
tags:
  - os
---
## Principle of Locality

사용자가 프로세스 주소 공간에 접근할 때, 모든 영역을 균등하게 접근하지 않습니다. 예를 들어 프로그램 내에 반복문이 있다면 주소 공간 중 코드영역에 많이 접근하고 배열과 같은 순차적인 자료구조가 있다면 그 근처에 있는 다른 메모리에도 접근합니다. 

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/d7b3ea5f-0a9c-4fee-a17a-07b82012ea4f">

이러한 캐시의 특성을 `locality`라고 합니다. 주소 공간의 특정 정보에 접근했을 때, 얼마 지나지 않아 동일한 데이터에 접근하는 것을 `temporal locality`, 접근한 데이터 근처 데이터에도 접근하는 것을 `spatial locality`라 합니다. 

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/44aac868-a0a3-40c7-8b73-c1748d0773c1">

하드웨어적인 관점에서 다시 살펴보면 `temporal locality` 로 인해 최근에 접근한 데이터일수록 CPU에 가깝습니다. `spatial locality`는 데이터를 접근하기 위해 점점 CPU에 가깝게 옮기는데 1byte가 아닌 여러 블록단위로 옮기는 것을 알 수 있습니다. `하드디스크 → Lower Leve Memory`에서는 페이지 단위로 옮기고 `Lower Level Memory → Upper Level memory`에서는 cache line단위로 옮깁니다. 

## Structure of cache

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/55cae39f-e912-43fe-89b1-371143cafa8c">

메모리는 byte 단위로 접근하고 캐시는 block 단위로 접근합니다. block은 cache line이라고도 하며 32byte 또는 64byte 크기를 갖습니다.

<img width="631" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/7df57c86-4e08-4ecd-9767-6b628b244edd">

만약 캐시안에 데이터가 없다면 메모리에서 데이터를 찾아서 가져와야합니다. 이때 필요한 1byte만 메모리에서 가져오는 것이 아니라 연속된 메모리 주소 32byte의 데이터를 가져와 캐시에 저장합니다. 때문에 `spatial locality`특성을 활용해 캐시를 효율적을 사용할 수 있습니다. 

캐시는 물리 메모리의 일부 내용을 빠르게 접근하기 위한 장치입니다. 캐시를 활용하기 위해서는 물리 메모리의 주소와 해당 주소에 담겨있는 데이터(cache line의 크기만큼)를 모두 알고 있어야합니다. 하지만 캐시는 비싼 자원이므로 32bit에 해당하는 주소 + 데이터를 모두 저장하기에는 부담스러우므로 효율적으로 위 정보들을 저장해야합니다.

## Index

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/50febb96-ab0b-4477-9ba4-26eb3d519f4a">

캐시 메모리도 데이터가 모여있으므로 배열처럼 indexing을 사용할 수 있습니다. 따라서 물리 메모리 주소 32bit를 캐시 라인에 전부 저장하는 대신 Index bit는 저장하지 않고 캐시 메모리 내에서 indexing을 하는데 사용합니다. 

즉 **Indexing이란 내가 접근하는 physical memory address가 캐시 메모리의 1024개 공간 중 어디에 위치하는지 찾는 방법**입니다. 캐시 라인 갯수가 1024개라면 $2^{10}$ 이므로 10bit가 필요합니다. 따라서 물리 메모리 주소 5번째 비트부터 14번째 비트를 index bit로 사용해 해당 비트들은 저장하지 않음으로써 캐시 메모리 공간을 절약할 수 있습니다.

## Block offset

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/5404b0b5-6f94-40f5-8b3a-b45780a23528">

우리가 원하는 정보를 사용하기 위해서 결국 byte단위로 접근해야합니다. 캐시라인에서  32byte 중 원하는 정보가 몇번째 byte인지 알기 위해 `block offset`을 사용합니다. 32개를 표현하기 위해 필요한 비트수는 5개 이므로 물리 메모리의 0번째부터 4번째까지 `block offset`으로 사용합니다.

## Tag

<img width="534" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/7acfc66b-e4d6-48d5-bc01-eeb8b0afbb02">

마지막으로 남은 비트들은 태그로 사용합니다. Index bit와 block offset을 사용하면 캐시 내에서는 유니크하게 원하는 정보를 가져올 수 있습니다. 하지만 해당 공간을 가리키는 물리 메모리 주소는 많습니다. 

따라서 물리 메모리의 여러 공간이 동일한 캐시라인을 가리킬 수 있으므로 현재 보고 있는 캐시라인이 내가 원하는 캐시라인이 맞는지 확인해야합니다. 이를 위해서 나머지 비트를 tag bit로 사용합니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/ea2c0fa2-c16b-4049-b75d-882cd5a20005">

다음과 같은 과정을 통해 정리해보겠습니다.

1. Cache Index를 통해 캐시 안에서 원하는 캐시라인을 찾는다.
2. 물리 메모리의 주소 중 Cache Tag에 해당하는 부분과 캐시에 저장되어있는 Cache Tag를 비교한다. 
3. 만약 태그가 같다면 캐시 라인의 valid bit를 확인한다. valid 하다면 cache hit이므로 block offset을 사용해 원하는 정보를 캐시에서 가져온다.

위 방식을 `Direct Mappepd Cache`라고 합니다. `Cache Index`가 가리키는 캐시 공간이 한 군데인 방식입니다. 만약 접근해야하는 물리 메모리 주소 목록이 있을 때 `Cache Index`가 동일한 주소들이 반복적으로 등장하면 캐시의 한 공간을 놓고 경쟁하게 됩니다. 결과적으로 `Cache miss`가 자주 발생하므로 효율적이지 않습니다.

## Set Associative Cache

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/053db6fc-b737-44be-b67d-4178c82dafee">

`Cache index`가 동일한 캐시라인을 가리키더라도 conflict가 발생하지 않도록 두 군데를 가리키는 방식입니다. 위 방식을 `2 way set associative cache`라 합니다. cache index 비트를 몇개 사용할지 조절함으로써 `4 way associatvie cache`, `fully associative cache`로 사용가능합니다.

## 정리

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/e9aac664-271f-492b-a40c-6d9852ddd5cb">

정리하자면 오늘 배운 캐시는 물리 메모리에 존재하는 내용을 빠르게 접근하는 하드웨어입니다. 이전에 배웠던 TLB도 캐시의 일종으로, 주소변환정보를 빠르게 얻기 위해 사용하는 캐시였습니다. 캐시와 마찬가지로 TLB hit, TLB miss라는 표현을 사용했던 것을 떠올리면 쉽게 알 수 있습니다. 참고로 TLB는 `fully associative cache`를 사용합니다.