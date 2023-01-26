---
layout: index
title: LINUX 내 파이썬 encoding 에러 해결
nav_order: 8
parent: Linux
permalink: /Linux7
---

작성일자: 2022/08/17

### LINUX 내 파이썬 encoding 에러 해결방법


Docker container 내부에서 한글로 된 string 값을 표준출력(std.out)하면 ascii encoding 관련 오류가 난다.

계속 헤매다가 이를 한번에 해결할 수 있는 환경변수를 찾았다..

### 에러 내용

```
UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)
```

### 해결 방법

`export PYTHONIOENCODING=utf8`

- PYTHONIOENCODING이란 환경변수를 통해 조정해주면 됐다..



-> 컨테이너 혹은 파드 생성 시 바로 적용을 해주려면, 유저 디렉토리의 `.bashrc` 나 `.bash_profile` 에 해당 값을 넣어주던지, 

yaml 파일 내에서 환경변수를 미리 지정해주면 될 거 같다.