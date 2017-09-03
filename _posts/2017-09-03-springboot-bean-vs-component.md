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
