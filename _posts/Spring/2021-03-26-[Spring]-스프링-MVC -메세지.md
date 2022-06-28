---
title: "[Spring] 스프링 MVC : 메세지"
tags: spring
categories: spring
use_math: true
---

### <spring:message>

이메일을 전달하기 위해 아래와 같은 폼을 사용했다고 가정하겠습니다.

```c++
	<label>이메일</label>
	<input type="text" name="email">
```

이메일을 사용하는 폼은 굉장히 많습니다. 만일 이메일을 이메일 주소로 변경하려고 한다면 이렇게 하드코딩된 코드들을 일일이 찾아 수정해야합니다. 또한 사용자의 접속 환경에 따라 보여주는 글자를 다르게 하고 싶을 때도 마찬가지입니다.

스프링은 이러한 문제점을 해결하기 위해 메세지 태그를 제공합니다. 메세지 태그를 사용한다면 다음과 같이 바꿀 수 있습니다. <spring:message code="email">을 사용한 곳은 메세지 파일에서 지정한 글자로 바뀌게 됩니다.

```c++
	<label><spring:message code="email"></label>
	<input type="text" name="email">
```

메세지 태그를 사용하기 위해 세가지를 준비해야합니다.

- 문자열을 담은 메세지 파일
- 메세지 파일에서 값을 읽어올 MessageSource 빈
- JSP에서 이를 사용하는 <spring:message> 태그

메세지 파일은 클래스패스안에 위치한 폴더에 .properties 파일을 생성해서 입력해주면 됩니다. 아래는 src/main/resources/message/label.properties 파일입니다.

```c++
member.register=회원가입

term=약관
term.agree=약관동의
next.btn=다음단계

member.info=회원정보
email=이메일
name=이름
password=비밀번호
password.confirm=비밀번호 확인
register.btn=가입 완료

register.done=<strong>{0}님 ({1})</strong>, 회원 가입을 완료했습니다.

go.main=메인으로 이동
```

다음은 MessageSource 빈 객체입니다. Config 폴더에 있는 아무 설정컨트롤러에 만들어도 상관 없습니다.

```java
	@Bean
	public MessageSource messageSource() {
		ResourceBundleMessageSource ms = new ResourceBundleMessageSource();
		ms.setBasename("message.label");
		ms.setDefaultEncoding("UTF-8");
		return ms;
	}
```

messageSource 빈 객체는 MessageSource 인터페이스를 구현한 객체입니다. 다국어 처리를 위해 MessageSource#getMessage()에는 locale 변수를 사용합니다. 이를 구현한 클래스는 자바의 프로퍼티 파일에서 값을 읽어오는 ResourceBundleMessageSource 클래스입니다.

마지막 단계는 <spring:message> 태그를 사용하는 것입니다. 스프링은 태그에서 코드를 읽어 messageSource를 통해 프로퍼티 파일과 일치하는 속성의 값을 출력하게 됩니다.

```c++
<%@ page contentType="text/html; charset=utf-8" %>
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>
<!DOCTYPE html>
<html>
<head>
    <title><spring:message code="member.register" /></title>
</head>
<body>
    <h2><spring:message code="term" /></h2>
    <p>약관 내용</p>
    <form action="step2" method="post">
    <label>
        <input type="checkbox" name="agree" value="true">
        <spring:message code="term.agree" />
    </label>
    <input type="submit" value="<spring:message code="next.btn" />" />
    </form>
</body>
</html>
```

<br>

### <spring:message> 태그의 메세지 인자 처리

label.properties를 보면 인덱스를 통해 값을 전달하는 부분이 있습니다.

```c++
register.done=<strong>{0}님 ({1})</strong>, 회원 가입을 완료했습니다.
```

MessageSource빈은 이를 처리하기 위해 Object 배열을 사용합니다.

```java
...
Object[] args = new Object[1];
args[0]="자바"
messageSource.getMessage("register.done",args,Locale.KOREA);
...
```

사용할 인자는 <spring:message> arguments 속성 등 다양한 방법을 통해 전달할 수 있습니다.

```c++
<spring:message code="register.done" arguments="${registerRequest.name}"/>
```
