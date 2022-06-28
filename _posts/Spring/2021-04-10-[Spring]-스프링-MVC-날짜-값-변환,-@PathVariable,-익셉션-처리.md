---
title: "[Spring] 스프링 MVC : 날짜 값 변환,@PathVariable, 익셉션 처리"
tags: spring
categories: spring
use_math: true
---

### 날짜 값 변환

사용자가 설정한 기간 사이에 회원가입을 한 멤버들을 확인하는 요청을 하고 싶은 경우가 있습니다. 이를 위해 MemberDao 클래스에 다음과 같은 메서드를 추가할 수 있습니다.

```java
private RowMapper<Member> memRowMapper =
	new RowMapper<Member>() {
		@Override
		public Member mapRow(ResultSet rs, int rowNum)
				throws SQLException {
			Member member = new Member(rs.getString("EMAIL"),
					rs.getString("PASSWORD"),
					rs.getString("NAME"),
					rs.getTimestamp("REGDATE").toLocalDateTime());
			member.setId(rs.getLong("ID"));
			return member;
		}
	};


public List<Member> selectByRegdate(LocalDateTime from, LocalDateTime to) {
	List<Member> results = jdbcTemplate.query(
			"select * from MEMBER where REGDATE between ? and ? " +
					"order by REGDATE desc",
			memRowMapper,
			from, to);
	return results;
}
```

이 메서드를 사용하기 위해 컨트롤러에서는 입력폼에서 from과 to를 커맨드 객체로 입력받아야합니다. 아래와 같은 커맨드 클래스를 작성합시다.

```java
package controller;

import java.time.LocalDateTime;

import org.springframework.format.annotation.DateTimeFormat;

public class ListCommand {
	private LocalDateTime from;
	private LocalDateTime to;

	public LocalDateTime getFrom() {
		return from;
	}

	public void setFrom(LocalDateTime from) {
		this.from = from;
	}

	public LocalDateTime getTo() {
		return to;
	}

	public void setTo(LocalDateTime to) {
		this.to = to;
	}
}
```

스프링 MVC는 입력폼에서 받은 문자열을 Long과 int와 같은 기본적인 데이터 형으로 바꿔주는 작업을 해줍니다. 하지만 LocalDateTime와 같은 데이터 형으로 바꾸기 위해선 추가적으로 @DateTimeFormat 어노테이션을 설정해야합니다.

```java
package controller;

import java.time.LocalDateTime;

import org.springframework.format.annotation.DateTimeFormat;

public class ListCommand {

	@DateTimeFormat(pattern = "yyyyMMddHH")
	private LocalDateTime from;
	@DateTimeFormat(pattern = "yyyyMMddHH")
	private LocalDateTime to;

	public LocalDateTime getFrom() {
		return from;
	}

	public void setFrom(LocalDateTime from) {
		this.from = from;
	}

	public LocalDateTime getTo() {
		return to;
	}

	public void setTo(LocalDateTime to) {
		this.to = to;
	}
}
```

커맨드 객체 설정과 이 객체들을 사용할 메서드(selectByRegDate())를 작성했으므로 요청 경로와 이를 엮어줄 컨트롤러를 작성합시다. 작성이 끝난 후에는 ControllerConfig에 @Bean 어노테이션을 통해 빈 객체를 추가해야 합니다. 컨트롤러가 리턴하는 뷰 페이지 또한 작성해줍시다.

```java
package controller;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.Errors;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;

import spring.Member;
import spring.MemberDao;

@Controller
public class MemberListController {

	@Autowired
	private MemberDao memberDao;


	@RequestMapping("/members")
	public String list(
			@ModelAttribute("cmd") ListCommand listCommand, Model model) {
		if (listCommand.getFrom() != null && listCommand.getTo() != null) {
			List<Member> members = memberDao.selectByRegdate(
					listCommand.getFrom(), listCommand.getTo());
			model.addAttribute("members", members);
		}
		return "member/memberList";
	}
}
```

```c++
<%@ page contentType="text/html; charset=utf-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ taglib prefix="tf" tagdir="/WEB-INF/tags" %>
<!DOCTYPE html>
<html>
<head>
    <title>회원 조회</title>
</head>
<body>
    <form:form modelAttribute="cmd">
    <p>
        <label>from: <form:input path="from" /></label>
        <form:errors path="from" />
        ~
        <label>to:<form:input path="to" /></label>
        <form:errors path="to" />
        <input type="submit" value="조회">
    </p>
    </form:form>

    <c:if test="${! empty members}">
    <table>
        <tr>
            <th>아이디</th><th>이메일</th>
            <th>이름</th><th>가입일</th>
        </tr>
        <c:forEach var="mem" items="${members}">
        <tr>
            <td>${mem.id}</td>
            <td><a href="<c:url value="/members/${mem.id}"/>">
                ${mem.email}</a></td>
            <td>${mem.name}</td>
            <td><tf:formatDateTime value="${mem.registerDateTime }"
                                   pattern="yyyy-MM-dd" /></td>
        </tr>
        </c:forEach>
    </table>
    </c:if>
</body>
</html>
```

커맨드 객체에 정보가 있을 때만 Model객체에 결과값을 담아 뷰 페이지에 출력하도록 했습니다. 즉 처음에 /컨텍스트경로/members 에 접근하면 from과 to를 입력하기 위한 폼만 존재합니다.

![img](https://blog.kakaocdn.net/dn/IFMxI/btq2ge48r3q/3nUH1r0zXwqOLEvA9w88l1/img.png)

입력 데이터 형식에 맞춰 작성한 후 조회를 누르게 되면 아래와 같은 결과가 발생합니다.

![img](https://blog.kakaocdn.net/dn/cXe7F6/btq2gEhYRM0/KjBzKRlrY9bk3ZCukkad61/img.png)

<br>

### 변환 에러 처리

위 예제에서는 입력데이터를 포맷에 정확히 맞추어 입력해서 오류가 발생하지 않습니다. 하지만 사용자가 입력한 데이터를 LocalDatetime으로 정확히 바꾸지 못한다면 에러가 발생할 것입니다. 이를 처리하기 위해 Controller에 Error 객체를 사용하면 처리가 가능합니다. Validation API에는 없지만 추가적으로 설정한 hibernate validator에 @DateTimeFormat이 존재하기 때문입니다. 만약 타입 변환에 실패한다면 에러 코드로 typeMismatch를 사용합니다. 파라미터로 error를 넘겨줄 때 유의할 점은 커맨드 객체 위에 error 파라미터는 적어야 하는 점입니다.

```java
	@RequestMapping("/members")
	public String list(
			@ModelAttribute("cmd") ListCommand listCommand,
			Errors errors, Model model) {
		if (errors.hasErrors()) {
			return "member/memberList";
		}
        ....
```

입력 폼을 커맨드 객체로 바꿀때 WebdataBinder가 관여합니다. 앞서 사용자 요청이 들어오면 DispatcherServlet이 사용자 요청과 매핑된 메서드를 찾기 위해 HandlerAdapter을 사용함을 살펴 보았습니다. HandlerAdapter은 데이터 변환을 위해 WebdataBinder을 사용하고 WebdataBinder은 DefaultFormattingConversionService 객체에 이를 위임합니다. 이 객체는 처음 @EnableWebMvc 어노테이션을 하는 과정에서 생성됩니다.

<br>

### 경로 변수 처리

회원중 id가 10인 회원을 조회하고 싶다면 /컨텍스트경로/members/id 와 같은 경로로 조회를 할 수 있습니다. 하지만 이렇게 되면 id마다 컨트롤러를 생성해야합니다. 요청경로중 일부만 다를 때, 경로 변수를 통해 이 요청을 처리할 수 있습니다.

```java
	@GetMapping("/members/{id}")
	public String detail(@PathVariable("id") Long memId, Model model) {
		Member member = memberDao.selectById(memId);
		if (member == null) {
			throw new MemberNotFoundException();
		}
		model.addAttribute("member", member);
		return "member/memberDetail";
	}
```

경로변수 {id}에 해당하는 값이 @PathVariable 어노테이션을 갖는 파라미터에 할당됩니다. String에 해당하는 값이 Long 타입으로 바뀌어 memId에 들어감을 알 수 있습니다.

<br>

### 익셉션 처리

만일 회원아이디에 해당하는 정보를 얻지 못한다면 MemberNotFoundException() 익셉션을 발생할 것입니다. 이러한 익셉션은 try-catch로 해결이 가능하지만 경로변수에 해당하는 {id}를 Long 타입으로 변환하지 못한 경우에도 에러가 발생할 것입니다. 예를 들어 'a'라는 문자는 Long으로 바꾸지 못하므로 에러가 발생합니다. 사용하는 컨트롤러에서 익셉션이 발생할 경우, @ExceptionHandler 어노테이션을 통해 익셉션을 처리할 수 있습니다. 아래 코드를 컨트롤러 내부에 구현합시다.

```java
	@ExceptionHandler(TypeMismatchException.class)
	public String handleTypeMismatchException() {
		return "member/invalidId";
	}

	@ExceptionHandler(MemberNotFoundException.class)
	public String handleNotFoundException() {
		return "member/noMember";
	}
```

하나의 컨트롤러에서 특정한 익셉션이 발생한다면 그 컨트롤러 안에 구현하면 좋지만 여러 컨트롤러에서 동일한 익셉션이 발생한다면 코드의 중복이 발생할 것입니다. 이를 해결하기 위해 @ControllerAdvice를 사용해 처리할 수 있습니다.

```java
package common;

import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice("spring")
public class CommonExceptionHandler {

	@ExceptionHandler(RuntimeException.class)
	public String handleRuntimeException() {
		return "error/commonException";
	}
}
```

spring 패키지와 그 하위 패키지들에서 RuntimeException이 발생한다면 위 클래스로 만들어진 빈 객체가 처리하게 됩니다. 이를 위해 빈 객체를 등록해야합니다.
