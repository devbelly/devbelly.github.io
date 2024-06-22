---
title: "volatile"
date: 2022-08-05T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - java
---

CPU는 1초에 여러 개의 instruction을 한 번에 수행할 수가 있습니다. 매 수행마다 RAM에 접근해서 데이터를 읽어 온 후 CPU에서 instruction을 수행하는 것은 비효율적이므로 성능 향상을 위해 다양한 전략을 사용하는데 그 중 한가지는 caching 입니다.

![image](https://user-images.githubusercontent.com/67682840/182785872-1205dc59-beac-4e9c-bdfc-06418b1a7e4c.png)

CPU는 instruction을 수행하면서 연관된 데이터들을 모아 cache에 저장합니다. 이로써 성능향상을 기대할 수 있지만 `cache coherence`, 즉 캐시가 동일한 내용을 갖고 있는지에 대한 문제에 부딪히게 됩니다.

Multithread 환경에서 `Reader thread`와 `Writer thread`가 존재한다고 가정해보겠습니다. `Writer thread`에서 `volatile` 이 아닌 변수를 변경했을 때 즉시 메모리에 반영되지 않습니다. 이는 `write policy cache` 때문입니다. CPU는 write request를 메모리에 즉시 반영하는 대신 write buffer에 request를 저장했다가 메인 메모리에 한번에 반영합니다.

이러한 이유로 `Reader Thread`는 자신이 읽어온 값이 write 이후에 반영된 값인지 write 되기 전에 메인 메모리에서 읽어온 값인지 보증할수 없는 문제가 발생합니다.

아래는 변수 running에 `volatile`을 사용하지 않았을 때 예시입니다.

```java
public class ThreadTest {
    boolean running = true;

    public void test() {
        new Thread(new Runnable() {
            public void run() {
                int counter = 0;
                while (running) {
                    counter++;
                }
                System.out.println("Thread 1 finished. Counted up to " + counter);
            }
        }).start();
        new Thread(new Runnable() {
            public void run() {

                System.out.println("Thread 2 finishing");
                running = false;
            }
        }).start();
    }

    public static void main(String[] args) {
        new ThreadTest().test();
    }
}
```

여러번 실행하면 아래와 같은 두 결과를 얻을 수 있습니다.

![image](https://user-images.githubusercontent.com/67682840/182791241-99f19e33-f288-4554-9073-434f9ef3de65.png)

![image](https://user-images.githubusercontent.com/67682840/182791367-bcd5cf87-ebe1-494f-ab46-83c308fdcf56.png)

첫번째 사진은 프로그래머 의도와는 다르게 `Thread 1`이 종료가 되고 있지 않습니다. JVM이 변수 running을 caching 하기 때문입니다. 만일 `volatile` 을 추가하면 어떻게 될까요?

```java
public class ThreadTest {
    volatile boolean running = true;

    public void test() {
        new Thread(new Runnable() {
            public void run() {
                int counter = 0;
                while (running) {
                    counter++;
                }
                System.out.println("Thread 1 finished. Counted up to " + counter);
            }
        }).start();
        new Thread(new Runnable() {
            public void run() {

                System.out.println("Thread 2 finishing");
                running = false;
            }
        }).start();
    }

    public static void main(String[] args) {
        new ThreadTest().test();
    }
}
```

![image](https://user-images.githubusercontent.com/67682840/182793938-778046de-49a9-4807-8809-b6d38a26b058.png)

여러번 실행하더라도 `Thread 1`이 끝남이 보장됩니다.

즉 `volatile`은 변수를 쓰거나 읽을 때 메모리에서 직접 읽거나 쓰는 것을 보장합니다.

`visibility`를 보장하기 위해 `volatile`은 `happen-before`을 채택합니다. 여기서 `visibility`란 동시성을 보장할 때 나오는 단어 중 하나로 하나의 Thread에 변화가 발생했다면 다른 Thread 들이 이러한 변화를 알 수 있다는 뜻입니다.

이를 가능케 하기 위해서는 아래를 따라야 합니다

> A write to a volatile field happens-before every subsequent read of that field.

쉽게 말하자면 write가 다 이루어진 후에 read를 한다는 의미입니다. 어찌보면 `synchronized` 를 보장해주는 것 처럼 보이지만 `volatile`은 race condition, 즉 atomic 문제를 해결 못해주기 때문에 여러 Thread가 write를 한다면 `volatile`에 잘못된 값이 할당됩니다. `volatile`을 사용하는 이상적인 경우는 하나의 Thread만 쓰기 연산을 하고 나머지 Thread는 읽는 경우입니다.

<br>

## 참고

- https://www.baeldung.com/java-volatile
- https://stackoverflow.com/questions/106591/what-is-the-volatile-keyword-useful-for
