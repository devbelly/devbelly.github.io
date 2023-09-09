---
title: Process API
date: 2023-09-03T23:53:44+09:00
draft: false
comments: true
toc: true
tags:
  - os
---
안녕하세요. 이번시간에는 Process API에 대해 알아보겠습니다. 첫번째는 프로세스를 생성할 때 사용하는 `fork()`입니다. 

## fork

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/d10d1c43-8526-4d18-af36-df4a7bd0afe3">

위 코드는 `rc` 값에 따라 다른 동작을 하는 코드입니다. `fork()`는 호출 시 새로운 프로세스를 생성하는데 결과값으로 생성한 자식 프로세스의 `pid`를 리턴합니다. 독특한 점은 새로 생성된 프로세스는 `fork()`를 호출한 프로세스와 **거의** 유사하다는 점입니다. 

<img width="518" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/f4110636-151b-4b08-99d2-e135730ce775">

위 그림은 물리 메모리에서 하나의 부모 프로세스가 실행되는 모습입니다.(물리 메모리가 이처럼 생기진 않았지만 설명을 위해 간단히 표현했습니다.) 간단히 보기 위해 `kernel space`는 생략했습니다. 이때 `fork()`를 호출하면 물리 메모리에 부모 프로세스가 갖고 있는 정보를 거의 그대로 복사합니다. 

<img width="524" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/ca8c3906-3c97-4fe9-a689-01ee0acd3763">

부모 프로세스를 복사하는 과정에서 `code`영역뿐 아니라 `Data`, `heap`을 그대로 복사하고 `stack` 영역 또한 `fork()`가 실행되기 이전의 상태 그대로 복사합니다. **거의** 그대로라고 했는데 그렇다면 무엇이 다를까요? 실행 결과를 봅시다.

<img width="548" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/976cdb66-bbe6-4737-b5e6-cbedbb47aef4">

프로세스가 복사 됐으므로 같은 일을 하는 프로세스가 두 개 실행되므로 `printf("hello world)`가 두 번 실행될 것으로 예상했지만 단 한 번만 출력된 것을 알 수 있습니다. 이는 부모 프로세스를 복사할 때 메모리 뿐만 아니라 PCB도 복사하며 이때 Context에 있는 PC도 복사하므로 자식 프로세스는 `main()`의 처음부터 실행하는 것이 아니라 마치 `fork()`라는 함수를 호출한 것 처럼 그 이후 라인부터 실행 됩니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/edd7f061-fe5b-4b5d-9a5e-3579be6065dd">

하지만 PCB를 복사했다 하더라도 완전히 동일할 수는 없습니다. 예를 들어 생성한 프로세스를 통제하기 위해 `pid`는 고유해야하기 때문입니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/3df168ff-2a96-469f-981d-b15e7b162a19">

또한 `fork()` 이후의 `stack`안에는 `rc`를 저장하는데 부모 프로세스의 `rc` 값에는 자식 프로세스의 `pid`가, 자식 프로세스의 `rc`에는 0을 저장함으로써 동일한 코드에 대해 다른 동작을 수행하게 됩니다.

## wait

프로세스가 2개 생성 되었는데 실행되는 순서는 어떻게 될까요? 단일 CPU 머신에서 어떤 프로세스가 스케줄링 되는지는 결정적이지 않습니다. 위 실행결과에서는 `hello, I'm parent` 이후 `hello, I'm child`가 실행 되었지만 자식 프로세스가 먼저 스케줄링 된다면 아래처럼 `child`가 먼저 출력됩니다.

<img width="542" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/094bf552-7a60-4ab2-ba68-2065c9653b68">

자식 프로세스가 실행된 후 부모 프로세스가 실행되어야 하는 경우도 분명 존재할 것입니다. 이를  위해 `wait` syscall을 사용할 수 있습니다. `wait`를 호출한 부모 프로세스는 자식 프로세스의 실행이 종료될 때까지 대기하는 역할을 수행합니다. 

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/795ad9a4-b6f3-4cc5-9e20-78ce1b971bb1">

이 프로그램은 몇 번을 실행 하더라도 위에서 표시한 번호 순서대로 콘솔에 찍히게 됩니다. 중요한 점은 `wait()`를 살펴보면 파라미터로 `NULL`을 전달받고 있는데 여기서 `wait`의 두번째 역할에 대해 알 수 있습니다. 각 프로세스가 종료되면 exit status를 갖게 됩니다. `fork`로 생성된 자식 프로세스가 정상적으로 종료되기 위해선 부모 프로세스가 자식 프로세스의 상태를 거둬들여야하는데 부모 프로세스가 자식 프로세스의 종료 상태를 거둬들이기 위해`wait` 를 사용합니다. 이때 종료상태를 어디에 받아올지 파라미터로 넘겨 줄 수 있습니다. 만약 `int s`가 선언된 상태라면 `wait(&s)` 를 통해 자식 프로세스의 상태를 받아오게 됩니다.

## zombie process

프로세스가 종료되고 리소스가 모두 회수되었지만 프로세스 테이블에 남아있는 프로세스를 `zombie process`라고 합니다. 프로세스 테이블의 공간은 한정되어 있으므로 좀비 프로세스가 많아진다면 더이상 프로세스를 생성할 수 없는 상태가 되기 때문에 zombie process를 release해야합니다.

자식 프로세스가 실행이 종료되면 `exit`를 호출해서 운영체제에게 삭제를 요청합니다. 프로세스의 종료 상태가 저장되는 시스템 프로세스 테이블에는 해당 프로세스가 남아있고 이는 부모 프로세스가 해당 상태를 거둬들일때까지 남아있게 됩니다. 프로세스의 상태를 거둬가는 것을 `reaping`한다고 하는데 단어의 뜻을 보면 `수확하다` 라고 되어있습니다. 즉, zombie process는 부모 프로세스가 자신의 상태를 reaping 해주도록 기다리는데 이는 위에서 언급한 `wait`의 두번째 역할을 통해 가능합니다. 부모 프로세스가 정보를 회수한 다음에야 자식 프로세스는 좀비 프로세스에서 벗어날 수 있게 됩니다.
## orphan process

자식 프로세스가 정상적으로 종료되기 위해선 부모 프로세스가 필요합니다. 만약 자식 프로세스를 처리해 줄 부모 프로세스가 어떠한 이유로 종료되었다면 자식 프로세스는 어떻게 될까요? 이렇게 부모가 사라진 프로세스를 orphan process. 고아 프로세스라고 부릅니다.

이 경우는 컴퓨터가 부팅될 때 OS가 처음 만든 `pid:1`인 `init` 프로세스가 고아 프로세스를 입양하여 관리하게 됩니다. `init` 프로세스는 입양 여부에 관계 없이 자식 프로세스를 `reaping`하기 때문에 오랫동안 `orphan` 이나 `zombie` 상태를 유지하지 않습니다.

## exec

마지막으로 살펴볼 API는 `exec`입니다. `fork`와 **자주** 같이 쓰이는 API인데 호출한 프로세스와 다른 동작을 하는 프로그램을 수행할 때 사용됩니다. 

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/50defb79-64c5-4500-8d30-9d80da6b6130">

현재 `p3.c` 를 컴파일하여 실행하는 모습을 나타낸 그림입니다. 하드디스크에는 `p3.c` 뿐만 아니라 단어 갯수를 세주는 프로그램인 `wc` 또한 있는 모습입니다. `fork()`를 했으므로 자식 프로세스에도 동일한 동작을 하는 코드가 실행됨을 알 수 있습니다. 이를 가시적으로 확인하기 위해 노란색으로 `code`영역을 표시했습니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/7464bd3f-0646-4987-9b0d-2ec7cb75a554">

자식 프로세스가 스케줄링 되어 `exec`가 실행된 모습입니다. 기존 프로그램 코드였던 `p3.c` 대신 단어의 갯수를 세주는 `wc`이 로드된 것을 확인할 수 있습니다. 이때 `data`, `heap`, `stack` 영역도 초기화됩니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/18495d18-3b7b-40cd-a5ee-c32348c77bc2">

`exec`는 실행 인자로 `myargs[0]`를 사용하여 현재 프로그램이 `wc`로 바뀔 것을 알리고 어떠한 프로그램의 단어 갯수를 셀 것인지 `myargs`를 통해 전달하고 있습니다. 배열의 마지막 부분을 알리기 위해 `myargs[2]`에는 `NULL`을 표시한 것도 확인할 수 있습니다.

자식 프로세스 입장에서는 현재 프로그램의 코드가 `wc`로 바뀌었으므로 `p3.c` 부분에 해당하는 `this shouldn't print out`은 출력되지 않습니다.

## Why? Motivating the API

프로세스를 `fork()`를 통해서 만들고 `exec()`코드를 변경하는 과정을 왜 분리했을까요? 이는 UNIX에서 `shell`을 통해 `fork()` 실행 이후와 `exec()`실행 사이에 코드를 실행함으로써 자식 프로세스의 실행환경을 변경할 수 있기 때문입니다.

<img width="326" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/ccaa3929-89c4-40f2-833c-8386f6ae4b86">

이 명령어는 `wc`의 실행인자로 `p3.c`를 받고나서 리다이렉션을 통해 `STDOUT`을 `newfile.txt`로 변경하는 명령어입니다. `shell`은 `fork()`이후 프로세스가 생성되었다면 해당 프로세스 내에 있는 `file descriptor table`의 1번 인덱스를 `exec`를 실행하기 전 `newfile.txt`로 변경함으로써 리다이렉션을 구현합니다. 리다이렉션 뿐만 아니라 파이프라인 또한 이와 유사하게 동작합니다. 즉, 이를 구분한 이유는 실행환경의 변경을 간단하게 할 수 있기 때문이라고 할 수 있겠습니다.

## Copy on write(COW)

지금까지 프로세스 API종류에 대해 살펴봤습니다. 잠깐 언급했지만 `fork`와 `exec`는 함께 자주 쓰이는 API들입니다. 프로세스가 실행되는 환경을 `fork`로 만들고 바로 `exec`로 새로운 코드 영역으로 초기화를 하는데, 한 가지 불편한 점은 `fork`입니다. `fork`를 하면 **물리 메모리**에 거의 동일한 내용으로 복사를 하여 프로세스를 생성한다고 말했는데요. `exec`와 함께 같이 쓰인다면 열심히 메모리 복사를 한 내용을 초기화 해버리므로 복사에 대한 오버헤드가 생기게 됩니다.

이러한 문제를 극복하기 위해 `COW`을 사용하게 되었습니다. Copy on write. 말 그대로 `write`시에 복사를 진행한다는 건데 그림을 통해 살펴보겠습니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/08686c93-5201-4ac2-88ed-9b65c7e552d1">

현재 1번 부모 프로세스가 `fork()`를 통해 2번 자식 프로세스를 생성한 직후의 모습을 나타낸 그림입니다. 맨 처음 설명했던 것처럼 물리메모리에 동일한 내용이 두 개 존재하는 것이 아니라 하나의 물리 메모리를 두개의 프로세스가 공유하고 있는 모습입니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/f632dc0b-9d67-4c9a-bb45-84505e7cc5b5">

만약 어떠한 프로세스에 `write` 요청이 발생한다면 기존 물리 메모리를 수정하는 것이 아니라 `copy on write`, 즉 복사본을 생성하여 달라진 내용을 복사본에 반영하여 사용하게 됩니다. 이러한 최적화 기술을 통해 `fork()`시 발생하면 메모리에 대한 오버헤드를 줄일 수 있습니다.
 
 ## 참고
- https://courses.cs.washington.edu/courses/cse451/02sp/section/notes/fork/
- https://www.youtube.com/watch?v=sA9NGMc2Kf8
- https://wslog.dev/fork-exec