---
title: "[Spring Boot] JPA"
tags: springboot
categories:
  - Spring boot
use_math: true
---

### 데이터베이스 연동

데이터베이스 연동에 사용되는 기술은 JDBC, 마이바티스, 하이버네이트와 같은 ORM에 이르기까지 매우 다양합니다. 특히 하이버네이트 같은 경우는 개발자가 직접 SQL을 작성하지 않아도 된다는 점에서 굉장히 편리합니다. 이후 여러 ORM이 등장하고 이를 표준화 한것이 JPA 입니다. 스프링 데이터 JPA는 복잡한 JPA를 간단하게 사용할 수 있게 해주는 스프링 모듈입니다.

연동 기술은 크게 SQL을 직접 다루는 기술과 프레임 워크가 SQL을 생성해주는 기술로 나눌 수 있습니다. 마이바티스는 SQL을 XML 파일에 직접 저장하여 사용하게 됩니다. 만일 추가적인 기능이 필요해 SQL을 수정하게 된다면 해당 SQL을 사용하는 코드또한 수정되어 번거롭습니다. 하지만 SQL을 직접 다루지 않는 ORM은 VO가 가지고 있는 정보를 데이터베이스에 저장한다기 보다는 MAP에 저장하는 것에 가깝습니다. 이럴 경우, 코드가 굉장히 단순해집니다.

<br>

### JPA

JPA는 자바 컬렉션에 객체를 저장하고 CRUD를 처리하는 개념과 비슷합니다. 하지만 결국 VO와 테이블의 매핑을 처리해주어야합니다. JPA는 개발자가 JDBC API에 직접 접근하여 처리하는 일을 대신해줍니다. 마지막으로 JPA를 구현해줄 프레임워크를 설정하면 됩니다.
![img](https://blog.kakaocdn.net/dn/c6DWix/btq8PuzNX6y/ZbbDYt3TG251o8WruGoNY0/img.png)

JPA를 사용하는 첫 걸음은 테이블과 매핑되는 엔티티 클래스를 작성하는 것입니다. 이클립스의 JPA 퍼스펙티브가 제공하는 엔티티 추가 기능을 사용하면 생성과 동시에 persistence.xml 파일에 엔티티를 자동으로 추가해줍니다.

```java
/**
 * Entity implementation class for Entity: Board
 *
 */
@Entity
@Table
public class Board  {
	@Id
	@GeneratedValue
	private Long seq;
	private String title;
	private String writer;
	private String content;
	private Date createDate;
	private Long cnt;
	~ 생략
}
```

<br>

### Persistence.xml

JPA는 META-INF폴더 안에 있는 persistence.xml 파일을 로딩하여 영속성 유닛에 대한 정보를 얻습니다. 영속성 유닛은 연동할 데이터베이스에 대한 정보가 담겨 있습니다.

<persistence>를 루트로 하여 <persistence-unit> 엘리먼트를 갖게 됩니다. 영속성 유닛은 연동할 데이터베이스당 하나씩 갖게 되므로 만일 여러 데이터베이스와 연동한다면 해당 수에 맞는 <persistence-unit> 엘리먼트를 사용하게 됩니다.

JDBC에서도 마찬가지로 해주어야하는 드라이버 설정과 JPA를 구현할 구현체 클래스를 설정해주면 됩니다.

```c++
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.1" xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd">
	<persistence-unit name="Chapter04">
		<class>com.rubypaper.domain.Board</class>

		<properties>
			<!-- 필수 속성 -->
			<property name="javax.persistence.jdbc.driver" value="org.h2.Driver" />
			<property name="javax.persistence.jdbc.user" value="sa" />
			<property name="javax.persistence.jdbc.password" value="" />
			<property name="javax.persistence.jdbc.url"	value="jdbc:h2:tcp://localhost/~/test" />
			<property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" />
			~생략~
		</properties>
	</persistence-unit>

</persistence>
```

JPA는 persistence.xml으로부터 영속성 유닛에 대한 정보를 얻고 이로부터 EntityManagerFactory 객체를 얻습니다. JPA로 CRUD기능을 처리하기 위해서는 EntityManager 객체가 필요한데, Factory 객체로부터 이를 얻어 올 수 있습니다. EntityManager로 BOARD객체를 추가하기 위해 persist 메서드를 사용합니다.

```java
		EntityManagerFactory emf=Persistence.createEntityManagerFactory("Chapter04");
		EntityManager em = emf.createEntityManager();

		try {
			Board board=new Board();
			board.setTitle("JPA 제목");
			board.setWriter("관리자");
			board.setContent("JPA 글 등록 잘 되네요");
			board.setCreateDate(new Date());
			board.setCnt(0L);

			em.persist(board);
		}catch(Exception e) {
			e.printStackTrace();
		}finally {
			em.close();
			emf.close();
		}
```

이를 실행하면 콘솔에서 H2 기반의 SQL들이 실행됨을 알 수 있습니다. 하지만 H2데이터베이스에 들어가서 BOARD 테이블을 조회해보면 아무런 값이 없는 것을 확인할 수 있습니다.

<br>

### 트랜잭션

JPA에서 BOARD 엔티티를 테이블에 저장하기 위해서는 CRUD 기능이 트랜잭션내에서 실행되어야합니다. EntityManager에서 트랜잭션 객체를 얻어와 다음과 같이 코드를 수정합니다.

```java
		EntityManagerFactory emf=Persistence.createEntityManagerFactory("Chapter04");
		EntityManager em = emf.createEntityManager();

		EntityTransaction tx = em.getTransaction();
		try {
			tx.begin();

			Board board=new Board();
			board.setTitle("JPA 제목");
			board.setWriter("관리자");
			board.setContent("JPA 글 등록 잘 되네요");
			board.setCreateDate(new Date());
			board.setCnt(0L);

			em.persist(board);
			tx.commit();
		}catch(Exception e) {
			e.printStackTrace();
			tx.rollback();
		}finally {
			em.close();
			emf.close();
		}
```

정상적으로 데이터베이스에 글이 등록되었음을 알 수 있습니다. 다음의 코드를 수정하여 검색기능을 구현해 봅시다.

```java
		try {
			Board searchBoard = em.find(Board.class, 1L);
			System.out.println(searchBoard.toString());
		}
```

검색은 트랜잭션 처리가 필요없으므로 트랜잭션 부분을 제외하고 try 블록안을 위와 같이 수정하면 됩니다. 콘솔에 다음과 같이 제대로 처리가 되었음을 확인할 수 있습니다.

![img](https://blog.kakaocdn.net/dn/AunWj/btq8TwrnfVZ/EaJ1ViRVPGPtjNjRfgQdcK/img.png)

<br>

### 엔티티 어노테이션

엔티티에 사용하는 어노테이션은 엔티티임을 나타내는 @Entity, 엔티티 이름과 테이블 명이 다를 때 매핑하기 위한 @Table, 임시검색변수와 매핑하지 않기 위한 @Transient, 날짜 타입과 매핑을 위한 @Temporal 등이 있습니다.
