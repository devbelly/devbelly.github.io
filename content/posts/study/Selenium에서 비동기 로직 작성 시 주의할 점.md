---
title: Selenium에서 비동기 로직 작성 시 주의할 점
date: 2024-08-22T18:44:49+09:00
draft: false
comments: true
toc: false
tags:
  - study
---
안녕하세요? 오늘은 Selenium 라이브러리를 활용해 비동기 로직을 구현할 때, 주의할 점에 대해 살펴보겠습니다.

Selenium은 웹 애플리케이션 테스트를 자동화하기 위한 도구입니다. Selenium을 통해 사람 대신 웹 페이지에서 버튼 클릭, 텍스트 입력, 페이지 탐색의 작업을 할 수 있습니다. 이러한 특징 덕분에 웹 페이지에서 필요한 정보를 동적으로 가져올 때 이용하기도 합니다.

‘동적으로 가져온다’는 말은 동적 컨텐츠를 가져오는 것을 의미합니다. 일정 시간이 지난 후에 로드되거나, 특정 이벤트(버튼 클릭, 스크롤 등)에 의해 추가로 로드되기도 합니다.

Selenium 예제 코드를 살펴보며 어떤 식으로 동작하는지 살펴보도록 하겠습니다. 

<br>

<img width="725" alt="image" src="https://github.com/user-attachments/assets/80d7a717-1538-423f-a409-39686249d922">

드라이버 객체를 생성한 후(1), 크롤링할 페이지를 설정합니다(2). 다음으로는 원하는 데이터를 가져오면 됩니다(3). 원하는 동작을 마쳤다면 `quit()`을 호출해서 리소스를 정리하는 과정을 거치게 됩니다(4).

<br>

## 드라이버 객체 생성

<img width="448" alt="image" src="https://github.com/user-attachments/assets/2c85abba-d093-4f17-8753-b42f22ad484a">



스프링에서는 객체 생성 비용이 크다면 싱글톤으로 설정하여 생성 및 소멸에 필요한 비용을 줄입니다. 이렇게 설정하면 생성자 주입 방식으로 필요한 클래스에서 `ChromeDriver`를 사용할 수 있게 됩니다. 하지만 싱글톤을 여러 쓰레드에서 사용할 때 주의할 점이 있습니다. 만약 여러 쓰레드에서 하나의 싱글톤의 상태를 변경할 수 있다면 동시성 문제가 발생합니다.

<br>

<img width="687" alt="image" src="https://github.com/user-attachments/assets/4ec3eb02-4fee-4c1c-aee0-273a1226c548">

드라이버는 내부적으로 사용하는 변수들이 많습니다. `thread-safe`하지 않을 것으로 예상할 수 있고 [공식 답변](https://www.selenium.dev/documentation/legacy/selenium_2/faq/#q-is-webdriver-thread-safe)에서도 `thread-safe`하지 않다고 설명합니다. 인스턴스에 대한 접근을 직렬화 하는 대신, 각 쓰레드마다 `WebDriver` 를 생성할 것을 권장합니다. 프로젝트에서 객체 생성이 복잡하므로 팩토리 메서드를 통해 필요한 곳에서 호출해 사용했습니다.

드라이버를 `thread-safe`하지 않게 사용하면 발생하는 문제 중 한 가지는 여러 HTML 코드가 겹쳐서 나타나는 문제가 있었습니다. 각 쓰레드마다 객체를 생성하고 사용을 마친 후 `quit()`을 통해 리소스를 정리하는 코드가 있다면 이제 비동기 처리를 할 준비가 완료됐습니다.

<br>

<img width="720" alt="image" src="https://github.com/user-attachments/assets/d8e8ca3a-75b6-4492-a16e-8601ddb32a39">

`find`로 원하는 데이터를 찾아온 후, 적절한 전처리를 해 데이터를 가공합니다. 데이터가 원하는 것이 아닐 수도 있고 LLM 모델 학습을 위해서 데이터를 전처리 해야할 필요가 있을 수도 있기 때문입니다. 

결과적으로 분당 80장의 이미지를 처리하는 기존 코드에서 분당 416장을 처리할 수 있었습니다. 프로젝트를 진행하며 느꼈던 점은 크롤링 할 때 고려할 점이 많다는 것입니다. 이 외에도 프로젝트를 진행하며 사이트에서 크롤링을 차단하거나 동일한 사이트, 위치에서도 HTML 구조가 일정하지 않아 엘리먼트 추출 시 어려움이 있었습니다.

<br>

## 참고

- [selenium](https://www.selenium.dev/documentation/webdriver/)
