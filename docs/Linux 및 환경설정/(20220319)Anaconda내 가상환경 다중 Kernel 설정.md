---
layout: index
title: Anaconda내 가상환경 다중 Kernel 설정
nav_order: 3
parent: Linux
permalink: /Linux2
---

작성일자: 2022/03/19

## Anaconda내 가상환경 다중 Kernel 설정 

#### 문제



가상환경에 Mecab 라이브러리를 설치한 후 Anaconda Prompt 내에서
아래와 같이 실행하면 작동하는데 jupyter notebook으로 접속하자 import가 실패했다.

```
>python
> import MeCab
```



원인은 jupyter notebook 실행 시 가상환경의 kernel이 아닌 base 환경의 kernel이 실행되도록 kernel.json 파일이 설계되어 있기 때문이다.



Jupyter 내 python kernel의 종류와 setting 파일을 경로를 확인하기 위한 커맨드는 아래와 같다.

```
(가상환경명) C:\Users\(사용자명)>jupyter kernelspec list
Available kernels:
  커널명         C:\Users\(사용자명)\AppData\Roaming\jupyter\kernels\가상환경명
  python3    C:\Users\(사용자명)\anaconda3\share\jupyter\kernels\python3
```

  ** 커널이 2개인 이유는 내가 하나 더 만들었기 때문



새로운 커널을 생성하는 방법은 아래와 같다. 

```
(가상환경명) ipython kernel install --user --name 가상환경 --display-name "Python(가상환경)"
```



하지만 이후 다시 jupyter notebook을 실행해도 바뀌지 않았다... 이유는 새롭게 생긴 kernel.json의 경로가 똑같이 기존 conda 환경의 python.exe를 가르켰기 때문이다.. 결국 위의 jupyter kernelspec으로 찾아낸 경로로 들어가 kernel.json 내의 python.exe 경로를 바꿨다.

```json
# 경로 ---> C:\Users\(사용자명)\AppData\Roaming\jupyter\kernels\가상환경명
{
 "argv": [
    "C:\\Users\\cony5\\anaconda3\\envs\\가상환경명\\python.exe",
  "-m",
  "ipykernel_launcher",
  "-f",
  "{connection_file}"
 ],
 "display_name": "Python",
 "language": "python",
 "metadata": {
  "debugger": true
 }
}
```

경로를 바꾸고 실행하자 



.json의 "ipykernel_launcher" 파라미터로부터 오류가남

가상환경 내에서 ipykernel을 다운받아줌

````
(가상환경명) pip install ipykernel
````



드디어 정상 작동한다....




#### 출처

* https://yahwang.github.io/posts/42