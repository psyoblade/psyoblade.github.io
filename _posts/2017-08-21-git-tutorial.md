---
layout: post
title:  "Git 기본 명령어 - '까먹지 말자'"
date:   2017-08-21 23:15:22 +0900
categories: psyoblade update
---
## Git 기본 명령어

### [원격 저장소에서 branch 가져오기](https://blog.outsider.ne.kr/641)

```git
1. 저장소 조회
    git branch      # 로컬
    git branch -r   # 리모트
    git branch -a   # 로컬/리모트

2. 브랜치 가져오기
    git checkout <remote-branch>    # 원격저장소의 브랜치로 작업트리를 변경 - detached HEAD상태로써 소스도 보고, 변경도 가능하지만, 잠시 확인하는 상태이며 저장되지 않음
    git checkout -b <remote-branch> <local-branch>      # 로컬에 브랜치도 만들고 체크아웃도 하는 경우

```

