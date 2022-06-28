---
title: "[Spring Boot] 로깅 / 빌드"
tags: springboot
categories: springboot
use_math: true
---

### 스프링 부트 로깅

애플리케이션을 운용하며 문제가 발생하면 로그 메세지를 보게 됩니다. 에러가 발생할때 뿐만 아니라 애플리케이션의 성능을 분석하는데에도 쓰는 등 다양한 용도로 사용됩니다.

마켓플레이스에서 ANSI Esacpe in Console을 설치하여 메인함수를 실행하면 로그 레벨별로 색깔이 구분되어 표기됩니다.

![img](https://blog.kakaocdn.net/dn/cJTTfq/btq8ujZrjBy/BBkXkBBcTiMt0Ibnnnojv1/img.png)

| Level | Color  | 의미                                  |
| ----- | ------ | ------------------------------------- | ------------------------- |
| ERROR | Red    | 사용자 요청을 처리하는 중 문제 발생   |
| WARN  | Yellow | 현재는 처리가능, 향후 문제가능성 존재 |
| INFO  | Green  | 상태변경과 같은 정보성 메세지         |
| DEBUG | Green  | 개발 시 디버깅 목적                   |
| TRACE | Green  | DEBUG                                 | 레벨보다 더 상세한 메세지 |

로그 레벨은 우선순위가 정해져 있어 특정 로그 레벨을 지정하면 해당 레벨을 포함한 상위 로그들이 모두 출력됩니다. 예를 들어 로그 레벨을 DEBUG로 설정했다면 1,2,3,4에 해당하는 로그들이 출력되고 WARN으로 설정했다면 1,2에 해당하는 로그들이 출력됩니다. 개발단계에서는 DEBUG와 TRACE 단계를 사용하지만 실제 운영에서는 WARN레벨로 설정해야합니다.

다음과 같이 로그를 출력하는 클래스도 직접 구현할 수 있습니다.

```java
@Service
public class LoggingRunner implements ApplicationRunner {
	private Logger logger = LoggerFactory.getLogger(LoggingRunner.class);

	@Override
	public void run(ApplicationArguments args) throws Exception {

		logger.trace("Trace 로그 메세지");
		logger.debug("Debug 로그 메세지");
		logger.info("Info 로그 메세지");
		logger.warn("Warn 로그 메세지");
		logger.error("Error 로그 메세지");

	}
}
```

slf4j에서 제공하는 LoggerFactory에서 logger를 얻어 온 뒤, run() 메서드에서 각 레벨에 해당하는 로그들을 직접 출력하고 있습니다.

로그 레벨은 외부 프로퍼티를 통해 설정할 수 있습니다. 또한 로그 파일도 외부 프로퍼티를 통해 설정이 가능합니다.

```java
#Logging Setting
logging.level.com.rubypaper=TRACE
logging.file.name=src/main/resources/board_log.log
```

만일 스프링 부트가 제공하는 기본 로그 설정을 사용하지 않고 직접 관리하고 싶다면 logback.xml 파일을 추가해야합니다. src/main/resources에 다음과 같이 logback.xml 파일을 추가해봅시다.

```c++
<?xml version="1.0" encoding="UTF-8"?>

<configuration>
	<include resoucre="org/springfamework/boot/logging/logback/base.xml"/>
	<logger name="com.rubypaper" level="DEBUG"/>
</configuration>
```

스프링부트에서 기본적으로 제공하는 base.xml을 이용하여 새로운 로깅설정을 하는 의미입니다. base.xml에는 어펜더를 비롯한 다양한 로깅 관련 설정들이 기본적으로 설정되어 있습니다. 아래 logger에서 로그를 출력할 패키지와 레벨을 설정해주면 DEBUG레벨 이상의 로그들이 출력됩니다.

base.xml을 이용하지 않고 모든 것을 제어하고 싶다면 직접 logback.xml 파일을 작성할 수도 있습니다. logback 설정은 크게 어펜더와 로거 두개로 나뉩니다. 어펜더는 어디에 어떤 패턴으로 로그를 출력할 것인지 결정합니다. 파일과 콘솔에 로그를 출력하려면 파일 어펜더와 콘솔 어펜더 두개를 만들고 각 어펜더마다 세부설정을 해주면 됩니다. 로거에서는 만든 어펜더를 사용하여 로그를 출력하도록 설정해주면 됩니다. 아래는 커스터마이징한 logback.xml 입니다.

```c++
<?xml version="1.0" encoding="UTF-8"?>

<configuration>

<appender name="fileAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
	<file>src/main/resources/logs/board_log.log</file>

	<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
		<fileNamePattern>
			src/main/resources/logs/myboard.%d{yyyy-MM-dd}.log.gz
		</fileNamePattern>
		<maxHistory>30</maxHistory>
	</rollingPolicy>
	<encoder>
		<pattern>
		%d{yyyy:MM:dd HH:mm:ss.SSS} %-5level --- [%thread] %logger{35} : %msg %n
		</pattern>
	</encoder>
</appender>

<appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
	<encoder>
		<pattern>
		%d{yyyy:MM:dd HH:mm:ss.SSS} %-5level --- [%thread] %logger{35} : %msg %n
		</pattern>
	</encoder>
</appender>

<logger name="com.rubypaper" level="warn" additivity="false">
	<appender-ref ref="consoleAppender" />
	<appender-ref ref="fileAppender" />
</logger>

<root level="error">
	<appender-ref ref="consoleAppender" />
</root>

</configuration>

```

<br>

### 스프링 부트 빌드

개발 완료된 프로젝트가 일반 자바 프로젝트면 JAR, 웹 프로젝트면 WAR로 패키징하여 사용합니다. 하지만 스프링부트는 웹 애플리케이션도 JAR 형태로 배포가 가능합니다. JAR로 패키징된 파일의 내부를 확인하며 이해해보도록 합시다.

Chapter03 폴더를 우클릭하여 Maven Install을 클릭합시다. 단, 테스트케이스에서 특정 클래스만 메모리에 올렸던 코드가 있다면 해당 코드를 삭제한 후 진행해야합니다.

![img](https://blog.kakaocdn.net/dn/xdPLA/btq8yst6dQw/5mbQdMuz3T6dTex5PGXJc1/img.png)

성공적으로 패키징이 끝났다면 target 폴더 아래 다음과 같이 생성됩니다. jar 파일의 이름은 pom.xml에서 설정했던 <artifactId>+ <version> + jar 형태의 이름으로 패키징 됩니다.

![img](https://blog.kakaocdn.net/dn/GBxma/btq8wJRcdRU/4VHGwMofEvk4XGE4YDRkjK/img.png)

파일의 내부 구조입니다.

![img](https://blog.kakaocdn.net/dn/bAt8Fd/btq8wlpyvDj/iRpdR9tIOUHTrasPh553Pk/img.png)

BOOT-INF 안에 있는 classes에는 src/main/java에서 작성한 클래스 파일과 src/main/resources에서 작성한 설정파일들이 모두 포함되어 있습니다. lib에는 해당 프로젝트에서 사용하는 모든 라이브러리들이 들어있습니다. JAR 파일에는 애플리케이션의 메타데이터가 포함된 매니페스트 파일이 포함되어있어야 합니다. 그 내용은 다음과 같습니다.

![img](https://blog.kakaocdn.net/dn/L0TQa/btq8ysOqvVg/3ipsc7MK5vKNMRUpmFRC71/img.png)

Main-Class는 패키징한 JAR 파일을 실행할 메인 클래스를 지정합니다. 또한 위에서 살펴보았던 클래스와 라이브러리의 위치도 지정하게 됩니다. 스프링 부트는 이 메타데이터를 확인하여 실행에 필요한 정보들을 얻게 됩니다.

<br>

### 스프링 부트 로더

기본적으로 JAR파일안에는 JAR파일을 포함할 수 없습니다. JAR파일에서 제공하는 클래스를 사용하기 위해서는 압축을 해제한 후 클래스 폴더를 옮긴 후 다시 패키징하여 사용해야합니다. 하지만 이렇게 사용하는 클래스가 많을수록 많은 시간이 소요됩니다. 스프링 부트는 이러한 문제를 해결하기 위해 로더클래스를 제공합니다. 해당 클래스는 org/springframework/boot/loader/jar 안에 JarFile.class입니다. 이 클래스를 통해 라이브러리안에 있는 JAR 파일의 수 많은 클래스들을 이용할 수 있게 됩니다. JAR 파일을 실행하기 위한 JarLauncher도 loader 폴더 안에 존재합니다.
