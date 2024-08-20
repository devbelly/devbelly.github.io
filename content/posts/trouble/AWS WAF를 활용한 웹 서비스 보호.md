---
title: AWS WAF를 활용한 웹 서비스 보호
date: 2024-08-20T09:31:09+09:00
draft: false
comments: true
toc: false
tags:
  - study
---
안녕하세요? 이번 시간에는 백엔드 API를 배포하며 겪었던 문제와 AWS WAF를 활용해 웹 API를 보호하는 방법에 대해 공유하고자 합니다.

<br>

<img width="902" alt="image" src="https://github.com/user-attachments/assets/669a6573-333c-4f80-8229-b7f960c111c7">

‘디클’ 프로젝트에서 처음 API를 배포한 후 다음날 `cloudwatch`로 로그를 확인해보니, 위와 같이 알 수 없는 URL로 요청이 들어오는 것을 확인했습니다. API를 배포한 지 두 시간만에 발생한 일이었습니다. 

시간당 2000번 정도 요청이 들어왔고 확장성을 위해 컨테이너의 CPU 혹은 Memory 사용률이 일정 수준을 넘어가면 새로운 컨테이너를 띄우도록 AutoScaling 설정을 했습니다. 불필요한 요청으로 인해 컨테이너 사용량이 증가하면 결국 비용 문제로 이어지므로 이를 해결해야 했습니다. 

<img width="922" alt="image" src="https://github.com/user-attachments/assets/311f8229-c8ef-474a-8cb4-3a6ed673a9c7">
<br>

당시에는 대부분 요청이 `/api` 로 시작하지 않는다는 것을 확인하고 로드 밸런서에 리스너 규칙을 추가하여 해결했습니다. 결과적으로 불필요한 요청이 눈이 띄게 감소했습니다.

<br>

<img width="1171" alt="image" src="https://github.com/user-attachments/assets/bd5b10c9-f35e-4617-bbed-a2265eae9ac4">

백엔드와 프론트엔드 개발 및 배포를 마무리하고 운영 단계에서 모니터링을 위해 `sentry`를 사용했습니다. 여러 이슈 중 NoResourceException이 `sentry`에 쌓이는 것을 확인했습니다. 

<br>

<img width="918" alt="image" src="https://github.com/user-attachments/assets/e5a2b583-7d64-400a-a867-a81456c4f155">

처음에는 도메인 주소를 알아내서 해당 주소로 요청을 보내는 줄 알았으나 로그를 보니 IP 주소를 통해서 접속 요청을 보내고 있었습니다. 하나의 봇이 존재하는 것이 아니므로 이러한 요청은 다양한 IP 주소에서 들어오고 있었습니다.

<br>

<img width="1173" alt="image" src="https://github.com/user-attachments/assets/32e0c0da-38e7-41ea-a23a-ad2414bb27d0">

한 가지 특징은 이상한 접속을 시도하는 모든 IP가 해외인 점입니다. ‘디클’ 프로젝트는 국내 대학생을 대상으로 하는 서비스이므로 해외 IP를 차단하기로 했습니다. 이를 위해서 AWS WAF를 사용했습니다.

AWS WAF는 웹 애플리케이션이나 API를 공격으로부터 보호하기 위한 방화벽입니다. 단순 방화벽은 TCP/IP 레벨에 포함된 정보들을 기반으로 차단 룰을 설정하는 반면 WAF는 HTTP 정보를 바탕으로 차단 룰을 설정합니다. [여러가지 기능](https://aws.amazon.com/ko/waf/features/)을 제공하지만 간단하게 해외 IP만 차단해 보도록 하겠습니다.

<br>

<img width="847" alt="image" src="https://github.com/user-attachments/assets/4c1dda19-f900-402d-814f-a7fb046371dc">

사용 방법은 Web ACLs를 생성하고 나서 아래 두 가지의 설정을 하면 됩니다. 
1. Rule을 설정
2. Rule을 적용할 AWS resource 설정(이번 실습에서는 ALB)

Rule에 대한 이름을 설정하고 한국에서 발생하는 IP가 아니라면 Block 행동을 취하는 규칙을 위와 같이 추가했습니다. 설정하는 규칙이 여러가지인 경우 우선 순위를 올바르게 설정해야합니다. 만약 상위 규칙에서 잘못된 설정으로 차단해야할 행동을 allow한다면 하위 규칙은 무시되기 때문입니다.

생성한 WAF를 확인하기 해외 IP에서 요청을 보내야 합니다. 아래 사이트에서 테스트가 가능합니다.
- [https://tools.pingdom.com/](https://tools.pingdom.com/)

<br>

<img width="1128" alt="image" src="https://github.com/user-attachments/assets/0d27c195-0aa1-4d4c-9c60-a1da35270976">
<img width="556" alt="image" src="https://github.com/user-attachments/assets/e7b0b9f7-ca47-4f19-be72-91d86d41c1bf">

테스트하면 올바르게 403 Forbidden을 반환하는 것을 확인할 수 있습니다. ‘디클’ 프로젝트에도 WAF를 적용하면 좋겠지만 [AWS WAF 요금](https://aws.amazon.com/ko/waf/pricing/)은 기본 설정이더라도 대략 6 달러가 매달 추가됩니다. 따라서 필요할 때 테라폼으로 AWS WAF를 추가하는 방향으로 결정했습니다. 

<br>

## 참고

- [실습으로 살펴보는 AWS WAF](https://www.youtube.com/watch?v=fweHx0vFCS8&t=768s)
- [AWS-WAF 사용하여 해외 IP 차단하기](https://baekji919.tistory.com/entry/AWS-WAF-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-%ED%95%B4%EC%99%B8-IP-%EC%B0%A8%EB%8B%A8%ED%95%98%EA%B8%B0)

