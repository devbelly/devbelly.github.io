---
title: "[Spring Boot] Querydsl, 양방향 다대일 매핑"
tags: springboot
categories:
  - Spring boot
---

### Querydsl

웹 애플리케이션에서 SQL은 개발/운용과정에서 수시로 바뀌게 됩니다. 다양한 검색쿼리를 미리 저장해서 사용하면 비슷한 쿼리들이 많아 관리하기가 어렵습니다. 이를 해결하기 위해 마이바티스에서는 동적 쿼리를 제공합니다. JPA에서는 @Query로 쿼리들을 관리하게 되는데 프로젝트를 로딩하는 시점에서 SQL들이 파싱되어 정적 SQL만 사용가능합니다. 따라서 동적쿼리를 이용하기 위해선 Querydsl을 사용해야합니다.

Querydsl을 사용하기 위해 querydsl-apt 와 querydsl-jpa를 추가한 후 버전정보를 삭제해주도록 합시다. 버전을 명시할 경우 스프링부트에서 참조하는 querydsl과 명시한 버전이 맞지 않으면 오류가 발생할 수 있습니다.

다음으로 쿼리용 클래스를 얻기 위해 플러그인을 추가하면 QBoard 클래스를 얻을 수 있습니다.

사용할 준비는 이로써 끝났습니다. 기존에 했던 것과 마찬가지로 DynamicBoardRepository 를 작성해주도록 합시다. CRUDRepository와 추가적으로 querydsl을 위한 QuerydslPredicateExecutor 인터페이스를 상속해야합니다.

```java
public interface DynamicBoardRepository extends CrudRepository<Board, Long>, QuerydslPredicateExecutor<Board>{

}
```

인터페이스를 추가적으로 상속했으므로 사용가능한 메서드들이 늘었습니다.

| 메소드                                 | 메소드 설명                  |
| -------------------------------------- | ---------------------------- |
| long count(Predicate p)                | 검색된 데이터의 전체 개수    |
| boolean exists(Predicate p)            | 검색된 데이터의 존재 여부    |
| Iterable<T> findAll(Predicate p)       | 조건에 맞는 데이터 목록      |
| Page<T> findAll(Predicate p)           | 조건에 맞는 데이터 목록      |
| Iterable<T> findAll(Predicat p,Sort s) | 조건에 맞는 데이터 목록 정렬 |
| T findOne(Predicate p)                 | 조건에 맞는 하나의 데이터    |

Predicate 인터페이스를 구현한 BooleanBuilder 클래스를 통해 검색조건을 설정하게 됩니다. Repository를 작성했다면 제대로 동작하는지 테스트하기 위해 테스트케이스를 작성합니다.

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class DynamicQueryTest {

	@Autowired
	DynamicBoardRepository boardRepo;

	@Test
	public void testDynamicQuery() {
		String searchCondition = "TITLE";
		String searchKeyword = "테스트 제목 10";

		BooleanBuilder builder = new BooleanBuilder();
		QBoard qboard = QBoard.board;

		if(searchCondition.equals("TITLE")) {
			builder.and(qboard.title.like("%"+searchKeyword+"%"));
		}else if(searchCondition.equals("CONTENT")) {
			builder.and(qboard.content.like("%"+searchKeyword+"%"));
		}
		Pageable paging=PageRequest.of(0, 5);
		Page<Board> info = boardRepo.findAll(builder,paging);

		System.out.println("검색결과-->");
		for(Board board : info) {
			System.out.println(board.toString());
		}
	}
}
```

<br>

### 연관관계 매핑

#### (1)단방향 다대일 관계

단방향 다대일 연관 매핑을 테스트하기 위해 다음과 같은 조건을 가정하겠습니다.

- 게시판과 회원 존재
- 한 회원은 여러개의 게시판 작성 가능
- 게시 글을 통해서 게시 글을 작성한 회원정보 조회 가능

회원 클래스를 아래와 같이 작성합니다. 추가함과 동시에 자동으로 QMember 클래스도 자동으로 생성됩니다.

```java
@Getter
@Setter
@ToString
@Entity
public class Member {
	@Id
	@Column(name="MEMBER_ID")
	private String id;
	private String password;
	private String name;
	private String role;
}
```

![img](https://blog.kakaocdn.net/dn/JfLnc/btq9dwJ2uAZ/k7pohaMRge9vYPU8AF6kBk/img.png)

Board 클래스를 아래와 같이 설정한 후 Member 엔티티를 위한 Repository를 작성해줍시다.

```java
public class Board {
	@Id @GeneratedValue
	private Long seq;
	private String title;
//	private String writer;
	private String content;
	@Temporal(value=TemporalType.TIMESTAMP)
	private Date createDate;
	private Long cnt;

	@ManyToOne
	@JoinColumn(name="MEMBER_ID")
	private Member member;
}
```

```java
public interface MemberRepository extends CrudRepository<Member,String>{

}
```

테스트 데이터를 넣은 후 테스트 케이스를 실행하면 다음과 같은 결과가 보이게 됩니다.

```java
	@Test
	public void testManyToOneSelect() {
		Board board = boardRepo.findById(5L).get();
		~생략~
	}
```

![img](https://blog.kakaocdn.net/dn/bVXp8s/btq9aF8SSYc/sJdIWeoyeB9erHJyhWgfqK/img.png)
LEFT OUTER JOIN을 사용하는 모습입니다. 외부 조인은 내부 조인에 비해 성능이 떨어집니다. Driving Table을 기준으로 이중 포문을 사용하기 때문입니다. 만일 참조변수에 null이 들어와 있지 않는 것이 확실하다면 nullable 속성을 추가해주면 됩니다. 실행결과는 INNER JOIN입니다.

```java
	@ManyToOne
	@JoinColumn(name="MEMBER_ID",nullable=false)
	private Member member;
```

![img](https://blog.kakaocdn.net/dn/9KdfR/btq9e6RtMeC/2w0HNIjFXTMKQ6AGYrrLfk/img.png)

<br>

#### (2) 양방향 다대일 관계

테이블에서는 하나의 외래키 설정으로 양방향 매핑이 설정되지만 엔티티에서는 양방향 매핑을 설정하기 위해서는 단방향 다대일 관계와 단방향 일대다 관계를 설정함으로써 양방향 매핑을 설정하게 됩니다. Member 엔티티에서 회원이 작성한 글을 조회하기 위해 일대다 관계를 설정해봅시다. 하나의 Member 객체는 여러 개의 글을 쓸 수 있으므로 List<Board>를 통해 작성 글을 참조하게 됩니다.

```java
	@OneToMany(mappedBy="member", fetch=FetchType.EAGER)
	private List<Board> boardList =new ArrayList<Board>();
```

OneToMany에서는 mappedBy속성과 fetch 속성을 사용할 수 있습니다. fetch 속성은 멤버들을 조회할 때 게시 글 정보도 같이 조회할지를 결정하는 속성입니다. @OneToMany에서는 기본적으로 LAZY가 설정되어 있지만 게시 글 정보도 함께 조회하기 위해 EAGER 속성을 사용했습니다.

중요한 것은 mappedBy 속성입니다. 앞서 설명한대로 테이블은 하나의 외래키 설정으로 양방향 관계를 매핑하지만 엔티티에서는 두 개의 단방향 매핑관계를 통해 양방향을 표현하게 됩니다. 두 개의 참조변수중 한 개를 정해서 테이블의 외래키를 관리해야합니다. 이를 연관관계 주인이라 합니다. 일반적으로 두 개의 테이블중 외래키가 있는 테이블을 연관관계 주인으로 하기 때문에 Board 엔티티의 Board.member 변수가 주인이 됩니다. 나머지 엔티티에서는 자신이 연관관계의 주인이 아님을 알리기 위해 사용하는 속성이 mappedBy 입니다.

실행하게 되면 StackOverflowError가 발생합니다.

![img](https://blog.kakaocdn.net/dn/bKKRp0/btq9cIjSjZ2/2Hbs6FQ9eTJOZAkLasAtPk/img.png)

이유는 Lombok에서 제공하는 ToString()에서 상호참조가 일어나기 때문입니다. Board 엔티티에서 Board.member을 출력하기 위해 Member 클래스의 ToString()을 호출하는데 Member 엔티티내에서 List<Board>를 출력하기 위해 Board의 ToString()을 참조하기 때문입니다. exclude 속성을 추가함으로써 상호 호출을 끊어야합니다.

```java
@ToString(exclude="member")
@Entity
public class Board {
}
```

```java
@ToString(exclude="boardList")
@Entity
public class Member {
}
```

상호 호출을 끊으면 제대로 실행됨을 알 수 있습니다.
![img](https://blog.kakaocdn.net/dn/IBqrh/btq9e5Zs9WX/IcP0rykzmRPRyvZ1XwVwWK/img.png)

<br>

### 영속성 전이

JPA에서는 부모 엔티티를 영속성 컨텍스트에 저장하면 자식 엔티티도 영속성 컨텍스트에 저장을 해주거나 부모 엔티티를 제거하면 자식 엔티티도 제거를 해주는 영속성 전이를 제공합니다. 데이터베이스에서 cascade 설정을 통해 연관 테이블들 동시에 삭제하는 것처럼 JPA도 cascade 속성을 통해 영속성 전이가 가능합니다. 부모 엔티티에 cascade 설정을 추가해줍시다.

```java
	@OneToMany(mappedBy="member", fetch=FetchType.EAGER,cascade = CascadeType.ALL)
	private List<Board> boardList =new ArrayList<Board>();
```

ALL 설정은 member 객체가 영속화되거나 수정, 삭제되면 board 객체또한 영향을 받는다는 의미입니다. Board 클래스에서 member을 설정할 때, member 객체에 자기자신이 List<Board>에 추가되기 위해 추가적으로 setMember() 메서드를 작성합시다.

```java
	public void setMember(Member member) {
		this.member=member;
		member.getBoardList().add(this);
	}
```

마지막으로 영속성 전이를 테스트해봅시다.

```java
	@Test
	public void testManyToOneInsert() {
		Member member1 = new Member();
		member1.setId("member1");
		member1.setPassword("member111");
		member1.setName("둘리");
		member1.setRole("User");
//		memberRepo.save(member1);

		Member member2 = new Member();
		member2.setId("member2");
		member2.setPassword("member222");
		member2.setName("도우너");
		member2.setRole("Admin");
//		memberRepo.save(member2);

		for (int i = 1; i <= 3; i++) {
			Board board = new Board();
			board.setMember(member1);
			board.setTitle("둘리가 등록한 게시글 " + i);
			board.setContent("둘리가 등록한 게시글 내용 " + i);
			board.setCreateDate(new Date());
			board.setCnt(0L);
//			boardRepo.save(board);
		}
		memberRepo.save(member1);

		for (int i = 1; i <= 3; i++) {
			Board board = new Board();
			board.setMember(member2);
			board.setTitle("도우너가 등록한 게시글 " + i);
			board.setContent("도우너가 등록한 게시글 내용 " + i);
			board.setCreateDate(new Date());
			board.setCnt(0L);
//			boardRepo.save(board);
		}
		memberRepo.save(member2);
	}
```

board를 영속화하는 코드를 제거하더라도 member만 영속화 하면 관련된 board 객체들 또한 영속화됨을 알 수 있습니다.

```java
	@Test
	public void testCascadeDelete() {
		memberRepo.deleteById("member2");
	}
```

실행해보면 member2와 관련된 BOARD가 삭제됨을 알 수 있습니다.
