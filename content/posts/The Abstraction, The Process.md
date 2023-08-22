---
title: "The Abstraction, The Process"
date: 2023-08-21T18:34:05+09:00
draft: false
comments: true
toc: true
tags:
  - os
---

## Process

저장장치에 있는 프로그램은 실행되기 위해 메모리에 올라와야합니다. CPU는 메모리에 있는 코드를 읽고 처리하는 과정을 반복하며 프로그램을 실행합니다.  실행중인 프로그램을 OS가 추상화한 것을 우리는 **프로세스** 라고 부릅니다. 

![image](https://github.com/devbelly/image-issue/assets/67682840/d8f38808-05c0-4978-a616-b87d2123808d)

## Process State

OS에 의해서 프로세스들이 관리되고 관리를 위해서는 여러 상태들을 갖게 됩니다. 여러 상태가 존재하지만 크게 아래 3가지 상태가 존재합니다. 

<img width="308" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/730ddf41-6091-424e-9435-23b09fe4b6e5">

- Running
	- CPU에 의해 처리되고 있는 프로세스
- Ready
	- 하나의 프로세스가 CPU를 사용하는 동안 나머지 프로세스들은 다른 준비는 다 끝났으나  CPU가 없어 대기해야합니다. 이를 Ready라고 합니다.
- Blocked
	- CPU 또는 메모리와 관련된 작업은 단순한 명령어로 처리가 가능하지만 하드디스크에서 파일을 읽어오거나 쓰는 작업은 커널의 도움을 받아 처리하므로 시간이 오래 걸립니다. 이 시간동안 CPU를 붙잡고 있는 것은 낭비이므로 해당 요청이 완료될 때 까지 `Blocked` 상태가 됩니다. 이 요청이 완료되면 다시 CPU를 가질 수 있는 상태인 `Ready`가 됩니다.

앞에서 메모리의 가상화에서 살펴보았던 예제를 통해 프로세스에 대해 살펴보겠습니다.

![image](https://github.com/devbelly/image-issue/assets/67682840/3d01fe4a-5322-44b7-8a89-7483825dcc93)

`mem`으로 생성된 오브젝트 파일을 `./mem &; ./mem &` 명령어를 입력하여 두 개의 프로세스를 생성해보겠습니다.

![image](https://github.com/devbelly/image-issue/assets/67682840/ed3835af-7929-45db-b3cf-46e42ae4c2de)

프로그램은 디스크에 저장되어있는 녹색 부분 하나입니다. 이를 메모리에 로드하면 프로그램이 실행되고 이를 추상화 한 것이 프로세스라고 했습니다. 추상화에 대한 개념을 표현하기 위해 구름안에 프로세스들을 표시한 모습입니다.

![image](https://github.com/devbelly/image-issue/assets/67682840/45c623ba-c7ec-473a-a44b-4d198114af72)

물리적 메모리에서는 페이지 단위로 데이터들이 나뉘어 있지만 추상화된 공간인 프로세스 안에서는 정리된 모습으로 표현가능합니다. 동일한 소스코드로 생성이 되었으므로 각 프로세스의 Address space 안 코드 영역이 동일한 것을 확인할 수 있습니다. (연두색)

![image](https://github.com/devbelly/image-issue/assets/67682840/75194282-d64d-46b6-87eb-7d99987c7061)

두 프로세스가 실행되기 위해서는 CPU에서 메모리에 저장되어있는 코드 영역의 명령어를 읽어와야합니다. 맨 처음 그림에서 보았듯, PC가 가리키고 있는 명령어를 IR 을 통해 명령어를 해독하고 명령어의 opcode에 따라 CPU는 명령을 수행합니다. 명령어를 수행하고 나면 다음 명령을 수행하기 위해 PC는 다음 명령어를 가리키게 됩니다.

CPU는 단순하게 `fetch`, `decoding`, `execution` 을 반복하는 물리적 기계일 뿐 어떤 프로세스를 실행하고 있는지 알 수 없습니다. 

## PCB

<img width="528" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/f708b6b0-cc53-44b8-9608-a376fd298b9d">

위 그림은 구름 모양안에 있는 하나의 프로세스를 자세히 나타낸 것입니다. 추상화된 프로세스에서는 자신이 CPU와 메모리를 독점하고 있다고 생각합니다. 간단히 언급했지만 CPU는 시분할 방식으로 메모리는 각 프로세스마다 address space를 줌으로써 이를 가능케 합니다. 실행 도중 다른 프로세스로 전환은 개념적으로 현재 `pid: 24113`의 상태가 `Ready` 또는 `Blocked` 상태로 전환되고 `pid: 24114`가 `Running`으로 바뀐다 설명할 수 있지만 실제 기계어 레벨에서는  단순히 상태만 바꾼다고 해서 이러한 점이 구현되진 않습니다. 추후 Context Switching에서 다시 다루도록 하겠습니다.

프로그램이 실행되기 위해서는 CPU의 사용이 필수적입니다. 메모리에서 CPU의 레지스터로 값을 가져와 연산을 수행하게 되죠. 그 외에서 `PC`, `SP`등 실행에 있어 다양한 레지스터를 사용하게 됩니다. 현재 실행중인 프로세스가 중단 된다면 메모리의 내용뿐만 아니라 현재 CPU에서 작업중인 레지스터의 정보 또한 어딘가에 저장한 후 다시 `Running` 상태가 됐을 때 이를 복원할 수 있어야 합니다.

이러한 내용을 저장하는 장소를 PCB, Process Control Block 라고 합니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/1e4fb349-9b27-4f67-80e5-233193cdd18e">
