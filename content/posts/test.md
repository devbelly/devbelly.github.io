---
title: "Test"
date: 2023-08-23T14:52:39+09:00
draft: true
comments: true
tags:
  - untagged
---

안녕하세요. 이번 시간에는 CPU의 가상화를 위해 고려해야할 점들에 대해 다뤄보겠습니다. 이전 글에서도 간단히 언급했지만 하나의 CPU를 통해 많은 프로세스를 실행하기 위해서는 시분할 방식을 사용한다고 했습니다. 이를 구현하기 크게 두 가지를 고려해야합니다. 첫번째는 Performance 입니다. 여러 프로세스가 CPU를 공유하는 과정에서 오버헤드가 없어야 하죠. 두번째는 Control입니다. 만약 제어가 없다면 프로세스는 운영체제가 허용하지 않는 정보에 엑세스하거나 사용자가 리소스를 사용하지 못하도록 다른 동작들을 계속 수행할 수도 있습니다. 

## Limited Direct Execution

성능이 뛰어나면서도 제어권을 갖기 위해 OS 개발자들은 `Limited Direct Execution`이라는 기술을 만들었습니다.  여기서 `Limited`를 뺀 `Direct Execution`는  제한 없이 **모든 작업**을 CPU에서 처리하는 것을 의미합니다. 

<img width="500" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/e81a722c-d865-4f45-9fc2-11c1d10aeb35">

위 프로그램은 프로세스 실행에 필요한 초기화 작업을 거친 후 OS에서 `main()`을 호출하고 있습니다. 겉보기에는 별 문제가 없어보이지만 두 가지 문제가 있습니다.

첫 번째는 `Program`에서 OS가 원하지 않는 작업을 수행할 때 통제를 할 수 없다는 것입니다. 단지 `Program`이 CPU에 대한 통제권을 다시 OS에 넘겨주어야하만 다음 작업을 수행할 수 있습니다. 이어지는 두번째 문제는 `Program`에서 적절한 작업을 수행한다 할지라도 시분할을 위해서는 일정량 사용 후 다른 프로그램에 CPU를 줘야하지만 통제권이 프로그램에 있어 시분할을 할 수 없다는 것입니다.

위 문제들을 해결하기 위해 CPU에 `limited`,  제한을 두었습니다. CPU를 `user mode`와 `kernel mode`로 구분하고 `user mode`에서는 대표적으로 `I/O`나 인터럽트 관련된 명령어의 존재 자체를 몰라 처리할 수 없습니다.

<img width="390" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/604e1ad7-5b04-417f-96d2-708e16867309">

`user mode`이므로 Program Counter가 `user space`에 있는 명령어를 가리키고 있는 모습입니다. 만약 코드에 `kernel mode`에서 동작하는 명령어가 존재한다면 컴파일러가 이를 맡아서 처리하게 됩니다. 이전 시간에 살펴본 메모리와 그림이 다른데 이는 뒤에서 설명하도록 하겠습니다.

<img width="612" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/628b8b50-3670-4085-bc42-3ac23b351968">

프로그래밍을 처음 하면 가장 먼저 하는 것이 `printf("hello, world!")`입니다. 이처럼 `I/O` 연산은 자주하게 되는데 만약 `user mode`에서 자신이 갖고 있는 이상의 권한, 즉 privilieged operation을 수행하기 위해 `kernel mode`로 변경하려면 어떻게 해야할까요?

<img width="620" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/4d2a33b8-8311-4460-83cc-4120d685db6b">

이때 등장하는 개념이 Trap입니다. CPU가 함정에 빠졌다고해서 Trap이라는 표현을 사용하며 유저 프로그램을 처리하다Trap이 걸리면 Program counter가 미리 약속된 지점인 Trap handler로 바뀌게 됩니다. PC가 Trap handler를 가리키기 위해서 CPU 안에는 trap handler의 주소가 어디인지 알고 있어야합니다. 이제 Trap hanlder가 위치하는 kernel space에 대해 자세하게 살펴볼까요?

<img width="532" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c6a958cb-f32d-4c4c-9672-5e70520ede4f">

메모리의 생김새가 이전 글들에서 살펴봤던 것과는 조금 다른 것을 알 수 있습니다.  이전에는 프로그램이 로드되는 `user space`만 표시했었는데 앞으로는 OS의 핵심인 `kernel`이 위치하는 `kernel space` 또한 같이 표시하도록 하겠습니다. 

Trap handler는 Trap이 트리거될 때 실행되는 코드입니다. 총 3가지 종류의 Trap이 있는데 간단히만 살펴보겠습니다.

- Exception
	- Internal Interrupt라고도 합니다. 여기서 Internal이란 CPU와 메모리 사이의 범위를 의미합니다.
	- division by zero
	- page fault

- Interrupt
	- CPU와 메모리를 제외한 영역에서 발생하는 인터럽트를 의미합니다.
	- CPU의 INT핀을 통해 외부 장치에서 오는 인터럽트를 확인할 수 있습니다.

- Syscall
	- SW Trap이라고 하며 사용자 프로그램을 실행하다가 `user mode`에서 처리할 수 없는 명령어를 만났을 때 발생합니다. 예를 들어 `INT x80`와 같은 어셈블리 명령어가 있습니다.

Trap Handler는 이러한 Trap을 구분해야 어떤 동작을 할지 결정할 수 있습니다.


---
작업이 완료되면 OS는 리턴 프롬 트랩을 수행한다.
트랩이 어떤 코드를 실행하는 방법? 직접 점프 주소를 지정할 수는 없다. 유저 스페이스에서 직접 커널의 주소에 접근하므로 구분한 의미가 없다.
트랩 테이블을 지정해서 이 문제를 해결한다. 예외 이벤트가 발생할 때 어떤 코드를 실행할지 하드웨어에게 알려준다 OSrk

정확한 시스템 호출을 지정하기 위해 일반적으로 각 시스템 호출에 시스템 호출 번호가 부호화됩니다. 따라서 사용자 코드는 원하는 시스템 호출 번호를 레지스터 또는 스택의 지정된 위치에 배치해야 하며, OS는 트랩 핸들러 내에서 시스템 호출을 처리할 때 이 번호를 검사하고 유효한지 확인한 다음, 유효한 경우 해당 코드를 실행합니다. 사용자 코드는 점프할 정확한 주소를 지정할 수 없고 번호를 통해 특정 서비스를 요청해야 하므로 이러한 수준의 간접지시는 일종의 보호 역할을 합니다.

그림 6.2의 타임라인(시간이 아래로 갈수록 증가함)은 프로토콜을 요약합니다. 각 프로세스에 커널 스택이 있고, 커널에 들어갔다 나올 때 레지스터(범용 레지스터와 프로그램 카운터 포함)가 저장되고 하드웨어에 의해 복원된다고 가정합니다.

p61