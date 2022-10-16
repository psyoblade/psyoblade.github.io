---
layout: post
title:  "스프링부트 어노테이션 탐구 'Bean vs Component'"
tags: [java, springboot, annotation]
date:   2017-09-03 20:38:22 +0900
comments: true
category: dev
---

## [스프링부트 @Bean vs. @Component 어노테이션의 차이점](http://jojoldu.tistory.com/27)

### @Bean vs @Component

> 외부 라이브러리들을 Bean으로 등록하고 싶은 경우 내가 접근할 수 없는 (소스코드가 없는) class에는 Component 어노테이션을 달 수 없기에, 이 객체를 생성하는 함수를 만들고 @Bean 어노테이션을 단다
```java
@Bean
public ObjectMapper objectMapper() { return new ObjectMapper(); }

@Bean
public RestTemplate restTemplate() { return new RestTemplate(); }

@Component
public class MyClass { ... }

```

## 스프링부트 프로파일 탐구

### 프로파일 active 설정을 한다는 의미?
> 현재 사용하는 프로파일이 여러가지인 경우에 { local, test, production } 이 가운데 active 설정이 안 되어 있는 경우는 application.properties 파일에 공통된 값을 참조합니다. 단, active 설정이 되어 있는 경우에는 application.properties 값들을 override 할 수 있습니다. 그리고. application.properties 내부에 active 설정이 있다고 하더라도 외부환경에서 -Dspring.profiles.active=test 설정이 되어 있다면 이를 따르게 됩니다.

```java

@Profile=(value={"prod","dev","test"})
public class MyService { .. }

java -jar -Dspring.profiles.active="dev" // 이클립스에서 테스트 할 때에 환경변수로 프로파일 활성화

@ActiveProfiles("test") // 단위테스트의 경우 별도의 환경변수 지정이 곤란할 수 있으므로 아예 코드에서 활성화

// 라이브 환경에서는 어떻게 적용하는 것이 좋은가?
```



