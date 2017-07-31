---
layout: post
title:  넷플릭스 마이크로서비스 가이드 - 혼돈을 지배하라
date:   2017-07-31 10:54:22 +0900
categories: psyoblade update
---

## 넷플릭스 마이크로 서비스 가이드라인 - 혼돈을 지배하라
원문: [Mastering Chaos - A Netflix Guide to Microservices](https://www.infoq.com/presentations/netflix-chaos-microservices)


### 마틴 파울러는 마이크로 서비스 아키텍처
 단일 응용프로그램을 개별 서비스로 실행하고, 경량 프로토콜(HTTP, API)을 통해 통신하는 작은 서비들의 모음으로 개발하는 접근방식이며, 완전 자동화된 배포방식으로 개별적인 서비스의 배포가 가능한 것을 말합니다. 전체 서비스를 하나의 덩어리로 고려하지 않아도 됩니다.
> In short, the microservice architectural style [1] is an approach to developing a single application as a suite of small services, each running in its own process and communicating with lightweight mechanisms, often an HTTP resource API. These services are built around business capabilities and independently deployable by fully automated deployment machinery. There is a bare minimum of centralized management of these services, which may be written in different programming languages and use different data storage technologies. - Martin Fowler


### 주요 서비스 핵심 키워드 
* Separation of concern: 모듈화, 캡슐화
	* 하나의 잘 못 배포된 서비스가 전체 서비스에 영향을 미치면 안된다
	* 잘못된 호출을 계속 시도하는 것 보다는 중지하는 것이 낫다 
	* 실패를 주입하는 (failure injection testing)을 통해 테스트를 위한 프레임워크 [Hystrix](https://github.com/Netflix/Hystrix)
	* api 가운데에서 핵심적인 종속성의 중간에 있는 api에 대해 FIT 테스트 한다
* Scalability
	* 수평확장, 워크로드 파티셔닝
	* 의사 결정에 있어 원칙은 CAP Theorem 참고
* Virtualization & elasticity
	* 자동화된 운영
	* on-demand provisioning (중요함)
* Avoiding chaos
	* 서비스는 지속적, 정기적으로 카오스 상태를 테스트 함으로써 강건해 진다.
	* stateless service : chaos monkey 등을 통해 특정 api가 장애인 상황을 미리 연출
	* stateful service : 비즈니스 로직과 상태를 하나의 어플리케이션에 두지 마라
	* operating drift : alert, timeout, fallback, throughput 등은 항상 모니터링해야 하는 요소이다. - continuous learning & automation 
* Cost of variance
	* 도구 생산성
	* 라이브러리/플랫폼 중복
	* 각종 도구의 개발비용
* Conway's law
	* 조직의 구조가 복잡하면 업무 단계도 그에 따라 복잡해 진다.
	* 해법이 중시되어야 하지, 조직이 우선시 되면 안된다.


### 라이브 서비스를 위한 준비사항

* Production ready
	* automated canary analysis
	* autoscaling
	* chaos monkey
	* healthcheck
	* squeeze testing
	* timeout, retry, fallback
	* staged, red/black deployment



### 현재 프로젝트에서 검토할 사항
* 아키텍처 설계 원칙은 무엇인가?
	* 모든 것을 다 가질 수 없을텐데, 어떤 부분이 가장 중요한 요소인가?
	* 그러면 기술부채로 가져갈 부분은 무엇인가?
* 우리가 가져가야 할 마이크로 서비스의 종류에는 무엇이 있는가?
	* [AWS제품](https://aws.amazon.com/ko/products/?hp=tile&so-exp=below)을 참고하면 좋을 듯
	* 마이크로 서비스 간의 종속성 혹은 활용이 제대로 되고 있는가? 모노리틱 아닌가?
* 서비스 단위로 배포 및 서비스 장애에 대한 대응이 가능한가?
	* 비즈니스 요구사항
	* 개발자의 기술적인 수준
* 기타 검토 사항
	* 99.99% 가용성은 1년에 53분만 허용하는 수치인데 우리의 기준은 어떠하면 좋은가?
	* 지속적으로 자동화된 학습 및 반영할 수 있는 기계학습 응용분야를 찾아볼 수 있겠다
	* 클라이언트 라이브러리 사용에 있어 아주 험난한 논의과정이 있었다고 한다 우리는?
	* 발생한 모든 장애에 대해서 분석하고 자동화하여 피할 수 있는 도구를 고안한다
