---
title: "HandlerMethodArgumentResolver"
date: 2023-01-10T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - spring
---

## HandlerMethodArgumentResolver

어떠한 요청이 컨트롤러에 들어왔을 때, 요청에 들어온 값으로부터 원하는 변수 또는 객체를 사용하기 위해 `RequestParam` 이나 `RequestBody` 를 사용할 수 있습니다. 프론트에서는 다양한 값들이 넘어올 수 있는데 이러한 값들에 대해 일관된 처리를 하는 로직을 작성할 필요가 있는 경우가 있습니다.

예를 들어 상품 검색의 경우, `title`에 `""`와 같은 빈 문자열이 들어오거나 `null`이 들어 올 수 있고 `price`의 경우 잘못된 입력이 들어 올 수 있습니다. 이러한 경우 컨트롤러에서 해당 값들에 대해 필터링을 하여 기본값으로 설정해야합니다.

하지만 컨트롤러에서 이러한 부가적인 처리를 한다면 컨트롤러의 책임이 번잡해지는 문제가 생깁니다. 이에 대한 해결책으로 `HandlerMethodArgumentResolver`을 사용할 수 있습니다.

`HandlerMethodArgumentResolver`은 인터페이스로 컨트롤러에 전달되는 파라미터를 가공할 경우 사용할 수 있습니다.

![image](https://user-images.githubusercontent.com/67682840/211475880-36d43dba-f0b3-4039-aaba-214da425c638.png)

`supportsParameter`은 어떤 파라미터에 대해 가공을 할 지 결장하는 메서드입니다. 이번 예시에서 사용할 파라미터 dto는 `SearchOption`이므로 해당 클래스가 파라미터로 전달되었다면 파라미터를 가공할 것이라는 코드를 작성하면 됩니다.

`resolveArgument`는 실제 로직을 수행하는 메서드입니다. `title`는 빈 문자열이거나 null인 경우 일정한 처리를 위해 기본값 `""` 으로 설정하였고 `minPrice`와 `maxPrice`에 올바르지 않은 값이 들어오거나 null인 경우 기본값을 `-1`로 설정했습니다. `toLong`을 살펴보면 올바르지 않는 값에 대해서도 처리함을 알 수 있습니다.

![image](https://user-images.githubusercontent.com/67682840/211476851-b4e6d122-efe0-448a-b6df-8c469713267b.png)

이후에는 `Resolver`가 동작하도록 설정해야합니다. 그 전에 `ArgumentResolver`가 어느 단계에서 작동하는지 파악할 필요가 있습니다.

1. 요청이 들어온다
2. `Dispatcher`이 해당 요청을 받아 `HandlerMapping`을 통해 해당 요청을 처리할 Controller을 찾는다.
3. 찾은 Controller를 실행할 `HandlerAdapter를 찾는다.

   이 때 controller실행에 필요한 파라미터를 생성하기 위해 `ArgumentResolver`가 실행된다

4. Controller의 실행결과를 `DispatcherServlet`이 받은 후 `ViewResolver`에게 전달한다

위 과정을 위해 Controller 실행 전 `ArgumentResolver`을 사용하도록 아래와 같이 설정합니다.

![image](https://user-images.githubusercontent.com/67682840/211479571-74fee1fa-6578-408c-87f5-920df1c8359a.png)

모든 설정을 마친 후 컨트롤러를 작성하고 `@RequestBody` 나 `@RequestParam` 없이 값이 올바르게 매핑되는지 확인해보겠습니다.

![image](https://user-images.githubusercontent.com/67682840/211479819-62abe873-3697-4e7d-bf55-ede828837012.png)

![image](https://user-images.githubusercontent.com/67682840/211480122-4341f3f4-6883-4fb2-aa7b-12e8f012be0f.png)

`minPrice`, `maxPrice`에 원하는대로 기본값이 설정됨을 확인할 수 있습니다.
