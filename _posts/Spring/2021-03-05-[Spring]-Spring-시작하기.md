---
title: "[Spring] Spring 시작하기"
tags: spring
categories:
  - Spring
---

### Maven

스프링이 사용하는 모듈과 플러그인에 대한 관리를 쉽게 해줍니다. 프로젝트의 루트폴더의 pom.xml을 통해 설정파일을 저장할 수 있습니다. 메이븐은 한개의 모듈을 아티팩트라는 단위를 통해 관리하게 되는데, 의존 모듈을 설정하는 것은 소스코드를 컴파일할 때 사용하는 클래스패스에 아티팩트를 추가하는 의미입니다. 아래 5.0.2RELEASE버전의 spring-context모듈을 클래스패스에 추가하기 위한 코드입니다.

```c++
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.0.2.RELEASE</version>
    </dependency>
```

### Repository

아티팩트 파일은 로컬 리포지토리와 원격 리포지토리에서 가져와 사용하게 됩니다. 1차적으로 로컬 리포지토리에서 해당 아티팩트 파일이 있는지 확인한 후 있다면 사용합니다. 만일 없다면 원격 리포지토리에서 찾아 로컬 리포지토리로 다운로드를 한 후 로컬 리포지토리에 있는 아티팩트 파일을 사용하게 됩니다.

프로젝트 폴더로 이동한 후 mvn compile을 수행하면 컴파일에 필요한 파일들을 다운로드 받게 됩니다. 또한 spring-context가 의존하고 있는 아티팩트들 또한 같이 다운로드 받게 됩니다. 이를 transitive dependencies라고 합니다.

<br>

### Spring 시작하기

기본적으로 실행을 위해선 사용할 객체를 위한 클래스,spring 설정 클래스, main 클래스가 필요합니다. 이를 위해 src/main/java 에 chap02 패키지를 만들고 `Greeter.java`, `AppContext.java`, `Main.java`를 작성하겠습니다.

```java
package chap02;

public class Greeter {
	private String format;

	public String greet(String guest) {
		return String.format(format,guest);
	}

	public void setFormat(String format) {
		this.format=format;
	}
}
```

```java
package chap02;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppContext {

	@Bean
	public Greeter greeter() {
		Greeter g = new Greeter();
		g.setFormat("%s, 안녕하세요!");
		return g;
	}
}
```

`@Configuration`은 해당 클래스를 스프링 설정 클래스로 설정합니다. 스프링은 빈 객체를 생성하고 초기화하는 기능을 제공합니다. 메서드에 `@Bean`를 사용하여 해당 메서드를 스프링이 빈 객체로 관리하기 시작합니다. 이때 메서드명은 객체를 구분할 때 사용됩니다.

<br>

```java
package chap02;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class main {

	public static void main(String[] args) {
		AnnotationConfigApplicationContext ctx =
				new AnnotationConfigApplicationContext(AppContext.class);

		Greeter g = ctx.getBean("greeter",Greeter.class);
		String msg = g.greet("스프링");
		System.out.println(msg);
		ctx.close();
	}
}
```

`AnnotationConfingApplicationContext`는 Configuration으로부터 설정을 읽어와 빈 객체를 생성하고 초기화하는 클래스 입니다. 초기화 이후 `getBean`을 통해 만들어진 객체들 중 원하는 객체를 검색해 사용하게 됩니다. `ApplicationContext`는 객체를 종합적으로 관리하여 컨테이너라 칭합니다.

<br>

### 싱글톤 객체

스프링은 하나의 @Bean 어노테이션에 대해 하나의 객체만을 생성하는 싱글톤 범위를 갖습니다.

```java
	Greeter g = ctx.getBean("greeter",Greeter.class);
	Greeter g2 = ctx.getBean("greeter",Greeter.class);
```

Main을 위와 같이 수정하여도 g와 g2는 같은 객체를 가리키게 됩니다.
