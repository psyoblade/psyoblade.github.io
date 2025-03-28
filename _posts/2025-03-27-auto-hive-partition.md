---
layout: post
title:  "03/27 (월) 동적으로 하이브 테이블 파티션 추가 프로그램 설계"
date:   2025-03-27 21:34:00 +0900
categories: psyoblade created
---

## 루틴: 2025년 03월 27일 (목)

>     하이브 익스터널 테이블의 임의의 파티션 경로가 생성될 때에 자동으로 하이브 파티션을 추가하는 프로그램 설계

### 설계

#### 시나리오

* Ranger 통한 Solr Audit 로그 조회를 통한 이벤트 탐지

  * HDFS 에 디렉토리 생성 이벤트 검색 

    * ```bash
      curl "http://<solr-host>:8983/solr/ranger_audits/select?q=repo:hdfs&fq=access:(WRITE OR CREATE)&fq=resource:/user/hive/warehouse/my_table&rows=10&sort=evtTime desc&wt=json"
      ```

    * 원하는 조건과 필터 등을 적용했을 때에 evtTime 기준으로 조회를 할 때에 조회 속도가 수 초 이내에 나오는지 확인

    * 조회 속도가 느리다면 원인을 분석하고 조회가 가능하도록 튜닝이 필요하다

  * 하루에 한 번 수행하되 해당 일자에 해당하는 plogdate 가 포함된 경로만 수행한다 약 25시간

    * 단 0시에 실행하고 해당 basedate 정보를 그대로 인자로 받아서 수행하는 것이 용이하다

  * 약 1분에 한 번씩 수행하되 evtTime 기준으로 약 10분 내에 생성된 모든 로그를 탐색한다

    * 실제 이벤트 대비 지연이 거의 없을 것으로 예상되며, 하이브 작업 시간을 고려한다
    * 하이브 파티션 추가에 오랜 시간이 걸릴 수 있기 때문에 충분히 긴 시간을 탐색한다

  * 중복된 파티션 생성을 피하기 위해서 시간 외에도 마지막 수행된 ID 값을 저장한다

    * 초기에는 특정 게임 하나만 모니터링 하고, 점차 일반화하여 규칙을 적용한다
    * solr 의 audit 로그에 id 가 있는지 확인하고 최대값이 얼마인지에 따라 해당 값을 저장해 두고 조회 필터로 사용한다
    * 다음 조회 시에 시간 + id 값을 기준으로 조회하면 누락 없이 조회가 가능하다

* 네트워크 문제로 인한 솔라 감사 로그 조회 실패에 대한 예외 처리

  * N회 이상 솔라 접속에 실패하는 경우는 즉각 장애로 알림을 발송한다
  * 복구 시에는 명시적으로 복구인 것을 인지할 방법을 명시해야 하는데 basetime 이 00시 00분이 아닌 것을 활용
  * 종료는 최대 25시간을 수행해도 무관하지만 임의의 타임아웃을 정해도 괜찮다
  * basedate 기준으로 0시 부터 다시 스캔을 하되 최대 1000개의 아이템을 조회하여 처리하도록 구성한다
  * 대기는 1분을 하되 조회는 마지막으로 성공한 시간보다 크거나 같고, id 는 커야 하는 조건으로 검색한다

* 네트워크 문제로 인한 하이브 접속 실패에 대한 예외 처리

  * N회 이상 하이브 접속에 실패하는 경우는 즉각 장애로 알림을 발송한다
  * 기본적으로 솔라 감사로그 작업과 동시에 수행되는 작업이므로 동일한 트랜잭션으로 판단해야 한다
  * 다만 하이브 접속 실패가 더 많이 발생할 가능성이 있으므로 재시도하는 로직이 포함되어야 한다

* 점검 시에 하둡은 살아있지만 하이브가 내려가 있는 경우 예외 처리

  * N회 이상 장애 시에 종료하고 장애로 알림을 발송하는 것이 운영에 유리할 듯
  * 복구를 통해서 일단위 작업 수행방식으로 전환하는 것도 좋겠다

* N회 재시도에도 실패하는 감사 실패 혹은 다른 오류에 의한 배치 처리 

  * 장애가 발생한 테이블에 대한 수작업 복구를 위한 DDL 생성을 위한 자동화 배치 하이브 작업을 등록해 둔다
  * min 경로를 지정하면 경로를 리스팅하여 자동으로 DDL 파티션 추가하는 스크립트를 생성해 둔다
  * hour 경로를 지정하면 경로를 리스팅하여 자동으로 파티션 경로를 변경하는 DDL 스크립트를 생성해두면 된다

* 하이브 파티션을 생성하기 위한 DDL 구성 및 전송

  * min 경로에 대한 모니터링
    * 경로 필터 "/datasource/country/category/type/log/min/plogdate=20230327" 하위를 모니터링
    * 이벤트가 인지되면 대상 경로를 통해서 대상 파티션 추가 DDL 을 생성한다
  * hour 경로에 대한 모니터링
    * 경로 필터 "/datasource/country/category/type/log/hour/plogdate=20230327" 하위를 모니터링
    * 이벤트가 인지되면 대상 경로를 통해서 대상 파티션을 변경하는 DDL 을 생성한다
  * 1분에 한 번 수행하기 때문에 1분에 한 번 하이브 커넥션을 맺고 cursor 를 통해서 N 개의 DDL 을 수행한다

#### 고찰

* 솔라 검색 방식이 0시 기준으로 일괄 조회를 하되 성공한 시간과 아이디를 기준으로 필터를 한다면 복구와 정규작업이 동일한 로직이 된다
  * 단, 해당 시간 기준 필터 조회가 충분히 속도가 잘 나와야 한다
* 이미 수행한 하이브 파티션 작업에 대한 조치는 다시 수행하더라도 문제가 없도록 `if not exists` 구문을 넣어 방어한다
  * 단, min 파티션과 hour 파티션은 복구 순서가 겹치면 문제가 될 수 있다
  * min 파티션 복구를 수행하고 

