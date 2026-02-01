---
layout: post
title:  "01/25 (일) 하루만에 끝내는 도커"
date:   2026-01-01 05:00:00 +0900
categories: psyoblade created
---

## 루틴: 2026년 1월 25일 (일)

>     

### 오늘의 작업 이력

#### 중요한 일

* 윈도우 11 환경에서 라이센스에 영향없는 도커 환경 구성하기
* 해당 도커 환경에서 데이터 엔지니어링 학습 수행 가능 여부 확인하기

#### 이력

* 17:00 ~ 18:30 

#### 환경

* WIndows 11 Version
* WSL (Window Subsystem for Linux)
* Ubuntu 22.04 LTS
* Windows Terminal
* Rancher Desktop

### 1. 윈도우 11 에서 도커 환경 구축하기

> 학습 관련 영상 링크와 내용 요약

#### 1-1. 윈도우 업데이트 및 바이오스 설정

* 시작 버튼 →  설정 → Windows 업데이트 → 업데이트 확인
* Windows 하드웨어 가상화 지원 활성화 : 작업 관리자 → 성능 → CPU → Virtualization: Enabled 확인
  * Disabled 되어 있다면 부팅 시에 Del 혹은 F2 키를 통해 "BIOS/UEFI 설정(CMOS 설정)" 화면으로 진입
    * **Intel**: `Intel Virtualization Technology` / `VT-x`
    * **AMD**: `SVM Mode` / `AMD-V`

#### 1-2. WSL2 설치 및 구성

* "Windows 기능 켜기/끄기" 설정을 통해서 기능 활성화를 통해 설치할 수도 있지만 직접 설치가 간편함

  * Linux용 Windows 하위 시스템
  * Windows 하이퍼바이저 플랫폼
  * 가상 머신 플랫폼 등

* 시작버튼 + 마우스 우클릭 + 터미널(관리자) → `wsl --list --online` 명령으로 설치 가능한 리눅스 배포판 확인

  1. 설치는 `wsl --install -d Ubuntu-22.04` 와 같이 원하는 `distro-name` 설치가 가능

     * 설치 후에는 설치된 우분투 버전에 따라서 윈도우 재시작이 필요할 수 있으므로 안전하게 재부팅

  2. 재부팅 후에 터미널 화면이 뜨게 되고 관리자 계정 생성합니다

     * `Enter new UNIX username:` 항목에서 `ubuntu`
     * `New password:`항목에서  `ubuntu` 와 같이 입력

  3. 계정 생성을 하고 터미널 상태에서 패키지 저장소 주소를 국내로 변경해야 업데이트가 빠름

     * `sudo vi /etc/apt/sources.list` 접속하여 `archive.ubuntu.com` 값을 `mirror.kakao.com` 으로 변경

     * `vi` 명령어로는 `:%s/archive.ubuntu.com/mirror.kakao.com` 엔터 치면 변경

     * `sudo apt update` 명령으로 패키지 목록 업데이트 후 `sudo apt install <package-name>` 으로 설치

     * ```bash
       # 아래는 우분투 SSH 접근을 위한 기본 도구 설치
       sudo apt install openssh-server openssh-client -y
       sudo apt install net-tools
       
       # 방화벽 구성을 위한 systemd 설치
       cat /etc/wsl.conf
       [boot]
       systemd=true
       
       # 자바 설치
       sudo apt update -y # 패키지 목록만 업데이트
       sudo apt upgrade -y # 기 설치된 패키지를 최신 버전으로 업데이트
       java -version
       
       # 파이썬 설치
       sudo apt install openjdk-17-jdk -y # 자바 설치
       sudo add-apt-repository ppa:deadsnakes/ppa # 파이썬 패키지 선택
       sudo apt install python3.9 -y # 파이썬 설치
       python3 --version
       ```

     * 우분투 설치가 끝나면 `exit` 명령으로 빠져나와서 윈도우 관리자 터미널에서 `wsl` 재시작

     * ```cmd
       wsl.exe --update
       wsl.exe --shutdown
       ```

  4. 종료 후에 윈도우 터미널에서 설치된 플랫폼 확인

     * `exit` 통해서 리눅스 접속 종료
     * `wsl -l` 명령을 통해서 설치된 플랫폼 확인
     * `wsl --set-default Ubuntu-22.04` 명령으로 기본 플랫폼 설정
     * `wsl` 명령으로 리눅스 접속이 가능함

  5. `Vmmem` 메모리 점유율 제한하기

     * `Windows + R` 누르고 `%USERPROFILE%` 입력하면 `C:\Windows\<사용자>` 경로로 이동

     * 해당 경로에 `.wslconfig` 파일이 없다면 아래와 같이 생성합니다

     * ```bash
       [wsl2]
       memory=16GB
       swap=4GB
       ```

     * VM 메모리를 16GB로 제한하고, SSD 용량을 가상 RAM으로 활용하는 정도를 4GB로 제한할 수 있습니다

     * 설정 후에는 마찬가지로 `wsl --shutdown` 명령으로 `WSL2` 를 재시작해야 합니다

#### 1-3. 실습

* 윈도우 재시작 이후에 설치된 도커 버전 확인

  * ```bash
    docker version
    ```

* 터미널 환경에서 도커 프로세스 학인

  * ```bash
    docker ps -a
    ```

* 도커를 통한 우분투 컨테이너 내부에서 헬로 월드 출력

  * ```bash
    docker run hello-world
    ```

### 2. 도커 컨테이너 활용하기

> 윈도우 인텔리제이 환경에서 우분투 서버의 `README.md` 파일 다운로드 애플리케이션을 구현합니다

#### 2-1. 우분투 서버 및 접근 환경 구성 하기

1. 우분투 24.04 LTS 도커 컨테이너 이미지 생성하기

   * ```dockerfile
     FROM ubuntu:24.04
     
     ENV DEBIAN_FRONTEND=noninteractive
     RUN apt-get update && apt-get install -y --no-install-recommends \
         openssh-server ca-certificates tzdata \
      && rm -rf /var/lib/apt/lists/*
     
     # sshd 준비
     RUN mkdir -p /var/run/sshd
     
     # ubuntu 사용자 생성 (홈: /home/ubuntu)
     RUN useradd -m -s /bin/bash ubuntu \
      && mkdir -p /home/ubuntu/.ssh \
      && chown -R ubuntu:ubuntu /home/ubuntu/.ssh \
      && chmod 700 /home/ubuntu/.ssh
     
     # 샘플 파일 (요구사항의 대상)
     RUN echo "# Hello from container" > /home/ubuntu/README.md \
      && chown ubuntu:ubuntu /home/ubuntu/README.md
     
     # SSH 보안/접속 설정
     # - root 로그인 금지
     # - 비밀번호 인증 비활성화(키 기반만 허용)
     # - ubuntu 사용자 허용
     RUN sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config \
      && sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config \
      && sed -i 's/^#\?PubkeyAuthentication.*/PubkeyAuthentication yes/' /etc/ssh/sshd_config \
      && echo "AllowUsers ubuntu" >> /etc/ssh/sshd_config
     
     EXPOSE 22
     
     CMD ["/usr/sbin/sshd","-D","-e"]
     ```

   * `cat > Dockerfile` 명령어를 통해서 붙여넣고 `Ctrl+D` 통해서 파일 생성

2. 도커 이미지 빌드 및 컨테이너 기동

   * ```bash
     docker build -t ubuntu24-sshd:1 .
     
     # 컨테이너 실행: Windows에서 localhost:2222로 접속
     docker run -d --name ubuntu24-sshd \
       -p 2222:22 \
       ubuntu24-sshd:1
     
     # 상태 확인
     docker ps --filter "name=ubuntu24-sshd"
     docker logs ubuntu24-sshd --tail 50
     ```

3. Windows에서 SSH 키 만들고 컨테이너에 주입

   * ```powershell
     # 키가 없으면 생성(이미 있으면 건너뛰어도 됨)
     ssh-keygen -t ed25519 -f $env:USERPROFILE\.ssh\id_ed25519 -N ""
     ```

4. WSL2에서 공개키를 컨테이너 ubuntu 계정 authorized_keys로 주입

   * ```bash
     # Windows 공개키 경로를 WSL 경로로 읽어오려면 보통 /mnt/c 를 사용
     PUBKEY_PATH="/mnt/c/Users/$(cmd.exe /c echo %USERNAME% | tr -d '\r')/.ssh/id_ed25519.pub"
     
     # 공개키 내용 확인(오타/경로 문제 방지)
     test -f "$PUBKEY_PATH" && head -c 80 "$PUBKEY_PATH" && echo || echo "Public key not found: $PUBKEY_PATH"
     
     # authorized_keys 주입
     docker exec -u root ubuntu24-sshd bash -lc "cat >> /home/ubuntu/.ssh/authorized_keys" < "$PUBKEY_PATH"
     
     # 권한 정리(SSH는 권한 엄격)
     docker exec -u root ubuntu24-sshd bash -lc "\
       chown -R ubuntu:ubuntu /home/ubuntu/.ssh && \
       chmod 700 /home/ubuntu/.ssh && \
       chmod 600 /home/ubuntu/.ssh/authorized_keys \
     "
     ```

5. Windows에서 SSH 접속 테스트 + 방화벽 체크

   * ```powershell
     ssh -p 2222 ubuntu@localhost
     # 컨테이너에 접속되면
     cat /home/ubuntu/README.md
     exit
     ```

     * 처음 접속이면 “신뢰할 호스트냐(yes/no)” 묻고, `known_hosts`에 등록됩니다.

#### 2-2. 접근 안되는 경우 체크할 항목

1. 포트 바인딩 확인 (WSL2)

   * ```bash
     # WSL2
     docker port ubuntu24-sshd
     # 22/tcp -> 0.0.0.0:2222 같은 형태가 떠야 정상
     ```

2. Windows 방화벽 인바운드 규칙

   * ```powershell
     New-NetFirewallRule -DisplayName "WSL2 Docker SSH 2222" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 2222
     ```

3. Windows OpenSSH Client 설치 여부

   * 대부분 기본 탑재지만, `ssh` 명령이 없다면 Windows 기능에서 “OpenSSH Client” 설치가 필요합니다.

#### 2-3. IntelliJ에서 “SSH로 README.md 다운로드” 앱 만들기

1. `build.gradle` 생성

   * ```groovy
     plugins {
         id 'java'
         id 'application'
     }
     
     repositories {
         mavenCentral()
     }
     
     dependencies {
         implementation 'com.hierynomus:sshj:0.39.0'
     }
     
     application {
         mainClass = 'demo.SshDownload'
     }
     ```

2. Java 코드 작성: `src/main/java/demo/SshDownload.java`

   * ```java
     package demo;
     
     import net.schmizz.sshj.SSHClient;
     import net.schmizz.sshj.transport.verification.PromiscuousVerifier;
     import net.schmizz.sshj.xfer.FileSystemFile;
     
     import java.nio.file.Path;
     
     public class SshDownload {
         public static void main(String[] args) throws Exception {
             String host = "localhost";
             int port = 2222;
             String user = "ubuntu";
     
             // Windows 기본 키 경로: C:\Users\<you>\.ssh\id_ed25519
             // 실행 환경마다 다를 수 있으니 필요하면 인자로 받도록 바꿔도 됨.
             Path privateKey = Path.of(System.getProperty("user.home"), ".ssh", "id_ed25519");
     
             String remotePath = "/home/ubuntu/README.md";
             Path localPath = Path.of(System.getProperty("user.dir"), "README.downloaded.md");
     
             try (SSHClient ssh = new SSHClient()) {
                 // 개발 편의: known_hosts 검증을 생략(운영에서는 절대 비추)
                 ssh.addHostKeyVerifier(new PromiscuousVerifier());
     
                 ssh.connect(host, port);
                 ssh.authPublickey(user, privateKey.toString());
     
                 ssh.newSCPFileTransfer().download(remotePath, new FileSystemFile(localPath.toFile()));
                 ssh.disconnect();
             }
     
             System.out.println("Downloaded to: " + localPath.toAbsolutePath());
         }
     }
     ```

### 3. 개발 환경 이해하기

#### 3-1. 아키텍처

```less
[Windows 11]                ← Host OS
   |
[WSL2 Hyper-V VM]           ← Guest OS (Linux Kernel)
   |
[Ubuntu 22.04]              ← User Space Linux
   |
[Docker Engine]
   |
[Containers]
```

#### 특징 및 참고 사항

* 윈도우 리부팅 시에 WSL IP 변경
* 윈도우 환경에서 WSL 환경 접근 시에 포트포워딩 필요
* `/mnt/c` 경로 접근 시에 `I/O` 병목이 있음
* 컨테이너 재시작 시에 키가 사라지므로 볼륨 마운트를 통해 유지할 필요 있음
* known_hosts 관련하여 컨테이너를 재생성하면 호스트키가 바뀌어서 “REMOTE HOST IDENTIFICATION HAS CHANGED”가 뜰 수 있음
  *  Windows의 `C:\Users\<you>\.ssh\known_hosts`에서 해당 엔트리를 제거하거나 `ssh-keygen -R "[localhost]:2222"`로 정리

### 레퍼런스

* [윈도우 11 에서 윈도우 서브시스템 포 리눅스 (WSL 2) 초간단 설치하기](https://www.youtube.com/watch?v=ob1nUrSy6LA)
* [Windows11 에 WSL 이용하여 Ubuntu 24.04 설치하기](https://www.youtube.com/watch?v=fMbChcFeGWs)
* [Windows 11에서 WSL2 설치하기](https://velog.io/@gynchoi17/WSL2-Windows-11%EC%97%90%EC%84%9C-WSL2-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0)
  * [Advanced settings configuration in WSL](https://github.com/MicrosoftDocs/wsl/blob/main/WSL/wsl-config.md)
  * [WSL을 사용하여 Windows에 Linux를 설치하는 방법](https://learn.microsoft.com/ko-kr/windows/wsl/install)
* [Rancher Desktop IO](https://rancherdesktop.io/)
