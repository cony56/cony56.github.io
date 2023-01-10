---
layout: index
title: Docker Container에서 한글 읽기
nav_order: 1
---

작성일자: 2022/10/06

### Docker Container에서 한글 읽기

#### 문제

centos base image의 container 내에서 파이썬 파일을 실행하면 문자가 깨져서 나왔다.



#### 해결방법

1) 구글링 내용중에

```
$ yum install -y glibc-common glibc
$ localectl list-locales | grep -i ko
$ locale
$ locale
LANG=ko_KR.UTF-8
LC_CTYPE=ko_KR.UTF-8
LC_NUMERIC="ko_KR.UTF-8"
LC_TIME="ko_KR.UTF-8"
LC_COLLATE="ko_KR.UTF-8"
LC_MONETARY="ko_KR.UTF-8"
LC_MESSAGES="ko_KR.UTF-8"
LC_PAPER="ko_KR.UTF-8"
LC_NAME="ko_KR.UTF-8"
LC_ADDRESS="ko_KR.UTF-8"
LC_TELEPHONE="ko_KR.UTF-8"
LC_MEASUREMENT="ko_KR.UTF-8"
LC_IDENTIFICATION="ko_KR.UTF-8"
LC_ALL=
```

를 통해 locale 이렇게 출력되게끔 하라고 나왔는데 `localectl list-locales` 를 했을 때

`Failed to dbus-connection no such file or directory` 와 같은 에러가 나왔다.

원인을 dbus 관련 파일이 제대로 생성이 되지 않아서였고 /var/run/dbus/~ 디렉토리가 없는 걸 확인했다.

-> dockerfile의 entrypoint에서 `/bin/bash` 대신 `/sbin/init` 을 실행해주니 localectl 명령어가 잘 작동하였다.

`/sbin/init`의 역할은?

* init 이라고도 불리며 나머지 부트 프로세스를 주관하며 사용자를 위한 환경을 설정하는 역할을 한다. /etc/rc.d/rc.sysinit 스크립트를 실행하여 환경 경로, 스왑, 파일 시스템 확인, 시스템 초기화에 필요한 여러 작업을 실행한다. -> 추가로 찾아보니 시스템 권한을 부여하는 역할도 한다고 한다.

  

2. 이후 localectl 명령어를 통해 한국어로 언어 설정을 바꿔줬다.

   `localectl set-locale LANG=ko_KR.utf8`

   설정을 해주고 다시 locale 로 설정을 확인해보니 모두 변경되어 있었다.

   이 값은 /etc/locale.conf 의 값에서도 수정이 가능하긴 하다.

   

   한가지 문제는 컨테이너 내부에서 /sbin/init이 실행된 후에는 적용이 가능하지만 dockerfile에선 Entrypoint 로 `/sbin/init` 을 실행한 후 localectl 명령어를 연속적으로 사용할 수 없었다.. 차선책으로 /etc/locale.conf에 환경 변수를 추가해주는 방식으로 실행을 하니 의도한대로 잘 동작하긴 하지만 컨테이너 접속 시 아래와 같은 warning이 뜬다.

   ```
   bash: warning: setlocale: LC_ALL: cannot change locale (ko_KR.utf8): No such file or directory
   /bin/sh: warning: setlocale: LC_ALL: cannot change locale (ko_KR.utf8): No such file or directory
   ```

   

   -> 이후 cat 명령어나 python 실행을 통해 파일의 return 값 혹은 파일 그 자체가 한글로 정상 출력 되는 것을 확인했다. 하지만 vi 에디터로 실행했을 때는 여전히 한글이 깨져 있었다.

3. vi 에디터는 .vimrc 파일에 별도로 설정이 필요했다.

   ```
   set encodig=utf-8
   set fileencodings=utf-8,euc-kr
   ```

   해당 문자를 /root/.vimrc에 넣어줘야 한다.

   dockerfile 상에서 이를 실행하기 위해

   ```dockerfile
   RUN echo -e "set encodig=utf-8 \n set fileencodings=utf-8,euc-kr" >> /root/.vimrc
   ```

   와 같이 작성했다. echo 명령어의 -e 옵션은 escape 문자열의 사용을 가능하게 하여 두 줄을 한 번에 입력할 수 있다.