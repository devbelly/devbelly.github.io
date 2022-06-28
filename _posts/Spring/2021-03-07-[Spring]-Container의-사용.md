---
title: "[Spring] Container의 사용"
tags: spring
categories: spring
use_math: true
---

### Container

이전 글에서 설명한 Assembler의 기능을 Spring이 제공을 해줍니다. 이를 위해 설정 클래스를 만들고 @Bean 어노테이션을 통해 객체를 생성합니다.

```java
package config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import spring.ChangePasswordService;
import spring.MemberDao;
import spring.MemberRegisterService;

@Configuration
public class AppCtx {

	@Bean
	public MemberDao memberDao() {
		return new MemberDao();
	}

	@Bean
	public ChangePasswordService changePwdSvc() {
		ChangePasswordService pwdSvc = new ChangePasswordService();
		pwdSvc.setMemberDao(memberDao());
		return pwdSvc;
	}

	@Bean
	public MemberRegisterService memberRegSvc() {
		return new MemberRegisterService(memberDao());
	}
}
```

`@Configuration` 어노테이션을 통해 해당 클래스를 설정 클래스로 만들고 `@Bean` 어노테이션을 통해 빈 객체를 생성합니다.

설정파일을 통해 컨테이너에 담을 객체를 정했다면 Main함수에서 Assembler대신 Container을 통해 객체에 접근합니다.

```java
...
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class MainForSpring {

	private static ApplicationContext ctx = null;

	public static void main(String[] args) throws IOException{
		Scanner scanner = new Scanner(System.in);
		ctx=new AnnotationConfigApplicationContext(AppCtx.class);//설정 클래스를 인자로 설정
        ...
```

<br>

### 생성자 vs 세터 메서드

어노테이션을 사용하지 않고 객체를 DI하는 방식으로는 생성자 방식과 세터 메서드 방식이 있습니다. 둘 중 어느 것을 사용해야할 지는 상황에 따라 다릅니다.

생성자 방식은 의존 객체를 여러개 주입할 때, 한 번에 Injection이 가능하다는 장점이 있습니다. 단점은 어느 객체를 몇 번째 인자로 주어야 하는지 알기가 어렵기 때문에 본인이 외우고 있거나 직접 소스코드를 확인해야한다는 점이다.

반대로 세터 메서드 방식은 굳이 어느 위치에 인자를 넣어야 하는지 알 필요가 없습니다. 본인이 넣을 의존 객체만 명확히 알고 있다면 set()을 호출하여 객체를 Injection하면 되기 때문입니다. 단점은 의존 객체를 넣지 않아도 사용할 객체가 생성되므로 프로그래머의 실수가 발생할 수 있다는 점입니다.

<br>

### 싱글톤의 원리

스프링은 하나의 @Bean 어노테이션에 하나의 객체만은 생성합니다. 그래서 아래 코드는 $memberDao()$함수를 호출해도 동일한 $memberDao$를 리턴하므로 문제가 없는 코드입니다.

```java
	...
    @Bean
	public ChangePasswordService changePwdSvc() {
		ChangePasswordService pwdSvc = new ChangePasswordService();
		pwdSvc.setMemberDao(memberDao()); // 이상없음
		return pwdSvc;
	}

	@Bean
	public MemberRegisterService memberRegSvc() {
		return new MemberRegisterService(memberDao()); // 이상없음
	}
    ...
```

이 방식이 가능한 이유는 스프링은 설정파일을 통해 컨테이너를 생성할 때, 설정 클래스를 그대로 사용하지 않기 때문입니다. 설정 클래스를 참고하여 스프링은 새로운 설정 클래스를 만들어 새로운 설정 클래스를 객체 생성에 이용합니다.

내부적으로는 더 복잡하나 아래와 같은 클래스를 생성합니다.

```java
public class AppCtxExt extends AppCtx{
	private Map<String,Object> beans = ...;

    @Override
    public MemberDao memberDao(){
    	if(!beans.containsKey("memberDao"))
        	beans.put("memberDao",super.memberDao());

        return (MemberDao)beans.get("memberDao");

    ...
}
```

<br>

### 두 개의 설정파일 사용하기

설정파일을 나누어 보자. 아래와 같이 설정할 수 있을 것입니다.

```java

//AppConf1.java
package config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import spring.MemberDao;
@Configuration
public class AppConf1 {

	@Bean
	public MemberDao memberDao() {
		return new MemberDao();
	}
}
```

```java
//AppConf2.java
package config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import spring.ChangePasswordService;
import spring.MemberRegisterService;

import spring.*;

@Configuration
public class AppConf2 {

	@Autowired
	private MemberDao memberDao;

	@Bean
	public MemberRegisterService memberRegSvc() {
		return new MemberRegisterService(memberDao);
	}

	@Bean
	public ChangePasswordService changePwdSvc() {
		ChangePasswordService changePwdSvc = new ChangePasswordService();
		changePwdSvc.setMemberDao(memberDao);
		return changePwdSvc;
	}
}
```

달라진 점은 AppConf2.java에 @Autowired라는 어노테이션을 사용했고 memberDao()함수를 호출하는 대신 memberDao필드를 이용하는 것입니다.

@Autowired는 어노테이션을 통해 DI를 가능하게 합니다. 즉, 스프링 빈에 의존하는 다른 빈을 주입하는 것입니다. @Configuration 또한 스프링 컨테이너는 내부적으로 빈으로 생성하여 관리하므로 설정 클래스 필드에 @Autowired를 사용해도 DI가 가능합니다(빈에 의존하는 다른 빈을 주입하므로).

설정 클래스를 나누었다면 컨테이너를 생성할 때, 두개의 설정파일을 배열을 통해 건네주면 됩니다.

```java
public class MainForSpring {

	private static ApplicationContext ctx = null;

	public static void main(String[] args) throws IOException{
		Scanner scanner = new Scanner(System.in);
		ctx=new AnnotationConfigApplicationContext(AppConf1.class,AppConf2.class);
        	...
```

또한 다른 방식으로 @Import를 사용할 수도 있습니다.

```java
package config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

import spring.MemberDao;
@Configuration
@Import(AppConf2.class)
public class AppConf1 {

	@Bean
	public MemberDao memberDao() {
		return new MemberDao();
	}
}
```

이 방식을 사용하게 되면 컨테이너 생성시 AppConf1.class만 인자로 전달하여도 됩니다.
