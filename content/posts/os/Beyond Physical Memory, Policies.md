---
title: Beyond Physical Memory, Policies
date: 2024-04-30T10:01:47+09:00
draft: false
comments: true
toc: true
tags:
  - os
---
지난 시간에는 메인 메모리의 부족한 공간을 극복하기 위해 하드디스크를 추가적인 메모리처럼 사용하는 Demand Paging에 대해 알아봤습니다. 하드디스크에 있는 내용을 메인 메모리에 읽어야할 때, 메인 메모리에 비어있는 프레임이 있다면 해당 공간에 페이지를 가져와서 저장하면 됩니다. 

문제는 새로운 페이지를 읽어와야할 때 메인 메모리에 여유 공간이 없는 경우입니다. OS는 어떤 페이지를 제거할지 결정해야하는데 이를 페이지 교체 정책이라고 합니다. 

## Cache Management

메인 메모리를 캐시처럼 바라볼 때 `cache miss`가 적게 발생해야합니다. 즉, 디스크에서 페이지를 가져와야하는 횟수가 적을수록 좋습니다. `cache hit` 또는 `cache miss`가 `AMAT(Average Memory Access Time)`에 어떤 영향을 미치는지부터 살펴보겠습니다.

<img width="436" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/27bf19a5-a5ec-4dc7-bbe3-6589a1c1c53a">

$T_M$ 은 메인 메모리에 접근하기 위해 걸리는 시간, $T_D$는 하드디스크에 접근하기 위해 걸리는 시간입니다. 각각 100ns, 10ms이라고 가정하겠습니다. 만약 $P_{Miss}$ 가 10%라면 AMAT는 1ms이고 $P_{Miss}$가 1%라면 AMAT는 100us입니다. 최신 시스템에서는 디스크 엑세스 비용이 굉장히 높기 때문에 `cache miss`의 미세한 차이에도 AMAT가 극명하게 달라집니다. 

## The Optimal Replacement Policy

최적의 교체 정책은 미래를 예측하는 방법입니다. 가장 나중에 사용될 페이지를 선택해서 버립니다. `cache miss`가 가장 적게 발생하지만 미래를 예측할 수가 없으므로 구현이 불가합니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/37d8991b-dcdd-4f5c-8596-99eebf592b78">

위 그림은 페이지 엑세스 패턴에 따른 Hit, Miss 여부를 나타낸 표입니다. 처음에 메인메모리에는 어떠한 페이지도 없다고 가정하면 `page fault`가 발생합니다. 이러한 종류의 Miss는 피할 수 없으므로 `cold miss` 또는 `compulsory miss`라고 합니다. 

<img width="553" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/4f87defe-00ad-4942-8e85-2118e335627d">

Access 3에서 메모리에 빈 공간이 없어 페이지 중 하나를 제거해야합니다. 미래에 접근할 Access 패턴을 확인 한 후 가장 나중에 접근하는 VPN은 2이므로 해당 VPN을 교체하게 됩니다. 이 교체 정책에서는 Miss가 5번 발생하고 이를 기준으로 다른 정책들은 Miss가 몇 번 발생하는지 알아봅시다.

## A Simple Policy : FIFO

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/291850c8-20f3-4302-bccb-3e31c5a8095e">

초기의 많은 시스템에서 최적화에 대한 복잡성을 피하고자 사용했던 FIFO입니다. 0, 1, 2 페이지에서 `cold miss`가 발생한 후 3번 페이지에 접근하면 메모리에는 원하는 페이지가 올라와있지 않은 상황입니다. FIFO 정책에 따라 가장 오래된 0번 페이지를 제거하고 3번 페이지를 메모리에 올립니다. 이렇게 끝까지 진행하면 Miss가 총 7번 발생하게 됩니다. 

FIFO는 구현하기 간단하지만 Locality를 고려하지 않는 문제가 있습니다. 또한 메모리를 크기를 늘려도  성능이 오히려 안좋아지는 `Belady's anomaly` 문제가 발생합니다. 이 문제는 뒤에서 다시 살펴보겠습니다.

## Using History: LRU

FIFO나 Random 정책은 Locality를 고려하지 않아 다시 참조될 중요한 페이지를 내보낼 수 있는 문제가 있습니다. 프로그래밍은 반복과 배열을 사용하므로 Locality 특징이 중요합니다. 따라서 미래를 예측할 수 없으면 과거에 기대어 성능을 개선해봅시다. 만약 과거에 어떤 페이지에 접근했다면 이는 미래에도 접근할 확률이 높기 때문입니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/f24ede51-d96e-48d5-a8b3-4d56cfab10d9">

이를 알고리즘으로 구현한 정책이 LRU(Least Recently Used)입니다. 메모리에 페이지가 올라온 이후 0번 페이지에 접근하면 메모리에 존재하므로 Hit 입니다. 뿐만 아니라 최근에 접근했으므로 LRU에서 MRU(Most Recently Used)로 위치가 변경됩니다.

<img width="570" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/348bc80d-00cd-47db-ba38-b0ebfc351a3d">

만약 메모리에 원하는 페이지가 존재하지 않는다면 LRU로 판단되는 2번 페이지를 제거하고 새로운 3번 페이지를 하드디스크로부터 읽어옵니다.

## Belady’s anomaly

캐시의 성능을 올리기 위해서는 캐시의 용량을 늘리는 방법이 있습니다. 일반적으로는 용량이 늘어나면 성능이 좋아질 것으로 기대할 수 있지만 FIFO 정책의 경우 그렇지 않은 현상이 있습니다. 이러한 현상을 `Belady's anomaly`라고 합니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/ec8f6059-306d-4700-8593-4d300f17472a">

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/3dcab27e-25fe-49d3-9206-5767eea9e124">

위 그림은 동일한 엑세스 패턴에 대해 캐시 크기를 3에서 4로 증가했을 때 `hitrate`를 나타냈습니다. 캐시 크기를 늘렸지만 오히려 `hitrate`는 감소하는 것을 알 수 있습니다. `OPT`나 `LRU`에서는 이러한 현상이 일어나지 않는데 그 이유는 두 정책은 `stack property`를 만족하기 때문입니다. 즉, `N+1`크기의 캐시는 `N`크기의 캐시가 가지고 있는 내용을 전부 포함하는 속성입니다. 하지만 `FIFO`나 `RANDOM`은 이러한 속성을 갖고 있지 않아 캐시 크기가 늘어나더라도 오히려 성능이 저하될 수 있습니다.

## Workload Example

<img width="472" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/9d063600-2d79-41fc-a078-33b390013313">

다양한 Workload 사례를 통해 각 정책을 살펴보겠습니다. 첫 번째 예시는 다소 비현실적인 `No-Locality Workload`입니다. 100개의 유니크한 페이지를 10000번 엑세스할 때, 캐시 크기에 따른 `hit rate`를 나타낸 그림입니다. 이 경우는 미래를 예측해 효율적으로 페이지를 선택할 수 있는 `OPT`를 제외하고는 캐시 크기에 단순히 정비례하는 성능을 관찰할 수 있습니다.

<img width="438" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/1d5d4423-92c5-4e74-8f47-d9fce61f48f2">

다음으로 살펴볼 워크로드는 `80-20 Workload`입니다. 80%의 참조가 20%의 페이지(Hot page)에서 발생하고 20%의 참조가 나머지 80%의 페이지에서 발생하는 워크로드입니다. 그림에서 볼 수 있듯이 `FIFO`나 `RAND`보다 `LRU`가 더 성능이 좋은 것을 확인할 수 있는데, 이는 과거에 자주 참조된 페이지가 `HOT PAGE`여서 미래에도 자주 참조될 가능성이 높기 때문입니다.

<img width="456" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c61989ef-4ce6-4382-9580-efaeb5d06fb5">

마지막으로 살펴볼 워크로드는 `Looping-Sequential Workload`입니다. 데이터베이스 뿐만 아니라 다른 상용 애플리케이션에도 이러한 패턴은 흔히 볼 수 있습니다. 0페이지에서 시작하여 49페이지까지 순차적으로 50페이지를 참조한 다음 다시 이러한 액세스를 반복합니다. 이 워크로드에서 `LRU`와 `FIFO`가 50페이지를 전부 담지 못하는 캐시 크기일때는 성능이 0%임을 알 수 있습니다. 독특한 점은 `RAND`가 어느정도 성능을 보인다는 것입니다. 이를 통해 `RAND`는 엣지 케이스가 존재하지 않고 어떠한 상황에서도 일정한 성능을 낼 수 있는 장점을 관찰할 수 있습니다.

## Implementing Historical Algorithms

위 내용을 살펴보면 적절한 캐시 크기가 보장됐을 때 `LRU`를 사용하는 건 좋은 선택이라고 생각할 수 있습니다. 하지만 과거를 기록하는 정책은 효율적으로 구현하기 위한 문제에 부딪히게 됩니다. 

<img width="250" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/8c125ebb-68f6-4a38-b838-caa5eb2b1c2f">

LRU는 일반적으로 `linked list`를 이용해 구현합니다. 4번 페이지에 접근했다면 다시 `MRU`위치로 옮겨주는 작업을 해야합니다. 소프트웨어 레벨에서는 이러한 자료구조를 구현하고 유지하는 건 어렵지 않지만 CPU가 1초에 수백만 개의 명령을 처리하면서 메모리 엑세스가 일어나는데 그때마다 자료구조를 업데이트 하는 건 결코 효율적이지 않습니다.

이를 해결할 수 있는 한가지 방법은 하드웨어의 도움을 받아 메모리에 엑세스할 때마다 타임스탬프를 페이지마다 기록하는 것입니다. 나중에 `page fault`가 발생했을 때 운영체제는 타임스탬프가 가장 빠른 것을 교체 대상으로 삼으면 `LRU`를 구현할 수 있습니다. 하지만 4GB의 메모리를 4KB 페이지로 나눈 시스템을 생각해보면 백만  개의 페이지 프레임이 존재합니다. 즉, 페이지를 교체할 때마다 백만 개의 페이지 프레임을 스캔해야하는 일이 발생합니다. 

## Approxiamting LRU

완벽한 `LRU`를 사용하는 대신 이를 근사하여 성능을 올릴 수 있는 방법이 있을까요? 즉, `oldest page`를 사용하는 대신 `old page`를 사용해서 교체하면 안될까요? 실제로 계산 오버헤드 관점에서 `LRU`를 근사하는 것이 더 좋고 실제로 많은 최신 시스템에서 이런 방식으로 구현하고 있습니다. 

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/2756a869-16ba-4a52-9d90-e56e374c2567">

완벽한 `LRU`를 근사하기 위해 추가적으로 하드웨어의 도움이 필요합니다. OS는 `use bit` 또는 `reference bit`를 사용해 `old page`를 추적합니다. 우리가 사용하는 메모리가 원형으로 배치되어있다고 상상해봅시다. 시계 바늘은 특정 페이지를 가리키고 해당 페이지가 사용된 적이 있는지 `use bit`를 통해 확인합니다. 만약 최근에 사용한 페이지라면 `use bit`가 1로 되어있고 교체 대상이 되어선 안됩니다. 따라서 그대로 지나가되, OS의 도움을 받아 `use bit`를 0으로 바꿔줍시다. 계속해서 `use bit`가 0일 때까지 시계 바늘을 이동하며 만약 `use bit`가 0인 페이지를 찾았다면 해당 페이지를 교체합니다.

이러한 알고리즘을 `clock algorithm`이라 합니다. `LRU`를 근사화 하는 방법 중 하나이며 페이지 교체를 위해 모든 메모리를 반복적으로 스캔하지 않아도 되는 장점이 있습니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/3f25aea4-cc70-4d29-844c-c47a93283dfd">

유사한 방법으로 `N'th chance algorithm`도 있습니다. 이 방법은 `use bit`가 0이면 바로 교체하는 것이 아니라 비록 `use bit`가 0 이더라도 `N`번 기회를 주는 방법입니다. `N`이 클수록 `LRU`에 근접할 수 있지만 `LRU`가 가지는 단점 또한 갖게 됩니다.

## Considering Dirty Pages

교체할 때 추가적으로 `dirty bit`를 확인하는 방법도 있습니다. `use bit`가 0일 때 해당 페이지 위를 덮어쓸 수 있지만 만약 해당 페이지가 `store`와 같은 명령으로 변경사항이 있다면 이를 디스크에 다시 반영해야합니다. 디스크에 반영하는 과정은 시간이 오래 걸리므로 교체할 페이지를 찾을 때 `dirty bit == 0 && use bit == 0`인 페이지를 우선적으로 찾는 전략을 취할 수 있습니다.  만약 그러한 페이지가 없다면 시간이 걸리더라도 디스크에 페이지 변경사항을 반영해야하는 `dirty bit == 1 && use bit == 0`인 페이지를 찾을 수도 있습니다.

## Thrashing

<img width="578" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/4704836f-99c6-4c25-8944-b19002aca9b1">

마지막으로, 실행중인 프로세스 집합의 메모리 요구량이 물리 메모리 크기를 초과한다면 시스템 전체적으로 하는 일은 `context-switching`입니다. `page fault`가 계속 발생하면서 결국 CPU의 사용률은 줄어들게 되는데 이를 `Thrasing`이라 합니다.

<img width="482" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/65376464-16f4-4564-b3fb-79ec15f4a78a">

이 문제를 해결하기 위해 각 프로세스마다 `Working set`을 구합니다. `Working set`은 특정 시간마다 어떤 `page`에 접근한지 기록한 집합을 의미합니다.  

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c76a7398-ae5c-4609-aaf2-20fd6ee33d1c">

매 순간마다 각 프로세스가 사용하는 `Working set`만을 메모리에 올려서 사용하므로 보다 많은 프로세스를 스케줄링하면서 `page fault`의 빈도를 낮춰 `thrashing`을 방지할 수 있습니다. 단, `Working set`의 크기 총합이 물리 메모리의 크기를 넘지 않도록 조절해야합니다.