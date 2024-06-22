---
title: "JVM architecture"
date: 2022-09-19T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - java
---

## JVM(Java Virtual Machine)

- 자바를 실행하기 위한 가상의 프로그램
- OS에 종속적이지 않기 위해 OS위에서 실행
- 자동으로 메모리를 관리하거나 최적화

## 특징

#### 1. 스택기반의 가상 머신

JVM specification을 구현했다면 누구든 JVM을 만들 수 있습니다. 핵심 개념은 물리적인 CPU에 의해 처리되는 동작을 흉내내는 것으로 아래와 같은 컨셉들을 가지고 있어야 합니다.

- 소스 코드를 VM이 실행가능한 바이트 코드로 변환
- 명령어와 피연산자를 포함하는 데이터구조
- 함수를 실행하기 위한 콜 스택
- 다음 실행할 곳을 지정하는 포인터

JVM을 구현하는 방법은 스택기반과 레지스터기반이 있는데 차이는 피연산자를 저장하고 다시 가져오는 메커니즘의 차이입니다. 스택기반의 가상머신이 갖는 장점 중 한가지는 피연산자들의 주소를 저장할 필요가 없다는 것입니다.

#### 2. 가비지 콜렉션

동적으로 할당된 메모리 영역중에서 어떠한 변수도 가리키지 않는 메모리 영역을 찾아 자동으로 해제하는 기술입니다.

#### 3. 심볼릭 레퍼런스

자바에서는 특정 객체를 참조할때 Memory address를 직접 참조하는 것이 아니라 객체의 이름으로 참조합니다. 이렇게 객체의 이름으로 참조하는 것을 Symbolic Reference라고합니다.

Symbolic Reference는 Constant pool에 저장되며 현재의 코드를 의존하는 다른 클래스와 연결할 때 JVM에 의해 사용됩니다. 예를 들어 `System.out.println("Hello, world!");`을 컴파일하여 바이트 코드를 얻는다면 다음과 같습니다.

```
   0:   getstatic       #2; //Field java/lang/System.out:Ljava/io/PrintStream;
   3:   ldc     #3; //String Hello, world!
   5:   invokevirtual   #4; //Method java/io/PrintStream.println:(Ljava/lang/String;)
```

여기서 `#n`에 해당하는 부분들이 symbolic reference이며 각각 `#2`는 System.out 필드, `#3` 은 `Hello, world!` 문자열, `#4`는 PrintStream.println()에 대한 symbolic reference입니다.

> **Constant Pool?** <br>
> Constant pool은 클래스의 코드를 실행하기 위해 필요한 constant를 저장하는 공간입니다. 이러한 constant들은 프로그래머에 의해 생성된 리터럴과 컴파일러에 의해 생성된 symbolic reference를 포함합니다.

## 컴파일 & 실행과정

![image](https://user-images.githubusercontent.com/67682840/191056318-5d9f34e9-a977-4341-af0d-4d79e1ae6478.png)

- 소스코드를 `javac` 컴파일러를 통해 바이트코드(.class)로 컴파일한다.
- 클래스로더는 바이트 코드를 메모리에 올린다.
- Execution Engine에서 메모리를 참조하여 실행한다

## 바이트 코드 vs 기계어

- CPU가 이해할 수 있는 언어가 기계어라면 바이트 코드는 가상 머신이 이해할 수 있는 언어
- CPU 종류에 따라 기계어가 달라지지만 바이트 코드는 어떤 플랫폼에 종속되지 않고 실행될 수 있는 가상머신용 기계어 코드
- 바이트코드는 JIT에 의해 바이너리 코드로 변환된다.

<br>

JVM의 구성요소를 살펴보며 컴파일 과정 및 실행과정을 자세히 살펴보겠습니다.

## 1. Class Loader

JVM에서 처음으로 살펴볼 부분은 클래스 로더입니다. 바이트 코드를 읽어들여 메모리에 로드하는 역할을 하고 크게 load, link, initialize로 나눌 수 있습니다.

![image](https://user-images.githubusercontent.com/67682840/190892657-6b6fa95c-af74-4706-a143-a633293a9999.png)

#### load

- 바이트 코드를 메모리에 올리는 역할을 담당
- 클래스 로더는 `.class`를 포함하는 `.jar` 파일을 읽어들인다.

자바에는 여러가지 종류의 클래스들이 존재합니다. 우리가 작성한 클래스들은 `java.lang.ClassLoader`의 instance에 의해서 loading이 됩니다. 하지만 이 또한 클래스인데 어떻게 loading이 될 수 있을까요? 다음 세가지 클래스 로더의 종류를 살펴봅시다.

1. bootstrap class loader

   - 최상위 클래스, JVM 시작 시 최초로 실행되는 클래스 로더
   - 자신을 로드해줄 부모 클래스로더가 없어 native code로 작성
   - `$JAVA_HOME/jre/lib` 안의 핵심 라이브러리를 로드(`rt.jar` 포함)
     - `rt.jar`은 java, javax등 필수 패키지 포함
     - Object, String, ClassLoader 클래스 로드

2. extension class loader

   - bootstrap class loader의 자식 클래스
   - `jre/lib/ext` 폴더에 있는 추가적인 `.jar` 파일 로딩

3. application class loader

   - extenstion class loader의 자식클래스
   - System class loader라고도 함
   - application level의 class들을 로드

#### link

로드된 클래스 파일을 검증하고 사용할수 있게 준비하는 과정입니다. link또한 verify, prepare, resolve 세 단계로 나눌 수 있습니다.

1. verify

   - 클래스로더에 의해 로딩된 바이트 코드를 보고 JVM specification에 적합한지 살펴봄
   - 적절하지 않은 경우 `VerifyError` 발생

2. prepare

   - 클래스나 인터페이스에 필요한 static field 메모리를 할당하고 이를 기본값으로 초기화
   - `public static boolean check = true`와 같은 문장이 있다면 false로 초기화 `=true`는 Initailization 단계에서 초기화 되므로 JVM에 탑제된 클래스 파일의 코드를 작동시키지 않음

3. resolve

   - 위 단계를 거치면 class, class instance, method은 symbolic reference로 전환
   - JVM이 실행되면 symbolic reference를 사용한다
   - resolve단계에서 symbolic reference를 actual reference로 교체

#### Initialization

이 단계에서는 클래스의 static initializer가 작동합니다. 위에서 언급한 `public static booelan check`의 값이 true로 초기화됩니다.

<br>

## 2. Runtime data area

다음으로 살펴볼 JVM의 두번째 부분은 Runtime data area입니다. JVM의 메모리에 해당하는 부분입니다. 기본적으로 5개 영역이 있고 크게 per thread, per JVM으로 나눌 수 있습니다.

![JVM_Internal_Architecture_small](https://user-images.githubusercontent.com/67682840/191770066-01d416ee-ea8a-490c-a88c-7602ff9dd298.png)

#### 모든 쓰레드에 공유되는 영역(per JVM)

1. Method area

   아래와 같은 클래스의 메타데이터 정보들이 저장됩니다. 모든 쓰레드들이 같은 Method area를 공유하므로 thread-safe를 보장해야합니다.

   - ClassLoader Referecne
   - Runtime Constant pool
     - Numberic Constant
     - Field references
     - Method References
   - Field data
     - Per field
       - Name
       - Type
       - Modifier
       - Attribute
   - Method data
     - Per method
       - Name
       - Return type
       - Parameter type
       - Modifier
       - Attribute
   - Method code
     - Per method
       - Bytecodes
       - Operand stack size
       - Local variable size
       - Local variable table
       - Exception table
         - Per exception handler
           - Start point
           - End point
           - PC offset for handler code
           - Constant pool index for exception class being - caught

   <br>

   **runtime constant pool**은 클래스와 인터페이스의 모든 Constant 정보를 가지고 있는 곳이며 여기서의 Constant는 상수만 의미하는 것이 아니라 Literal Constant, Type Field(Local Variable, Class Variable),Method로의 모든 심볼릭 레퍼런스를 의미합니다.

2. HEAP

   클래스의 인스턴스나 배열을 사용할 때 사용되는 영역입니다. 메서드의 호출이 끝나도 HEAP에서 사라지지 않으며 오직 Garbage collector에 의해서 제거됩니다. GC를 위해 HEAP을 세 영역으로 나눌 수 있습니다.

   - Young Generation
     - Eden
     - Survivor
   - Old Generation
   - Permanent Generation

#### 각 쓰레드마다 갖는 영역(per Thread)

1. pc register

   program counter register, 다음에 실행될 instruction들을 가리키고 있습니다.

2. java stack

   current method에 상응하는 stack frame을 가지고 있습니다. 스택 프레임은 현재실행하고 있는 메서드와 관련된 데이터들을 저장합니다.

   - Local variable array
   - Return value
   - Operand stack
   - Reference to runtime constant pool for class of the current method

3. native method stack

   JVM은 native method를 지원하므로 native method를 위해 사용되는 스택이 따로 있습니다. 네이티브 메서드란 자바가 아닌 C/C++로 구현된 메서드를 의미합니다.

   - 네이티브 메서드 인터페이스인 JNI와 연결되어있어 JNI가 사용되면 네이티브 메서드 스택에 저장

   native method 예시

   ```java
   @HotSpotIntrinsicCandidate
   public static native Thread currentThread();​
   ```

<br>

## 3. Execution engine

마지막으로 메모리에 로드된 클래스파일을 실행시키는 역할을 하는 Execution Engine입니다. 실행 방법은 크게 두가지가 존재합니다.

- 인터프리터(Interpreter)
- JIT compiler

인터프리터의 단점을 극복하기 위해 자바에서는 여러 최적화를 제공합니다. 그중 하나가 JIT compiler입니다.

JIT은 just in time의 약자로 직역하자면 그때 그때라고 할 수 있습니다. 자바 바이트 코드는 인터프리터로 한줄 한줄 실행된다면 속도가 굉장히 오래 걸립니다. 이러한 성능을 개선하기 위해서 Oracle Hotspot VM에서는 바이트 코드의 "hot" area들을 찾아서 기계어로 컴파일을 하는 최적화를 진행했습니다. 기계어들은 non-heap memory의 code cache 부분에 저장을 함으로써 반복적으로 수행되는 instruction에 대해서는 매번 기계어 코드가 생성되는 것을 방지해 인터프리팅 시간을 단축시킵니다.

> **code cache** <br>
> 컴파일된 코드가 캐싱 되는 곳을 의미합니다. 크기는 고정 값이며 가득차면 JVM은 더이상 코드를 컴파일 할 수 없습니다.

즉 인터프리터 방식과 JIT compiler 방식을 적절히 혼합해서 개선을 하는데 어떤 것을 기준으로 위에서 언급한 "hot" area들을 찾을 수 있을까요?

JVM은 호출되는 메서드 각각에 대해 호출 횟수를 누적해서 그 횟수가 특정 수치를 초과할 경우 컴파일을 하게됩니다. 즉, 얼마나 자주 호출되는지 검사한 후 '컴파일이 필요한 시점이다' 라고 생각하는 기준치에 도달하면 컴파일을 하게 되는데 이러한 기준치를 컴파일 임계치(Compile Threshold)라고 합니다.

예를 들어 자바에서 루프가 길 경우 매 루프마다 메서드의 실행횟수를 카운트 하다가 컴파일 임계치를 넘어가면 전체 메서드가 아닌 루프만을 컴파일하여 컴파일된 버전으로 바로 실행하는 것입니다. 아래 예시를 살펴보겠습니다.

```java
for (int i = 0; i < 500; ++i) {
  long startTime = System.nanoTime();

  for (int j = 0; j < 1000; ++j) {
 	new Object();
  }

  long endTime = System.nanoTime();
  System.out.printf("%d\t%d\n", i, endTime - startTime);
}
```

루프가 반복됨에 따라 일정 시점에서 실행속도가 줄어드는 부분을 확인할 수 있습니다.

![image](https://user-images.githubusercontent.com/67682840/191144214-a70a6725-2234-4f14-89bb-cc476315d84a.png)

JIT 컴파일러는 사용하는 옵션에 따라 서버 컴파일러와 클라이언트 컴파일러로 나눌 수 있고 두 컴파일러의 장점을 얻기 위해 Tiered 컴파일러를 사용할 수도 있습니다.

#### 서버 컴파일러

- 컴파일전에 많은 정보를 수집하여 최적화에 중점을 둔다
- 서버 컴파일러는 절대 모든 코드를 컴파일 하지 않느다

#### 클라이언트 컴파일러

- 서버 컴파일러보다 먼저 컴파일을 시작한다
- 최적화를 위한 대기 시간이 짧다
- Start-up 시간이 빠르다. 하지만 최적화를 덜 하기 때문에 코드실행은 서버가 더 빠르다

#### Tiered 컴파일러

- "-server -xx:+TieredCompilation" 옵션으로 명시한다
- 먼저 클라이언트 컴파일러로 스타트업 시간을 빠르게 하고 많이 쓰이는 부분을 서버 컴파일러로 다시 컴파일 하여 대체한다
- JAVA8 부터는 기본 옵션이다

일반 서버 컴파일러를 사용할 때는 컴파일 대상이 되는 클래스의 개수가 코드 캐시를 가득 채울 일은 그다지 없습니다. 하지만 클라이언트나 티어드 컴파일을 사용할 때는 주의해야하는데, 코드 캐시가 부족한 상황이 나타날 가능성이 크기 때문입니다.

<br>

## JDK vs JRE

#### JRE

- Java Runtime Environmnet
- JVM, Java Class Library, java command 포함
- 프로그램 실행 가능, 생성 불가

#### JDK

- Java Development Kit
- JRE 포함
- compiler(javac)와 tools(javadoc, jdb) 포함
- 프로그램 생성 및 실행 가능

로컬 머신에서 자바 프로그램을 실행하기 위한 목적이라면 JRE만 설치하면 되지만 Java programming을 하기 위해서는 JDK가 필요하다고 요약할 수 있습니다.

## 출처

- https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=kbh3983&logNo=221292870568
- https://catsbi.oopy.io/df0df290-9188-45c1-b056-b8fe032d88ca
- https://tecoble.techcourse.co.kr/post/2021-07-15-jvm-classloader/
- https://www.youtube.com/watch?v=ZBJ0u9MaKtM
- https://steady-coding.tistory.com/593
- https://tecoble.techcourse.co.kr/post/2021-07-15-jvm-classloader
- https://imbf.github.io/interview/2021/03/02/NAVER-Practical-Interview-Preparation-4.html
  https://jithub.tistory.com/40
- https://blog.jamesdbloom.com/JVMInternals.html#constant_pool
- https://beststar-1.tistory.com/3
- https://xzio.tistory.com/1953
- https://stackoverflow.com/questions/1906445/what-is-the-difference-between-jdk-and-jre
- https://stackoverflow.com/questions/10209952/what-is-the-purpose-of-the-java-constant-pool
