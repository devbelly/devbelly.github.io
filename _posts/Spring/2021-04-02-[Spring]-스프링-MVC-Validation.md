---
title: "[Spring] 스프링 MVC : Validation"
tags: spring
categories: spring
use_math: true
---

### Validator

클라이언트가 작성한 폼 객체를 검사하지 않으면 데이터베이스에 잘못된 데이터가 들어가 오류를 일으킬 수 있습니다. 또한 확인을 통해 잘못된 값이 들어왔다는 것을 확인하더라도 클라이언트가 에러 메시지를 받지 못하면 어떠한 이유에서 가입을 실패했는지 알 수 없어 서비스를 제대로 이용할 수 없게 됩니다.

스프링은 이를 해결하기 위해 다음의 두 인터페이스를 제공합니다.

- org.springframework.validation.Validator
- org.springframework.validation.Errors

  Validator은 아래와 같은 인터페이스를 사용합니다.

```java
public interface Validator{
	boolean supports(Class<?> clazz);
	void validate(Object target,Errors errors);
}
```

supports()는 해당 클래스가 검증 가능한 객체인지 확인하고 validate()는 target 객체에 대한 검증을 진행하고 errors 파라미터에 에러를 담게 됩니다.

```java
package controller;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

import org.springframework.validation.Errors;
import org.springframework.validation.ValidationUtils;
import org.springframework.validation.Validator;

import spring.RegisterRequest;

public class RegisterRequestValidator implements Validator {
	private static final String emailRegExp =
			"^[_A-Za-z0-9-\\+]+(\\.[_A-Za-z0-9-]+)*@" +
			"[A-Za-z0-9-]+(\\.[A-Za-z0-9]+)*(\\.[A-Za-z]{2,})$";
	private Pattern pattern;

	public RegisterRequestValidator() {
		pattern = Pattern.compile(emailRegExp);
		System.out.println("RegisterRequestValidator#new(): " + this);
	}

	@Override
	public boolean supports(Class<?> clazz) {
		return RegisterRequest.class.isAssignableFrom(clazz);
	}

	@Override
	public void validate(Object target, Errors errors) {
		System.out.println("RegisterRequestValidator#validate(): " + this);
		RegisterRequest regReq = (RegisterRequest) target;
		if (regReq.getEmail() == null || regReq.getEmail().trim().isEmpty()) {
			errors.rejectValue("email", "required");
		} else {
			Matcher matcher = pattern.matcher(regReq.getEmail());
			if (!matcher.matches()) {
				errors.rejectValue("email", "bad");
			}
		}
		ValidationUtils.rejectIfEmptyOrWhitespace(errors, "name", "required");
		ValidationUtils.rejectIfEmpty(errors, "password", "required");
		ValidationUtils.rejectIfEmpty(errors, "confirmPassword", "required");
		if (!regReq.getPassword().isEmpty()) {
			if (!regReq.isPasswordEqualToConfirmPassword()) {
				errors.rejectValue("confirmPassword", "nomatch");
			}
		}
	}
}
```

32~39번째 줄은 커맨드 객체의 속성 값 email을 검사하는 과정입니다. if에서 이메일이 null값을 가지거나 빈 문자열이라면 에러코드를 삽입합니다. 빈 문자열이 아니더라도 이메일에 해당하는 정규표현식을 갖지 못한다면 역시 errors.rejectValue()를 통해 에러코드를 삽입하게 됩니다.

스프링은 속성 값은 간결하게 검사해줄 수 있는 ValidationUtils 클래스를 제공하기도 합니다. 40, 41, 42행에서 이를 사용하여 속성을 검사하고 있습니다.

```c++
		ValidationUtils.rejectIfEmptyOrWhitespace(errors, "name", "required");
		ValidationUtils.rejectIfEmpty(errors, "password", "required");
		ValidationUtils.rejectIfEmpty(errors, "confirmPassword", "required");
```

특이한 것 중 하나는 ValidationUtils는 커맨드 객체를 입력받지 않았음에도 불구하고 커맨드 객체의 속성에 접근한다는 점입니다. 이유는 Validator을 사용하는 메서드에 있습니다.

```java
package controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.Errors;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import spring.DuplicateMemberException;
import spring.MemberRegisterService;
import spring.RegisterRequest;

@Controller
public class RegisterController {
	...

	@PostMapping("/register/step3")
	public String handleStep3(RegisterRequest regReq, Errors errors) {
		new RegisterRequestValidator().validate(regReq, errors);
		if (errors.hasErrors())
			return "register/step2";

		try {
			memberRegisterService.regist(regReq);
			return "register/step3";
		} catch (DuplicateMemberException ex) {
			errors.rejectValue("email", "duplicate");
			return "register/step2";
		}
	}
}
```

매핑 어노테이션이 사용된 메서드의 파라미터에 Errors 타입이 있다면 스프링 MVC는 커맨드 객체와 연결된 errors 객체를 해당 메서드의 인자로 전달하게 됩니다. 단, 커맨드 객체의 파라미터 뒤에 Error타입의 파라미터가 위치해야 합니다. errors는 getFieldValue() 메서드를 통해 커맨드 객체의 속성에 접근하므로 ValidationUtils는 커맨드 객체를 전달받지 않아도 error객체에서 이미 커맨드 객체에 접근할 수 있으므로 따로 커맨드 객체를 입력받지 않아도 됩니다.

validate() 실행 후 hasError() 메서드를 사용합니다. 만약 rejectValue()가 한 번이라도 실행된 적이 있다면 true를 리턴하게 됩니다.

커맨드 객체의 속성이 잘못되었을 때는 rejectValue()를 사용하지만 커맨드 객체가 잘못되었을 때는 reject()를 사용합니다. 예를 들어 아이디와 패스워드가 일치하지 않을 때 사용할 수 있습니다.

<br>

### <form:error>

에러 객체에 담았다면 사용할 차례입니다. jsp에서는 <form:error>을 통해 에러코드를 확인할 수 있습니다. 다음은 step2.jsp의 일부입니다.

```c++
    <p>
        <label><spring:message code="email" />:<br>
        <form:input path="email" />
        <form:errors path="email"/>
        </label>
    </p>
```

path 속성은 해당 객체의 프로퍼티의 에러코드가 있는지 확인합니다. 만일 에러코드가 있다면 이에 대응하는 메세지코드를 찾아 리턴하게 됩니다. 메세지코드를 찾을 때는 다음의 우선순위를 통해 찾게 됩니다.

1. 에러코드 . 커맨드객체이름 . 필드명
2. 에러코드 . 필드명
3. 에러코드 . 필드타입
4. 에러코드

메세지를 찾을때는 MessageSource를 이용하게 됩니다. label.properties에 메세지코드에 대응하는 값을 넣어주도록 합시다.

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

required=필수항목입니다.
bad.email=이메일이 올바르지 않습니다.
duplicate.email=중복된 이메일입니다.
nomatch.confirmPassword=비밀번호와 확인이 일치하지 않습니다
```

<br>

### 글로벌 범위 Validator

스프링 MVC는 글로벌 Validator을 사용할 수 있도록 제공합니다. 글로벌 Validator을 설정하고 검증할 커맨드 객체에 @Valid 어노테이션을 사용하면 컨트롤러마다 사용자가 직접만든 Validator 객체를 생성하는 코드를 적지 않아도 됩니다. 스프링은 WebMvcConfig 인터페이스가 제공하는 getValidator() 메서드를 구현하면 해당 객체를 글로벌 Validator로 사용하게 됩니다. 다음은 설정 클래스인 MvcConfig에서 getValidator()을 구현한 코드입니다.

```java
	@Override
	public Validator getValidator() {
		return new RegisterRequestValidator();
	}
```

이후에 커맨드 객체에 @Valid을 추가하면 됩니다. 이 어노테이션은 Bean Validation API에 포함되어 있으므로 아래의 의존성을 추가한 후 사용해야합니다.

```c++
		<dependency>
			<groupId>javax.validation</groupId>
			<artifactId>validation-api</artifactId>
			<version>1.1.0.Final</version>
		</dependency>
```

```java
	@PostMapping("/register/step3")
	public String handleStep3(@Valid RegisterRequest regReq, Errors errors) {
		...
	}
```

객체에 대한 검증은 handleStep3가 실행되기 이전에 수행되고 해당 결과는 Errors 객체에 담습니다. 그러므로 handleStep3의 파라미터에 Errors 파라미터가 존재하지 않다면 에러가 발생하게 됩니다.

<br>

### @InitBinder을 통한 컨트롤러 범위 Validator

모든 커맨드 객체가 글로벌 Validator을 사용하는 것은 적절하지 않습니다. @InitBinder을 통해 글로벌 범위 대신 컨트롤러 범위의 Validator을 설정할 수 있습니다.

```java
@Controller
public class RegisterController {
	...
	@InitBinder
	protected void initBinder(WebDataBinder binder) {
		binder.setValidator(new RegisterRequestValidator());
	}
}
```

WebDataBinder은 내부적으로 Validator 목록을 갖게 됩니다. 이 안에는 글로벌 범위의 Validator이 포함된 상태입니다. setValidator()메서드를 통해 기존에 갖고 있었던 Validator을 제거하고 인자로 얻게 된 새로운 Validator을 목록에 추가하게 됩니다. 이로써 글로벌 Validator 대신 컨트롤러 범위 Validator사용이 가능해집니다.

<br>

### Bean Validation을 통한 값 검증

위의 두 방법은 결국엔 사용자가 Validator을 작성해야하는 번거로움이 있습니다. 특히 이메일이 유효한 값인지 확인하기 위해서는 정규표현식을 직접 사용자가 입력해야합니다. 이러한 번거로움을 막고자 Bean Validation API가 제공됩니다. 사용하기 위해서는 API를 정의한 모듈과 구현한 프로바이더를 의존설정에 추가해야합니다.

```c++
		<dependency>
			<groupId>javax.validation</groupId>
			<artifactId>validation-api</artifactId>
			<version>1.1.0.Final</version>
		</dependency>

		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-validator</artifactId>
			<version>5.4.2.Final</version>
		</dependency>
```

그리고 검증할 커맨드 객체에 어노테이션을 사용합니다.

```java
public class RegisterRequest {

	@NotBlank
	@Email
	private String email;
	@Size(min=6)
	private String password;
	@NotEmpty
	private String confirmPassword;
	@NotEmpty
	private String name;

	...
}
```

커맨드 객체를 검증하기 위해서 기존에는 사용자가 직접 만든 Validator을 사용하는 대신 OptionalValidatorFactory 클래스를 글로벌 범위 Validator로 설정하면 사용자가 정의한 Validator은 사용하지 않아도 됩니다. 이 설정은 @WebMvcConfig 어노테이션 안에 들어있으므로 추가적으로 설정할 것은 없습니다. 다만 기존에 getValidator()메서드로 사용하고 있는 글로벌 Validator가 있다면 해당 Validator을 우선적으로 사용하므로 코드에서 없애주어야 합니다.

```java
//	@Override
//	public Validator getValidator() {
//		return new RegisterRequestValidator();
//	}
```

정상적으로 에러메세지를 출력하는지 확인하기 위해 일부러 비밀번호를 짧게 설정하여 가입을 진행하면 다음과 같은 에러메시지를 만나게 됩니다.

![img](https://blog.kakaocdn.net/dn/vGglg/btq1E8Kz9ch/JnLBeSV4OW8kwDKK2OT2C0/img.png)

프로퍼티에 따로 해당 메세지를 등록하지 않았지만 저런 메세지를 출력하게 됩니다. 이는 에러코드에 해당하는 메세지가 존재하지 않을때 프로바이더가 기본적으로 제공하는 메세지를 사용하기 때문입니다.
