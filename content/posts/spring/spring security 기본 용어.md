---
title: "spring security 기본 용어"
date: 2022-07-16T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - spring
---

프로젝트의 코드 중 시큐리티에 대한 내용이 잘 이해가 되지 않아 복습하기 위해 작성한 글입니다.

## SecurityContextHolder

SecurityContext를 담고 있는 클래스입니다. ThreadLocal 전략을 사용하고 아래와 같은 구조를 띱니다.

![image](https://user-images.githubusercontent.com/67682840/179336516-fb7bc74e-48ca-4ac4-bfcf-86d560b4394c.png)

위 구조를 아래 사진의 코드를 통해 확인할 수 있습니다.

![image](https://user-images.githubusercontent.com/67682840/179339760-37dcb3bc-3f6f-498e-b05e-2e71895a847b.png)

## @EnableWebSecurity

해당 어노테이션을 통해 스프링이 찾을 수 있게 하고 해당 클래스를 global websecurity에 적용할 수 있도록 합니다.

## WebSecurityConfigurerAdapter

자신만의 시큐리티 설정을 하고 싶을 때 상속해서 사용하면 됩니다. 상속시 auto-configuration을 사용하지 않게 됩니다. cors, csrf, filter, authenticate, exception handler 등을 설정할 수 있게 됩니다.

## AuthenticationManager

어떻게 authenticate를 할지 정의한 인터페이스입니다. 반환된 authentication 객체는 AuthenticationManager를 호출한 컨트롤러에 의해 SecurityContextHolder 안에 저장됩니다.

## UsernamePasswordAuthenticationToken

UsernamePasswordAuthenticationToken은 AbstractAuthenticationToken을 상속받고 AbstractAuthenticationToken은 Authentication을 구현한 추상 클래스입니다.

![image](https://user-images.githubusercontent.com/67682840/179337293-bf7117ea-b185-4342-a247-8e2740a9b372.png)

![image](https://user-images.githubusercontent.com/67682840/179337334-cf77b602-78b7-4862-aa5f-d106a65327a3.png)

UsernamePasswordAuthenticationToken을 이용해서 AuthenticationManager은 인증을 진행하고 인증된 객체를 SecurityContextHolder에 담게 됩니다. 이 토큰을 생성하는 부분은 코드에서 두 군데가 있습니다. 첫번째는 login API를 호출하는 부분입니다.

![image](https://user-images.githubusercontent.com/67682840/179337558-7141c570-9ef7-4a99-85f2-39d2748bd518.png)

이는 Token의 두가지 생성자 중 첫 번째 생성자를 호출하게 됩니다. `setAuthenticated(false);` 가 호출됨을 알 수 있습니다. 즉 인증되지 않은 토큰을 생성한 부분입니다.

![image](https://user-images.githubusercontent.com/67682840/179337619-e5076eab-25e5-4a4d-997e-252621a58f09.png)

두번째는 커스텀 필터를 구현한 부분입니다. 요청 헤더에 JWT 필드가 존재한다면 사용자의 정보를 파싱해서 토큰을 생성한 후 SecurityContextHolder에 UsernameAuthenticationToken을 담는 것을 확인할 수 있습니다.

![image](https://user-images.githubusercontent.com/67682840/179339676-ca96d459-bf6d-4207-81e2-f62912f4c02c.png)

이때 호출되는 생성자를 확인해보면 `setAuthenticated(true)`가 포함됨을 알 수 있습니다.

![image](https://user-images.githubusercontent.com/67682840/179339704-1bdbb581-74c2-42c5-a501-b78324eb9cf0.png)

그리고 시큐리티 설정 클래스에 loginForm() 설정을 하게 되면 UsernamepasswordAuthenticationFilter가 활성화 되지만 하지 않는다면 활성화 되지 않는 것을 확인할 수 있습니다.

![formLogin 없을때](https://user-images.githubusercontent.com/67682840/179339955-d77dde90-38e6-4b9c-9d0b-5945127fa1cb.png)

<p align=center>
	[fomrLogin() 없을 때]
</p>

![loginForm 설정시](https://user-images.githubusercontent.com/67682840/179340009-bab3b58a-88bd-4634-8ee1-b1b185c882bf.png)

<p align=center>
		[fomrLogin() 있을 때]
</p>

프로젝트에서는 jwt인증방식을 사용하므로 UsernamePasswordAuthenticationFilter를 대체할 custom Filter을 사용하게 됩니다. 해당 필터가 작동되는 순서를 정하기 위해 `addFilterBefore()`을 사용하게 됩니다.

![image](https://user-images.githubusercontent.com/67682840/179340064-cfff55d4-5b09-4351-a05e-aea4a70932f2.png)

## UserDetailsService

AuthenticationManager의 구현체인 ProviderManager은 인증에 대한 구현을 AuthenticationProvider에게 위임합니다. AuthenticationProvider은 사용자 이름을 기반으로 사용자 세부 정보를 검색하기 위해 UserDetailsService를 사용합니다. 이는 interface로 데이터베이스에서 사용자 정보를 얻기 위해 loadUserByUsername 메서드를 구현해야합니다. UserDetails를 리턴하며 아래 사진은 `DaoAuthenticationProvider`가 UserDetailsService를 사용하는 코드입니다.

![image](https://user-images.githubusercontent.com/67682840/179340114-b4e7bd75-ea38-4a6d-a6cc-c50e1604e19f.png)

## authenticationManagerBean
AuthenticationManager을 외부에서 사용하기 위해 authenticateManagerBean을 사용했습니다. 시큐리티에 아래와 같은 설정을 추가함으로써 Controller에서 AuthenticationManager을 주입하여 사용할 수 있게 됩니다.

![image](https://user-images.githubusercontent.com/67682840/179340177-6fab3f6b-0631-4f80-9db9-c9891936c4ca.png)

![image](https://user-images.githubusercontent.com/67682840/179340418-e1d9cac1-4c0c-4476-808a-d1c918ef90ca.png)

## authenticationManagerBuilder

애플리케이션에 적합한 UserDetails를 구현한 클래스를 얻기 위해 UserDetailsService를 customize를 했다면 AuthenticationProvider에게 알려주어야합니다. 이는 아래 메서드를 override함으로써 가능합니다.

![image](https://user-images.githubusercontent.com/67682840/179340976-76ec8c83-5270-4da4-84cc-4130c198fda3.png)

## exceptionHandling.authenticationEntryPoint

ExceptionTranslationFilter은 2가지 종류의 예외에 대해서 처리합니다. 인증예외처리와 인가예외처리가 있는데 이 중 authenticationEntryPoint는 인증예외처리를 담당합니다.

![image](https://user-images.githubusercontent.com/67682840/179341000-0d89e3b9-2a7b-4b86-bb2f-0b6d52be914b.png)

#### 참고

- https://stackoverflow.com/questions/64526372/when-should-i-override-the-configureauthenticationmanagerbuilder-auth-from-spr

- https://doozi0316.tistory.com/entry/Spring-Security-Spring-Security%EC%9D%98-%EA%B0%9C%EB%85%90%EA%B3%BC-%EB%8F%99%EC%9E%91-%EA%B3%BC%EC%A0%95

- https://www.bezkoder.com/spring-boot-jwt-authentication/
