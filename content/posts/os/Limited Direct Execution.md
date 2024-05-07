---
title: Limited Direct Execution
date: 2023-08-23T14:52:39+09:00
draft: false
comments: true
tags:
  - os
toc: true
---

안녕하세요. 이번 시간에는 CPU의 가상화를 위해 고려해야할 점들에 대해 다뤄보겠습니다. 이전 글에서도 간단히 언급했지만 하나의 CPU를 통해 많은 프로세스를 실행하기 위해서 시분할 방식을 사용한다고 했습니다. 이를 구현하기 위해 두 가지를 고려해야합니다. 첫번째는 Performance 입니다. 여러 프로세스가 CPU를 공유하는 과정에서 오버헤드가 없어야 하죠. 두번째는 Control입니다. 만약 제어가 없다면 프로세스는 운영체제가 허용하지 않는 정보에 엑세스하거나 사용자가 리소스를 사용하지 못하도록 다른 동작들을 계속 수행할 수도 있습니다. 

## Limited Direct Execution

성능이 뛰어나면서도 제어권을 갖기 위해 OS 개발자들은 `Limited Direct Execution`이라는 기술을 만들었습니다.  여기서 `Limited`를 뺀 `Direct Execution`는  제한 없이 **모든 작업**을 CPU에서 처리하는 것을 의미합니다. 

<img width="500" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/e81a722c-d865-4f45-9fc2-11c1d10aeb35">

위 프로그램은 프로세스 실행에 필요한 초기화 작업을 거친 후 OS에서 `main()`을 호출하고 있습니다. 겉보기에는 별 문제가 없어보이지만 두 가지 문제가 있습니다.

첫 번째는 `Program`에서 OS가 원하지 않는 작업을 수행할 때 통제를 할 수 없다는 것입니다.  이어지는 두번째 문제는 `Program`에서 적절한 작업을 수행한다 할지라도 다른 프로세스 실행을 위해 CPU를 회수해야할 경우 실행이 종료될 때 까지 계속해서 `Program`이 CPU를 갖고 있다는 점입니다. 두번째 문제는 조금 후에 다루도록 하겠습니다.

원치않는 작업에 대한 수행을 통제하기 위해 CPU에 `limited`,  제한을 두었습니다. CPU를 `user mode`와 `kernel mode`로 구분하고 `user mode`에서는 대표적으로 `I/O`나 인터럽트 관련된 명령어의 존재 자체를 몰라 처리할 수 없습니다. 아래 그림에서 CPU가 `user mode`와 `kernel mode` 로 구분된 것을 확인할 수 있습니다.

<img width="390" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/604e1ad7-5b04-417f-96d2-708e16867309">


`user mode`이므로 Program Counter가 `user space`에 있는 명령어를 가리키고 있는 모습입니다. 만약 코드에 `kernel mode`에서 동작하는 명령어가 존재한다면 컴파일러가 이를 사전에 확인하여 컴파일에러를 발생시킵니다. 이전 시간에 살펴본 메모리와 그림이 다른데 이는 뒤에서 설명하도록 하겠습니다.

<img width="612" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/628b8b50-3670-4085-bc42-3ac23b351968">

프로그래밍을 처음 하면 가장 먼저 하는 것이 `printf("hello, world!")`입니다. 이처럼 `I/O` 연산을 자주하게 되는데 만약 `user mode`에서 자신이 갖고 있는 이상의 권한, 즉 privilieged operation을 수행하기 위해 `kernel mode`로 변경하려면 어떻게 해야할까요?

<img width="620" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/4d2a33b8-8311-4460-83cc-4120d685db6b">

이때 등장하는 개념이 Trap입니다. CPU가 함정에 빠졌다고해서 Trap이라는 표현을 사용하며 유저 프로그램을 처리하다 Trap이 걸리면 Program counter가 미리 약속된 지점인 Trap handler로 바뀌게 됩니다. PC가 Trap handler를 가리키기 위해서 CPU 안에는 trap handler의 주소가 저장되어 있습니다. 이제 Trap handler가 위치하는 kernel space에 대해 자세하게 살펴볼까요?

<img width="532" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c6a958cb-f32d-4c4c-9672-5e70520ede4f">

메모리의 생김새가 이전 글들에서 살펴봤던 것과는 조금 다른 것을 알 수 있습니다.  이전에는 프로그램이 로드되는 `user space`만 표시했었는데 앞으로는 OS의 핵심인 `kernel`이 위치하는 `kernel space` 또한 같이 표시하도록 하겠습니다. `kernel space`은 OS가 부팅될 때 해당 내용들로 채워지게 됩니다.

Trap handler란 Trap이 트리거될 때 실행되는 코드입니다. 총 3가지 종류의 Trap이 있는데 간단히만 살펴보겠습니다.

- Exception
	- Internal Interrupt라고도 합니다.  Internal이란 CPU와 메모리 사이의 범위를 의미합니다.
	- division by zero
	- page fault

- Interrupt
	- CPU와 메모리를 제외한 영역에서 발생하는 인터럽트를 의미합니다.
	- CPU의 INT핀을 통해 외부 장치에서 오는 인터럽트를 확인할 수 있습니다.

- Syscall
	- SW Trap이라고 하며 사용자 프로그램을 실행하다가 `user mode`에서 처리할 수 없는 명령어를 만났을 때 발생합니다. 예를 들어 `INT x80`와 같은 어셈블리 명령어가 있습니다.
	  <img width="804" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/26075132-92b1-4513-919d-e25a533631cf">
  
Trap Handler는 이러한 Trap을 구분해야 어떤 동작을 할지 결정할 수 있습니다. 따라서 사용자 코드에서 시스템 호출 번호를 레지스터 또는 스택의 지정된 위치에 배치합니다. 이후 어떠한 이유에서 CPU가 `kernel mode`로 바뀌었다면 OS는 Trap handler내에서 시스템 호출을 처리할 때 이 번호가 유효한지 검사한 후 해당 번호에 맞는 코드를 실행합니다. 사용자 코드에서 직접 `kernel space`에 접근하는 것이 아닌 번호를 통해 특정 함수 조각을 호출해야하므로 이러한 간접 지시는 보호 역할을 하게 됩니다. 

<img width="606" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/fab1ccaa-4cdb-4753-b60c-046d5256942e">

이 그림은 프로그램에서 `syscall`을 호출하기 위한 흐름을 시간순으로 배치했습니다. 1번에서 프로세스 초기화를 위해 `user stack`과 `kernel stack`을 초기화 한 후 `return-from-trap`을 호출하고 있습니다. `kernel stack`은 `kernel space`에 존재하는 스택이라고 생각하면 됩니다. `return-from-trap`을 호출하면 CPU의 모드가 `kernel mode`에서 자신이 바뀌기 이전의 모드인 `user mode`로 변경됩니다. 

2번에서 `user mode`로 바뀌기 전 `restore regs`가 있는데 이건 `kernel space`에 있는 `kernel stack`의 내용을 CPU의 레지스터에 저장한다는 의미입니다. CPU에 적절한 값, 예를 들어 PC가 `main()`을 가리키고 있어야 `user mode`에서 `main()`을 실행할 수 있기 때문입니다. 

PC가 `main()`을 가리키고 있으므로 코드를 수행하다가 자신이 작업범위 밖인 `syscall`을 만나게 되어 `trap`이 호출됩니다. 

3번에서는 현재 CPU의 레지스터 값들을 `kernel stack`에 다시 저장한 후 CPU는 `kernel mode`로 진입하여 적절한 `syscall`을 처리합니다. 이후 내용은 직접 살펴보시면 금방 파악할 수 있으므로 생략하겠습니다.

---
`Direct Execution`의 첫번째 문제, 원치 않는 동작을 하는 것은 Trap Handler를 통해 제어가 가능함을 살펴봤습니다. 두번째 문제는 나중에 언급한다 했었는데 지금 그 문제를 살펴보겠습니다. 만약 프로그램이 크게 문제가 없다 하더라도 CPU는 소중한 자원이기 때문에 여러 프로세스가 사용하고 싶어합니다. 특히 프로세스가 CPU를 갖고 있다면 OS는 논리적으로 실행이 되고 있지 않는 상태입니다. 이 상태에서 회수할 수 있는 방법이 없으므로 CPU를 다른 프로세스에게 스케줄링하는 건 불가능하다고 볼 수 있습니다.
## A Cooperative Approach: Wait for Syscall 

초기에는 프로세스가 OS에게 주기적으로 CPU를 반납하는 협조적인 방식으로 이를 해결했습니다. 프로그램은 파일을 열어 읽거나 다른 프로세스를 생성하는 것과 같은 시스템 호출을 자주 사용하고 이 과정에서 CPU를 반납하므로 다시 CPU는 OS가 얻어 통제할 수 있게 됩니다. 뿐만 아니라 `yield()`를 통해 명시적으로 CPU를 반납하기도 했습니다. 하지만 악의적인 프로그래머가 커널이 필요한 연산을 하지 않고 무한 루프와 같이 CPU 집약적인 연산만 사용한다면, 즉 비협조적으로 나온다면 OS는 여전히 CPU를 얻어올 수 없는 문제가 있습니다.
## A Non-Cooperative Approach: The OS Takes Control

운영체제는 유저 프로그램이 CPU를 자진 반납하는 유토피아 세계를 기대하지 않고 타이머 인터럽트를 통해 이 문제를 해결했습니다. 타이머가 몇 밀리초마다 인터럽트를 걸면 똑같이 Trap이 걸려서 현재 실행중인 프로세스가 중단되고 OS의 Trap Handler가 실행됩니다. OS는 제어권을 찾아왔으므로 어떤 프로세스를 실행할지 결정할 수 있는데 이는 OS의 일부인 스케줄러가 결정합니다. 어떤 프로세스로 전환할지 결정하면 해당 프로세스에게 제어권을 넘겨줄 수 있는데 이를 Context Switching이라 합니다.

<img width="618" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/0b20d33a-19f3-487b-9c47-0e40c243cf18">

위 그림은 Context Switching이 일어나는 과정을 살펴본 것입니다. 1번에서 OS가 부팅되며 `kernel space`에 trap table을 초기화합니다. 이후 CPU가 `syscall handler`와 `timer handler`의 위치를 기억한다고 나와있는데 간단히 말해 CPU가 `Trap Handler`의 위치를 미리 알고 있어야 한다는 뜻입니다. 위치를 모른다면 Trap이 발생했을 때 CPU의 PC가 Trap Handler의 위치를 가리키지 못하기 때문이죠. 그 이후 timer가 실행된 것을 볼 수 있습니다.

3, 4번 과정에서는 프로세스가 실행되고 timer interrupt가 발생합니다. 현재 CPU의 레지스터를 `kernel stack(A)`에 저장한 후 Trap이 발생했으므로 CPU를 `kernel mode`로 변경합니다.

5번과정은 자신이 Trap이 걸린 이유를 파악하고 timer에 의해 interrupt가 걸렸다는 것을 알게 됩니다. OS의 스케줄러가 다음에 실행할 프로세스를 결정하면 현재 CPU의 레지스터를 Process Control Block(PCB)안 Context에 저장한 후 프로세스 B의 PCB안 Context를 현재 CPU의 레지스터로 복원합니다.

여기서 그림만으로는 조금 이해하기 어려운 것이 있는데 바로 4번의 `save regs(A) → k-stack(A)`와 5번의 
`save regs(A) → proc_t(A)`입니다. 왜 CPU 레지스터값을 두번이나 저장하는 걸까요? 그 이유는 **대부분**의 정보를 `kernel stack`에 저장할 뿐 모든 정보를 저장하진 않습니다. 그렇다면 무엇을 저장하지 않을까요? 바로 **kernel stack**의 메모리 주소입니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c83ce731-51e8-4e5b-b544-f4415ba0bd9e">

이 그림은 4번 과정을 나타낸 것입니다. `user mode` 상태의 레지스터를 `kernel-stack(A)`에 저장하고 있습니다. `kernel stack`의  시작 주소가  6000임을 눈여겨 봅시다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/fce98fbe-fc5d-454f-93ec-0f838e072032">

CPU가 `kernel mode`로 변경되고 Trap Handler로 진입한 5번 상태입니다. 자신이 불려온 이유를 파악하고 Context Switching을 해야한다고 판단해 CPU의 레지스터 정보를 PCB에 저장하고 있습니다. 레지스터 값들이 `user mode`일때와 다르고 특히 `esp`는 `kernel stack`의 시작주소인 6000을 가리키고 있습니다. 즉 PCB의 esp에는 해당 프로세스의 `kernel stack`의 시작주소가 담겨있습니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/884daeae-2e5e-4c72-afa9-ba7e6b968784">

프로세스 B가 실행될 것으로 결정됐으므로 프로세스 B의 PCB에서 `kernel stack`의 시작주소를 가져옵니다. 이때 시작주소가 8500인 것을 알 수 있습니다.  

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/9c63eaef-bc2a-4ba3-922c-ed18ba9833a9">

6번과정에서 이제 `kernel stack(B)`의 주소를 파악했으므로 해당 주소로 접근하여 `user mode`에서 사용하는 레지스터를 복원할 수 있게 됩니다.

## 참고
- https://stackoverflow.com/questions/67955845/during-a-context-switch-does-the-os-use-pcb-or-kernel-stack-to-restore-register
