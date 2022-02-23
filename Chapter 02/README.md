# Chapter 01. 쿠버네티스 소개

## 2.1 쿠버네티스란?
쿠버네티스(kubernetes, k8s) : 여러 서버로 구성된 클러스터 환경에서 컨테이너화된 프로세스를 관리하기 위한 컨테이너 오케스트레이션(Orchestration) 플랫폼
 - 컨테이너는 가상 먼신과는 다르게 호스트 운영체제를 공유
 - 가상머신에 비해 훨씬 더 가볍지만 가상 머신과 마찬가지로 환경을 독립적
 - 가상화된 실행 환경을 가지는 컨테이너의 특징 덕분에 컨테이너를 쉽게 복제하거나 배포

### 2.1.1 컨테이너 오케스트레이션이란?
컨테이너 오케스트레이션 : 다수의 서버 위에서 컨테이너의 전반적인 라이프 사이클을 관리해주는 플랫폼

쿠버네티스 : 대표적인 컨테이너 오케스트레이션 플랫폼

쿠버네티스는 컨테이너의
- 실행 및 배포를 책임
- 이중화와 가용성을 보장
- 수평확장 및 축소를 관리
- 스케줄링을 담당
- 네트워크 설정을 관리
- health 상태를 모니터링
- 설정값을 관리

### 2.1.2 데이터 센터 운영체제
쿠버네티스 : 데이터 센터 운영체제 / 클러스터 운영체제

데이터센터 (클러스터)
- 여러 컴퓨터의 집합체
- 리소스 컴퓨팅 자원의 군집

운영 체제
- 하드웨어 수상화
- 프로세스 스케줄링
- 컴퓨팅 자원 관리
- 사융자 인터페이스(UI)

## 2.2 쿠버네티스 기본 개념
쿠버네티스는 일반적인 운영체제와 같이 운영체제로서 지원하는 기능들을 비슷하게 제공

사용자가 개별 노드(서버)를 직접 제어하지 않고 쿠버네티스라는 추상화된 레이어를 통해 클러스터를 제어할 수 있는 하드웨어 추상화 기능을 제공하고 컨테이너의 스케줄링, 자원할당 관리 등의 기능을 제공

쿠버네티스에서는 "kubectl" 이라는 유저 인터페이스를 이용하여 쿠버네티스를 제어

### 2.2.1 애완동물 vs 가축
서버를 애완동물이 아닌 가축
- 세심한 관리보단 방목하여 키움
- 정해진 이름보단 무리로 관리
- 죽는 것에 괜찮음

1. 서버마다 특별한 이름을 부여하지 않음
2. 한두 개의 서버가 망가져도 문제 없음

### 2.2.2 바라는 상태(Desired State)
"바라는 상태"라는 개념

쿠버네티스가 바라는 상태와 동일해지도록 사전에 미리 정의된 특정 작업을 수행

### 2.2.3 컨트롤러(Controller)
현재 상태를 "바라는 상태"로 변경하는 주체를 '컨트롤러(Controller)'

컨트롤러는 control-loop라는 루프를 돌며 특정 리소스를 지속적으로 모니터링하다가, 사용자가 생성한 리소스의 이벤트에 따라 사전에 정의된 작업을 수행

Job라는 리소스를 사용자가 생성하면(바라는 상태) 그에 맞는 배치 작업 프로세스를 실행(특정 작업 수행)
## 2.3 아키텍처

## 2.4 장점

## 2.5 마치며




## 1.1 도커 소개
도커(Docker) : 가상실행 환경을 제공해주는 오픈소스 플랫폼 (컨테이너, Container) 
- 어디서든 실행할 수 있는 장점
- 온프레미스와 클라우드 서버간의 간극 줄이는데 사용
- 표준화(Standard) : 프로세스의 실행을 표준화
- 이식성(Portability) : 동일한 실행 환경으로 프로세스를 작동
- 가볍다(Lightweight) : 애플리케이션별로 커널을 공유해, 다른 제품에 비해 가벼움
- 강한 보안 : 컨테이너라는 고립된 환경에서 실행되므로 보안 측면 유리

### 1.1.1 컨테이너와 가상머신
가상머신(VM) : 하이퍼바이저 설치, 그 위에 가상 OS와 APP을 패키징한 VM을 만들어 실행하는 방식인 '하드웨어 레벨의 가상화'

컨테이너 : 운영체제를 제외한 나머지 애플리케이션 실행에 필요한 모든 파일을 패키징, 'OS 레벨 가상화'

### 1.1.2 CD 플레이어, 도커
도커는 마치 CD 플레이어와 같다.

도커 이미지(Image) : 사용자가 실행할 코드가 들어있는 바이너리, CD와 마찬가지로 생성하면 수정 불가능

도커 파일(Dockerfile) : 도커 이미지를 생성하기 위해 필요한 문서

도커 컨테이너(Container) : 도커 이미지가 메모리 위에 상주하여 실제 코드가 수행되는 프로세스, 종료 시 휘발

도커 데몬(Docker Daemon) : 컨테이너가 정상적으로 수행될 수 있게 실행 환경 제공

원격 레지스트리(Registry) : 도커 이미지를 저장할 수 있는 원격 저장소

### 1.1.3 도커 설치
```bash
#apt docker 설치
sudo apt update && sudo apt install -y docker.io net-tools

#계정에 sudo 권한 주기
sudo usermod -aG docker $USER

# 서버를 재시작
sudo reboot
```

## 1.2 도커 기본 명령
도커 컨테이너를 하나 실행한다는 것은 일반적인 프로세스 한 개를 실행하는 것과 마찬가지

### 1.2.1 컨테이너 실행
```bash
docker run <IMAGE>:<TAG> [<args>]
```
일반적인 리눅스에서 cowsay 실행
```bash
sudo apt install -y cowsay
cowsay hello world!
#  ______________
# < hello world! >
#  --------------
#         \   ^__^
#          \  (oo)\_______
#             (__)\       )\/\
#                 ||----w |
#                 ||     ||
# 
```
도커 컨테이너 버전의 cowsay
```bash
docker run docker/whalesay cowsay 'hello world!'
# Unable to find image 'docker/whalesay:latest' locally
# latest: Pulling from docker/whalesay
# 23cwc732thk4: Pull complete
# t4nb4f93jc42: Pull complete
# ...
#  ______________
# < hello world! >
#  --------------
#     \
#      \
#       \
#                     ##        .
#               ## ## ##       ==
#            ## ## ## ##      ===
#        /""""""""""""""""___/ ===
#   ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
#        \______ o          __/
#         \    \        __/
#           \____\______/
```
- 기본 레지스트리 주소 : docker.io
- 기본 사용 TAG : latest

```bash
<레지스트리 이름> / <이미지 이름> : <TAG>
docker/whalesay == docker.io/docker/whalesay:latest
```
docker/whalesay 이미지에 다른 파라미터를 전달
```bash
docker run docker/whalesay echo hello
# hello
```
-d 옵션으로 컨테이너를 백그라운드 실행, 사용할 도커 이미지는 (nginx)
```bash
# '-d' {.bash} 옵션 추가
docker run -d nginx
# Unable to find image 'nginx:latest' locally
# latest: Pulling from library/nginx
# 5e6ec7f28fb7: Pull complete
# ab804f9bbcbe: Pull complete
# ...
# d7455a395f1a4745974b0be1372ea58c1499f52f97d96b48f92eb8fa765bc69f
```
nginx 이미지의 전체 주소
```bash
nginx == docker.io/nginx:latest
```

### 1.2.2 컨테이너 조회
```bash
docker ps
# 실행한 컨테이너에 대해서 조회
```

### 1.2.3 컨테이너 상세 정보 확인
```bash
# docker ps를 통해 얻은 <CONTAINER_ID>로 상세 정보 확인
docker inspect d7455a395f1a
# [
#   {
#     "Id": "d7455a395f1a0f562af7a0878397953331b791c229a31b7eba0",
#     "Created": "2020-07-12T05:26:23.554118194Z",
#     "Path": "/",
#     "Args": [
#     ],
#     "State": {
#       "Status": "running",
#       "Running": true,
#       "Paused": false,
#       ...
```

### 1.2.4 컨테이너 로깅
컨테이너에서 출력되는 로그 기록
```bash
docker logs <CONTAINER_ID>
```
백그라운드로 실행된 컨테이너의 로그를 직접 확인. -f는 follow output 옵션
```bash
docker logs -f d7455a395f1a
# /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, ...
# /docker-entrypoint.sh: Looking for shell scripts in ...
# ...
```
CTR> + C 로 로깅을 종료

### 1.2.5 
간혹 실행된 컨테이너에 새로운 패키지를 설치하거나 설정을 수정해야 하는 경우
```bash
docker exec <CONTAINER_ID> <CMD>
```
exec 명령으로 nginx 컨테이너에 wget을 설치하고 localhost로 요청
```bash
# 새로운 패키지 설치
docker exec d7455a395f1a sh -c 'apt update && apt install -y wget'
docker exec d7455a395f1a wget localhost
# ..
# Saving to: 'index.html'
#      0K .......... ..           166M=0s
# 2020-07-16 15:15:43 (166 MB/s) - 'index.html' saved [13134]
```
### 1.2.6 컨테이너 / 호스트간 파일 복사
실행한 컨테이너와 호스트서버간에 파일을 주고 받을 수 있음
```bash
docker cp <HOST_PATH> <CONTAINER_ID> : <CONTAINER_PATH>
```
호스트 서버의 /ect/password 파일을 컨테이너 내부의 /usr/share/nginx/html/ 위치로 복사
