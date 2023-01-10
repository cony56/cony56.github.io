---
layout: default
title: Local Database와 Docker 내 웹서버 연결
nav_order: 1
---

작성일자: 2022/04/07

### Docker image Offline 환경에 build


#### ONLINE 서버

1. docker pull로 해당 명령어 받기

2. docker image save 명령어로 이미지 파일을 tar 파일로 저장

```
docker save <image_name>:<tag_name> > <filename>.tar
```

#### OFFLINE 서버

3. tar 파일을 다음 명령어로 docker image에 등록한다

```
docker image load -i < <file_name>.tar
```

