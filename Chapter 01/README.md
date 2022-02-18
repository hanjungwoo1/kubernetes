#Chapter 01. 도커 기초
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

```bash
# 호스트에서 컨테이너로 파일 복사
docker cp /etc/passwd d7455a395f1a:/usr/share/nginx/html/.

# 확인
docker exec d7455a395f1a curl -s localhost/passwd
# root:x:0:0:root:/root:/bin/bash
# daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
# bin:x:2:2:bin:/bin:/usr/sbin/nologin
# ...

# 반대로 컨테이너에서 호스트로 파일 복사
docker cp d7455a395f1a:/usr/share/nginx/html/index.html .

# 확인
cat index.html
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...
```

### 1.2.7 컨테이너 중단
사용이 완료된 컨테이너를 중단
```bash
docker stop <CONTAINER_ID>
```
다음 명령을 통해 백그라운드로 실행된 nginx 컨테이너를 중단
```bash
docker stop d7455a395f1a
# d7455a395f1a

docker ps
# CONTAINER ID    IMAGE   COMMAND   CREATED   STATUS   PORTS    NAMES

# 종료된 것까지 확인 가능
docker ps -a
# CONTAINER ID   IMAGE  COMMAND       ...  STATUS       PORTS   NAMES
# d7455a395f1a   nginx  "/docker..."  ...  Exited (0)           ...
# ...
```

### 1.2.8 컨테이너 재개
종료된 컨테이너 다시 재시작
```bash
docker start <CONTAINER_ID>
```
start 명령으로 중단된 컨테이너 재개
```bash
docker start d7455a395f1a
# d7455a395f1a

docker ps
# CONTAINER ID    IMAGE    COMMAND                  CREATED         ...
# d7455a395f1a    nginx    "/docker-entrypoint.."   4 minutes ago   ...
```

### 1.2.9 컨테이너 삭제
중단된 컨테이너를 완전히 삭제
```bash
docker rm <CONTAINER_ID>
```
컨테이너 삭제 후에는 더는 컨테이너 재개 불가능
```bash
# 컨테이너 중단
docker stop d7455a395f1a
# d7455a395f1a

# 컨테이너 삭제
docker rm d7455a395f1a
# d7455a395f1a

# 컨테이너 조회, nginx가 사라졌습니다.
docker ps -a
# CONTAINER ID  IMAGE            COMMAND       ...  STATUS      ...
# fc47859c2fdc  docker/whalesay  "echo hello"  ...  Exited (0)  ...
# 32047a546124  docker/whalesay  "cowsay ..."  ...  Exited (0)  ...
```

### 1.2.10 Interactive 컨테이너
이미지를 실행 시, -it 옵션을 통해 직접 컨테이너 안으로 접속하여 작업 가능

-it는 interactive (stdin, stdout 연결), tty(터미널 연결)의 약자
```bash
docker run -it ubuntu:16.04 bash
# Unable to find image 'ubuntu:16.04' locally

# 컨테이너 내부
$root@1c23d59f4289:/#

$root@1c23d59f4289:/# cat /etc/os-release
# NAME="Ubuntu"
# VERSION="16.04.6 LTS (Xenial Xerus)"
# ID=ubuntu
# ID_LIKE=debian

# 컨테이너에서 빠져 나오기
$root@1c23d59f4289:/# exit
```
이미 생성한 컨테이너 에도 접속이 가능, exec -it 명령을 이용
```bash
# 컨테이너 실행
docker run -d nginx
# c4484sc501e9a

# exec 명령을 통해서 bash 접속
docker exec -it c4484c501e9a bash

# 컨테이너 내부
$root@c4484c501e9a:/#
```

- 컨테이너는 휘발성 프로세스, 컨테이너 내부의 파일시스템에 저장하였더라도 삭제 시 모든 데이터가 사라짐
- 패키시를 설치하더라도 컨테이너를 종료하면 설치된 패키지가 없어짐

## 1.3 도커 저장소
도커 허브(Docker Hub)는 도커 이미지 원격 저장소

사용자들은 도커 허브에 이미지를 업로드하고, 다른 곳에서 자유롭게 재사용 가능

### 1.3.1 도커 허브 계정 만들기
도커 허브(https://hub.docker.com/)에 접속하여 계정을 생성

### 1.3.2 이미지 tag 달기
tag 명령을 이용하여 기존의 이미지에서 새로운 이름을 부여

nginx 이미지에 새로운 이름 USERNAME/nginx
```bash
docker tag <OLD_NAME>:<TAG> <NEW_NAME>:<TAG>
```

nginx 이미지에 새로운 이름 (USERNAME/nginx:1)를 부여
```bash
docker tag nginx:latest <USERNAME>/nginx:1
```

tag를 부여할 때 TAG를 생략할 수 있음
```bash
docker tag nginx <UNSERNAME>/nginx
```

생략 시, 기본 tag인 latest이 입력

### 1.3.3 이미지 확인

```bash
docker images
```

원격 저장소로 부터 다운로드 받아 로컬 디스크에 저장된 이미지 리스트를 확인

지금까지 사용하고 생성한 이미지 리스트 확인
```bash
docker images
# REPOSITORY         TAG        IMAGE ID        CREATED          SIZE
# nginx              latest     89b0e8f41524    2 minutes ago    150MB
# hongkunyoo/nginx   latest     89b0e8f41524    2 minutes ago    150MB
# hongkunyoo/nginx   1          89b0e8f41524    2 minutes ago    150MB
# ubuntu             16.04      330ae480cb85    8 days ago       125MB
# docker/whalesay    latest     6b362a9f73eb    5 years ago      247MB
```

### 1.3.4 도커 허브 로그인
이미지 업로드를 위해 도커허브에 로그인
```bash
docker login
# Username: <USERNAME>
# Password: ****
# WARNING! Your password will be stored unencrypted in ...
# Configure a credential helper to remove this warning. See
# https://docs.docker.com/engine/reference/commandline/login/#credentials-store

# Login Succeeded
```

### 1.3.5 이미지 업로드
```bash
docker push <USERNAME>/<NAME>
```
이미지를 업로드, 앞에서 생성한 USERNAME/nginx 이미지를 업로드
```bash
docker push <USERNAME>/nginx
# The push refers to repository [docker.io/hongkunyoo/nginx]
# f978b9ed3f26: Preparing
# 9040af41bb66: Preparing
```

### 1.3.6 이미지 다운로드
```bash
docker pull <IMAGE_NAME>
```
반대로 다음과 같이 이미지를 다운로드

예제에서는 redis 이미지를 다운로드

run 명령을 사용하여 자동으로 이미지를 받을 수 있지만, pull 명령을 통해서 명시적으로 다운로드 가능

```bash
docker pull redis
# Using default tag: latest
# latest: Pulling from library/redis
# ...
# 85a6a5c53ff0: Pull complete

docker images
# REPOSITORY         TAG         IMAGE ID        CREATED          SIZE
# hongkunyoo/nginx   latest      89b0e8f41524    2 minutes ago    150MB
# hongkunyoo/nginx   1           89b0e8f41524    2 minutes ago    150MB
# nginx              latest      89b0e8f41524    2 minutes ago    150MB
# redis              latest      235592615444    3 weeks ago      104MB
# ubuntu             16.04      330ae480cb85    8 days ago       125MB
# docker/whalesay    latest      6b362a9f73eb    5 years ago      247MB
```
### 1.3.7 이미지 삭제
```bash
docker rim <IMAGE_NAME>
```
로컬 서버에 존재하는 이미지를 삭제, (docker images 명령으로 조회되는 이미지들)
```bash
docker rmi redis
# Untagged: redis:latest
# Untagged: redis@sha256:800f2587bf3376cb01e6307afe599ddce9439deafbd...
# Deleted: sha256:2355926154447ec75b25666ff5df14d1ab54f8bb4abf731be2fcb818c7a7f145

docker images
# REPOSITORY         TAG       IMAGE ID         CREATED          SIZE
# nginx              latest    89b0e8f41524     2 minutes ago    150MB
# hongkunyoo/nginx   latest    89b0e8f41524     2 minutes ago    150MB
# hongkunyoo/nginx   1         89b0e8f41524     2 minutes ago    150MB
# ubuntu             16.04     330ae480cb85     8 days ago       125MB
# docker/whalesay    latest    6b362a9f73eb     5 years ago      247MB
```

## 1.4 도커 파일 작성
도커 이미지를 생성하기 위해서는 Dockerfile 이라는 텍스트 문서를 작성

사용자는 Dockerfile에 특정 명령을 기술하여 원하는 도커 이미지 생성

특정명령

- Dockerfile에 기반 이미지(base Image)를 지정
- 원하는 소프트웨어 및 라이브러리를 설치하기 위한 명령을 기술
- 컨테이너 실행 시 수행할 명령을 기술하는 것





## 1.5 도커 실행 고급