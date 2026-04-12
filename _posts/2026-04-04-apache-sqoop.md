---
layout: post
title:  "04/04 (토) 아파치 스쿱 하둡 2.9.2 버전 빌드"
date:   2026-04-04 11:00:00 +0900
categories: psyoblade created
---

# 아파치 스쿱 하둡 2.9.2 버전 빌드

>      해외 데이터 수집을 위한 데이터 파이프라인 구축과 레이턴시가 극단적인 상황에서도 안정적인 적재가 가능한 수집 서비스 구축

## 목적

* 멀티캠퍼스 강좌 홍보를 위한 쇼츠 혹은 블로그 작성을 위한 스토리보드 작성
* 해외 데이터 수집을 위한 파일럿 프로그래밍

## 목표

* 아파치 스쿱 하둡 2.9.2 버전 빌드
* 로컬 수집 및 Split-By 수집 테스트

### 도커로 빌드된 아파치 스쿱을 통해서 외부 명령으로 로컬 저장이 가능할까?

* [apache-sqoop](https://github.com/psyoblade/apache-sqoop) 하둡 2.9.2 버전 의존성 추가하여 스쿱 1.5.4 버전을 빌드

* [data-engineer-docker-stacks/sqoop](https://github.com/psyoblade/data-engineer-docker-stacks/tree/master/sqoop) 1.5.4 버전 적용 및 `avrò-1.7.7.jar` 의존성 제거하여 독립 실행 가능한 도커 이미지 빌드

* 외부 명령어 실행으로 테이블 수집 테스트

  * ```bash
    # 환경변수 통해서 실행
    docker run --rm \
      --network global_ingestion_network \
      -v $(pwd)/target:/target \
      -e SQOOP_QUERY="SELECT category, id, name, address, naddress, tel, tag FROM seoul_popular_trip WHERE \$CONDITIONS" \
      -e SQOOP_CONNECT="jdbc:mysql://mysql:3306/default" \
      -e SQOOP_USERNAME="scott" \
      -e SQOOP_PASSWORD="tiger" \
      psyoblade/data-engineer-sqoop:2.2 \
      sh -c '
        sqoop import -jt local -m 1 \
          --connect "$SQOOP_CONNECT" \
          --username "$SQOOP_USERNAME" \
          --password "$SQOOP_PASSWORD" \
          --query "$SQOOP_QUERY" \
          --target-dir /target/seoul_popular_trip \
          --as-parquetfile \
          --delete-target-dir
      '
    # 직접 실행
    docker run --rm \
      --network global_ingestion_network \
      -v $(pwd)/target:/target \
      psyoblade/data-engineer-sqoop:2.2 \
      sqoop import -jt local -m 1 \
        --connect "jdbc:mysql://mysql:3306/default" \
        --username "scott" \
        --password "tiger" \
        --table "seoul_popular_trip" \
        --target-dir /target/seoul_popular_trip \
        --as-parquetfile \
        --delete-target-dir
    ```

  * 로컬 마운트된 target 경로에 파일이 생성됨

* 아래와 같이 환경 구성이 되어 있어야 함

  * ```yaml
    # cat docker-compose.yml
    services:
      mysql:
        container_name: mysql
        hostname: mysql
        image: psyoblade/data-engineer-mysql:1.3
        restart: always
        environment:
          MYSQL_ROOT_PASSWORD: $MYSQL_ROOT_PASSWORD
          MYSQL_DATABASE: $MYSQL_DATABASE
          MYSQL_USER: $MYSQL_USER
          MYSQL_PASSWORD: $MYSQL_PASSWORD
        ports:
          - '3306:3306'
        networks:
          - default
        healthcheck:
          test: ["CMD", "mysqladmin" ,"ping", "-uroot", "-p${MYSQL_ROOT_PASSWORD}", "-h", "localhost"]
          interval: 3s
          timeout: 1s
          retries: 3
        volumes:
          - ./mysql/etc:/etc/mysql/conf.d
    
    networks:
      default:
        name: global_ingestion_network
    ```

  * ```bash
    .
    ├── docker-compose.yml
    ├── mysql
    │   └── etc
    ├── README.md
    ├── sqoop
    │   └── jars
    │       ├── mssql-jdbc-8.2.0.jre11.jar
    │       ├── mssql-jdbc-8.2.0.jre13.jar
    │       ├── mssql-jdbc-8.2.0.jre8.jar
    │       ├── mysql-connector-java-8.0.19.jar
    │       ├── parquet-tools-1.8.1.jar
    │       ├── postgresql-42.2.12.jar
    │       └── protobuf-java-3.6.1.jar
    └── target
        └── seoul_popular_trip
            ├── _common_metadata
            ├── _metadata
            ├── _SUCCESS
            └── part-m-00000.parquet
    ```

  * 와 같이 동일한 네트워크 접근을 위한 yaml 설정 및 mysql 기동이 필요합니다

### 아파치 스쿱 로컬 저장 시에 split-by 수집이 가능할까?

* ```bash
  # split-by id 기준으로 mapper 2개 설정
  docker run --rm \
    --network global_ingestion_network \
    -v $(pwd)/target:/target \
    -e SQOOP_QUERY="SELECT category, id, name, address, naddress, tel, tag FROM seoul_popular_trip WHERE \$CONDITIONS" \
    -e SQOOP_CONNECT="jdbc:mysql://mysql:3306/default" \
    -e SQOOP_USERNAME="scott" \
    -e SQOOP_PASSWORD="tiger" \
    psyoblade/data-engineer-sqoop:2.2 \
    sh -c '
      sqoop import -jt local -m 2 \
        --connect "$SQOOP_CONNECT" \
        --username "$SQOOP_USERNAME" \
        --password "$SQOOP_PASSWORD" \
        --query "$SQOOP_QUERY" \
        --split-by id \
        --target-dir /target/seoul_popular_trip_split_by_2 \
        --as-parquetfile \
        --delete-target-dir
    '
  ```

* ```bash
   ~/work/global_ingestion  tree target/seoul_popular_trip_split_by_2
  target/seoul_popular_trip_split_by_2
  ├── _common_metadata
  ├── _metadata
  ├── _SUCCESS
  ├── part-m-00000.parquet
  └── part-m-00001.parquet
  
  0 directories, 5 files
  ```

  * 위와 같이 정상적으로 동작함을 확인

