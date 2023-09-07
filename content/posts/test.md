---
title: "Test"
date: 2023-09-03T23:53:44+09:00
draft: true
comments: true
toc: true
tags:
  - untagged
---
안녕하세요. 이번시간에는 Process API에 대해 알아보겠습니다. 첫번째는 프로세스를 생성할 때 사용하는 `fork()`입니다. 

## fork

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/d10d1c43-8526-4d18-af36-df4a7bd0afe3">

위 코드는 `rc` 값에 따라 다른 동작을 하는 코드입니다. `fork()` 호출 시 새로운 프로세스를 생성하는데 결과값으로 생성한 자식 프로세스의 `pid`를 리턴합니다. 독특한 점은 새로 생성된 프로세스는 `fork()`를 호출한 프로세스와 **거의** 유사하다는 점입니다. 

<img width="518" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/f4110636-151b-4b08-99d2-e135730ce775">

위 그림은 물리 메모리에서 하나의 부모 프로세스가 실행되는 모습입니다. 간단히 보기 위해 `kernel space`는 생략했습니다. 이때 `fork()`를 호출하면 물리 메모리에 부모 프로세스가 갖고 있는 정보를 거의 그대로 복사합니다. 

<img width="524" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/ca8c3906-3c97-4fe9-a689-01ee0acd3763">

부모 프로세스를 복사하는 과정에서 `code`영역뿐 아니라 `Data`, `heap`을 그대로 복사하고 `stack` 영역 또한 `fork()`가 실행되기 이전의 상태 그대로 복사합니다. **거의** 그대로라고 했는데 그렇다면 무엇이 다를까요? 실행 결과를 봅시다.

<img width="548" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/976cdb66-bbe6-4737-b5e6-cbedbb47aef4">

프로세스가 복사 됐으므로 같은 일을 하는 프로세스가 두 개 실행되므로 `printf("hello world)`가 두 번 실행될 것으로 예상했지만 단 한 번만 출력된 것을 알 수 있습니다. 이는 부모 프로세스를 복사할 때 메모리 뿐만 아니라 PCB도 복사하며 이때 Context에 있는 PC도 복사하므로 자식 프로세스는 `main()`의 처음부터 실행하는 것이 아니라 마치 `fork()`라는 함수를 호출한 것 처럼 그 이후 라인만 실행이 됩니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/edd7f061-fe5b-4b5d-9a5e-3579be6065dd">

하지만 PCB를 복사했다 하더라도 완전히 동일할 수는 없습니다. 예를 들어 생성한 프로세스를 통제하기 위해 `pid`는 고유해야하기 때문입니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/3df168ff-2a96-469f-981d-b15e7b162a19">

또한 `fork()` 이후의 `stack`안에는 `rc`를 저장하는데 부모 프로세스의 `rc` 값에는 자식 프로세스의 `pid`가, 자식 프로세스의 `rc`에는 0을 저장함으로써 동일한 코드에 대해 다른 동작을 수행하게 됩니다.

## wait

프로세스가 2개 생성 되었는데 실행되는 순서는 어떻게 될까요? 단일 CPU 머신에서 어떤 프로세스가 스케줄링 되는지는 결정적이지 않습니다. 위 실행결과에서는 `hello, I'm parent` 이후 `hello, I'm child`가 실행 되었지만 자식 프로세스가 먼저 스케줄링 된다면 실행결과가 다음과 같습니다.

<img width="542" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/094bf552-7a60-4ab2-ba68-2065c9653b68">




- 내용
	- 좀비 & 고아 프로세스 추가
	- 좀비 프로세스가 되기 위한 조건이 뭘까?
	- wait exit

## 참고
- https://courses.cs.washington.edu/courses/cse451/02sp/section/notes/fork/
- https://www.youtube.com/watch?v=sA9NGMc2Kf8
- https://wslog.dev/fork-exec