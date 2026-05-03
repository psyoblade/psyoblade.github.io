---
layout: post
title:  "05/03 (일) 안드레 카파시 클로드 가이드 & 글로벌 데이터 수집 아키텍처"
date:   2026-05-03 21:00:00 +0900
categories: psyoblade created
---

# 안드레 카파시 클로드 가이드  & 글로벌 데이터 수집 아키텍처

>      최근 핫 하다는 카파시 클로드 가이드 통해서 기본 설정을 바꿔보고 테스트도 해보고 하는데 생각보다 하드하게 사용하지 않아서 그런지 잘 못 느끼고 있다 다만, 실무에 적용 시에 얼마나 차이가 나는지 경험해보면 좋겠다.

## 목적

* 안드레 카파시 클로드 코드 가이드라인 리뷰
* 글로벌 데이터 수집 아키텍처 수정
* 인프런 스프링 시큐리티
* 중고 물품 정리
* 주식 포트폴리오 정리
* 투자 원칙 초안 작성
* 전인구 강의 듣기
* 멀티 캠퍼스 강좌 광고 쇼츠 작성
* 멀티 캠퍼스 강의 일정 조율 문의

## 목표

* 가이드라인 적용 테스트
* 글로벌 수집 아키텍처 개선

### 1. 안드레 카파시 클로드 가이드

#### Andrej Karpathy의 [LLM 코딩 문제 관찰에서 도출된 4가지 원칙](https://github.com/forrestchang/andrej-karpathy-skills/blob/main/CLAUDE.md)

------

1. **Think Before Coding (코딩 전에 생각)** 가정을 숨기지 말고, 모호할 땐 침묵 대신 질문. 혼란스러우면 멈추고 명확히 할 것.
2. **Simplicity First (단순함 우선)** 요청한 것만 구현. 추측성 기능, 불필요한 추상화, 안 쓰는 유연성 금지. 200줄이 50줄로 줄 수 있으면 줄여라.

3. **Surgical Changes (외과적 수정)** 요청된 부분만 건드릴 것. 인접 코드, 주석, 포맷 "개선" 금지. 내 변경으로 생긴 고아 코드(미사용 import 등)만 정리하고, 기존 dead code는 언급만.

4. **Goal-Driven Execution (목표 기반 실행)** "이렇게 해줘" 대신 검증 가능한 성공 기준을 정의. 예: "버그 수정" → "재현 테스트 작성 후 통과시켜라". LLM은 명확한 목표가 있으면 스스로 루프를 돈다.

#### 클로드 코드 적용 및 활용 방법

* 플러그인 설치

  ```bash
  # claude terminal 상에서 아래와 같이 플러그인 설치
  /plugin marketplace add forrestchang/andrej-karpathy-skills
  /plugin install andrej-karpathy-skills@karpathy-skills
  /reload-plugins
  ```

* 플러그인 설치 후 적용

  ```bash
  # /ka 만 치면 설치된 플러그인 확인이 가능하며 아래와 같이 활성화된다
  /karpathy-guidelines
  
  ⏺ The Karpathy guidelines are now loaded and active for this session. I'll apply them going forward:
  
    1. Think before coding — surface assumptions and tradeoffs before implementing
    2. Simplicity first — minimum code that solves the problem, nothing speculative
    3. Surgical changes — touch only what the task requires, match existing style
    4. Goal-driven execution — define verifiable success criteria before starting
  ```

* 명시적으로 항상 적용하는 방법

  ```bash
  # 모드 클로드 프로젝트에 적용하는 방법 (Global)
  curl -o ~/.claude/CLAUDE.md https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md
  
  # 특정 프로젝트에만 적용하는 방법 (Project)
  curl -o CLAUDE.md https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md
  ```

###  2. 글로벌 파일 수집 아키텍처 개선

#### 스트리밍 데이터인 로그 수집이 API 통한 상태관리가 필요한가?

1. **도커 스웜 서비스 통한 로그 동기화**
   1. 컨테이너 내부의 rsync 는 지속적으로 수행하기만 하면 됨
   2. 컨테이너의 상태는 도커 로그 드라이버 통해서 그라파나로 전송하고 이는 국내에서 모니터링 및 알림을 받음
2. **로그 전송의 일 별 완료 여부 기준**
   1. 전 일자의 로그가 적재 완료된 마커 `_SUCCESS` 혹은  `_ROWCOUNT=1234` 와 같은 선언적인 방식이 가장 좋지만 레거시는 적용이 어려움
   2. 일정 시간 경과 이후에 즉 수집 `SLA` 수준 초과 시에 적재 완료되었다고 간주하는 것이 가장 간편함
   3. 파일 변경 없음 감지 `Quiescence` 통해서 `find /logs/dt=20250502 -newer /tmp/checkpoint -type f | wc -l` 같은 방식으로 대기하는 방법도 있으나 `SLA` 수준을 만족시키지 못할 수 있음
3. **일 단위 로그 정합성 검증 방안**
   1. `rsync --checksum src/ dst/` 와 같이 체크섬 전송 시에 보장하는 것
      1. 전송 중 데이터 손상
      2. mtime 만 다르고 동일한 파일 스킵
      3. 내용이 다른데 +mtime 동일 감지
      4. 목적지 파일이 변조된 경우 재전송
   2. 반대로 `rysnc` 전송이 보장하지 않는 것
      1. 전송 이후에 추가된 파일
      2. 쓰고 있는 파일을 원격 파일을 읽는 경우
      3. 일정 시간 이후에 백필 되는 경우
   3. `SLA` 수준이 30분이라고 하면 0시 30분 시점에 검증을 수행
      1. 일 단위 적재는 체크섬과 30분 지연 적재
      2. 백필 전송은 최대 7일 과거 데이터에 대해서 하루에 1회만 수행
      3. 백필에 대해서는 unsent 방식으로 별도로 저장할 것인지 그대로 덮어쓸 것인지 검토
4. **여기까지 정리된 이상적인 수집 방향**
   1. 10분 주기 rsync (크기+mtime, 체크섬 없음)
      1.  → 누락/유실: 걱정 안 해도 됨 ✅
      2.  → Silent Corruption: 이론적으로 존재하나 현실적으로 희소
   2. 7일치 일 1회 배치 (--checksum)
      1.  → 백필로 인한 변경 감지: ✅
      2.  → 전날까지의 Silent Corruption 사후 감지: ✅
      3.  → 당일 발생한 Silent Corruption: 다음날 배치에서 잡힘
5. **여전히 발생할 수 있는 문제점**
   1. 10분 동기화 작업의 중복 실행에 따른 위험
      1. `flock -n /var/lock/rsync_job.lock rsync ...`
   2. 10분 동기화 스크립트와 7일치 배치 동기화 작업과 충돌 위험
   3. 다른 스크립트가 같은 서버를 동기화하는 경우
   4. 너무 오랜 시간 동기화 작업이 수행되는 경우 인지
   5. 컨테이너 내부의 스크립트 실행 오류에 대한 인지
   6. 특정 서버의 동기화 스크립트 오류의 운영 담당자에게로의 알림 기능
6. **일 단위 수집의 트리거 상태 변경**
   1. 매일 0시 30분에 서버 별 파일의 카운트 수와 전체 크기를 측정하고 `_SUCCESS` 파일에 정보를 저장한다
   2. 전일자와 오늘의 통계정보를 확인하여 증감이 통계적으로 큰 차이가 난다면 경고 메일을 그렇지 않으면 정상 메일을 발송한다
   3. 해당 `_SUCCESS` 파일을 기준으로 정상적재 여부를 판단하고 국내 동기화 및 하이브 작업을 수행한다

### 3. 스프링 시큐리티 학습

#### 프로젝트 생성 및 SecurityBuilder & SecurityConfigurer

* 개발 환경

  * `brew install openjdk@17`

* 실행 환경

  * ```bash
    # add symbolic link
    sudo ln -sfn /usr/local/opt/openjdk@17/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-17.jdk
    
    # cat ~/.zshrc
    export DEFAULT_USER="$(whoami)"
    
    alias j17="export JAVA_HOME=`/usr/libexec/java_home -v 17`  ; java -version"
    alias j11="export JAVA_HOME=`/usr/libexec/java_home -v 11`  ; java -version"
    alias  j8="export JAVA_HOME=`/usr/libexec/java_home -v 1.8 `; java -version"
    
    # docker-compose alias
    alias d="docker-compose"
    
    # git alias
    alias gl="git log --all --graph --oneline --decorate"
    alias gs="git status"
    alias gd="git diff"
    alias gb="git branch"
    alias gp="git pull"
    ```

###  회고

>  멀티 캠퍼스 강의가 AI 도구가 창궐하는 이 시기에 과연 얼마나 유효한 것인지 고민해볼 필요가 있겠다. 그럼에도 불구하고 필요한 부분도 있다고 생각은 한다. AI 도구가 만능은 아니며 여전히 엔지니어가 해야 할 일들이 남아있다고 생각하기 때문이다

