---
layout: post
title:  "04/01 (수) DuckDB 가 뭘까?"
date:   2026-04-01 22:30:00 +0900
categories: psyoblade created
---

# DuckDB 를 어디에 써먹어볼까?

>      회사 동료분으로부터 DuckDB 가 핫하다고 추천을 받아 둘러보는 중이다 어디에 써먹으면 좋을지 고민 중이었는데 우연히 parquet 읽기에 편한 CLI 추천해 달라고 했더니 duckdb 라고 하길래 일단 설치하고 사용해 보는 중이다

![image-20260401234902009](/private/images/2026-04-01-what-is-duckdb/image-20260401234902009.png)

## 목적

* DuckDB 설치 및 사용법
* 기술 자료 및 지피티를 통해서 어떤 기능을 갖고 있고 어떤 내부 구조를 가지고 있는지 이해하기
* 원격지 컨테이너 환경의 테이블 로컬 저장소 수집을 위한 엔진으로 아파치 스쿱 대체가 가능한 지 검토

## 목표

* DuckDB 의 특징을 이해하고, 어디에 써먹을 수 있을지 생각해보기

### [How to install](https://duckdb.org/install/?platform=macos&environment=cli) & run DuckDB

* `brew install duckdb` 통해서 설치 후 `duckdb` 통해서 전용 터미널 접속이 가능하며 `duckdb -c "SELECT * FROM './1.parquet' LIMIT 10"` 명령으로 바로 조회가 가능하다

* 온프레미스 환경에서 설치가 문제가 없다면 기존 parquet-tools 보다 훨씬 강력하고 쓰기에도 편할 것 같다

###  [파이썬 클라이언트](https://duckdb.org/docs/current/clients/python/overview#using-an-in-memory-database)도 한 번 써볼까?

* `pip install duckdb` 통해서 설치하고 간단한 예제 코드 탐험과 테스트 결과 아주 간단하게 가상 테이블 생성 parquet 저장 읽기 그리고 특히 file.db 생성 후 wal 방식으로 데이터 처리를 하고 있어어 트랜잭션 개념도 장착하고 있는 것 같다. 스레드로 여러 데이터를 읽고 쓰기 시에 락 처리는 가능한지 여부는 추후에 테스트 해봐야 겠다

  * 예제 코드의 `duckdb.sql` 혹은 `duckdb.connect(':default:')` 통한 접근은 shared global in-memory 접속을 사용하라는 의미라서 thread-safe 하지 않으므로 반드시 ` con = duckdb.connect()` 통해서 신규 생성된 `con` 통해서 조회하는 습관을 가지자

* 특히 connect 시에 다양한 [설정](https://duckdb.org/docs/current/configuration/overview#configuration-reference)(TimeZone, access_mode 등)을 넣어줄 수 있는 듯 한데 필요 할 때에 찬찬히 찾아봐야 겠다

* 무엇보다도 강력한 외부 확장 기능인데 아래와 같이 외부 확장 기능 설치 사용이 가능하다

  * ```python
    #!/usr/env python
    # -*- coding:utf-8 -*-
    
    import duckdb
    
    con = duckdb.connect()
    con.install_extension("quack")
    con.load_extension("quack")
    con.sql("SELECT quack('꾸엑');").show()
    ```

  * [Community-Extention Repo](https://github.com/duckdb/community-extensions)

    * [mssql](https://duckdb.org/community_extensions/extensions/mssql)


#### DuckDB 의 특징

* 데이터베이스에 접속 하는 개념이 아니라, 프로세스 안에 DB를 “열어서(embed)” 사용하는 구조

* 3가지 연결방식
  * `duckdb.connect()`
    * 메모리에만 존재
    * 프로세스 종료 시 데이터 사라짐
    * 테스트/임시 분석용
  * `duckdb.connect("test.duckdb")`
    * 로컬 파일 생성됨
    * SQLite처럼 동작
    * 재사용 가능
  * `duckdb.connect("test.duckdb", read_only=True)`
    * 읽기 전용
    * 동시 접근 안전성 확보
* 접속 객체의 의미
  * `con = duckdb.connect()` : 쿼리를 실행하는 세션 + 상태 + extension 로딩 컨텍스트
    * SQL 실행
    * extension load
    * 설정 관리
* DuckDB 사용시 유의 사항
  * 멀티 프로세스 제약 : SQLite 와 유사하게 하나의 DB 파일은 **동시에 여러 프로세스 write 불가** 
  * 병렬 처리 : **내부적으로 vectorized + multi-thread 실행** 하므로 connection 하나로도 충분히 빠름
  * 개별 접속 : `con1 = duckdb.connect(file.db)` 와 `con2 = ducdb.connect(file.db)`는 서로 다른 세션과 상태를 가진다
  * DuckDB 는 `Single Process, Multiple Threads` 컨셉으로 **하나의 producer 와 다수의 consumer 구성이 적절하다**

#### DuckDB 추천 사용 예제

| 용도        | DuckDB 역할       |
| ----------- | ----------------- |
| 데이터 검증 | parquet 직접 조회 |
| 샘플링      | 빠른 local 분석   |
| 디버깅      | 특정 파티션 확인  |
| ETL 테스트  | lightweight 실행  |

#### Apache Sqoop 을 대체할 수 있을까?

* Sqoop 을 통해서 수행하고 있는 기능을 나열해보자
  * 하나의 물리적인 테이블을 특정 컬럼을 기준으로 분할 해서 다수의 파일로 저장하는 기능
  * 다수의 작업을 YARN 이라고 하는 분산 처리 엔진을 통해서 안정적으로 스케줄링 할 수 있는 점
  * JDBC 통해서 일정한 데이터 크기 (fetch-size) 통해서 검증된 안정적인 수집 기능을 제공한다
  * 현재 운영 중인 데이터베이스 서비스와 태그를 통한 안정적인 관리 기능과 잘 구현되어 있는 점
  * 과거에 발생했던 다양한 질의문 혹은 데이터베이스 타입 등을 안전하게 지원하는 검증된 도구라는 점
* DuckDB 를 사용할 때의 단점은 뭐가 있을까?
  * 기존 레거시 서비스는 구현 과정에서 대부분 비지니스 적으로 발생한 것들을 해결하고 개선되어 왔다는 점 외에는 큰 이슈는 없어 보인다
  * 그러한 기능들을 안전하게 풀어내고 동일하게 구현하는 것이 가장 큰 과제일 것 같다
  * 베이스라인 엔진을 스쿱으로 가져가고 전체적인 파이프라인이 안정적으로 운영되는 시점에 엔진만 교체하는 것이 좋을 것 같다

### 회고

> 

