---
title: "[Spring] Autowired 어노테이션"
tags: spring
categories:
  - Spring
---

### @Autowired at field

이전 글에서 Autowired를 사용해 빈을 DI할 수 있다고 설명했습니다. 또한 사용하면 설정 클래스에서 생성자나 세터 메서드를 통해 DI를 하는 코드를 없앨 수 있습니다. @Autowired는 필드나 세터 메서드에 위치할 수 있는데 아래 코드는 필드에 적용한 예시입니다.

```java
public class MemberRegisterService {

	@Autowired
	private MemberDao memberDao;
    ...
```

위와 같이 했다면 설정 클래스에서 DI를 하는 코드를 제거할 수 있습니다.

```java
	@Bean
	public MemberRegisterService memberRegSvc() {
		return new MemberRegisterService();
//		return new MemberRegisterService(memberDao()); 제거!
	}
```

<br>

### @Autowired at setter method

```java
public class ChangePasswordService {

	private MemberDao memberDao;

	@Autowired
	public void setMemberDao(MemberDao memberDao) {
		this.memberDao=memberDao;
	}
    ...
```

역시 설정 클래스에서 DI코드를 제거하도록 합시다.

```java
	@Bean
	public ChangePasswordService changePwdSvc() {
		ChangePasswordService pwdSvc = new ChangePasswordService();
//		pwdSvc.setMemberDao(memberDao()); 제거!
		return pwdSvc;
	}
```

정상적으로 실행이 되는 것을 알 수 있습니다.

<br>

### @Qualifier

@Autowired는 자신이 가지고 있는 컨테이너를 확인해 빈을 DI 해줍니다. 자동 주입가능한 빈이 2개 이상일 경우는 Exception이 발생합니다. 아래는 에러의 일부입니다.

```java
@Configuration
public class AppCtx {

    //MemberDao Bean이 2개 존재
	@Bean
	public MemberDao memberDao1() {
		return new MemberDao();
	}
	@Bean
	public MemberDao memberDao2() {
		return new MemberDao();
	}
    ...
```

```c++
Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'spring.MemberDao' available: expected single matching bean but found 2: memberDao1,memberDao2
	at org.springframework.beans.factory.config.DependencyDescriptor.resolveNotUnique(DependencyDescriptor.java:215)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1113)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1062)
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredMethodElement.inject(AutowiredAnnotationBeanPostProcessor.java:659)
	... 14 more
```

오른쪽에 보시면 expected single matching bean but found 2에 명확한 내용이 나와 있습니다.

이를 해결하기 위해 @Qauilfier 어노테이션을 사용합니다. 닉네임과 비슷한 개념으로 생각하시면 됩니다. 컨테이너에 담는 빈에 닉네임을 붙이고 자동 주입을 위해 설정한 @Autowired에서 사용할 닉네임을 적는 것입니다.

설정 클래스의 @Bean 어노테이션과 함께 @Quailfier을 통해 닉네임을 짓고

```java
@Configuration
public class AppCtx {

	@Bean
	@Qualifier("mainDatabase")
	public MemberDao memberDao1() {
		return new MemberDao();
	}
	@Bean
	public MemberDao memberDao2() {
		return new MemberDao();
	}
    ..
```

주입을 위한 @Autowired에 사용할 닉네임을 적습니다.

```java
public class MemberRegisterService {

	@Autowired
	@Qualifier("mainDatabase")
	private MemberDao memberDao;
    ...
```

```java
public class ChangePasswordService {

	private MemberDao memberDao;

	@Autowired
	@Qualifier("mainDatabase")
	public void setMemberDao(MemberDao memberDao) {
		this.memberDao=memberDao;
	}
```

<br>

### 기본 한정자

다음 예시는 MemberDao에 해당하는 Bean이 2개임에도 불구하고 에러가 나지 않습니다.

```java
@Configuration
public class AppCtx {

	@Bean
	public MemberDao memberDao() {
		return new MemberDao();
	}
	@Bean
	public MemberDao memberDao2() {
		return new MemberDao();
	}
```

```java
public class MemberRegisterService {

	@Autowired
	private MemberDao memberDao;
```

```java
public class ChangePasswordService {

	private MemberDao memberDao;

	@Autowired
	public void setMemberDao(MemberDao memberDao) {
		this.memberDao=memberDao;
	}
```

이유는 @Qualifier 을 사용하지 않으면 기본 한정자를 생성하기 때문입니다. 설정 클래스에서 @Qualifier 이 없으므로 기본 한정자를 함수의 이름으로 생성하게 됩니다. 마찬가지로 DI를 받아야하는 클래스에서 사용하지 않으면 필드명 또는 세터 메서드의 함수인자를 기본 한정자로 사용합니다. 세 가지 모두 memberDao로 일치하므로 오류가 발생하지 않습니다.

<br>

### 상위/하위 타입관계의 자동 주입

MemberDao를 상속하는 CachedMemberDao를 만들고 설정 클래스에서 다음과 같이 두 개의 Dao를 만들어 봅시다.

```java
@Configuration
public class AppCtx {

	@Bean
	public MemberDao memberDao1() {
		return new MemberDao();
	}

	@Bean
	public CachedMemberDao cachedMemberDao2() {
		return new CachedMemberDao();
	}
```

이를 실행하면 Dao를 사용하는 빈 객체들은 MemberDao사용한다고 적어놓았지만 일치하는 빈을 특정할 수 없다는 에러메세지를 발생시킵니다.

```c++
Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'spring.MemberDao' available: expected single matching bean but found 2: memberDao1,cachedMemberDao2
	at org.springframework.beans.factory.config.DependencyDescriptor.resolveNotUnique(DependencyDescriptor.java:215)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1113)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1062)
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredMethodElement.inject(AutowiredAnnotationBeanPostProcessor.java:659)
	... 14 more
```

이유는 일치하는 빈 객체를 찾기 위해 MemberDao 클래스의 빈들을 확인할 때, CachedMemberDao의 객체는 MemberDao를 상속했으므로 스프링은 @Autowired가 작동할 때, 어느 객체와 매칭되는지 특정할 수 없기 때문입니다.

@Qualifier을 사용하거나 @Autowired에서 사용하는 필드나 세터 메서드 부분을 CachedMemberDao로 더 정확하게 적게되면 해결됩니다.

<br>

### @Autowired 필수 여부

다음과 같은 프로그램이 있다고 해봅시다. 객체가 null이 아니라면 데이터가 있다고 출력하고 null이라면 그에 따른 출력 포멧을 따라야합니다.

```java
package spring;

import org.springframework.beans.factory.annotation.Autowired;

public class testPrint {

	@Autowired
	private FastMemberDao memberDao;

	public void print() {
		if(memberDao!=null) {
			System.out.print("data exist\n");
		}
		else {
			System.out.println("no data\n");
		}
	}
}
```

하지만 @Autowired는 알맞은 빈 객체를 찾지 못하면 Exception을 발생시킵니다. 이때는 세가지 방식으로 Autowired에 대한 강제성을 해제할 수 있습니다.

#### 1)required

```java
package spring;

import org.springframework.beans.factory.annotation.Autowired;

public class testPrint {

	@Autowired(required=false)
	private FastMemberDao memberDao;

	...
}
```

#### 2)Optional (자세한 것은 자바 8의 Optional을 참고해주세요)

#### 3)@Nullable 어노테이션

<br>

### @Nullable vs required

@Autowired에 대한 강제성을 해제한 후, @Autowired를 통해 객체을 넣는 필드를 초기화 할 때 조심해야할 것이 있습니다. @Nullable은 일치하는 빈 객체를 찾지 못하면 해당 필드에 null을 넣어 초기화 하는 반면 required = false는 해당 필드에 대해 아무런 행동을 취하지 않습니다. 만일 기본생성자를 통해 필드에 값이 들어가 있다면 해당 필드를 사용하게 됩니다.
