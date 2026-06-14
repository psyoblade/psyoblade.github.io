---
title:  "05/31 (일) 스프링 시큐리티 스타터"
date:   2026-05-31 12:00:00 +0900
categories: psyoblade created
---

# 스프링 시큐리티 스타터

>      스프링 시큐리티 최신 버전을 프로젝트 도입을 위한 기본 지식 정리 및 향후 방향성을 정리한다

## 목적

* 스프링 시큐리티 기본 지식
* 스프링 시큐리티 적용 방향

## 목표

* 용어 내 맘대로 정리
* 어노테이션 사용 목적
* 프로젝트 도입에 필요한 기능 검토

### 기본 이해

* `SpringBoot` 란?
  * 스프링 MVC 가 API 서버 구성을 위한 모든 기능을 가진 프레임워크이고 스프링부트 = `스프링 MVC` + `서블릿 엔진`(톰캣) + `자동화 도구` (AutoConfiguration) + `스타터` 통한 의존성 자동 주입 + `application.yml` 통한 외부 설정 통합

* 

###  어노테이션

| 어노테이션        | 간략 설명                                         | 보충 설명                                                    |
| ----------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| Configuration     | 이 클래스가 Bean 설정 클래스임을 선언             |                                                              |
| Bean              | 지정한 객체나 메서드가 싱글톤으로 관리되도록 보장 |                                                              |
| EnableWebSecurity | Spring Security 웹 보안 활성화                    | Spring Boot 환경에서는 반드시 추가하지 않아도 SecurityFilterChain Bean 객체 선언 만으로도 동작하지만 순수 MVC 환경이거나 debug 활성화 혹은 명시적인 선언을 하기 위해서 사용한다 |
| Data              | DTO 보일러플레이트 코드                           | Lombok 통한 getter, setter, equals, hashCode, toString 자동 생성 |
| Entity            | DB 매핑 목적 JPA 전용 객체                        | `@Entity` + `@Getter` + `@NoArgsConstructor`                 |
| Value             | Value Object                                      |                                                              |
|                   |                                                   |                                                              |
|                   |                                                   |                                                              |
|                   |                                                   |                                                              |

### 액션 아이템

* `Gradle` 내에서 `Starter` 통한 의존성 자동 관리
* `SecurityFilterChain` 구현에 포함되어야 할 항목
* 유지되어야 할 부분
  * `LDAP` 인증
  * `JWT` 인증
* 개선되어야 할 부분
  * 이용자 별 혹은 그룹 별 권한 및 노출
  * 수정 가능한 작업과 사용 가능한 작업의 구분
* 

### 회고

* 레거시 시스템 분석
  * 어떤 기능이 어떻게 구현되어 있는지
  * 어떤 흐름으로 인증, 인가가 이루어지는지
*  기본 이해 및 구현
  * `AuthenticationFilter` - AuthenticationManager - `AuthenticationProvider` - `UserDetailsService` 위계 및 동작 방식
    * 주로 사용해야 하는 필터의 종류에는 뭐가 있고 어떻게 구현하는가?
  * 인증 정보는 `SecurityContextHolder` 통해서 언제든지 인증 객체 접근이 가능하다
  * Spring MVC - Vue.js , JavaScript 사용 방법 그리고 DTO 및 Controller 연계의 사용
  * Spring Data - Controller, Service 및 Repository 통하여 Entity 를 저장소에 CRUD 사용
  * 타임리프 확장에서 UI 에 인증 객체를 사용할 수 있는데 vue.js 에서는 안 되는가?
* 이용자 관리
  * UserDetails 구현하되 이용자 그룹에 대한 정보를 추가하여 이용자 생성하도록 구현
* 기타 기술 사용에 대한 검토
  * CORS 는 필수이나 CSRF 사용을 하는가?
  * CSRF 를 꺼도 되는지, GET 방식으로 로그아웃 하면 어떤가?

