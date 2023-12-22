---
title: Scheduling, Proportional Share
date: 2023-09-14T08:45:59+09:00
draft: false
comments: true
toc: true
tags:
  - os
---
안녕하세요. 이번시간에는 `proportional-share` 스케줄러에 대해 알아보겠습니다. 지금까지 살펴본 스케줄러들은 Performance에 중점을 두었습니다. `proportional-share` 스케줄러는 반환 시간이나 응답 시간을 최적화하는 대신 Fairness에 초점을 맞춰 각 작업이 일정한 비율로 CPU를 획득 하는 것을 목표로 합니다.
## Basic Concept: Tickets Represent your share

일정한 비율로 CPU를 획득하기 위한 기본적인 아이디어는 프로세스마다 티켓을 주는 것입니다. 매번 추첨을 통해 다음에 실행할 프로세스를 결정하고 더 자주 실행해야하는 프로세스에는 티켓을 더 나눠주면 됩니다. 티켓을 통해 당첨여부를 판단해 이를 `lottery scheduling`이라 부릅니다.

예를 들어 프로세스 A와 B가 75:25 비율로 실행되기를 바란다면 A에게는 75장의 티켓을 주고 B에게는 25장의 티켓을 줍니다. 스케줄러는 발행된 티켓의 총 갯수인 100개만 알고 있으면 이제 어떤 프로세스에게 CPU를 줄 지 결정할 수 있습니다.

<img width="572" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/4e8689f7-d92d-48cb-83eb-68bad66f184c">

`time-slice`마다 스케줄러는 0~99 사이의 랜덤한 숫자를 고릅니다. 만약 0~74 사이의 숫자가 나온다면 A에게 75~99 사이의 숫자가 나온다면 B에게 CPU를 할당해줍니다. 

## Ticket Mechanism

티켓 스케줄링은 여러 상황에 응용하기 위해 여러 메커니즘을 적용할 수 있습니다.
### Ticket currency

첫 번째는 티켓 통화라는 개념입니다. 각 사용자마다 가지고 있는 job이 다르고 각 job마다 얼마만큼 CPU를 할당하고 싶은지도 다릅니다. 만약 사용자 A와 사용자 B가 CPU를 50:50 비율로 사용하기 위해 동일한 티켓을 나눠 가졌습니다. 사용자 A는 A1과 A2 라는 job이 있고 사용자 B는 B1이라는 job 1개만 가지고 있다 해봅시다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/34dbdf38-050e-42ad-aafb-ed87bfd20c84">

위 사진에서는 사용자 A가 A1과 A2를 같은 시간만큼 실행하고 싶어 티켓을 500장씩 동일하게 배부했습니다. 여기서 배부한 1000장의 티켓은 사용자 A만 사용하는 단위로 이는 곧 A의 통화라고 할  수 있습니다. 예제와는 달리 490개와 510개 비율로 나눠줄 수도 있겠습니다.

사용자 B는 실행할 job이 1개 뿐이므로 10장의 티켓을 만들어 전부 B1에게 할당했습니다. 이제 위 티켓을 가지고 스케줄링이 되어야하는데 A가 사용하는 통화와 B가 사용하는 통화의 가치가 다르기 때문에 사용자가 발행한 통화를 `global currency`로 환산해야합니다. 따라서 맨 오른쪽에 보이는 A1 50장, A2 50장, B1 100장을 기준으로 스케줄링 됩니다. 

### Ticket transfer

두 번째로 사용되는 메커니즘은 ticket transfer입니다. 문자 그대로 티켓을 양도한다는 건데 주로 클라이언트와 서버 환경이 존재할때 유용합니다. 일반적으로 클라이언트가 서버에게 요청을 보내면 서버에서 필요한 연산을 해서 응답하게 됩니다. 요청을 보낸 이후 클라이언트는 서버에 응답이 올때까지 기다려야 하는데 이때 자신이 가진 티켓을 서버에게 양도하면 서버가 스케줄링이 더 많이 되어 빠른 응답을 받을 것이라 기대할 수 있습니다.

서버에서 연산이 끝나게 되면 다시 티켓을 클라이언트에게 돌려주며 이전과 동일한 상태가 됩니다.

### Ticket inflation

마지막으로는 ticket inflation입니다. 프로세스가 서로 신뢰하는 환경하에서 특정 프로세스가 어떤 이유로 CPU를 많이 필요로 하면 `global currency`를 발행합니다. 티켓이 많으므로 더 많이 스케줄링되며 처리가 끝난 이후 다시 발행했던 `global currency`를 반납합니다. 만약 프로세스간 신뢰가 없는 상황이라면 서로 더 많은 `global currency`를 발행해 더 많은 CPU를 차지하려고 하므로 인플레이션에 대한 효과를 기대하긴 어렵습니다.

## Fairness?

확률에 의존하는 lottery scheduling의 공정성은 어떻게 될까요? 동일한 실행 시간을 가진 2개의 job이 완료되기를 경쟁하는 상황을 가정해봅시다. 공정함에 대해 정량적으로 평가하기 위해 $U$를 `첫 번째 작업이 완료된 시간 / 두 번째 작업이 완료된 시간`로 정의하겠습니다.

만약 job의 실행 시간이 R, R 이라면 첫 번째 job이 먼저 스케줄링 되어 R에 끝났다면 두 번째 job은 2R에 끝나게 되고 이때 $U$는 0.5가 됩니다. 공정하다는 것은 두개의 job이 거의 동시에 끝나야하므로 원하는 것은 $U$가 1에 가장 가까울수록 공정합니다. 

<img width="366" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/4892b68e-8300-4c6e-a8cd-7fe604564052">

주사위를 많이 굴릴수록 수학적인 기댓값과 실제 눈금이 나온 횟수가 일치해지는 것처럼 lottery scheduling 역시 실행시간이 길면 우리가 원하는 Fairness에 근접하지만 짧다면 불공정성이 생기는 문제가 있습니다.

## How To Assgin Tickets?

본질적으로 돌아와서 결국 티켓을 어떻게 배분할 것인가에 대한 문제가 있습니다. 사용하는 job이 길다면 여러 티켓을 주고 싶지만 SJF, STCF와 마찬가지로 얼마나 실행될 것인지에 대해 알고 있어야 여러 티켓을 주는 행동이 가능합니다. 또한 고정적인 티켓을 주고 끝내는 것처럼 설명했지만 스케줄링 도중 새로운 job이 도착했다면 이 job에 대해서는 티켓을 얼만큼 줄지 등 여러 고려사항들이 있습니다. 

## Stride scheduling

운영체제에서는 랜덤을 통해 문제를 해결하는 경우도 종종 있습니다. 책에서도 랜덤의 장점에 대해 간략히 소개하기도 합니다.

> - 기존 알고리즘이 처리하기 어려운 엣지 케이스를 처리할 수 있다.(예시는 LRU의 worst case가 있다)
> - 구현이 간단해지고 필요한 상태가 많지 않아 가볍다
> - 랜덤 알고리즘만 빠르게 동작한다면 실행이 빠르다

lottery scheduling의 단점은 실행 시간이 짧다면 불공정성이 커지는 것 외에도 `non-deterministic`하다는 점입니다. 비결정적이면 해당 스케줄링에서 문제가 발생했을때 재연이 불가하여 디버깅이 까다롭다는 단점이 있습니다.

이러한 비결정성을 없앤 스케줄러가 Stride scheduling입니다. stride는 보폭이라는 뜻인데 티켓 수가 많은 프로세스는 짧은 보폭으로 CPU를 사용하지만 티켓 수가 적은 프로세스는 한번 CPU를 할당 받을 때 큰 보폭처럼 오래 CPU를 사용합니다.

<img width="596" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/d46e613d-6b43-49e5-af64-fed8483edd32">


예시를 통해 이해를 해볼까요? 위 사진은 프로세스 리스트를 나타냅니다. 티켓들의 공배수인 10000을 각 티켓 수로 나누면 프로세스마다 stride를 구할 수 있습니다. 각각 100, 200, 40임을 알 수 있습니다.

<img width="450" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/88c7b751-d767-4be4-9727-507fce3af6b5">

이제 스케줄러가 실행되어 각 프로세스마다 얼마만큼의 보폭을 사용했는지를 기록합니다. 현재까지 걸어온 보폭을 Pass라고 부르기도 합니다. 맨 처음에는 각 프로세스의 pass가 0 이므로 모든 프로세스가 실행이 될 수 있습니다. 큐에 A, B, C 순서로 있어 A가 먼저 스케줄링이 되었다고 해봅시다.

A의 stride가 100이므로 pass가 100만큼 증가합니다. 다음에 실행될 프로세스는 pass값이 가장 작은 B와 C중에 선택되고 B가 선택되었다면 pass가 200으로 증가합니다. 다음 스케줄링에서는 pass값이 가장 작은 C가 스케줄링되며 40이 기록됩니다. 

lottery scheduling은 확률적으로 proportional-share을 달성하지만 stride scheduling은 확정적으로 보유한 티켓에 비례하여 정확하게 CPU를 나눠서 사용하는 모습을 알 수 있습니다. 

## Problem?

stride scheduling은 lottery scheduling보다 우월하다고 할 수 있을까요? 실행 도중 새로운 프로세스 $D$가 들어온다면 문제가 발생할 수 도 있습니다. 만약 단순하게 pass값을 0으로 설정한다면 한동안은 CPU를 프로세스 $D$가 점령하는 문제가 있습니다. 적절한 pass를 설정해줘야하는데 이는 지금까지 겪었던 문제와 동일합니다. **적절한** pass란 도대체 뭘까요?

lottery scheduling에서는 단순히 프로세스 리스트를 업데이트하고 랜덤값을 결정하는 변수였던 `total_ticktes`만 업데이트해주면 시간이 지남에 따라 적절한 비율로 스케줄링이 될 것이라 기대할 수 있습니다.

## The Linux Completely Fair Scheduler(CFS)

FreeBSD 운영체제에는 MLFQ 기반으로 스케줄러를 사용하는 반면 Linux에서는 CFS를 통해 스케줄링합니다. 스케줄링은 전체 CPU 사용량의 5%를 차지할만큼 높은 비율을 차지합니다. 리눅스에서는 스케줄링에서 사용하는 CPU 사용량을 줄이고자 CFS 스케줄러를 개발했습니다.

앞서 살펴봤던 Round Robin이나 MLFQ에서는 큐마다 **고정된** `time-slice`를 사용하는 반면, CFS에서는 현재 경쟁중인 프로세스에게 동일한 CPU 시간을 나눠주도록 해 동적으로 `time-slice`를 결정하도록 합니다. 동적으로 `time-slice`가 달라진다는 면에서 stride scheduling과 유사하기도 합니다. 이 뿐만 아니라 stride scheduling에서 사용되는 pass 역할을 CFS에서는 `vruntime`이 수행합니다.

<img width="492" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/1e6ed504-6bd5-4d03-b472-ccd4dea669a7">

CFS가 어떻게 동작하는지 알기 위해서는 `sched_latency`(또는 target latency)에 대해 먼저 알아보겠습니다. CFS는 현재 경쟁중인 프로세스에게 동일한 CPU시간을 나눠주기로 했는데 경쟁중인 프로세스가 N개라면 프로세스마다 CPU를 사용하는 시간은 `X / N` 일 것입니다. 여기서 X가 의미하는 것이 `sched_latency`입니다. 

X 역시 **적절한**(?) 숫자로 조절해야합니다. 너무 짧다면 컨텍스트 스위칭이 많아져 오버헤드가 발생하고 너무 길다면 Fairness가 떨어지기 때문입니다. 일반적으로는 `sched_latency`를 48ms로 설정하고 이는 위 사진에서도 확인 가능합니다. 현재 실행중인 프로세스는 4개 이므로 프로세스마다 12ms씩 사용하게 됩니다.

stride scheduling에서 공정하게 스케줄링하기 위해 현재까지 사용한 cpu양을 pass로 기록하듯이 CFS에서는 `vruntime`을 기록합니다. 지금까지 사용한 CPU양과 일치하진 않지만 사용 시간에 비례하여 저장합니다.

적절한 시간을 위해 `sched_latency` 말고도 사용하는 장치가 있습니다. 어떠한 복잡한 원리로 `sched_latency`를 정하였더라도 실행되는 프로세스의 수가 많아진다면 어쩔수 없이 프로세스당 CPU 사용시간은 엄청 짧아지게 되고 컨텍스트  스위칭이 자주 발생하는 문제가 있습니다. 이를 해결하기 위해 추가적으로 `min_granularity`라는 변수를 사용합니다. 덕분에 `time-slice`가 아무리 짧아져도 `min_granularity`보다 짧아지지 않도록합니다. 

## Weighting(Niceness)

단순하게 `1/n`로 CPU를 나눠쓰는 것 뿐만 아니라 우선순위에 따라 스케줄링을 할 수도 있습니다. 지금까지 lottery scheduling이나 stride scheduling에선 티켓을 사용해 우선순위를 부여했지만 CFS에서는 `nice` 파라미터를 사용합니다. 

<img width="532" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/743928c9-4409-4e20-8f20-669f52e81d1b">

`nice` 파라미터는 -20부터 19까지 설정가능하며 기본값은 0입니다. 설정값이 클수록 우선순위가 낮아짐을 알 수 있습니다. 책에서는 `너가 너무 nice하면 scheduler를 신경 쓰지 않아` 라고 설명하네요.

만약 프로세스 A, B의 `nice`가 각각 0과 5라면 위 배열에 따라 가중치는 각각 1024,와 335입니다. 이제 아래 공식을 사용하여 동적으로 `time-slice`를 정할 수 있습니다. 가중치가 클 수록 더 긴 `time-slice`를 사용하는 것을 알 수 있습니다.

<img width="418" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/ccc61b6b-5dc2-4afb-b3b0-81d1e575baf3">

`vruntime`을 계산할 때도 가중치를 역으로 반영하여 우선순위가 높을 수록 `vruntime`은 짧게 증가합니다.

<img width="418" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/9f7383d0-018e-47b7-9be8-7aaba004a493">

왜 실제 실행시간 대신 `vruntime`을 사용할까요? 커널 입장에서는 할당된 `time-slice`만큼 사용했으니 다음 스케줄링에서 프로세스를 공평하게 보기 위해서입니다. 가중치가 커서 실제 실행시간(`runtime`)이 길더라도 `vruntime`에 증가되는 값은 작은 것을 알 수 있고 가중치가 작아 실제 실행시간이 짧더라도 `vruntime`에 증가되는 값은 큰 것을 알 수 있습니다. 즉, `vruntime`의 값이 비슷해지므로 다음 스케줄링에서 두 프로세스의 입지가 비슷해집니다. 

## Other Features

CFS는 효율적인 구현을 위해 다양한 방법들을 사용합니다. ***현재 실행중인*** 프로세스를 추적하기 위해 단순한 큐가 아닌 Red-Black-Tree를 활용해 수천개의 프로세스 내에서 빠른 속도로 프로세스들을 관리하도록 도와줍니다. 

또한 특정 프로세스가 장기간 `blocked` 되었다가 돌아온다면 다른 프로세스보다 `vruntime`이 뒤처지게 되어 다른 프로세스들이 CPU를 사용하지 못하는 `starvation` 현상이 일어납니다. 이에 대한 해결책으로 `vruntime`을 Tree에서 발견되는 가장 작은 `vruntime`으로 업데이트 해줍니다