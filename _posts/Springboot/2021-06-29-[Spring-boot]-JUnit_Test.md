---
title: "[Spring Boot] JUnit Test"
tags: springboot
categories:
  - Spring boot
use_math: true
---

### 스프링부트 테스트

2
웹 애플리케이션에서 개발자가 만든 컨트롤러가 정상적으로 동작하기 위해서는 서블릿 컨테이너가 구동되어야하고 브라우저를 통해 요청/응답 결과를 확인해야합니다. 하지만 컨트롤러가 수정될때마다 이러한 수행을 할 수 없기에 JUnit 기반의 단위테스트를 통해 컨트롤러를 검증하게 됩니다. 이를 위해 JUnit은 서버를 구동하지 않고 컨트롤러를 테스트하거나 컨트롤러와 연관된 비즈니스 컴포넌트를 실행하지 않고 컨트롤러를 독립적으로 테스트할 수 있어야합니다. 기존에 작성한 BoardController의 검증을 위해 다음과 같은 테스트케이스를 작성했습니다.

```java
package com.rubypaper;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.core.env.Environment;
import org.springframework.test.context.junit4.SpringRunner;

import com.rubypaper.controller.BoardController;

@RunWith(SpringRunner.class)
@SpringBootTest
public class PropertiesTest {

	@Test
	public void testMethod() {
	}
}
```

@Runwith 어노테이션은 실행할 러너를 설정합니다. 기본적으로 JUnit에서 제공하는 러너를 사용하는 대신 스프링 러너를 사용했습니다. 이 표시를 해야 @SpringBootTest 어노테이션을 사용할 수 있습니다. 이 어노테이션은 @SpringBootApplication과 같이 테스트케이스에 사용되는 빈을 초기화하며 자동설정을 해줍니다.

<br>

### 외부 프로퍼티 참조/재정의

테스트 케이스를 작성하다 보면 여러 테스트 케이스에서 사용하는 공통적인 데이터가 있습니다. 이러한 데이터들은 외부 프로퍼티로 등록하면 여러 테스트 데이터들을 재사용하거나 변경하기 쉽습니다.
application.properties에 다음과 같이 있다고 하겠습니다.

```c++
##WebApplication Type Setting
spring.main.web-application-type=servlet

##Server Setting
server.port:8080

##Test Property Setting
author.name=TESTER
author.age=53
```

아래와 같이 작성할 때, @SpringBootTest의 인자로 아무것도 넘겨주지 않는다면 외부 프로퍼티를 참조하여 결과를 출력하고 인자로 프로퍼티를 넘겨주면 재정의되어 재정의한 데이터가 출력되게 됩니다. 외부 프로퍼티를 사용하기 위해 Environment 클래스를 추가적으로 사용했습니다.

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes=BoardController.class,
				properties= {"author.name=Gurum","author.age=45","author.nation=Korea"})
public class PropertiesTest {

	@Autowired
	Environment environment;

	@Test
	public void testMethod() {
		System.out.println("이름 :" + environment.getProperty("author.name"));
		System.out.println("나이 :"+environment.getProperty("author.age"));
		System.out.println("국가 :"+environment.getProperty("author.nation"));
	}
}
```

<br>

### 목 객체로 테스트하기

웹 애플리케이션에서 사용되는 컨트롤러는 다른 소프트웨어의 도움이 필요하거나 생성하는데 많은 시간이 소요될 수 있습니다. 이러한 컨트롤러들은 매번 테스트하는 것은 어려우므로 필요한 기능만을 테스트하기 위해 목 객체를 만들어내어 테스트 합니다. 이 과정을 모킹이라고 합니다. 당연히 테스트를 위해서는 객체가 메모리에 올라와 있어야합니다. 실제 컨트롤러가 사용될 때는 서블릿 컨테이너가 구동되어야 하지만 테스트 환경에서는 서블릿 컨테이너를 모킹하여 사용하게 됩니다.

#### 1)@WebMvcTest

WebMvcTest는 @Controller와 @RestController가 설정된 클래스를 찾아 메모리에 생성합니다. 반면 @Service나 @Repository는 대상이 아니므로 메모리에 생성하지 않습니다. 아래 코드에서 서블릿 컨테이너를 모킹한 MockMvc 객체를 목업하여 테스트케이스에서 사용하게 됩니다.

```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class BoardControllerTest {

	@Autowired
	private MockMvc mockMvc;

	@Test
	public void testHello() throws Exception{
		mockMvc.perform(get("/hello").param("name","둘리"))
        .andExpect(status().isOk())
        .andExpect(content().string("hello:둘리"))
        .andDo(print())
	}
}
```

#### 2)@AutoConfigureMockMvc

@WebMvcTest와 비슷하게 사용할 수 있는 어노테이션입니다. 앞선 어노테이션과의 차이는 테스트 하지 않는 @Repository와 @Service 클래스 또한 메모리로 올리는 것입니다. 따라서 간단하게 컨트롤러만 테스트하려면 @WebMvcTest를 사용해야합니다. 이때 WebMvcTest와 SpringBootTest를 동시에 사용해선 안됩니다. 서로의 MockMvc 객체를 모킹하기 때문입니다. webEnvironment 설정으로 서블릿 컨테이너를 사용하지 않고 목 객체를 사용한다면 AutoConfigureMockMvc로 의존성을 주입받아 사용해야합니다.

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment=WebEnvironment.MOCK)
@AutoConfigureMockMvc
public class BoardControllerTest {

	@Autowired
	private MockMvc mockMvc;

	@Test
	public void testHello() throws Exception{
		mockMvc.perform(get("/hello").param("name","둘리"))
        .andExpect(status().isOk())
        .andExpect(content().string("hello:둘리"))
        .andDo(print())
	}
}
```

#### 3)내장 톰캣으로 테스트

목 객체로 테스트하는 대신 톰캣을 이용해서 테스트를 진행하고 싶을 때는 WebEnvironment.RANDOM_PORT로 설정을 바꾸어 테스트를 진행하면 됩니다. 목 객체를 대신할 객체로 TestRestTemplate 객체를 이용합니다. 목 객체를 사용하지 않아 DI를 할 필요가 없어 AutoConfigureMockMvc 설정 또한 없습니다.

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)
public class BoardControllerTest {

	@Autowired
	private TestRestTemplate restTemplate;

	@Test
	public void testHello() throws Exception{
		String result = restTemplate.getForObject("/hello?name=둘리",String.class);
		assertEquals("hello:둘리",result);
	}
}
```

<br>

### 서비스 계층을 연동하는 컨트롤러 테스트

실제 프로젝트에서 컨트롤러는 단순히 사용자의 요청을 받아들이는 역할만 합니다. 실제 비즈니스 로직을 처리하기 위해서는 비즈니스 컴포넌트를 호출해야합니다. 이를 위해 일반적으로 사용자를 위한 인터페이스를 정의한 후 이 인터페이스를 구현하는 방식으로 비즈니스 컴포넌트를 만들게 됩니다. 아래 두 코드는 각각 인터페이스와 해당 인터페이스를 구현한 클래스입니다.

```java
public interface BoardService {
	String hello(String name);
	BoardVO getBoard();
	List<BoardVO> getBoardList();
}
```

```java
@Service
public class BoardServiceImpl implements BoardService{

	@Override
	public String hello(String name) {
		return "hello:"+name;
	}

	@Override
	public BoardVO getBoard() {
		BoardVO board = new BoardVO();
		board.setSeq(1);
		board.setTitle("테스트 제목");
		board.setWriter("테스터");
		board.setContent("테스트 내용");
		board.setCreateDate(new Date());
		board.setCnt(0);
		return board;
	}

	@Override
	public List<BoardVO> getBoardList() {
		List<BoardVO> boardList = new ArrayList<BoardVO>();
		for(int i=1;i<=10;++i) {
			BoardVO board = new BoardVO();
			board.setSeq(i);
			board.setTitle("제목"+i);
			board.setWriter("테스터");
			board.setContent(i+"번 내용입니다.");
			board.setCreateDate(new Date());
			board.setCnt(0);
			boardList.add(board);
		}
		return boardList;
	}

}
```

즉 컨트롤러에서는 BoardService에 맞는 객체를 찾아 Autowired로 주입받아 메서드를 호출하여 사용합니다. 요청을 받아 처리를 서비스 레이어에 맡기고 결과를 받아 리턴하는 역할만 하는 것을 알 수 있습니다.

```java
@RestController
public class BoardController {
	public BoardController() {
		System.out.println("==> BoardController 생성");
	}

	@Autowired
	private BoardService boardService;

	@GetMapping("/hello")
	public String hello(String name) {
		return boardService.hello(name);
	}

	@GetMapping("/getBoard")
	public BoardVO getBoard() {
		return boardService.getBoard();
	}

	@GetMapping("/getBoardList")
	public List<BoardVO> getBoardList(){
		return boardService.getBoardList();
	}
}
```

비즈니스 컴포넌트가 미완성이거나 생성하는데 많은 시간이 필요하다면 비즈니스 컴포넌트 또한 @MockBean을 통해 모킹할 수 있습니다.
