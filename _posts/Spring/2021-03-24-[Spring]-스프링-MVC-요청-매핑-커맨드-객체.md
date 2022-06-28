---
title: "[Spring] 스프링 MVC : 요청 매핑, 커맨드 객체"
tags: spring
categories:
  - Spring
---

### 요청 매핑

웹 개발은 크게 클라이언트 요청 URL을 처리할 코드를 작성하거나 처리 결과를 페이지에 출력해줄 코드를 작성하는 것입니다. 첫 번째는 dispatcherServlet이 요청받은 URL을 처리해줄 Controller을 작성하는 것입니다. 컨트롤러 클래스는 요청 매핑 어노테이션을 이용해서 메서드가 처리할 경로를 지정할 수 있습니다.

```java
package controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import spring.DuplicateMemberException;
import spring.MemberRegisterService;
import spring.RegisterRequest;

@Controller
public class RegisterController {

	@RequestMapping("/register/step1")
	public String handleStep1() {
		return "register/step1";
	}

}
```

컨트롤러 클래스는 요청 매핑 어노테이션을 적용한 메서드를 여러개 가질 수 있습니다. 회원가입을 처리해야할 때, 약관 동의 - 가입 정보 - 가입완료 와 같이 크게 세 단계로 나눌 수 있습니다. 이 경우 하나의 컨트롤러 클래스에서 각각을 처리할 메서드를 정의하게 되면 관리가 편해집니다.

```java
package controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import spring.DuplicateMemberException;
import spring.MemberRegisterService;
import spring.RegisterRequest;

@Controller
public class RegisterController {

	@RequestMapping("/register/step1")
	public String handleStep1() {
		//something...
	}

	@PostMapping("/register/step2")
	public String handleStep2(
			@RequestParam(value="agree",defaultValue="false") Boolean agree) {
		//something...
	}

	@PostMapping("/register/step3")
	public String handleStep3(RegisterRequest regReq) {
		//something...
	}
}
```

같은 일(회원가입)을 수행하므로 요청하는 URL은 비슷하기 마련입니다. 위 코드에서 "/register"이 중복되는것을 확인할 수 있는데 아래와 같이 클래스 어노테이션에 요청 매핑 어노테이션을 사용하면 중복되는 코드를 막을 수 있습니다. 클래스에 사용한 요청 매핑 어노테이션과 메서드에 적용한 요청 매핑 어노테이션이 합쳐져 경로를 찾게 됩니다.

```java
package controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import spring.DuplicateMemberException;
import spring.MemberRegisterService;
import spring.RegisterRequest;

@Controller
@RequestMapping("/register")
public class RegisterController {

	@RequestMapping("/step1")
	public String handleStep1() {
		//something...
	}

	@PostMapping("/step2")
	public String handleStep2(
			@RequestParam(value="agree",defaultValue="false") Boolean agree) {
		//something...
	}

	@PostMapping("/step3")
	public String handleStep3(RegisterRequest regReq) {
		//something...
	}
}
```

<br>

### GET vs POST

회원가입을 위한 컨트롤러 클래스를 작성해보겠습니다. 첫번째 단계는 약관동의 페이지를 보여주는 것입니다. 별 다른 처리 없이 약관내용을 보여주면 되므로 바로 뷰 페이지 주소를 리턴했습니다.

```java
@Controller
@RequestMapping("/register")
public class RegisterController {

	@RequestMapping("/step1")
	public String handleStep1() {
		return "register/step1";
	}
}
```

처리하는 뷰 페이지는 WEB-INF/view/register/step1.jsp 입니다.

```c++
<%@ page contentType="text/html; charset=utf-8" %>
<!DOCTYPE html>
<html>
<head>
	<title>회원가입</title>
</head>
<body>
	<h2>약관</h2>
	<p>약관 내용</p>
	<form action="step2" method="post">
	<label>
		<input type="checkbox" name="agree" value="true"> 약관 동의
	</label>
	<input type="submit" value="다음 단계" />
	</form>
</body>
</html>
```

폼을 POST 방식으로 처리할 것을 요청하고 있습니다. @RequestMapping을 통해 요청된 방식을 구분하지 않고 요청된 경로를 확인하고 처리를 하게 됩니다. POST 요청에 대해서만 컨트롤러가 처리하기를 원한다면 @PostMapping 어노테이션을 통해 지정할 수 있습니다. 마찬가지로 GET 요청에 대해서만 처리하기 원한다면 @GetMapping 어노테이션을 활용할 수 있습니다.

<br>

### 요청 파라미터 접근1. HttpServletRequest

약관 페이지에서 위 내용에 동의를 한다면 다음 단계인 회원 가입을 위한 정보를 입력하는 페이지를 보여주고 동의하지 않았다면 다시 약관 페이지를 보여주게 처리하고 싶습니다. 이를 위해선 컨트롤러에서는 요청 파라미터의 agree 값을 읽어와서 확인을 해야합니다. 이를 위해 사용할 수 있는 방식은 HttpServletRequest 입니다. getParameter을 통해서 값을 읽어올 수 있습니다.

```java
	@PostMapping("/step2")
	public String handleStep2(HttpServletRequest request) {
		String agree = request.getParameter("agree");
		if(agree==null||!agree.equals("true")) {
			return "register/step1";
		}
		return "register/step2";
	}
```

<br>

### 요청 파라미터 접근2. @RequestParam

또 다른 방식으로 @RequestParam 어노테이션을 사용하는 것입니다. 요청 파라미터가 적을 때 사용하기 적합한 방법입니다. String만으로 값을 처리했던 HttpServletRequest 방식과는 달리 스프링이 적합한 자료형을 찾아 변환해주기 때문에 파라미터를 사용하기 더 편리합니다.

```java
	@PostMapping("/step2")
	public String handleStep2(
		@RequestParam(value="agree",defaultValue="false") Boolean agree) {
		if(!agree) {
			return "register/step1";
		}
		return "register/step2";
	}
```

동의가 확인되면 다음 뷰 페이지를 화면에 출력합니다.

```c++
<%@ page contentType="text/html; charset=utf-8" %>
<!DOCTYPE html>
<html>
<head>
    <title>회원가입</title>
</head>
<body>
    <h2>회원 정보 입력</h2>
    <form action="step3" method="post">
    <p>
        <label>이메일:<br>
        <input type="text" name="email" id="email">
        </label>
    </p>
    <p>
        <label>이름:<br>
        <input type="text" name="name" id="name">
        </label>
    </p>
    <p>
        <label>비밀번호:<br>
        <input type="password" name="password" id="password">
        </label>
    </p>
    <p>
        <label>비밀번호 확인:<br>
        <input type="password" name="confirmPassword" id="confirmPassword">
        </label>
    </p>
    <input type="submit" value="가입 완료">
    </form>
</body>
</html>
```

<br>

### 리다이렉트

서버를 실행한 후 http://localhost:8080/sp5-chap11/register/step2 주소를 입력하게 되면 다음과 같은 에러메세지를 만나게 됩니다.

![img](https://blog.kakaocdn.net/dn/bGfgf2/btq0SkLCyiu/Wzdb27rjjkL5y7SDq6gGik/img.png)

GET 방식으로 요청하는 것은 기본적으로 동일한 결과를 확인할 때 주로 사용합니다. 조회에 어울리는 요청방식이며 주소창에 직접 링크를 입력하는 것은 GET 방식으로써 서버에 요청됩니다. step2를 처리할 때 위에서 작성한 것과 같이 POST메서드에 대해서만 처리하도록 작성했으므로 GET 방식을 찾을 수 없다는 에러메세지를 만나게 됩니다. 아래와 같이 @GetMapping 요청 매핑을 통해 step1 페이지로 리다이렉트를 하게 되면 에러 메세지 대신 약관동의 페이지로 돌아가게 됩니다.

```java
	@GetMapping("/step2")
	public String handlestep2Get() {
		return "redirect:/register/step1";
	}
```

<br>

### 요청 파라미터 접근3. 커맨드 객체

step2.jsp를 보면 클라이언트가 입력한 정보를 email, name, password, confirmPassword를 통해서 전달하고 있습니다. 이를 컨트롤러에서 사용하려면 getParameter 또는 @RequestParam을 사용해야하는데 전달되는 정보가 많을수록 여러번 적어야하는 번거로움이 있습니다. 이에 대한 방식으로 스프링은 커맨드 객체를 통해 컨트롤러에 정보를 전달합니다. 커맨드 객체는 요청매핑을 한 메서드의 파라미터부분에 위치합니다. 커맨드객체는 입력받을 정보에 대한 세터메서드들을 갖고 있어야 합니다.

```java
package spring;
//커맨드 객체 클래스
public class RegisterRequest {

	private String email;
	private String password;
	private String confirmPassword;
	private String name;

	public String getEmail() {
		return email;
	}

	public void setEmail(String email) {
		this.email = email;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	public String getConfirmPassword() {
		return confirmPassword;
	}

	public void setConfirmPassword(String confirmPassword) {
		this.confirmPassword = confirmPassword;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public boolean isPasswordEqualToConfirmPassword() {
		return password.equals(confirmPassword);
	}
}
```

```java
	@PostMapping("/step3")
	public String handleStep3(RegisterRequest regReq) {
		try {
			memberRegisterService.regist(regReq);
			return "register/step3";
		}catch(DuplicateMemberException ex) {
			return "register/step2";
		}
	}
```

<br>

### 뷰 JSP 코드에서 커맨드 객체 사용하기

회원가입을 완료하면 가입완료 페이지가 뜨게 되는데, 조금 더 추가해서 가입한 사용자 이름을 출력하는 뷰 코드를 작성하고 싶다고 하겠습니다. 이를 위해서는 컨트롤러에서 사용한 커맨드 객체를 가져와 뷰 페이지에서 사용해야하는데 커맨드 객체 클래스의 이름을 속성 그대로 사용하게 됩니다. 단 맨 앞글자는 소문자로 바꾸어 사용하게 됩니다.

```c++
<%@ page contentType="text/html; charset=utf-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html>
<head>
    <title>회원가입</title>
</head>
<body>
    <p><strong>${registerRequest.name}님</strong>
        회원 가입을 완료했습니다.</p>
    <p><a href="<c:url value='/main'/>">[첫 화면 이동]</a></p>
</body>
</html>
```

@ModelAttribute을 이용해 커맨드 객체의 이름을 바꾸어 사용할 수도 있습니다.

```java
	@PostMapping("/step3")
	public String handleStep3(@ModelAttribute("info") RegisterRequest regReq) {
		try {
			memberRegisterService.regist(regReq);
			return "register/step3";
		}catch(DuplicateMemberException ex) {
			return "register/step2";
		}
	}
```

또한 step2에서 중복된 이메일로 인해 회원가입에 실패했을 때도 커맨드 객체를 활용할 수 있습니다. 현재 step2.jsp 페이지는 회원가입에 실패했다면 모든 정보를 다시 입력해야하는데 이는 클라이언트에게 굉장히 번거로운 일입니다. step2.jsp에서 step3 페이지를 요청하는 과정에서 커맨드 객체가 생성되는데 이 커맨드 객체를 다시 step2.jsp에서 value 속성을 이용함으로써 다시 정보를 기입하는 일을 막을 수 있습니다.

```c++
    <p>
        <label>이메일:<br>
        <input type="text" name="email" id="email" value="${registerRequest.email}">
        </label>
    </p>
    <p>
        <label>이름:<br>
        <input type="text" name="name" id="name" value="${registerRequest.name}">
        </label>
    </p>
```

이를 간결하게 위해 스프링은 form 태그를 제공하기도 합니다. 이를 사용하기 위해선 커맨드 객체가 미리 생성되어 있어야 하는데 처음 step2.jsp 페이지는 커맨드 객체가 없는 상태입니다. 이 때문에 step2.jsp에 접근하기전 @PostMapping(/step2) 부분에서 Model 파라미터를 추가적으로 이용해 커맨드 객체를 생성하여 Model에 전달해야 합니다.
