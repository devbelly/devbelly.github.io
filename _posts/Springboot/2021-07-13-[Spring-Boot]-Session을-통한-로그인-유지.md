---
title: "[Spring Boot] Session을 통한 로그인 유지"
tags: springboot
categories:
  - Spring boot
use_math: true
---

### 로그인 인증

게시 글 목록을 회원만 조회하기 위해서는 접근하는 사용자가 데이터베이스 Member 테이블에 있는 사용자인지 확인을 하는 과정을 거쳐야합니다. 이를 위한 비즈니스 컴포넌트를 만든 후, LoginController을 작성해줍시다. Repository와 Service 인터페이스 및 구현에 대한 코드는 생략하도록 하겠습니다.

```java
@SessionAttributes("member")
@Controller
public class LoginController {

	@Autowired
	private MemberService memberService;

	@GetMapping("/login")
	public void loginView() {

	}

	@PostMapping("/login")
	public String login(Member member,Model model) {
		Member findMember = memberService.getMember(member);

		if(findMember!=null && findMember.getPassword().equals(member.getPassword())) {
			model.addAttribute("member",findMember);
			return "forward:getBoardList";
		}
		else {
			return "redirect:login";
		}
	}
}
```

SessionAttributes는 세션에 데이터를 저장할 때 사용합니다. Model 객체에 "member"라는 이름으로 저장된 데이터를 자동으로 세션에 저장합니다. 로그인 폼에서 입력받은 커맨드객체를 서비스객체를 통해 확인하여 유효하다면 Model 객체에 조회한 데이터를 담고 유효하지 않다면 다시 로그인 페이지로 리다이렉트합니다.

<br>

### 로그인 상태 유지

localhost:8080/getBoardList 와 같은 요청을 보내면 로그인을 하지 않았음에도 불구하고 게시 글 목록이 불러와지는 것을 확인할 수 있습니다. 우리가 저장한 세션을 BoardController에서 활용하도록 해봅시다.

```java
@Controller
@SessionAttributes("member")
public class BoardController {

	@Autowired
	private BoardService boardService;

	@ModelAttribute("member")
	public Member setMember() {
		System.out.println("*********setMember()*******");
		return new Member();
	}

	@RequestMapping("/getBoardList")
	public String getBoardList(@ModelAttribute("member") Member member,Model model,Board board) {
		System.out.println("getBoardList");
		if(member.getId()==null) {
			return "redirect:login";
		}
		List<Board> boardList =boardService.getBoardList(board);
		model.addAttribute("boardList",boardList);
		return "getBoardList";
	}
```

@SessionAttribute는 Model에 "member"라는 이름으로 저장된 데이터를 세션에 저장합니다. method level에서 사용된 @ModelAttribute는 @RequestMapping이 실행되기 전 Model의 "member" 키 값을 초기화해주는 역할을 합니다. parameter 에서 사용된 @ModelAtrritube는 세션에 저장된 member 객체를 확인하여 바인딩하게 됩니다. 만일 id값이 null이라면 세션이 존재하지 않는 것이므로 올바르지 않은 요청입니다. 즉 로그인 상태에서만 getBoardList 요청을 할 수 있게 되었습니다.

<br>

### 로그아웃

sessionStatus 객체를 받아 setComplete() 메서드를 호출하여 세션을 종료하도록 구현하면 됩니다.

```java
	@GetMapping("/logout")
	public String logout(SessionStatus status) {
		status.setComplete();
		return "redirect:index.html";
	}
```
