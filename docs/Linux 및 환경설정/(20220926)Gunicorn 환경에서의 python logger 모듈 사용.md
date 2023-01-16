
layout: index
title: Gunicorn 환경에서의 python logger 모듈 사용
nav_order: 9
parent: Linux
permalink: /Linux8
---

작성일자: 2022/09/26

Gunicorn 환경에서의 python logger 모듈 사용



Gunicorn worker가 8개 떠 있는 Flask webserver에서 python logging 모듈의 TimedRotatingFileHandler를 사용해 어플리케이션 로그를 찍고 있었다. 운영 환경에서 모니터링을 하던 중 2가지 오류를 파악했다.



#### 문제

1. 분명 자정(midnight)에 파일 rotating을 하는건데 오후 2시쯤(api 실행시간)에 rotating을 하기 시작했다.. 
2. 이 때를 기점으로 로그 파일에 로그가 안 쌓였다. 오전부터 api는 호출이 됐었는데 갑자기 이 시기에온 api 요청 이후 아무리 요청을 보내도 log 파일에 쌓이지 않았다. 그러다가 2시간 후 부터 다시 쌓이기 시작했다.. 레퍼런스를 찾아보니 멀티프로세스 환경에서 로그가 과거 파일과 현재 파일에 뒤죽박죽 쌓인다고 한다.



#### 해결방안

1. stackoverflow에 나온 여러 답변은 프로세스가 자정까지 켜졌다는 가정하에 로그 파일이 rotating 된다고 한다. 웹서버 프로세스는 죽지 않고 떠있는데 안되는 걸 봐서 api 호출이 필요하단 말인가? 해서 자정에 api를 호출해서 file rotating이 잘 되는지 확인하려 한다.
2. gunicorn 환경에서 logger 사용 방법에 관한 레퍼런스를 찾아보던 중 stackoverflow에서 아래와 같은 글을 찾았다.

```
Each worker is an isolated process with its own memory so you can't really share the same logger across different workers.

The master process is a simple loop that listens for various process signals and reacts accordingly. It manages the list of running workers by listening for signals like TTIN, TTOU, and CHLD. TTIN and TTOU tell the master to increase or decrease the number of running workers.
```

마스터 프로세스의 역할은 외부로부터 시그널을 받고 워커의 수를 늘릴지 결정하는 것이며 다른 자식 워커들은 별도의 메모리에서 운영된다. 때문에 같은 로거를 사용하는 것은 불가능하다.. 하지만 logger 들이 동일한 경로에 log를 작성하기 때문에 파일이 변경되는 시점에 오류가 생기는 것 같다..

multiprocess 환경에서 각 worker 별로 log를 찍을 수 있는 방안을 고민하거나 다른 로그 모듈을 고민,혹은 다른 handler를 사용하는 걸 고민해야 할 거 같다.