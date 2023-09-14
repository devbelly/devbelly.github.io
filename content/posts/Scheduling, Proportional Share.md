---
title: Scheduling, Proportional Share
date: 2023-09-14T08:45:59+09:00
draft: true
comments: true
toc: true
tags:
  - os
---
안녕하세요. 이번시간에는 `proportinal-share` 스케줄러에 대해 알아보겠습니다. 지금까지 살펴본 스케줄러들은 Performance에 중점을 두었습니다. SJF, STCF는 short job부터 스케줄링하여 turnaround time을 최적화 하고 RR은 interactive user를 위해 response time을 개선했습니다. 

Turnaround time과 response time은 모두 Performance 관점에서 스케줄러를 평가하기 위한 지표입니다. 메트릭에 대해 설명하면서 Performance 말고도 Fairness도 있다 설명했는데 이번시간에는 Fairness에 초점을 맞춘 스케줄러를 살펴봅시다.


## Basic Concept



로터리는 랜덤넘버, 리셋 후 다시 재연하면 그대로 실행되지 않는다. 비결정적

정책
사용자가 발행한 티켓을 그대로 인정하면 정상적인 비율로 CPU 배분을 기대하기 어렵다
사용자가 발행한 통화를 global currency로 환산해야한다

티켓 인플레이션
자기가 좀 급하다 싶으면 로컬 대신 글로벌 커런시를 발행한다. 서로간에 신뢰가 있어야 발행이 가능하다.

초기에 티켓을 어떻게 줄지에 대한 문제가 있따
유연하긴 하지만 티켓 할당을 해결하기 어려운 한계가 있다.

불공정 지표 , 숫자가 클수록..공평한거 아닌가?

9.6
비결정적이다. 
stride scheduling을 통해 비결정적 → 결정적 으로 바꿔보자
보폭
뱁새가 개이득
pass , 지금까지 얼마나 왔는지를 확인하고 CPU를 줄지 말지 결정한다.

공배수 설정

결정적이라면 디버깅이나 관리하는데 이점이 있다.

문제점?
중간에 들어온 D의 pass값을 어떻게 설정해야하는지 문제 → 로터리에서는 total ticket만 바꿔주면 해결된다.


—
리눅스에서는 CFS를 사용한다.

보폭이 100이라면 time slice를 100으로 설정

RR이나 MLFQ에서는 time slice를 큐 단위로 고정시켜놨지만

CFS나 stride에서는 특정 job을 지정해서 timer를 맞춰놓고 실행시켜준다.

CFS에서는 vruntime 개념을 pass 대체

731