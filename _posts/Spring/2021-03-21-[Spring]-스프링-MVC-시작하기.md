---
title: "[Spring] 스프링 MVC 시작하기"
tags: spring
categories:
  - Spring
---

### web.xml

웹 요청이 발생하면 우선적으로 톰캣이 해당 요청을 받게 됩니다. web.xml은 톰캣의 환경설정에 해당하는 파일입니다. 들어가야할 내용은 Servlet에 대한 설정들입니다.

```c++
<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
             http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
	version="3.1">

	<servlet>
		<servlet-name>dispatcher</servlet-name>
		<servlet-class>
			org.springframework.web.servlet.DispatcherServlet
		</servlet-class>
		<init-param>
			<param-name>contextClass</param-name>
			<param-value>
				org.springframework.web.context.support.AnnotationConfigWebApplicationContext
			</param-value>
		</init-param>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>
				config.MvcConfig
				config.ControllerConfig
			</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>dispatcher</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

	<filter>
		<filter-name>encodingFilter</filter-name>
		<filter-class>
			org.springframework.web.filter.CharacterEncodingFilter
		</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>encodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

</web-app>
```

<br>

### HandlerMapping

DispatcherServlet은 클라이언트의 요청을 처리할 컨트롤러를 찾습니다. 이때 직접찾는것이 아니라 HandlerMapping 빈 객체에 그 일을 위임하고 HandlerMapping 빈 객체는 이를 처리할 컨트롤러 빈 객체를 리턴하게 됩니다.

이름에 대한 의문중 하나는 이 일을 수행하는 빈 객체의 이름은 ControllerMapping 가 더 어울릴 것 같은데 사용하는 이름은 HandlerMapping 입니다. 우리는 편의를 위해 스프링 프레임워크에서 제공해주는 @Controller가 적용된 객체를 통해 클라이언트의 요청을 처리합니다. 하지만 Dispatcher입장에서 처리한 객체가 무조건 @Controller가 적용된 객체일 필요는 없습니다. 원한다면 사용자가 Controller 인터페이스를 구현한 클래스를 사용하거나 HttpRequestHandler 타입을 사용해서 컨트롤러를 구현해도 되기 때문입니다.

즉 스프링은 클라이언트 요청을 처리할 실제 객체들을 Handler라고 표현하므로 HandlerMapping이라는 이름을 사용합니다.

<br>

### HandlerAdapter

위에서 설명했듯, 여러종류의 Handler(@Controller 객체, Controller 인터페이스를 구현한 객체, HttpRequestHandler 타입의 객체)에서 HandlerMapping을 통해 사용자 요청을 처리할 Handler을 찾았으면 HandlerMapping은 해당 객체를 DispatcherServlet에 반환하게 됩니다. DispatcherServlet은 이 객체를 실행한 결과를 담은 ModelAndView 객체를 필요로 하는데 Handler의 구현 타입에 따라 반환하는 타입이 다릅니다. 예를 들어 Controller 인터페이스를 구현한 객체는 ModelAndView를 리턴하게 됩니다.

```java
package chap09;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

public class ImpController implements Controller {

	@Override
	public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
		ModelAndView mv = new ModelAndView();
		//do something..
		return mv;
	}
}
```

반면 @Controller 어노테이션을 사용한 객체는 String만 리턴하게 됩니다.

```java
package chap09;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class HelloController {

	@GetMapping("/hello")
	public String hello(Model model,@RequestParam(value="name",required=false)String name) {
		model.addAttribute("greeting","안녕하세요, "+name);
		return "hello";
	}
}
```

즉 어떠한 Handler가 요청을 수행해도 DispatcherServlet은 ModelAndView를 받아야합니다. 이를 위해 HandlerAdapter 를 사용합니다.

<br>

### ViewResolver

```java
package config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ViewResolverRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer{
	@Override
	public void configureViewResolvers(ViewResolverRegistry registry) {
		registry.jsp("WEB-INF/view/",".jsp");
	}
}
```

DispatcherServlet은 HandlerAdapter에게 전달받은 ModelAndView 객체를 갖고 ViewResolver에게 View 객체를 요구하게 됩니다. registry.jsp를 통해 viewResolver를 생성하고 뷰 이름에 suffix와 prefix를 붙여 뷰 코드로 사용할 경로를 만들게 됩니다. 만들어진 뷰 코드를 사용하는 뷰 객체를 만들게 되고 이를 DispatcherServlet에게 돌려주게 됩니다.

<br>

### View

마지막으로 View 객체를 받은 DispatcherServlet은 View 객체에게 응답 생성을 요청하게 됩니다. 요청함과 동시에 Map 객체를 전달하게 되고 안에는 Model에서 설정한 키 값을 담고 있습니다. 뷰 객체는 request.setAttribute()를 통해 request 속성에 해당 값을 저장하게 되고 JSP는 속성이름을 통해 해당 값을 사용하게 됩니다.

<br>

### DispatcherServlet

web.xml에서 contextConfiguration 초기화 파라미터를 이용해 스프링 설정 클래스를 전달했습니다. DispatcherServlet은 이를 통해 스프링 컨테이너를 생성하고 이 안에서 빈 객체를 찾게 됩니다. 즉 앞서 사용했던 HandlerMapping, HandlerAdapter, 컨트롤러들과 ViewResolver 빈 객체들은 설정 파일안에 정의되어야 합니다. 우리는 이러한 복잡한 설정들을 @EnableWebMvc 어노테이션을 이용해 한번에 처리했습니다.

톰캣은 클라이언트의 요청 URL을 보고 이에 맵핑된 servlet이 처리를 하는 구조입니다. 그래서 web.xml에 우리는 아래와 같은 servlet-mapping을 사용했습니다.

```c++
	<servlet-mapping>
		<servlet-name>dispatcher</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
```

위와 같이 매핑 경로가 '/' 인 경우 .jsp로 끝나는 요청을 제외한 모든 요청을 DispatcherServlet이 처리하게 됩니다.

하지만 @EnableWebMvc 어노테이션이 등록하는 HandlerMapping은 @Controller 어노테이션을 적용한 빈 객체가 처리할 수 있는 요청 경로만 대응할 수 있습니다. 따라서 "/index.html"이나 "/css/bootstrap.css"와 같은 요청을 처리할 수 있는 컨트롤러를 찾지 못해 404에러를 뱉게 됩니다.

이러한 정적인 경로를 처리하기 위해 configureDefaultServletHanding 메서드를 사용합니다. 이를 통해 두 개의 빈 객체가 생성됩니다.

- DefaultServletHttpRequestHandler
- SimpleUrlHandlerMapping

기존에 존재하던 RequestMappingHandlerMapping은 SimpleUrlHandlerMapping보다 우선순위가 높기 때문에 다음과 같은 순서로 요청이 처리됩니다.

1. RequestMappingHandlerMapping을 통해 사용자 요청을 처리할 Handler을 검색합니다. 만일 존재한다면 해당 Handler을 이용해 처리합니다.

2. 존재하지 않는다면 SimpleUrlHandlerMapping을 통해 사용자 요청을 처리할 Handler을 검색합니다. SimpleUrlHandlerMapping은 모든 경로에 대해 DefaultServletHttRequestHandler을 반환합니다. 이 Handler은 디폴트 서블릿에게 처리를 위임하게 됩니다.
