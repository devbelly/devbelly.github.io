---
layout: post
title: "[Spring Boot] Spring Boot 시작하기"
tags: springboot
categories: springboot
use_math: true
---

### WebApplicationType

스프링부트는 웹 애플리케이션이나 일반 자바 애플리케이션으로 실행가능합니다. 아무런 설정을 하지 않았을 때는 웹 애플리케이션으로 실행이 되어 내장된 톰캣서버가 실행됩니다. 일반 자바 애플리케이션으로 실행하기 위해서는 SpringApplication의 WebApplicationType을 NONE으로 수정해야합니다. 만일 값이 NONE이 아닌 SERVLET으로 되어있다면 다시 웹 애플리케이션으로 실행됩니다.

```java
@SpringBootApplication
public class Chapter01Application {
	public static void main(String[] args) {
		SpringApplication application =
				new SpringApplication(Chapter01Application.class);
		application.setWebApplicationType(WebApplicationType.NONE);
		application.run(args);
	}
}
```

<br>

### 외부 프로퍼티

매번 애플리케이션을 수정할때마다 소스코드를 일일이 수정하는 것은 굉장히 번거롭습니다. 이를 위해 스프링부트는 모든 설정을 담당하고 있는 application.properties를 제공합니다. 프로젝트내에 src/main/resources안에 있으며 자바 소스코드의 설정보다 우선순위가 높습니다. 이 파일을 확인하여 Properties 객체에서 Setter 메서드를 호출하게 의존성 주입을 하게 됩니다. 아래 외부 프로퍼티는 웹 애플리케이션으로 동작을 위한 servlet과 기본 포트 8080을 8000으로 바꾼 것 입니다.

```c++
##WebApplication Type Setting
spring.main.web-application-type=servlet

##Server Setting
server.port:8000
```

<br>

### 자동 컴포넌트 스캔

사용자가 만든 컨트롤러가 빈으로 등록되는 것을 확인하기 위해 다음과 같은 컨트롤러를 작성했습니다. 포트 충돌을 막기 위해 재실행을 하면 아래 사진과 같은 결과를 얻을 수 있습니다.

```java
package com.rubypaper.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class BoardController {
	public BoardController() {
		System.out.println("==> BoardController 생성");
	}

	@GetMapping("/hello")
	public String hello(String name) {
		return "hello: "+name;
	}
}
```

![img](https://blog.kakaocdn.net/dn/bP9HfS/btq8f92EiKI/9lEmZhfGS7mZUmcSbvMf5k/img.png)

스프링에서는 @RestController 어노테이션을 사용했더라도 xml 설정파일에 <context:component-scan>을 설정하지 않으면 컨테이너가 컨트롤러를 빈으로 등록하지 않습니다. 하지만 위 사진에서도 보이듯 정상적으로 빈 객체가 생성됨을 확인할 수 있습니다. 이것이 가능한 이유는 main 메서드 위에 있는 @SpringbootApplication 어노테이션 덕분입니다.

ctrl키를 눌러 해당 어노테이션을 확인해보면 다음과 같습니다.

![img](https://blog.kakaocdn.net/dn/dpI7WU/btq8hkQrijP/zOqCKD3sf0f3bCwblfqWzK/img.png)

@ComponentScan 어노테이션이 포함되어있음을 알 수 있습니다. 이는 현재 main메서드가 포함된 패키지를 베이스 패키지로하여 해당 패키지를 포함한 하위 패키지들에 포함된 어노테이션들을 확인하여 컨테이너에 빈 객체를 만들게 됩니다. 이 때문에 Spring boot에서는 클래스를 만들 때, 어느 패키지에 해당 클래스를 포함할 지 고려해야합니다. 만일 com.rubypaper이 아닌 다른 패키지에 컨트롤러를 만들었다면 빈 객체로 등록되지 않았을 것입니다. 해당 패키지를 스캔범위에 포함하려면 아래와 같이 적을 수 있습니다.

![img](https://blog.kakaocdn.net/dn/cTR5CB/btq8gynRo3l/Ij0bKOXF3MXmX3S4PLgQDK/img.png)

<br>

### Lombok 라이브러리 사용하기

@RestController 어노테이션은 리턴값이 문자열이면 해당 문자열을 그대로 리턴하게 되고 VO(value Object)일 경우에는 해당 클래스를 json 형식으로 변환하여 리턴하게 됩니다. 다음은 json 변환을 테스트하기 위해 작성한 BoardVO 클래스 입니다.

```java
package com.rubypaper.domain;

import java.util.Date;

public class BoardVO {
	private int seq;
	private String title;
	private String writer;
	private String content;
	private Date createDate=new Date();
	private int cnt=0;
	public int getSeq() {
		return seq;
	}
	public void setSeq(int seq) {
		this.seq = seq;
	}
	public String getTitle() {
		return title;
	}
	public void setTitle(String title) {
		this.title = title;
	}
	public String getWriter() {
		return writer;
	}
	public void setWriter(String writer) {
		this.writer = writer;
	}
	public String getContent() {
		return content;
	}
	public void setContent(String content) {
		this.content = content;
	}
	public Date getCreateDate() {
		return createDate;
	}
	public void setCreateDate(Date createDate) {
		this.createDate = createDate;
	}
	public int getCnt() {
		return cnt;
	}
	public void setCnt(int cnt) {
		this.cnt = cnt;
	}

}
```

하지만 매번 함수를 추가해야하는 것은 굉장히 까다롭습니다. 이를 편리하게 관리하게 해주는 Lombok 라이브러리를 사용해보도록 합시다. pom.xml에 lombok 라이브러리를 추가하고 별도로 이클립스 설치폴더에 lombok 라이브러리를 또 추가해주어야합니다. 설치가 끝났다면 BoardVO는 다음과 같이 변경할 수 있습니다.

```java
package com.rubypaper.domain;

import java.util.Date;

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString
public class BoardVO {
	private int seq;
	private String title;
	private String writer;
	private String content;
	private Date createDate=new Date();
	private int cnt=0;
}
```

lombok 사용을 통해 기존보다 간결해졌음을 확인할 수 있습니다.
