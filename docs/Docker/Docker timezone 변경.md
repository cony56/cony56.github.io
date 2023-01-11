---
layout: index
title: Docker timezone 변경
nav_order: 6
parent: Docker
permalink: /Docker5
---

작성일자: 2022/10/06

### Docker timezone 변경

도커 컨테이너 상에선 timezone이 UTC로 기본 설정 되어있다. 파이썬 어플리케이션 내에서 로그를 찍을 때 정확한 시간을 기록해야 모니터링이 편하다.

설정 방법은 간단하여 원리만 알고 있으면 된다.



#### 설정 방법

image: centos:7

1) 도커파일에서 심볼릭 링크 생성

`sudo ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime`

* sudo 패키지가 없다면 도커파일에서 설치해준다.
* `ln -sf` 는 심볼릭 링크를 지정해주는 명령어이며 `/usr/share/zoneinfo/` 내에는 여러 시간대가 있어 이 중 원하는 시간대의 파일을 `/etc/localtime` 이란 위치에 심볼릭 링크로 연결해준다

2. 도커 실행 환경변수에 TZ 변수 선언

   도커 실행 시 env에 tz=Asia/Seoul 까지 입력하여 환경변수에 추가해줘야 잘 적용이 된다.

3. timezone 확인

   배쉬셀에서 `date` 명령어를 통해 잘 바뀐 것을 확인할 수 있다.

   `Thu Oct  6 11:14:11 KST 2022`

#### 이외의 방법

python에도 timezone 관련 내장 파일들이 존재한다

```
/usr/local/lib/python3.5/dist-packages/pytz/zoneinfo/Asia/Seoul
```

동일하게 해당 파일을 /etc/localtime에 심볼릭 링크를 걸어주면 된다.



#### 출처

* https://proni.tistory.com/entry/%F0%9F%90%B3-Docker-Timezone%EC%8B%9C%EA%B0%84-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0

