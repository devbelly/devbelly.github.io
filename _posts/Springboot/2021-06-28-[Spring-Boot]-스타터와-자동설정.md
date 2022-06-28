---
title: "[Spring Boot] 스타터와 자동설정"
tags: springboot
categories:
  - Spring boot
use_math: true
---

### 스타터 이해하기

스프링부트는 복잡한 라이브러리 관련 설정을 위해 스타터를 사용합니다. 스타터는 실행에 필요한 라이브러리들을 관련된 것끼리 묶어서 제공합니다. 만일 데이터베이스 연동을 위해 JPA 라이브러리만 다음과 같이 추가했다고 해봅시다.

```c++
<!-- https://mvnrepository.com/artifact/org.hibernate/hibernate-entitymanager -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-entitymanager</artifactId>
    <version>5.4.32.Final</version>
</dependency>
```

하지만 JPA와 연동하기 위해서는 하이버네이트말고도 spring-orm.jar 과 spring-data-jpa.jar와 같은 라이브러리가 추가적으로 필요합니다. 이러한 문제를 해결하기 위해 스프링부트는 관련된 라이브러리를 묶어 스타터로 제공합니다.

```c++
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

<br>

### POM파일 상속구조

스타터에는 version정보가 명시되어있지 않음에도 정상적으로 작동합니다. 이를 이해하기 위해서는 pom파일 상속구조에 대해 알아야합니다. <ctrl>키를 눌러 해당 소스코드를 확인합니다. 누르게 되면 스프링부트가 제공하는 스타터와 의존성들이 보이게 됩니다.

![img](https://blog.kakaocdn.net/dn/cKPn9N/btq8fXiF29x/ihc47HtxIXOCV67cKG4lA1/img.png)

따라서 우리 프로젝트의 pom.xml에서 버전정보가 명시되어 있지 않더라도, version정보를 상속받아 사용하게 됩니다. 여기서 다시 <ctlr>키를 누른 후 링크를 따라 이동하면 스프링 MVC관련 의존성들을 볼 수 있습니다.

또한 메이븐은 자바 클래스와 마찬가지로 복잡한 pom설정을 부모로부터 상속받아 재사용할 수 있습니다. 이때 사용하는 엘리먼트는 <parent>입니다.

```c++
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.5.2</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
```

스타터의 버전을 변경하거나 스타터에 묶여있는 라이브러리중 일부 버전을 변경하려면 버전을 재정의하거나 프로퍼티를 재정의하는 것 또한 가능합니다.

<br>

### 자동설정

스타터를 통해 라이브러리를 등록했다하더라도 빈을 등록하여 적절히 의존성 주입을 해야 모듈을 정상적으로 사용할 수 있습니다. 이에 대한 처리는 스프링부트가 자동으로 해줍니다. 구체적으로 이에 대한 처리를 해주는 @SpringBootApplication 안에 포함된 @EnableAutoConfiguration을 살펴봅시다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
~
~
}
```

중요한 어노테이션은 아래있는 3개입니다. @SpringBootConfiguration은 환경설정 빈 클래스임을 나타내기 위한 @Configuration과 동일합니다. 단지 차이는 일반 환경설정이 아닌 스프링부트 환경설정임을 나타내기 위함입니다.

스프링 컨테이너 초기화를 위한 어노테이션은 @EnableAutoConfiguration과 @ComponentScan 입니다. 우선 @ComponentScan은 TypeExcludeFilter와 AutoConfigurationExcludedFilter을 제외한 @Configuration, @Repository, @Service, @Controller, @RestController 클래스의 객체를 메모리에 올리는 역할을 합니다.

다음으로 @EnableAutoConfiguration은 사용자가 만든 빈 객체를 사용하기 위한 빈 객체들을 초기화합니다. 예를 들어 파일 업로드를 처리하기 위해서는 사용자가 만든 컨트롤러가 MultipartFile 객체를 이용해야하는데 MultipartFile 객체는 사용자가 업로드할 파일을 설정하기 위해 MultipartFileResolver 객체를 사용해야합니다. ComponentScan이 사용자가 만든 객체를 스캔하여 초기화 한다면 EnableAutoConfiguration은 기본적으로 사용할 빈 객체를 초기화 한다고 볼 수 있습니다.
