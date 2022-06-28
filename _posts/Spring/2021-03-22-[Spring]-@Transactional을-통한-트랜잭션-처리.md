---
title: "[Spring] @Transactional을 통한 트랜잭션 처리"
tags: spring
categories: spring
use_math: true
---

### Transaction

두 개 이상의 쿼리를 한 작업으로 실행해야 할 때 사용하는 것이 트랜잭션입니다. 한 트랜잭션에 묶인 쿼리들 중 한개라도 실패하면 이전상태로 돌아가는 rollback을 수행하고 모든 쿼리들이 성공하면 해당 상태를 업데이트하는 commit을 수행합니다.

<br>

### @Transactional

트랜잭션 처리를 편리하게 하기 위해 @Transactional 어노테이션을 사용합니다. 트랜잭션 범위에서 실행하고 싶은 메서드에 어노테이션을 붙이면 됩니다.

```java
package spring;

import org.springframework.transaction.annotation.Transactional;

public class ChangePasswordService {

	private MemberDao memberDao;

	@Transactional
	public void changePassword(String email, String oldPwd, String newPwd) {
		Member member = memberDao.selectByEmail(email);
		if (member == null)
			throw new MemberNotFoundException();

		member.changePassword(oldPwd, newPwd);

		memberDao.update(member);
	}

	public void setMemberDao(MemberDao memberDao) {
		this.memberDao = memberDao;
	}

}
```

changePassword()와 update()가 하나의 트랜잭션 범위에 속하게 됩니다.

<br>

### 추가 설정

제대로 동작하기 위해서는 추가적인 두가지 설정을 해야합니다.

1. PlatformTransactionManager

2. @Trasaction 어노테이션 활성화 설정

```java
package config;

import org.apache.tomcat.jdbc.pool.DataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import spring.ChangePasswordService;
import spring.MemberDao;

@Configuration
@EnableTransactionManagement
public class AppCtx {

	@Bean(destroyMethod = "close")
	public DataSource dataSource() {
		..
	}

	@Bean
	public PlatformTransactionManager transactionManager() {
		DataSourceTransactionManager tm = new DataSourceTransactionManager();
		tm.setDataSource(dataSource());
		return tm;
	}

	@Bean
	public MemberDao memberDao() {
		..
	}

	@Bean
	public ChangePasswordService changePwdSvc() {
		..
	}
}
```

PlatformTransactionManager은 스프링이 제공하는 인터페이스입니다. JDBC는 DataSourceTransactionManager 클래스를 PlatformTransactionManager로 사용하게 됩니다. 또한 connection 설정을 위해 DataSource를 주입하여 사용하게 됩니다.

@EnableTransactionManagement 어노테이션은 @Transactional 어노테이션을 찾아 트랜잭션 범위를 활성화하는 기능을 하게 됩니다.

<br>

### Log확인을 통한 트랜잭션 확인

위 설정들을 마치고 다음 메인함수를 실행하면 제대로 트랜잭션 처리가 됐는지 알기 힘듭니다. Logback 모듈을 추가해서 자세한 실행결과를 확인해봅시다.

```java
package main;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import config.AppCtx;
import spring.ChangePasswordService;
import spring.MemberNotFoundException;
import spring.WrongIdPasswordException;

public class MainForCPS {

	public static void main(String[] args) {
		AnnotationConfigApplicationContext ctx =
				new AnnotationConfigApplicationContext(AppCtx.class);

        ChangePasswordService cps =
                ctx.getBean("changePwdSvc", ChangePasswordService.class);
        try {
            cps.changePassword("devbelly@naver.com", "1111", "1234");
            System.out.println("암호를 변경했습니다.");
        } catch (MemberNotFoundException e) {
            System.out.println("회원 데이터가 존재하지 않습니다.");
        } catch (WrongIdPasswordException e) {
            System.out.println("암호가 올바르지 않습니다.");
        }

		ctx.close();

	}
}
```

루트디렉토리에 존재하는 pom.xml에 다음의 아키텍쳐들을 추가하고 메이븐 프로젝트를 업데이트합시다.

```c++
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>1.7.25</version>
		</dependency>

		<dependency>
			<groupId>ch.qos.logback</groupId>
			<artifactId>logback-classic</artifactId>
			<version>1.2.3</version>
		</dependency>
```

Logback 모듈은 로그 메세지 형식과 기록 위치를 설정파일에서 읽어옵니다. src/main/resources에 logback.xml을 추가합시다. level의 값을 DEBUG로 설정하면 자세히 볼 수 있고 INFO로 설정하면 일반적인 실행과 동일하게 설정됩니다.

```c++
<?xml version="1.0" encoding="UTF-8"?>

<configuration>
	<appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
		<encoder>
			<pattern>%d %5p %c{2} - %m%n</pattern>
		</encoder>
	</appender>
	<root level="INFO">
		<appender-ref ref="stdout" />
	</root>

	<logger name="org.springframework.jdbc" level="DEBUG" />
</configuration>
```

<br>

실행하게 되면 콘솔창에 다음 내용과 비슷한 결과를 얻을 수 있습니다.

```c++
2021-03-22 15:29:56,194  INFO o.s.c.a.AnnotationConfigApplicationContext - Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@48ae9b55: startup date [Mon Mar 22 15:29:56 KST 2021]; root of context hierarchy
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by org.springframework.cglib.core.ReflectUtils$1 (file:/C:/Users/devbe/.m2/repository/org/springframework/spring-core/5.0.2.RELEASE/spring-core-5.0.2.RELEASE.jar) to method java.lang.ClassLoader.defineClass(java.lang.String,byte[],int,int,java.security.ProtectionDomain)
WARNING: Please consider reporting this to the maintainers of org.springframework.cglib.core.ReflectUtils$1
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
2021-03-22 15:29:56,569 DEBUG o.s.j.d.DataSourceTransactionManager - Creating new transaction with name [spring.ChangePasswordService.changePassword]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT; ''
3월 22, 2021 3:29:56 오후 org.apache.tomcat.jdbc.pool.ConnectionPool checkPoolConfiguration
WARNING: maxIdle is larger than maxActive, setting maxIdle to: 10
2021-03-22 15:29:56,848 DEBUG o.s.j.d.DataSourceTransactionManager - Acquired Connection [ProxyConnection[PooledConnection[com.mysql.jdbc.JDBC4Connection@71104a4]]] for JDBC transaction
2021-03-22 15:29:56,851 DEBUG o.s.j.d.DataSourceTransactionManager - Switching JDBC Connection [ProxyConnection[PooledConnection[com.mysql.jdbc.JDBC4Connection@71104a4]]] to manual commit
2021-03-22 15:29:56,860 DEBUG o.s.j.c.JdbcTemplate - Executing prepared SQL query
2021-03-22 15:29:56,860 DEBUG o.s.j.c.JdbcTemplate - Executing prepared SQL statement [select * from MEMBER where EMAIL = ?]
2021-03-22 15:29:56,900 DEBUG o.s.j.c.JdbcTemplate - Executing prepared SQL update
2021-03-22 15:29:56,901 DEBUG o.s.j.c.JdbcTemplate - Executing prepared SQL statement [update MEMBER set NAME = ?, PASSWORD = ? where EMAIL = ?]
2021-03-22 15:29:56,904 DEBUG o.s.j.c.JdbcTemplate - SQL update affected 1 rows
2021-03-22 15:29:56,904 DEBUG o.s.j.d.DataSourceTransactionManager - Initiating transaction commit
2021-03-22 15:29:56,904 DEBUG o.s.j.d.DataSourceTransactionManager - Committing JDBC transaction on Connection [ProxyConnection[PooledConnection[com.mysql.jdbc.JDBC4Connection@71104a4]]]
2021-03-22 15:29:56,910 DEBUG o.s.j.d.DataSourceTransactionManager - Releasing JDBC Connection [ProxyConnection[PooledConnection[com.mysql.jdbc.JDBC4Connection@71104a4]]] after transaction
2021-03-22 15:29:56,910 DEBUG o.s.j.d.DataSourceUtils - Returning JDBC Connection to DataSource
암호를 변경했습니다.
2021-03-22 15:29:56,911  INFO o.s.c.a.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@48ae9b55: startup date [Mon Mar 22 15:29:56 KST 2021]; root of context hierarchy
```

<br>

17, 18번째줄에서 트랜잭션을 처리한 후 commit되었음을 알 수 있습니다.

동일한 main함수를 한번 더 실행하게 되면 비밀번호가 바뀐 후 이므로 WrongPasswordException이 발생하게 됩니다.

```c++
2021-03-22 15:34:53,105  INFO o.s.c.a.AnnotationConfigApplicationContext - Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@48ae9b55: startup date [Mon Mar 22 15:34:53 KST 2021]; root of context hierarchy
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by org.springframework.cglib.core.ReflectUtils$1 (file:/C:/Users/devbe/.m2/repository/org/springframework/spring-core/5.0.2.RELEASE/spring-core-5.0.2.RELEASE.jar) to method java.lang.ClassLoader.defineClass(java.lang.String,byte[],int,int,java.security.ProtectionDomain)
WARNING: Please consider reporting this to the maintainers of org.springframework.cglib.core.ReflectUtils$1
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
2021-03-22 15:34:53,492 DEBUG o.s.j.d.DataSourceTransactionManager - Creating new transaction with name [spring.ChangePasswordService.changePassword]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT; ''
3월 22, 2021 3:34:53 오후 org.apache.tomcat.jdbc.pool.ConnectionPool checkPoolConfiguration
WARNING: maxIdle is larger than maxActive, setting maxIdle to: 10
2021-03-22 15:34:53,751 DEBUG o.s.j.d.DataSourceTransactionManager - Acquired Connection [ProxyConnection[PooledConnection[com.mysql.jdbc.JDBC4Connection@71104a4]]] for JDBC transaction
2021-03-22 15:34:53,754 DEBUG o.s.j.d.DataSourceTransactionManager - Switching JDBC Connection [ProxyConnection[PooledConnection[com.mysql.jdbc.JDBC4Connection@71104a4]]] to manual commit
2021-03-22 15:34:53,764 DEBUG o.s.j.c.JdbcTemplate - Executing prepared SQL query
2021-03-22 15:34:53,765 DEBUG o.s.j.c.JdbcTemplate - Executing prepared SQL statement [select * from MEMBER where EMAIL = ?]
2021-03-22 15:34:53,786 DEBUG o.s.j.d.DataSourceTransactionManager - Initiating transaction rollback
2021-03-22 15:34:53,786 DEBUG o.s.j.d.DataSourceTransactionManager - Rolling back JDBC transaction on Connection [ProxyConnection[PooledConnection[com.mysql.jdbc.JDBC4Connection@71104a4]]]
2021-03-22 15:34:53,787 DEBUG o.s.j.d.DataSourceTransactionManager - Releasing JDBC Connection [ProxyConnection[PooledConnection[com.mysql.jdbc.JDBC4Connection@71104a4]]] after transaction
2021-03-22 15:34:53,787 DEBUG o.s.j.d.DataSourceUtils - Returning JDBC Connection to DataSource
암호가 올바르지 않습니다.
2021-03-22 15:34:53,788  INFO o.s.c.a.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@48ae9b55: startup date [Mon Mar 22 15:34:53 KST 2021]; root of context hierarchy
```

14, 15번째줄에서 해당 트랜잭션이 rollback되었음을 알 수 있습니다.

<br>

### @Transactional 적용 메서드의 롤백처리

프록시를 통해 실제 객체를 호출하여 트랜잭션을 처리하는 도중 RumtimeException이 발생하면 처리를 종료하고 rollback을 수행하게 됩니다. 이러한 이유때문에 WrongIdPasswordException을 구현할 때 RumtimeException을 상속받아 처리했습니다.
