---
layout: index

title: 컨테이너 환경에서의 AI 서빙엔진 구축기(2)

nav_order: 2

parent: Tech_Study

permalink: /Tech_Study2

---


작성일자: 2023/05/17



#  컨테이너 환경에서의 AI 서빙엔진 구축기(1)



이번 시간에는 TA 서비스에서 AI 모델 운영을 위한 워크플로우, 환경 구축, 그리고 서버 내부 아키텍처를 알아보겠습니다.



## CMS를 통한 AI 모델 관리

TA 서비스의 주요 기능인 "VOC 유형 예측"을 수행하기 위해서 데이터 구축부터 AI 모델 서빙까지의 워크플로우가 진행됩니다. 이런 ML 워크플로우는 학습 데이터가 누적됨에 따라 반복적으로 수행됩니다. 이러한 과정을 시스템화하기 위해 TA CMS(Content Management System) 내에 AI 모델 관리 화면을 구성했습니다. CMS를 통한 AI 모델 관리 워크플로우는 아래와 같이 진행됩니다.



1.개통 처리

포탈에서 개통 신청을 받으면 TA DB에 업체의 파티션 테이블을 생성하고, 메타정보(회사 코드, 상담센터 코드, 상담그룹 정보, 상담사 id 정보)와 함께 업체의 VOC 유형 코드를 전달받습니다.


2.학습 데이터 저장

ELK에 저장된 통화별 상담원문과 VOC 유형 정답 데이터를 불러와 txt 파일로 가공합니다. 가공된 파일은 NAS(Network Attached Storage)의 지정된 디렉토리에 저장됩니다.


3.모델 학습

적재된 학습 데이터(txt파일) 중 일부를 선택하여 모델을 학습합니다. 이 때 데이터 분포가 높은 유형에 대한 언더샘플링과 검증 데이터 비율을 조정할 수 있습니다. 학습이 완료되어야 모델에 대한 검증 진행이 가능합니다.


4.모델 검증

검증 데이터셋으로 검증을 실행해 모델의 precision 값을 반환합니다.



5.서빙 모델 등록/해지

검증 후 관리자의 기준에 따라 모델을 서빙 영역에 등록할 지 결정합니다. 기존에 업체에 대한 서빙이 진행중이었다면 모델 해지를 먼저 실행합니다. 이후 모델 등록을 실행하면 TA 서비스에서 AI를 통한 VOC 유형 분류가 가능합니다.



## ML 서비스 환경 구축



### GPU 환경 구축



자연어 AI 모델의 학습과 추론을 위해 GPU 사용은 필수입니다. AI 프레임워크(tensorflow, keras, pytorch)에서 gpu를 사용하기 위해 우선 Cuda 환경과 Nvidia 드라이버를 설치합니다. 

**Cuda**는 c++로 개발되었으며 병렬 컴퓨팅과 gpu 관련 프로그래밍을 할 수 있는 플랫폼입니다. Cuda를 설치할 때는, Cuda와 호환 가능한 Nvidia Driver 버전을 확인 후 함께 설치해야합니다. Cuda 버전을 설치하기 전에 사용하려는 AI 프레임워크가 Cuda 버전과 호환이 가능한지도 확인이 필요합니다.

현재 개발 서버에서는 아래와 같이 환경 세팅을 진행했습니다.

```
GPU 사양: NVIDIA TESLA A100 40GB
CUDA Version: 11.4
Driver Version: 470.82.01
```

Cuda 및 드라이버 설치를 위한 여러 방법 중 가장 심플한 방법은 Cuda Toolkit을 사용하는 방법입니다.

Cuda 11.4 Toolkit은 Nvidia 공식 홈페이지에서 다운 가능합니다.

<img src="/docs/Indb_Study/image/gpu1.png" width="2100" height="1000">

https://developer.nvidia.com/cuda-11-4-0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=CentOS&target_version=7&target_type=rpm_local

서버에 접속하여 wget을 통해 rpm 파일을 받은 후 설치를 진행합니다.

```
wget https://developer.download.nvidia.com/compute/cuda/11.4.0/local_installers/cuda-repo-rhel7-11-4-local-11.4.0_470.42.01-1.x86_64.rpm
sudo rpm -i cuda-repo-rhel7-11-4-local-11.4.0_470.42.01-1.x86_64.rpm
sudo yum clean all
sudo yum -y install nvidia-driver-latest-dkms cuda
sudo yum -y install cuda-drivers
```

설치경로를 따로 지정하지 않으면 cuda는 `/usr/local/` 경로에 설치됩니다. Cuda 환경을 사용하기 위해선`{사용자명}/.bash_profile`에 

```
PATH=$PATH:/usr/local/cuda-11.4/lib64
LD_LIBRARY_PATH=/usr/local/cuda-11.4/lib64
export PATH
export LD_LIBRARY_PATH
```

와 같이 환경변수를 지정해주어야 합니다.



이후 bash shell에서 `nvidia-smi` 명령어를 통해 driver와 cuda의 설치 여부를 확인할 수 있습니다.

 <img src="/docs/Indb_Study/image/gpu2.png" width="1600" height="1000">

아래 예시 사진을 보면 현재 장착된 gpu의 사용량과 해당 gpu를 사용중인 Process, driver version과 Cuda Version 등을 확인할 수 있습니다.



### Docker 환경 구축


1.도커란
Docker는 프로세스 격리 기술들을 사용해 컨테이너로 실행하고 관리하는 오픈 소스 프로젝트입니다. 기존의 가상화 기술과 다르게 도커는 경량화된 OS 환경 위에서 어플리케이션을 빠르게 배포할 수 있으며 이식성이 높은 장점을 가집니다.

2.도커 의존성 패키지 다운로드
기본적인 네트워크 환경에선 Centos7 환경에서는 간단한 명령어로 도커 및 의존성이 있는 하위 라이브러리를 설치 가능합니다.

`yum install docker=={version}`

하지만 보안성 목적으로 네트워크가 차단된 환경에선 모든 의존성 패키지를 개별적으로 받아야 합니다. 이 작업을 수월하게 하기 위해 yum 패키지 매니저에서 yum-utils를 설치합니다. yum-utils 패키지를 통해 docker의 모든 의존성 패키지를 rpm 파일로 한번에 받을 수 있습니다.

`yum install yum-utils`

우선 yum repository에 도커 repository를 추가합니다.

`yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`

이후 아래 명령어를 통해 docker 패키지를 rpm 파일로 다운받습니다.

`yumdownloader --downloadonly --resolve --destdir={저장경로}`



3.nvidia-docker 관련 패키지 다운로드

docker 환경에서 gpu 사용을 하려면 nvidia 관련 패키지를 추가로 설치해주어야 합니다.

우선 패키지 다운을 위해 nvidia 리포지토리를 구성합니다.

`curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | \ tee /etc/yum.repos.d/nvidia-docker.repo`

nvidia-docker2 관련 의존성 패키지를 다운 받습니다. 사실 nvidia-docker2는 docker 19.03 이후 버전에 대해서는 필요하지 않지만, 의존성 패키지인 nvidia runtime 등은 여전히 필요합니다.

`yumdownloader --downloadonly --resolve --downloaddir=/app/nvidia-docker nvidia-docker2`

명령어 실행을 하면 아래와 같이 4가지 rpm 파일을 받을 수 있습니다. 

```
libnvida-container~.rpm
libnvidia-container-tools~.rpm
nvidia-container-toolkit~.rpm
nvidia-docker2-~.rpm(docker 버전에 따라 필수 여부 결정됨)
```

4.폐쇄망 환경에서의 도커 설치

위에서 받은 모든 파일을 폐쇄망 서버로 이전하여 rpm 설치 작업을 진행합니다. 설치는 아래와 같이 rpm cli 명령어로 진행합니다. -i 옵션을 사용하면 의존성이 있는 패키지 여러개를 한번에 설치할 수 있습니다.

`rpm -i {패키지1} {패키지2} ...`


이제 도커 설치가 완료되었습니다.

5.docker와 gpu 환경 연결

만약 docker version이 19.03 이하라면 도커에서 nvidia-container-runtime을 사용할 수 있게 하기 위해 도커 설정 파일에 아래 값을 추가해줘야 합니다.

```json
# /etc/docker/daemon.json

{"runtimes":{
    "nvidia": {
        "path":"nvidia-container-runtime"
    }
}}
```

해당 값을 입력한 후 docker restart를 진행해주어야 docker cli 명령어 옵션을 통해 사용하고자 하는 gpu를 지정할 수 있습니다.

예를 들어 도커를 실행할 때 `docker run --runtime=nvidia --gpus=0` 과 같은 지정을 할 수 있게 됩니다.

하지만 docker 19.03 이상의 버전부터는 libnvidia-container-toolkit만 깔려있어도 docker CLI에서 "--gpus" 명령어를 사용할 수 있기 때문에 위의 작업이 불필요합니다.



위의 작업을 반영한 후 systemctl 혹은 service를 통해 docker를 실행합니다.

`sudo systemctl start docker` or `sudo service start docker`

6.도커 기동 확인

`docker ps -a` 는 도커 위에 떠있는 모든 컨테이너를 조회하기 위한 명령어입니다. 해당 명령어가 잘 작동하면 도커 환경의 구축이 끝난 걸 알 수 있습니다.


7.컨테이너와 이미지의 개념

도커 환경이 구축 되었으면 딥러닝 어플리케이션을 담은 컨테이너를 띄워야 합니다. 컨테이너를 띄우기 위해선 도커 이미지가 필요합니다.

**이미지**는 간단하게 하나의 소프트웨어 프로그램을 가동하기 위한 경량화된 실행환경이라 보시면 됩니다. 예를 들어 linux 환경에서 실행중인 파이썬 프로세스가 있다고 가정해보면, 해당 프로세스는 python3.7 binary 파일, 그리고 그 파일을 실행하기 위한 ENV 설정파일, pip 패키지, 그리고 여러가지 python 라이브러리 파일들이 존재할 것입니다. 현재 시점의 OS 실행환경을 하나의 사진처럼 캡쳐해놓은 것을 **이미지**로 생각하면 이해가 편합니다.

**컨테이너**는 **이미지**를 통해 실행한 인스턴스입니다. 컨테이너는 **이미지**라는 경량화된 OS 환경 위에서 하나 이상의 프로세스를 띄우고 있는 인스턴스입니다. 컨테이너는 **이미지**만 있다면 언제든지 정지, 삭제, 실행이 가능하기 때문에 도커가 깔린 어떤 OS 위에서도 환경에 대한 제약없이 재현 가능합니다.


이미지는 Docker Community에서 만든 클라우드 기반의 이미지 저장소 **DockerHub**에서 받을 수 있습니다. 이미지가 어플리케이션 환경을 구성하기에 부족하다면(대부분의 상황에서 부족합니다..) Dockerfile을 통해 해당 이미지(Base Image) 위에 여러 리눅스 명령어를 추가하여 새로운 이미지를 생성할 수 있습니다.

8.도커 이미지 로드

도커 리포지토리와 네트워크 연결이 안되어 있을 시 외부에서 이미지를 생성한 후 tar 파일로 받아오는 방법도 있습니다. 제가 작업한 폐쇄망 환경에서는 tar 파일을 이미지로 로드하는 방법을 사용했습니다.

`docker load -i {파일경로/파일명.tar}`

이미지 파일을 로드한 후 Docker CLI 명령어로 간단하게 컨테이너를 실행할 수 있습니다.

9.컨테이너 기동

`docker run -d -rm --init --gpus-=all --names={컨테이너명} -e LC_ALL=C.UTF-8 -e WORKSPACE={WORKSPACE} -v /app/data/ta-engine:/data/ta-engine/scripts/start.sh`

도커 컨테이너를 기동할 때는 `docker run` 명령어를 사용합니다. 도커 컨테이너는 그 자체가 하나의 격리된 프로세스이기 때문에 바이너리 파일 혹은 실행 파일을 마지막에 입력하여 합니다. 이 외에도 컨테이너명, 환경변수, gpu 사용값, 데몬 프로세스 여부 등 여러가지 설정값으로 컨테이너를 제어할 수 있습니다.

10.컨테이너 기동여부 확인

이후 `docker ps -a`를 통해 현재 컨테이너 상태를 조회하면 컨테이너가 정상적으로 올라온 것을 확인할 수 있습니다.



## 딥러닝 서빙 아키텍처



다음은 기동된 도커 컨테이너 내부의 서빙 아키텍처를 알아보겠습니다. 

컨테이너 내부에는 외부로부터 데이터를 받아 AI 결과값을 반환하는 API 서버가 있습니다. 파이썬에서는 일반적으로 API 서버 구축을 위해 Flask, FastAPI와 같은 웹프레임워크를 사용합니다. 하지만 딥러닝을 활용하는 서비스에서 해당 웹프레임워크 만으로 통신을 주고 받으면 GPU 자원을 효율적으로 쓰기 어렵고, 이로 인해 API 성능 또한 현저하게 떨어집니다. 이를 해결하기 위해 현재 진행중인 서비스에서는 아래와 같이 3가지 계층으로 서빙 아키텍처를 구축하였습니다.

 <img src="/docs/Indb_Study/image/gpu3.png" width="1830" height="1000">

TorchServe는 pytorch로 생성한 모델을 프로덕션 환경에서 쉽게 배포하고 서빙하기 위한 프레임워크입니다. Torchserve는 API를 통해 **모델 관리**와 **추론**(Inference) 기능을 지원합니다. **모델 관리**란 복수의 모델을 gpu에 등록, 해지하는 작업, 그리고 각 모델당 워커의 개수(GPU 수)를 할당하는 등의 조정을 할 수 있습니다. **추론**이란 AI가 입력값을 받아 예측값을 반환하는 작업을 의미합니다.

<img src="/docs/Indb_Study/image/gpu5.png" width="1540" height="1000">

Torchserve를 기동하면 내부적으로 2개의 포트(모델 관리, 추론)를 활용하여 클라이언트와 통신 합니다. 이 때 통신방법은 rest api 혹은 grpc 중 하나를 선택할 수 있으며 세부적인 인터페이스는 공식 홈페이지에서 참고 가능합니다.

https://pytorch.org/serve/management_api.html#register-a-model

이 프레임워크의 가장 큰 이점은 미리 명시된 API를 통해 GPU 자원을 효율적으로 관리하고 모니터링 할 수 있다는 점입니다. 일반적인 웹프레임워크에서 pytorch 모델로 추론을 할 때는 모델을 호출하는 부분에서는 배포시 저장되어 있는 모델만을 활용할 수 있습니다. 또한 추론 API를 요청할 때 gpu 사용 시점에서 모델이 GPU 메모리에 올라갔다 내려갔다를 반복합니다. 하지만 torchserve  서비스 환경에서 모델의 생성과 해지를 배포없이 무중단으로 진행할 수 있으며, 여러 모델이 GPU 메모리에 올라간 상태에서 프레임워크의 룰에 따라 자원을 효율적으로 사용합니다.

<img src="/docs/Indb_Study/image/gpu4.png" width="1540" height="1000">

GRPC는 구글에서 언어에 제약없이 빠른 통신을 하기 위해 만든 통신 프레임워크입니다. json 파일을 주고 받는 Rest API와 다르게 GRPC는 IDL을 통해 메시지의 형식을 미리 정해두고 이를 Protocol Buffer라는 방식으로 직렬화합니다. 이 때 직렬화란 전송하고자 하는 데이터의 형태를 정해진 포맷으로 변경하는 것을 의미합니다. Protocol Buffer는 데이터를 바이너리 파일로 압축하여 직렬화하는 과정에서 데이터 사이즈를 축소하기 때문에 텍스트, 이미지와 같은 대용량 데이터 전송에서 성능의 이점을 갖게 됩니다. 현재 TA 서비스에서는 10분 이상의 대화내역을 input으로 받는 경우가 있기 때문에 GRPC를 통해 torchserve와 통신하는 방법을 적용하였습니다.



위와 같은 딥러닝 아키텍처를 적용하면서 프로덕션 환경에서 AI 모델을 서빙하기 위해 다양한 고려사항이 있다는 걸 파악할 수 있었습니다. 올해 적용할 모델에서는 Torchserve 대신 Nvidia에서 제공하는 Triton server와 Fast-transformer 프레임워크를 도입중입니다. 다음 시간에는 Triton server와 Torchserver의 차이, 젠킨스를 통한 CI/CD 환경에 대해 적어보겠습니다.











