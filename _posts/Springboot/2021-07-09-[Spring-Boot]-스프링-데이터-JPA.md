---
title: "[Spring Boot] 스프링 데이터 JPA"
tags: springboot
categories: springboot
use_math: true
---

### Repository

스프링 데이터 JPA를 테스트하기 위해 새로운 프로젝트를 만든 후 엔티티 클래스를 작성하도록 합시다. 그 다음으로 Repository 인터페이스를 작성해야하는데 비즈니스 클래스에서는 Repository 인터페이스를 이용하여 실질적인 데이터베이스 연동작업을 하게 됩니다. Repository 인터페이스는 스프링에서 제공하는 네 개의 인터페이스중 하나를 선택하여 사용하면 됩니다.

![img](https://blog.kakaocdn.net/dn/bHusuu/btq85TStRx3/SnXYa7Ta3kt9FeQd9lVWh1/img.png)

위 세 개는 스프링 데이터 모듈, 아래 JpaRepsitory는 스프링 데이터 JPA에서 제공하는 인터페이스입니다. 인터페이스들은 공통적으로 두 개의 제네릭 타입을 지정하게 됩니다. 첫 번째는 매핑할 엔티티 클래스를 적고 두 번째는 해당 엔티티의 Id 타입을 적게 됩니다.

일반적으로 인터페이스는 구현할 클래스를 작성하기 위해 사용합니다. 하지만 위 인터페이스를 상속하게 되면 직접 클래스를 작성하지 않아도 사용할 수 있습니다. 이는 스프링 부트가 내부적으로 구현 객체를 자동으로 생성해주기 때문입니다. 또한 JPA를 이용한 데이터베이스 연동을 위해 사용했던 EntityManagerFactory, EntityManager, EntityTransaction 역시 스프링 데이터 JPA에서 자동으로 객체를 생성하여 활용하기 때문에 필요 없습니다. 다음과 같이 BoardRepository 인터페이스를 정의한 후 CRUD 기능을 하나씩 테스트 해봅시다.

```java
package com.rubypaper.persistence;

import org.springframework.data.repository.CrudRepository;

import com.rubypaper.domain.Board;

public interface BoardRepository extends CrudRepository<Board,Long>{

}
```

<br>

### 등록기능 테스트

등록기능 테스트를 위해 다음과 같은 테스트 케이스를 작성합시다. 기존에 영속성 컨텍스트에 저장하기 위해 persist() 메서드를 사용했지만 CRUDRepository 에서는 save() 메서드를 사용합니다.

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class BoardRepositoryTest {
	@Autowired
	private BoardRepository boardRepo;

	@Test
	public void test() {
		Board board = new Board();
		board.setCnt(0L);
		board.setContent("잘 등록되나요?");
		board.setCreateDate(new Date());
		board.setWriter("테스터");
		board.setTitle("첫 번째 게시글");

		boardRepo.save(board);
	}
}
```

실행해보면 잘 처리되었음을 알 수 있습니다.

![img](https://blog.kakaocdn.net/dn/r6gVU/btq85UKCvRA/j5ByKtgCw12nKkyywMcKV0/img.png)

<br>

### 조회기능 테스트(1개)

1개의 데이터를 조회하기 위해서는 findById() 메서드를 사용합니다. 리턴값으로 Optional 타입의 객체가 리턴되는데 추가적으로 get() 메서드를 통해 영속성 컨텍스트에 저장된 Board 객체를 얻을 수 있습니다.

```java
	@Test
	public void testGetBoard() {
		Board board = boardRepo.findById(1L).get();
		System.out.println(board);
	}
```

<br>

### 수정기능 테스트

객체를 영속성 컨텍스트에서 가져와야하므로 처음에 조회기능을 통해 원하는 객체를 영속성 컨텍스트에 넣은 후 수정하고 다시 save() 메서드를 통해 수정을 반영한다

```java
	@Test
	public void testUpdateBoard() {
		Board board=boardRepo.findById(1L).get();

		board.setTitle("제목을 수정했습니다");
		boardRepo.save(board);
	}
```

<br>

### 삭제기능 테스트

```java
	@Test
	public void testDeleteBoard() {
		boardRepo.deleteById(1L);
	}
```

<br>

### 쿼리 메서드

애플리케이션을 개발할 때는 목록 검색에 대한 기능이 필요하며 스프링은 JPQL을 이용하여 목록 검색에 대한 기능을 처리합니다. SQL과 달라진 것은 테이블에 질의를 하는 대신 엔티티에 질의를 하는 것입니다. 복잡한 JPQL을 이용하기 위해 쿼리 메서드를 제공하여 메서드의 이름으로 필요한 쿼리를 제공해줍니다.

가장 많이 사용되는 쿼리 메서드는 엔티티에서 특정 변수만 조회하는 것입니다. 메서드 이름을

<p align=center>
	find + 엔티티 이름 + By + 변수이름
</p>

와 같이 한다면 특정 변수만을 조회할 수 있고 엔티티 이름은 생략가능합니다. Title 변수를 조회하는 쿼리 메서드를 작성한 후 테스트 해봅시다.

```java
public interface BoardRepository extends CrudRepository<Board,Long>{
	List<Board> findBytitle(String searchKeyword);
}
```

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class QueryMethodTest {
	@Autowired
	private BoardRepository boardRepo;

	@Before
	public void dataPrepare() {
		for(int i=1;i<=200;++i) {
			Board board = new Board();
			board.setTitle("테스트 제목 " + i);
			board.setWriter("테스트");
			board.setContent("테스트 내용" +i);
			board.setCreateDate(new Date());
			board.setCnt(0L);
			boardRepo.save(board);
		}
	}

	@Test
	public void testFindByTitle() {
		List<Board> boardList =boardRepo.findBytitle("테스트 제목 10");

		System.out.println("검색결과");
		for(Board board: boardList) {
			System.out.println(board);
		}
	}
}
```

<br>

### 특정 단어를 포함하는 질의

SQL에서 LIKE 기능을 사용하기 위해서는 쿼리 메서드에서 Containing 을 활용합니다. BoardRepository에 findByContentContaining() 메서드를 추가한 후 content에 "17"이 포함된 Board 객체들의 목록을 조회해보도록 합시다.

```java
public interface BoardRepository extends CrudRepository<Board,Long>{
	List<Board> findBytitle(String searchKeyword);
	List<Board> findByContentContaining(String searchKeyword);
}
```

```java
	@Test
	public void testFindByContentContaining() {
		List<Board> boardList = boardRepo.findByContentContaining("17");
		for(Board board : boardList) {
			System.out.println(board);
		}
	}
```

<br>

### 검색결과 정렬

검색결과를 정렬하기 위해서는 OrderBy + 변수 + Asc / Desc을 사용하면 됩니다. Title에 17이 포함된 board를 Seq를 기준으로 내림차순 정렬하기 위해선 다음과 같은 쿼리 메서드를 작성하면 됩니다.

```java
	List<Board> findByTitleContainingOrderBySeqDesc(String searchKeyword);
```

마찬가지로 테스트 케이스를 작성후 실행하면 다음과 같은 결과가 출력됩니다.

![img](https://blog.kakaocdn.net/dn/qEhkW/btq89kDdGRa/mSLl6MB3m30HQKYDL0ntmk/img.png)

<br>

### 페이징 및 정렬

모든 쿼리 메서드는 마지막 파라미터로 Pageable 인터페이스와 Sort 인터페이스를 추가할 수 있습니다. 다음은 Title의 검색결과를 한 페이지에 5개를 보여주기 위한 쿼리 메서드입니다. 해당 쿼리 메서드는 1페이지입니다.

```java
List<Board> findByTitleContaining(String searchKeyword,Pageable paging);
```

```java
	@Test
	public void testFindByTitleContaining() {
		Pageable paging = PageRequest.of(0, 5);
		List<Board> boardList = boardRepo.findByTitleContaining("17", paging);

		for(Board board:boardList) {
			System.out.println(board);
		}
	}
```

추가적으로 정렬된 검색결과를 페이징하기 위해서는 다음과 같이 수정하면 됩니다.

```java
	@Test
	public void testFindByTitleContaining() {
		Pageable paging = PageRequest.of(0, 5,Sort.Direction.DESC,"Seq");
		List<Board> boardList = boardRepo.findByTitleContaining("17", paging);

		for(Board board:boardList) {
			System.out.println(board);
		}
	}
```

![img](https://blog.kakaocdn.net/dn/bVLV15/btq88xwGpEx/Y9lKTSqnl5H1O4SffvPoDk/img.png)
