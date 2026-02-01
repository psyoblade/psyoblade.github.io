---
layout: post
title:  "04/25 (금) 도커 멀티 플랫폼 빌드"
date:   2025-04-25 22:10:00 +0900
categories: psyoblade created
---

## 루틴: 2025년 04월 25일 (금)

>      AMD64 플랫폼에서 ARM64 도커 이미지 빌드

### 기술

#### 환경구성

* QEMU 등록 (1회만 하면 됨)

  * ```bash
    bash docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
    ```

  * QEMU를 Docker 시스템에 등록하며, `--privileged` 플래그는 QEMU 바이너리를 커널에 등록하기 위해 필요

* AMD64 환경에서 ARM64 에뮬레이션

  * ```bash
    docker run --rm --platform linux/arm64 <image>:<tag>
    ```


#### 빌드

* `Dockerfile` 을 아래와 같이 작성합니다

  * ```bash
    ARG OWNER=jupyter
    ARG TAG=ubuntu-22.04
    FROM $OWNER/scipy-notebook:$TAG
    LABEL maintainer="Suhyuk Park <park.suhyuk@gmail.com>"
    
    RUN uname -m && dpkg --print-architecture
    ```

* AMD64 환경에서 ARM64 플랫폼 `--push` 혹은 `--load` 하지 않고 단순히 빌드만 테스트 합니다

  * ```bash
    docker buildx build --no-cache --platform linux/arm64 --progress=plain -f Dockerfile .
    ```

    * `--no-cache` 옵션과 --progress=plain` 옵션으로 에코 등의 디버깅을 출력할 수 있습니다

* 로컬에서 테스트가 완료 되었다면 도커허브에 푸시합니다

  * ```bash
    docker buildx build --no-cache --platform linux/amd64,linux/arm64 --progress=plain -f Dockerfile -t ${IMAGE}:${TAG} --push .
    ```

#### 할일

1. 도커 이미지 목록

   1. 멀티 플랫폼 지원 여부를 확인 결과, 일부 (namenode, datanode, turnilo) 멀티플랫폼 지원 불가

      ```bash
      $ find . -name docker-compose.yml -exec grep name {} \; | sort | uniq
      
      (X) image: bde2020/hadoop-datanode:1.1.0-hadoop2.8-java8
      (X) image: bde2020/hadoop-namenode:1.1.0-hadoop2.8-java8
      (O) image: cassandra:4.1.8
      (O) image: elasticsearch:7.17.28
      (O) image: httpd:2.4.63
      (O) image: kibana:7.17.28
      (O) image: mongo:6.0.22
      (O) image: psyoblade/data-engineer-druid:1.0
      (O) image: psyoblade/data-engineer-fluentd:3.1
      (O) image: psyoblade/data-engineer-kafka:1.2
      (O) image: psyoblade/data-engineer-mysql:1.4
      (O) image: psyoblade/data-engineer-notebook:1.9.0
      (O) image: psyoblade/data-engineer-ubuntu:22.04-slim
      (O) image: psyoblade/data-engineer-sqoop:2.0
      (O) image: zookeeper:3.5.9
      (X) image: uchhatre/turnilo
      (O) image: zookeeper:3.5.9
      ```

2. 멀티플랫폼 미지원 컨테이너 이미지 실습에서 제외 및 대체 서비스로 전환

   1. `uchhatre/turnilo` 멀티 플랫폼 지원 되지 않아 3일차 실습을 위한 `superset` 사용하도록 전환
   2. `bde2020/hadoop-namenode:1.1.0-hadoop2.8-java8` 하둡 네임노드 및  `bde2020/hadoop-datanode:1.1.0-hadoop2.8-java` 하둡 데이터노드 멀티플랫폼 지원이 안되므로 `day2/ex9` 예제 제거
   3. `druid` 실습은 최소 `3G~6G` 가 필요하므로 `docker-compose` 수준에서 명시할 것

3. 데이터 엔지니어링 기술 동향

   1. 국내 데이터 엔지니어링 최신 기술 동향
   2. 해외 데이터 엔지니어링 최신 기술 동향 featured by DATA+AI 2024
      1. 데이터브릭스 워크플로우의 새로운 기능
      2. **Apache Spark 3.5의 새로운 기능 심층 분석**
      3. <u>델타 라이브 테이블을 활용한 지능형 데이터 파이프라인을 위한 베스트 프랙티스</u>
      4. **데이터브릭스 활용한 대규모 스트리밍 애플리케이션 마이그레이션 및 최적화**
      5. **데이터 인텔리전스 플랫폼에서 데이터 엔지니어링을 위한 가이드**
      6. <u>델타 레이크 및 관련 도구를 활용한 효과적인 레이크하우스 스트리밍</u>
      7. <u>구조화 스트리밍(Structured Streaming)에서 발생하는 문제 극복 방법</u>
      8. Delta Lake와 마이크로서비스를 활용한 반구조화 커뮤니케이션 데이터 처리
      9. <u>Apache Spark™용 새로운 Python 데이터 소스 API 소개</u>
      10. <u>Unity Catalog 오픈 소스화 발표</u>
   3. 데이터 엔지니어링 미래 기술
   4. AWS 연계 주제 ?

4. 강의 자료 수정 및 보완

   1. day1 전체 도커 이미지를 한 번에 내려받는 매뉴얼 전달
   2. day1 굳이 docker-compose up -d 제거 특히 docker rm -f `docker ps -a` 같은 명령은 모두 수정
   3. day1 랜처 데스크톱 사용 시 dps 명령어 alias 추가 특히 dps, dpa 같은 명령어를 넣어야 할까?
   4. day1 우분투가 슬림만 사용해도 된다면 굳이 예제에서 풀 셋 넣지 말자
   5. day2 ex9 의 hdfs, yarn 관련 예제는 제거하고 컴포즈에서도 제거
   6. day5 카프카 통한 예제 실습 반드시 해보고 Kraft 버전에 문제가 없는지 확인
   7. Day5 드루이드 내부에서 python 이 없어서 직접 인덱스 수행은 안되므로 ui 통해서 하도록 교재 변경

#### 고찰

* 

#### Q&A

1. 빌드시에 정상적으로 잘 되었다고 생각했지만 M1 맥북에서 정상 동작하지 않고 X86 으로 동작하는 현상
   1. 인텔 맥북 환경에서 에뮬레이션 환경에서 플랫폼 확인 시에도 여전히 `x86_64` 로 뜨고 있음
   2.  `scipy-notebook:ubuntu-22.04` 이미지는 정상적으로 `aarch64` 가 뜨고 있음
   3. `FROM --platform=$BUILDPLATFORM $BASE_CONTAINER` 때문에 오히려 잘못 빌드되고 있었음
   4. `BUILDPLATFORM` 관련 인자 모두 제거하면 `--platform linux/amd64,linux/arm64` 실행 시에 정상 빌드
