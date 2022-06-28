---
title: "[Spring] Dependency Injection"
tags: spring
categories:
  - Spring
---

### Dependency

한 클래스 내부에서 다른 클래스의 함수를 호출할 때 의존관계가 있다고 할 수 있습니다. 구체적으로 호출한 함수 명을 바꿀 때 다른 클래스 소스코드에서도 변경을 요구하는 관계를 의존이라 합니다.

**방식1)**  
하나의 객체에 다른 객체를 넣는 것을 조립이라 칭하겠습니다. 조립을 할 때는 크게 두가지로 나뉩니다. 첫 번째 방식은 클래스 내부에서 직접 조립할 객체를 생성하는 방식입니다.

```java
package spring;

import java.time.LocalDateTime;

public class MemberRegisterService {

	//방식1)
	private MemberDao memberDao=new MemberDao();
    ...
```

<br>

**방식2)**  
위처럼 서비스 함수 내에 직접 Dao객체를 생성하는 것은 편리하지만 유지보수 측면에서는 그닥 좋지 못한 방법입니다. 이유는 Dao클래스는 수 많은 서비스 클래스에서 사용을 하기 마련인데, 만약 Dao클래스를 상속한 다른 Dao클래스를 사용할 일이 생기면 모든 서비스 클래스에서 Dao객체를 생성하는 코드를 수정해야하기 때문입니다. 그래서 아래와 같이 세터 메서드를 작성하거나 생성자를 통해 조립을 하는 방식을 채택합니다.

```java
package spring;

import java.time.LocalDateTime;

public class MemberRegisterService {

	private MemberDao memberDao;

	public MemberRegisterService(MemberDao memberDao) {
		this.memberDao=memberDao;
	}
    ...
```

아래와 같이 Assember의 한 부분만 수정하면 모든 서비스 객체는 cachedMemberDao를 사용할 수 있습니다.

```java
package assembler;

import spring.*;

public class Assembler {
	private MemberDao memberDao;
	private MemberRegisterService regSvc;
	private ChangePasswordService pwdSvc;

	public Assembler() {
		memberDao = new MemberDao(); // new cachedMemberDao(); 로 수정해주면 해결
		regSvc=new MemberRegisterService(memberDao);
		pwdSvc=new ChangePasswordService();
		pwdSvc.setMemberDao(memberDao);
	}
    ...
```

<br>

### Assembler

다양한 객체들의 조립은 Main함수에서 직접해주는 것보다 Assembler 클래스를 따로 두어 구현하면 더 좋습니다. Assembler 클래스는 객체 DI를 관리해주며 서비스 객체를 사용할 수 있도록 도와줍니다.

```java
package assembler;

import spring.*;

public class Assembler {
	private MemberDao memberDao;
	private MemberRegisterService regSvc;
	private ChangePasswordService pwdSvc;

	public Assembler() {
		memberDao = new MemberDao(); // new cachedMemberDao(); 로 수정해주면 해결
		regSvc=new MemberRegisterService(memberDao);
		pwdSvc=new ChangePasswordService();
		pwdSvc.setMemberDao(memberDao);
	}

	public MemberDao getMemberDao() {
		return memberDao;
	}


	public MemberRegisterService getRegSvc() {
		return regSvc;
	}


	public ChangePasswordService getPwdSvc() {
		return pwdSvc;
	}

}
```
