---
layout: index
title: PyKoSpacing docker container내에서 적용하기.
nav_order: 7
parent: Linux
permalink: /Linux6
---

작성일자: 2022/07/27

### 띄어쓰기 모듈 docker container 내에서 적용하기


기존 띄어쓰기 엔진을 전희원님이 개발한 PyKoSpacing 모듈로 교체하면서 찾은 문제



개발환경: centos7 image의 docker container 상에서 개발을 진행함

1. 구글링을 했을 때 `pip install git+https://github.com/haven-jeon/PyKoSpacing.git `으로 설치하게 되어있지만 이 모듈 하나를 위해 docker container 내에 git을 깔면 무거워지기 때문에 소스코드를 다운받아 수동설치 하는 방법을 선택함

2. 기존 파이썬 버전(3.6.10)이 낮아 tensorflow 2.8.2 버전을 사용하지 못하는 문제를 발견 -> 3.7x 버전으로 변경하면 사용가능하다고 함

​	-> 일단 3.6점대 버전으로 설정된 pip가 tensorflow 2.6.2버전까지 밖에 찾지 못해서 다운받을 수가 없었다..

3. python을 3.7점대로 설치하고 pip3를 설치하였지만 계속해서 tensorflow 2.6.2 이상 찾지 못하는 걸 확인

   -> 문제는 centos7 내부에 이미 내장된 python 패키지(3.6.8)가 존재하고 (usr/local/lib/python3.6)

   이 주소를 참조하여 python을 실행하기 때문이었다.

   이를 해결하기 위한 방법은 유저의 bashrc에 alias 라는 기능을 통해 변수가 어떤 주소를 참조할 지 지정해주는 것이다.

```
alias python3 = "/usr/local/bin/python3.7"
alias pip='python3.7 -m pip'
```

와 같이 vi 에디터를 통해 입력해주고 source ~/.bashrc 를 통해 적용해주면 python 버전과 pip 버전이 모두 변경된 것을 볼 수 있다.

4. 하지만 이를 dockerfile에서 실행해줬을 때, 계속해서 pip에 관련된 명령어에서 오류가 났다.

```dockerfile
RUN echo alias python3="/usr/local/bin/python3.7" >> /root/.bashrc
RUN echo alias pip="python3.7 -m pip" >> /root/.bashrc
```

이유는 dockerfile 상에서 위와 같이 입력했을 때 큰 따옴표가 제대로 인식되지 않기 때문이다.

```
alias python3 = /usr/local/bin/python3.7
alias pip=python3.7 -m pip
```

위와 같이 값이 반영되어 pip=python3.7 까지만 인식하고 그 뒤의 값들을 에러로 처리한다.

 vi 에디터로 적은 것과 같은 문자열을 입력해주려면 

```
RUN echo alias python3="/usr/local/bin/python3.7" >> /root/.bashrc
RUN echo alias pip='"python3.7 -m pip"' >> /root/.bashrc
```

와 같이 한번 더 감싸줘야 한다.



5. 이후 container 내부로 들어가 pip install tensorflow==2.7.2를 실행하면 pip가 python3.7.1의 pip로 실행되어 잘 작동하는 것을 볼 수 있었다. 이렇게 문제가 해결되는지 알았으나 dockerfile로 작성하자 바로 실패하는 것을 볼 수 있었다.
6. 아직도 명확한 이유는 모르겠으나 dockerfile로 작성할 때 python, pip에 alias가 먹히지 않아 pip install -r requirements.txt로 패키지를 다운받으면 python3.6/site-packages 밑으로 다운되는 것을 확인했다. 또한 이후 python3 setup.py install 을 통해 PykoSpacing을 수동 설치할 때도 같은 문제가 일어났다.
7. 이에 대한 해결법은 결국 install을 할 때 python3.7 명령어를 사용하는 것이었다.

```dockerfile
# python 관련 패키지 설치
RUN python3.7 -m pip install --upgrade pip
RUN python3.7 -m pip install -r requirements.txt
RUN pip install tensorflow==2.7.2
# PyKoSpacing 설치
WORKDIR /applibrary/ta-engine-analysis/PyKoSpacing/
RUN python3.7 setup.py install
```

이렇게 python3.7을 명시해주면 3.7.1을 통해 pip를 실행하고, 라이브러리를 수동 설치하는 것을 볼 수 있었다.

8. 안타깝게도 3.7.1로 모든 패키지를 설치해줬을 때 API 서버의 다른 라이브러리가 작동하지 않았다....

9. 생각해보니 이 모든 원인은 tensorflow 버전이 너무 높아 파이썬과 호환되지 않는 것이었고 혹시나해서 PykoSpacing 모듈의 setup 파일에 지정된 2.8.2 버전을 2.6.2로 강제로 낮춰서 설치를 시도했다.

10. 정말 다행히 작동에 문제가 없었고 결국 기존의 Python 3.6.8 환경에서 띄어쓰기 기능을 추가할 수 있었다.

11. PyKospacing 관련글을 찾아보면 클래스를 불러올 때 변수값으로 미리 띄어쓰기에 적용해야 할 신조어를 등록하는데, 이렇게 하면 AS-IS 구조상 API를 호출할 때 마다 객체를 생성해야 하므로 성능이 떨어졌다.

    **[변경전]**

    ```python
    @app.route('/spacing',method=['POST'])
    def apply_spacing():
        sentence = json.loads(request.data)['sentence']
        custom_dict_rule = custom_dict_reload(query)
        space = Spacing(rule=custom_dict_rule)	
        new_sentence = space(sentence)
        return jsonify(new_sentence)
    ```

    이런식으로 매번 새롭게 객체를 생성하는 구조로 예시가 나와있었고, 왠지 rule이 attribute으로 구성되어 있을 거 같아 소스를 찾아봤다. 소스코드는 간단했고 __ init __ 함수에서 attribute를 수정해주므로 해당 로직을 통해 객체가 생성된 후에도 충분히 룰로 쓰일 리스트를 변경해줄 수 있었다.

    **[변경후]**

    ```python
    space = Spacing(rule=custom_dict_rule)	
    
    app.route('/spacing',method=['POST'])
    def apply_spacing():
        sentence = json.loads(request.data)['sentence']
        custom_dict_rule = custom_dict_reload(query)
        for r in custom_vocab:
                space.rules[r] = re.compile('\s*'.join(r))
        new_sentence = space(sentence)
        return jsonify(new_sentence)
    ```



8. 고생을 많이 했지만 리눅스에 다수의 python 버전이 설치되어 있을 때, 그리고 dockerfile에서 실행하는 경우 다르게 작동하는 것을 알 수 있었고 github에 있는 소스코드를 독해하고 필요에 따라 수정하는 방법을 익힐 수 있었다.