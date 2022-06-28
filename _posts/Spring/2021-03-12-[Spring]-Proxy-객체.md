---
title: "[Spring] Proxy 객체"
tags: spring
categories:
  - Spring
---

### AOP를 위한 모듈 추가

aop 프로그래밍을 위해서 추가해야하는 모듈은 spring-aop와 aspectjweaver 모듈입니다. 각각은 AOP의 실행과 설정에 필요한 모듈입니다. spring-context를 추가할 때, spring-aop 모듈 또한 적용이 되므로 aspectjweaver을 dependency에 추가하도록 합시다. 이를 통해 설정에 필요한 어노테이션을 사용할 수 있게 됩니다.

```java
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.8.13</version>
    </dependency>
```

<br>

### 문제점1) 기존 코드의 수정

다음 두 함수의 실행시간을 비교를 해야한다고 가정하겠습니다. 하나는 계승을 반복문을 통해 구하는 함수이고 다른 하나는 재귀를 통해 구하는 것입니다.

```java
package chap07;

public class ImpeCalculator implements Calculator{

	@Override
	public long factorial(long num) {
		long ans =1;
		for(int i=1;i<=num;++i) {
			ans*=i;
		}
		return ans;
	}
}
```

```java
package chap07;

public class REcCalculator implements Calculator{
	@Override
	public long factorial(long num) {
		if(num==0) return 1;
		return num*factorial(num-1);
	}
}
```

함수가 시작한 직후와 반복문 종료후에 다음과 같은 코드를 추가하여 실행시간을 구할 수 있습니다.

```java
package chap07;

public class ImpeCalculator implements Calculator{

	@Override
	public long factorial(long num) {
		long start = System.currentTimeMillis();
		long ans =1;
		for(int i=1;i<=num;++i) {
			ans*=i;
		}
		long end = System.currentTimeMillis();
		System.out.printf("실행시간은 %d 입니다.\n", end-start);
		return ans;
	}
}
```

하지만 재귀함수로 구현한 부분에서 위와 같이 구현하면 출력문이 여러번 나오는 문제가 발생합니다. 이를 해결하기 위해 실행코드에서 직접 시간을 구하는 것보다 이를 호출하는 메서드에서 계승 메서드를 호출하기 전과 후에 시스템 시간을 기록하면 반복 출력되는 문제를 해결할 수 있습니다

```java
package main;

import chap07.*;

public class halfProxy {
	public static void main(String[] args) {
		Calculator ex1 = new ImpeCalculator();
		long st = System.currentTimeMillis();
		ex1.factorial(5000);
		long ed = System.currentTimeMillis();

		System.out.printf("실행시간은 %d입니다\n", ed-st);

		Calculator ex2 = new REcCalculator();

		st = System.currentTimeMillis();
		ex2.factorial(5000);
		ed = System.currentTimeMillis();

		System.out.printf("실행시간은 %d입니다\n", ed-st);
	}
}
```

<br>

### 문제점2) 코드의 중복

하지만 위 코드 또한 효율적이지 못한 부분이 존재합니다. Millies를 구하는 설정에서 Nano로 바꾸고 싶다면 8, 10, 16, 18째 줄을 System.NanoTime()으로 전부 고쳐야하기 때문입니다.

#### 프록시 객체

기존코드의 수정을 피하고 중복되는 코드를 제거하려면 어떡해야할까요? 이러한 문제점을 해결하기 위해 프록시 객체를 사용합니다. 프록시 객체의 특징은 핵심 기능은 다른 객체에 위임하고 자신은 여러 객체가 공통적으로 실행할 수 있는 공통 기능을 구현합니다. 아래는 이 내용들을 구현한 프록시입니다.

```java
package chap07;

public class ExecCalculator implements Calculator{
	private Calculator delegate;

	public ExecCalculator(Calculator calcu) {
		this.delegate=calcu;
	}

	@Override
	public long factorial(long num) {
		long start = System.nanoTime();
		long exec = delegate.factorial(num);
		long end= System.nanoTime();
		System.out.printf("실행시간은 %d 입니다\n", end-start);
		return exec;
	}
}
```

메인함수에서 프록시 객체를 다음과 같이 사용하면 됩니다.

```java
package main;

import chap07.*;

public class Proxy {
	public static void main(String[] args) {
		ExecCalculator cal1 = new ExecCalculator(new ImpeCalculator());
		cal1.factorial(3);

		ExecCalculator cal2= new ExecCalculator(new REcCalculator());
		cal2.factorial(3);
	}
}
```
