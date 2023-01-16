---
layout: index
title: Local Database와 Docker 내 웹서버 연결
nav_order: 3
parent: Docker
permalink: /Docker2
---

작성일자: 2022/05/26

### Remote Database와 Docker 내 웹서버 연결


#### 1) ipconfig로 ip 확인 후 입력

#### 2) postgresql config 파일 수정

windows 기준으로 `C:\Program Files\PostgreSQL\14\data` 경로 접근

-> `pg_hba.conf` 파일 수정

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             0.0.0.0/0            scram-sha-256
```

-> 추가해주기



-> `postgresql.conf`

```
listen_address = '*'
```

-> 모든 remote 서버에서 접근가능하게 함

​	-> 보안이 필요한 경우 listen_address에 해당 remote 서버의 ip를 입력하면 됨



#### 3) postgresql 재기동

`(C:\Program Files\PostgreSQL\14\bin) pg_ctl.exe reload -D C:\Program Files\PostgreSQL\14\Data `

