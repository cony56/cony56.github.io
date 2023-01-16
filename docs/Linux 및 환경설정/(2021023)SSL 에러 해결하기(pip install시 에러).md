---
layout: index
title: SSL 에러 해결하기(pip install시 에러)
nav_order: 2
parent: Linux
permalink: /Linux1
---

작성일자: 2021/10/23

# SSL 에러 해결하기

## 에러 내용

노트북 메인보드가 나가고 새 환경에 아나콘다를 설치했다. 교육을 빨리 들어야하는데.. `pip install`이 안되는거 때문에 이틀동안 구글링을 한 결과 겨우 에러를 해결했다. 트러블 슈팅을 잘하기 위해서 구글링도 중요하지만 SSLError가 뭔지, 내가 해결한 방식이 어떻게 해결한건지 알아야 다른 오류들도 더 잘 수정할 수 있을 거 같아 알아보려 한다.



### 문제 상황)

```powershell
WARNING: pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.",)': /simple/pip/
WARNING: Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.",)': /simple/pip/
WARNING: Retrying (Retry(total=2, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.",)': /simple/pip/
WARNING: Retrying (Retry(total=1, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.",)': /simple/pip/
WARNING: Retrying (Retry(total=0, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.",)': /simple/pip/
Could not fetch URL https://pypi.org/simple/pip/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/pip/ (Caused by SSLError("Can't connect to HTTPS URL because the SSL module is not available.",)) - skipping
Requirement already up-to-date: pip in /opt/conda3/lib/python3.5/site-packages (19.3.1)
WARNING: You are using pip version 19.3.1; however, version 20.2.2 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.

```

-> `pip install` 명령어에 대해서 ssl module in python is not available이 뜨고 여러번의 retry 끝에 ssl certificate 확인에 문제가 있다는 에러가 떴다.



### 구글링하며 시도한 해결방법

1) pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org (패키지명) 

* pip 명령어를 쓸때마다 뒤에 파라미터를 붙여줘야함

2) 영구적으로 신뢰할 수 있는 호스트를 저장해둠

```
[global]
trusted-Host = pypi.python.org
               pypi.org
               files.pythonhosted.org
```

-> pip.ini 파일에다가 해당 내용을 추가해줌

2) `C:\Users\user(사용자이름)\anaconda3\Lib\site-packages\pip\_vendor\requests\sessions.py` 에 들어감 -> self.verify = True 를 False로 변환

3) 환경변수 설정 추가

환경변수 편집에 들어가

* `C:\anaconda3\Scripts`

* `C:\anaconda3`
* `C:\anaconda3\Library\bin`

을 추가함

4) Anaconda 하위 경로에 파일 복사

아나콘다 하위 폴더 중 /DLLs/ 경로 안에다가

C:\Users\user\anaconda3\Library\bin로부터

* libcrpyto-1_1-x64.dll

* libcrpyto1_1-x64.pdb

* libssl-1_1-x64.dll

* libssl-1_1-x64.pdb

4개 파일을 붙여넣기

------> 마지막 방법으로 pip install을 마침내 성공하였다!

문제가 해결되었으니 에러 내용을 살펴보고 SSL이 뭔지, 왜 DLLs 저기 파일을 넣으니 해결되니

파악해보자



### SSL이란?

* SSL(Secure Socket Layer)은 웹사이트와 브라우저 사이에 전송된 데이터를 암호화하여 인터넷 연보안을 유지하는 표준 기술이다. 이는 해커가 개인 정보 및 금융 정보를 포함한 전송되는 모든 정보를 열람하거나 훔치는 것을 방지한다.
* 다른 말로는 사용자 컴퓨터와 웹사이트 간에 암호화된 연결을 수립함으로써 인증되지 않은 사용자로부터 각 방문(Session) 중 교환한 중요 데이터를 보호한다.

### SSL의 작동 원리와 절차

최종 사용자가 볼 수 없는 'SSL Handshake'라는 프로세스를 통해 웹 서버와 브라우저 간 연결이 수립된다. 3가지 키를 사용하여 대칭 세션 키를 만들고 키는 데이터를 암호화하는데 사용한다.

1. 서버에서 비대칭 공개 키(첫 째 키)의 복사본을 브라우저로 전송
2. 브라우저에서 대칭 세션 키(둘 째 키)를 만들어 서버의 비대칭 공개 키로 암호화한 다음 서버로 전송
3. 서버는 대칭 세션 키를 얻기 위해 비대칭 비공개 키(셋 째 키)를 사용해 세션키를 해독
4. 이를 통해 서버, 브라우저 간 대칭 세션 키를 사용해 전송된 모든 데이터를 암호화, 해독 가능함

이는 특정 세션에서만 유효하며 다음날 브라우저가 동일세션에 연결되면 새로운 세션 키가 생김

* TLS(Transport Layer Security)는 가장 최신 기솔로 SSL의 강력한 버전이다.
*  HTTPS는 웹사이트를 SSL/TTL  인증서로 보안하는 경우, URL 창에 표시된다.

사용자는 브라우저 바의 잠금 기호를 클릭해 인증서 발행, 웹사이트 소유 기업명을 포함한 인증서의 세부 내용을 확인할 수 있다.



### PIP란?

- Python 패키지 관리자 중 하나로 python 3.4버전부터 기본으로 포함되어 있고 파이썬으로 작성된 패키지를 설치하는 역할을 함



### DLL이란?

라이브러리란 란 소프트웨어 개발에서 자주 쓰고 기초적인 함수들의 중복개발을 피하려고 표준화된 함수 및 데이터 타입을 만들어서 모아 놓은 것

-> 이런 라이브러리는 언제 메인 프로그램에 연결하느냐에 따라 Static Link와 Dynamic Link로 나뉨

스태틱 링크(Static Link Library)란 정적 링크라고도 하며 컴파일 시점의 라이브러리가

링커에 의해 연결되어 실행 파일의 일부분이 됨

**DLL(Dynamic LInk Library)**이란 동적 링크라고 하며 해당 라이브러리의 기능을 사용시에만 라이브러리 파일을 참조하여 기능을 호출함

-> 정적 링크와 다른 점은 컴파일 시점에 실행 파일 함수를 복사하지 않고 함수의 위치정보로 함수를 호출함

DLL의 이점

1) 적은 리소스 사용 - 한코드를 여러 프로그램이 동시에 사용하여 메모리가 절약됨

운영체제와 프로그램이 더 빠르게 로드 및 실행되고 디스크 공간이 절약됨

2) 모듈식 아키텍처 - DLL 사용 시 모듈식 프로그램을 효과적으로 개발할 수 있음

3) 손쉬운 배포와 설치 - 여러 프로그램이 같은 DLL을 사용하는 경우 모든 프로그램에 업데이트나 수정 내용이 일괄적으로 적용됨, 재사용성이 뛰어나 배포가 쉬움

출처) https://goddaehee.tistory.com/185

###PDB란?

PDB(Program DataBase)란 프로그램에 대한 디버깅 정보를 저장하기 위한 파일 형식임

*.exe, *.dll 등의 프로그램은 *.pdb 없이도 실행가능하지만 함수, 변수의 이름, 소스파일, 소스 행정보(line number) 등이 포함되어 있어 프로그램 개발 시 오류를 찾거나 기능을 테스트하기에 좋음

출처) https://hi098123.tistory.com/333



--------------------------------------------------------------------------------------------------------

그렇다면 밑에 얘네는 왜 DLLs에 들어가야 했는가?

* libcrpyto-1_1-x64.dll

* libcrpyto1_1-x64.pdb
* libssl-1_1-x64.dll
* libssl-1_1-x64.pdb

Openssl 라이브러리에 의존성이 있는 파일들이기 때문에 필요했던 것이다.

환경변수에 anaconda3/ 경로가 등록 되어있었고 또 다르게 지정한 경로인 Scripts/ 내에 pip 파일에서 아마 내부적으로 ssl작업을 할 때 DLLs 폴더 내에 4가지 파일을 실행하도록 설계가 되어있는 것 같다.

** 또한 trusted-host를 쓰는 이유는 pip가 서버에 접근할 때 막혀져 있는 host 정보를 풀어주기 위해host를 등록해놓는 과정인 것 같다.