---
title: git rebase
date: 2025-01-19T12:21:13+09:00
draft: false
comments: true
toc: false
tags:
  - study
---
안녕하세요. 오늘은 `git rebase`를 사용하면서 겪었던 문제를 공유하고자 합니다.

디클 프로젝트를 진행하면서 `git` 과 관련된 이야기를 종종 하는 경우가 있었습니다.  커밋 규칙부터 브랜치 관리 및 전략에 대한 이야기를 나눴고 이력 관리를 위해 `merge` 대신 `rebase`를 사용하자고 이야기를 나눴습니다. 

<br>

# 문제 상황

<img width="689" alt="Image" src="https://github.com/user-attachments/assets/bd999e4a-4e78-4809-8b5b-fe6974dd3141" />


작업을 하던 도중 커밋 이력에 이상한 내용이 보이기 시작했습니다. 보통은 `devbelly comitted 1 hours age` 처럼 보이지만 ` A authored and B comitted 1 hour age` 와 같이 보이는 문제가 발생했습니다. 

단순히 글자만 저렇게 나타나면 상관이 없겠지만, 브랜치를 만들어서 작업하고 있던 다른 사람들에게도 영향을 끼쳐서 문제 원인을 파악해야했습니다.

<br>

# 원인

<img width="915" alt="Image" src="https://github.com/user-attachments/assets/c3d1861f-0fa8-4a51-95ff-5955ca697515" />

`git rebase`는 커밋 해시를 변경합니다. 참고로 `git rebase`는 “나를 쟤 위에 올려” 로 이해하면 쉽습니다. 실제로 변경하는지 확인해 볼까요?

1. `640d2e` 에서 `devbelly-work`라는 새로운 브랜치 생성
2. `c3265f` 커밋
3. `git rebase main` 명령을 수행
4. `c3265f(나)` 를 `3cae8d(쟤)` 위에 새롭게 올리고 해시는 `ea9158`로 변경


<br>

# 재연

<img width="639" alt="Image" src="https://github.com/user-attachments/assets/ff85d9c7-f348-485b-8a6e-d48c877caf05" />

두 명의 사람이 하나의 레포에 작업을 하는 상황입니다. `subbelly`가 휴가를 갔다 온 사이 `main` 에 다른 작업들이 반영되어 있습니다. 자신이 해야할 작업을 마치고 `subbelly working hard`로 커밋을 만듭니다.

<br>
<img width="639" alt="Image" src="https://github.com/user-attachments/assets/e7a826be-bf09-41cb-9f8f-1684a390e824" />

작업 내용을 커밋하기 전, `main` 과 충돌이 있는지 확인해야 합니다. `main` 에 있는 변경 사항을 자신의 `feature/sub`로 가져오기 위해 `git rebase feature/sub`를 수행합니다.

<br>

<img width="638" alt="Image" src="https://github.com/user-attachments/assets/9b9064ab-4d2b-4b07-9332-a10b3d32f4ca" />

`main`은 직접 푸시할 수 없도록 막아놨습니다. 따라서 `fast-forward`로 변경 내용을 `feature/sub`에 반영합니다.

<br>

<img width="909" alt="Image" src="https://github.com/user-attachments/assets/f2ec8f45-82dc-42bd-8fcd-f2f4cad5c774" />

브랜치를 푸시하고 `PR`을 확인해보면 `A authored and B comiited`로 나와있는 것을 확인할 수 있습니다. 이 경우 머지하게 된다면 `main`의 베이스가 망가져 다른 작업자들에게 영향을 미칠 수 있습니다. 따라서 이미 공유 되어있는 커밋을 `rebase`로 가져오는 작업을 하지 않도록 조심해야 합니다.


