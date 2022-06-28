---
title: "[Spring] 스프링 MVC : Session, Interceptor, Cookie"
tags: spring
categories: spring
use_math: true
---

### Session

웹 프로그래밍은 여러 웹서버 프로그램들이 모여 이루어지는 것이라고 볼 수 있습니다. 클라이언트에서 특정한 요청을 하면 웹서버는 알맞는 프로그램을 찾아 실행한 후 종료하게 됩니다. 이 때 실행되는 프로그램은 실행이 완료된 후 메모리에서 제거됩니다. 이 때문에 다음 요청에서 이전에 실행한 결과를 담은 값들에 대해 접근을 할 수가 없습니다. 이 문제를 해결하기 위해 전역변수와 같이 저장공간을 두어야합니다. 이러한 저장공간은 크게 HttpSession과 Cookie로 나눌 수 있습니다. 우선 알아볼 것은 Session을 활용하여 로그인 상태를 유지하는 방법에 대해 알아보겠습니다.

우선적으로 세션에 저장할 객체를 정의해야합니다.

```java
package spring;

public class AuthInfo {
	private Long id;
	private String email;
	private String name;

	public AuthInfo(Long id, String email, String name) {
		this.id = id;
		this.email = email;
		this.name = name;
	}

	public Long getId() {
		return id;
	}
	public String getEmail() {
		return email;
	}

	public String getName() {
		return name;
	}


}
```

이 객체를 확인할 서비스 객체를 구현합니다.

```java
package spring;

public class AuthService {

	private MemberDao memberDao;

	public void setMemberDao(MemberDao memberDao) {
		this.memberDao=memberDao;
	}

	public AuthInfo authenticate(String email,String password) {
		Member member = memberDao.selectByEmail(email);
		if(member==null) {
			throw new WrongIdPasswordException();
		}
		if(!member.matchPassword(password)) {
			throw new WrongIdPasswordException();
		}
		return new AuthInfo(member.getId(),member.getEmail(),member.getName());
	}
}
```

마지막으로 위 서비스 객체를 사용할 컨트롤러를 구현해야합니다. 이 때, 로그인폼을 저장할 커맨드 객체와 Validator또한 구현해줍니다. Validator은 이전 챕터에서 설명한 것처럼 어노테이션을 활용해 처리할 수도 있습니다. 여기서는 따로 Validator 클래스를 만들어 사용했습니다. 아래는 각각 로그인 컨트롤러와 로그인 정보를 담는 LoginForm.jsp 입니다. LoginForm 페이지는 form 태그를 활용하여 구현했는데 form 의 디폴트 method는 POST여서 submit후에는 로그인 컨트롤러 후의 submit 메서드와 매핑이 됨을 알 수 있습니다

```java
@Controller
@RequestMapping("/login")
public class LoginController {

	@Autowired
    private AuthService authService;

    public void setAuthService(AuthService authService) {
        this.authService = authService;
    }
    @GetMapping
    public String form(LoginCommand loginCommand) {
    	return "login/loginForm";
    }

    @PostMapping
    public String submit(
    		LoginCommand loginCommand, Errors errors) {
        new LoginCommandValidator().validate(loginCommand, errors);
        if (errors.hasErrors()) {
            return "login/loginForm";
        }
        try {
            AuthInfo authInfo = authService.authenticate(
                    loginCommand.getEmail(),
                    loginCommand.getPassword());

            return "login/loginSuccess";
        } catch (WrongIdPasswordException e) {
            errors.reject("idPasswordNotMatching");
            return "login/loginForm";
        }
    }
}
```

```c++
<%@ page contentType="text/html; charset=utf-8" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>
<!DOCTYPE html>
<html>
<head>
    <title><spring:message code="login.title" /></title>
</head>
<body>
    <form:form modelAttribute="loginCommand">
    <form:errors />
    <p>
        <label><spring:message code="email" />:<br>
        <form:input path="email" />
        <form:errors path="email"/>
        </label>
    </p>
    <p>
        <label><spring:message code="password" />:<br>
        <form:password path="password" />
        <form:errors path="password"/>
        </label>
    </p>
    <p>
        <label><spring:message code="rememberEmail" />:
        <form:checkbox path="rememberEmail"/>
        </label>
    </p>
    <input type="submit" value="<spring:message code="login.btn" />">
    </form:form>
</body>
</html>
```

위 코드는 데이터베이스에서 입력한 값만 비교하여 존재한다면 로그인성공 페이지를 보여주게 됩니다. 로그인 성공이후에 할 일들(글 작성하기, 회원정보 조회 등)을 위해 로그인 상태를 유지해야하지만 아직 Session에 아무것도 담지 않아 해당 요청들을 처리할 수 없습니다. 이를 위해 Session에 접근하여 authInfo 객체를 넣는 작업을 해야합니다.

스프링 MVC 에서 Session에 접근하는 방법은 크게 두 가지가 있습니다.

- 매핑 어노테이션이 적용된 메서드에 HttpSession 파라미터를 추가
- 매핑 어노테이션이 적용된 메서드에 HttpServletRequest 파라미터를 추가한 후 HttpSession 구하기

두 방법의 차이점은 아래 HttpServletRequst는 원하는 시점에서 HttpSession을 얻을 수 있다는 것입니다. 파라미터에 HttpSession이 존재하면 스프링 MVC는 현재 Session이 존재하지 않으면 세션을 새로 생성한 후 파라미터에 전달하고 이미 존재하고 있는 Session이 있다면 해당 세션을 전달하게 됩니다. 위 로그인 컨트롤러에서 파라미터에 Httpsession을 파라미터로 추가한 후 authInfo 객체를 세션에 저장합시다. 아래는 수정된 submit 메서드입니다.

```java
    @PostMapping
    public String submit(
    		LoginCommand loginCommand, Errors errors, HttpSession session) {
        new LoginCommandValidator().validate(loginCommand, errors);
        if (errors.hasErrors()) {
            return "login/loginForm";
        }
        try {
            AuthInfo authInfo = authService.authenticate(
                    loginCommand.getEmail(),
                    loginCommand.getPassword());

            session.setAttribute("authInfo", authInfo);

            return "login/loginSuccess";
        } catch (WrongIdPasswordException e) {
            errors.reject("idPasswordNotMatching");
            return "login/loginForm";
        }
    }
```

로그인 상태가 유지가 되어있다면 해당 상태를 종료할 로그아웃 컨트롤러도 필요합니다. 새로운 컨트롤러가 생성되었으므로 빈 객체로 추가해야합니다.

```java
package controller;

import javax.servlet.http.HttpSession;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class LogoutController {

	@RequestMapping("logout")
	public String logout(HttpSession session) {
		session.invalidate();
		return "redirect:/main";
	}
}
```

<br>

### Interceptor

로그인 상태가 유지되었을 때, 하는 일 중 한가지는 비밀번호 변경입니다. 메인페이지에서 아직 로그인을 하지 않은 채로 비밀번호 변경을 요청을 하면 잘못된 요청입니다. 만약 로그인 세션이 존재하지 않는다면 비밀번호 변경 요청을 위한 페이지를 리턴해서는 안됩니다. 이를 위해 컨트롤러의 시작 부분에 세션이 존재한다면 비밀번호 변경을 위한 메서드를 실행하고 존재하지 않는다면 메인페이지로 리다이렉션 하는 코드를 추가할 수 있습니다.

하지만 컨트롤러들이 로그인 세션을 검사해야할 경우는 굉장히 많습니다. 이때마다 컨트롤러 시작부분에 세션이 존재하는지 여부를 구현하면 굉장히 많은 중복이 발생하게 됩니다. 이를 피하기 위해 컨트롤러마다 동일한 작업을 수행해야할 때, 인터셉터를 활용할 수 있습니다.

인터셉터를 사용하기 위한 인터페이스는 다음의 세 가지 메서드로 나눌 수 있습니다.

- preHandle()
- postHandle()
- afterCompletion()

preHandle() 메서드는 컨트롤러가 실행되기 전 수행할 메서드를 정의할 수 있습니다. 로그인 세션을 검사하는 것도 이 메서드를 구현해서 해결할 수 있습니다.

postHandler() 메서드는 컨트롤러가 수행된 이후에 실행되는 메서드입니다. 실행 중간에 Exception이 발생하면 실행되지 않습니다.

afterCompletion()은 컨트롤러가 수행되고 뷰 페이지를 출력한 이후에 수행되는 메서드입니다. 실행 중간에 Exception이 발생하면 파라미터에 해당 에러를 저장하므로 로그를 확인할 때 자주 사용됩니다.

로그인 세션을 확인하기 위해 Interceptor 인터페이스를 상속받아 구현한 예시입니다.

```java
package interceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.springframework.web.servlet.HandlerInterceptor;

public class AuthCheckInterceptor implements HandlerInterceptor {

	@Override
	public boolean preHandle(
			HttpServletRequest request,
			HttpServletResponse response,
			Object handler) throws Exception {
		HttpSession session = request.getSession(false);
		if (session != null) {
			Object authInfo = session.getAttribute("authInfo");
			if (authInfo != null) {
				return true;
			}
		}
		response.sendRedirect(request.getContextPath() + "/login");
		return false;
	}
}
```

구현을 했다면 이를 적용할 컨트롤러를 설정해야합니다. 이는 WebMvcConfigurer 인터페이스에서 addInterceptors() 메서드를 구현해서 설정할 수 있습니다.

```java
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		registry.addInterceptor(authCheckInterceptor()).addPathPatterns("/edit/**");
	}
```

addInterceptor은 handlerInterceptor 객체를 설정합니다. 이 객체를 authCheckInterceptor 객체로 설정합니다. 또한 addInterceptor 메서드는 InterceptorRegistration 객체를 리턴합니다. 이 객체의 addPathPatterns() 메서드는 인터셉터를 적용할 컨트롤러를 찾기 위해 Ant 경로를 설정할 수 있습니다. edit 경로에 해당하는 요청들은 인터셉터로 세션을 확인하게 됩니다.

<br>

### Cookies

웹 페이지들은 이전에 사용자가 접근한 적이 있다면 편리한 접근을 위해 사용자 아이디를 미리 로그인 폼에 입력해주는 기능을 제공하기도 합니다. 이는 Cookies를 통해 구현이 가능합니다. 로그인 폼에서 사용자가 해당 이메일에 대해 기억할 것은 체크했다면 다음 로그인때 Cookies를 활용해 바로 이메일을 폼에 넣어주는 기능을 구현해보도록 합시다.

이를 위해서는 로그인할 때, 이를 쿠키로 만들 것인지를 정하는 부분을 설정해야합니다.

```c++
    <p>
        <label><spring:message code="rememberEmail" />:
        <form:checkbox path="rememberEmail"/>
        </label>
    </p>
```

로그인 GET 요청이 들어온다면 로그인 컨트롤러는 form() 메서드를 실행하게 됩니다. 이 메서드의 파라미터에는 커맨드 객체가 있는데, 기존에 쿠키가 있다면 커맨드 객체에 쿠키정보를 꺼내어 담습니다.

```java
    @GetMapping
    public String form(LoginCommand loginCommand,
    		@CookieValue(value = "REMEMBER", required = false) Cookie rCookie) {
		if (rCookie != null) {
			loginCommand.setEmail(rCookie.getValue());
			loginCommand.setRememberEmail(true);
		}
    	return "login/loginForm";
    }
```

스프링 MVC에서는 요청 매핑 어노테이션이 적용된 메서드에 Cookie를 사용하는 인자에 @CookieValue 어노테이션을 사용하여 쿠키에 접근할 수 있습니다. value에 해당하는 쿠키가 존재한다면 쿠키를 rCookie에 담게 됩니다. 쿠키가 존재하지 않을 수도 있으므로 required를 false로 설정해야합니다. default값은 true이기 때문입니다.

실제로 쿠키를 만드는 것은 로그인을 수행하는 submit() 메서드입니다. 로그인폼에서 이메일 기억하기의 체크여부에 따라 쿠키의 유지시간을 설정하면 됩니다. 쿠키를 담기 위해서 파라미터에 추가적으로 HttpServletResponse를 사용합니다.

```java
	Cookie rememberCookie =
			new Cookie("REMEMBER", loginCommand.getEmail());
	rememberCookie.setPath("/");
	if (loginCommand.isRememberEmail()) {
		rememberCookie.setMaxAge(60 * 60 * 24 * 30);
	} else {
		rememberCookie.setMaxAge(0);
	}
		response.addCookie(rememberCookie);
```
