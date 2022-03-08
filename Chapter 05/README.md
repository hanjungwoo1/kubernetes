# Chapter 05. Pod  살펴보기

## 5.1 Pod 소개
Pod : 쿠버네티스의 최소 실행 단위
- 가상머신 : Instance
- 도커 : Container
- 쿠버네티스 : Pod

### 5.1.1 Pod 특징
#### 1개 이상의 컨테이너 실행
보통은 1개 Pod 내에 한 개 컨테이너 실행, 2~3(3개 이상 거의 없음)

#### 동일 노드에 할당
Pod 내에 실행되는 컨테이너들은 반드시 동일한 노드에 할당, 동일한 생명 주기

#### 고유의 Pod IP
Pod 리소스는 노드 IP와는 별개로 클러스터 내에서 접근 가능한 고유의 IP를 할당, 도커 컨테이너인 경우
다른 노드에 위치한 컨테이너간의 통신을 하기 위해서 일반적으로 포트 포워딩을 이용하여 노드 IP와 포워딩 포트를 이용하여 접근

쿠버네티스에서는 다른 노드에 위치한 Pod라 하더라도 NAT 통신 없이도 Pod 고유의 IP를 이용하여 접근 가능

#### IP 공유
Pod 내에 있는 컨테이너들은 서로 IP를 공유, Pod 내의 컨테이너끼리는 localhost를 통해 서로 네트워크 접근이 가능하며 포트를 이용하여 구분

#### Volume 공유
Pod 안의 컨테이너들은 동일한 볼륨과 연결이 가능하여 파일 시스템을 기반으로 서로 파일을 구조 받음


--dry run과 -o yaml 옵션을 조합하면, Pod를 실제로 생성하지 않고 템플릿 파일 생성
```bash
# mynginx.yaml 이라는 YAML 정의서 생성
kubectl run mynginx --image nginx --dry-run=client -o yaml > mynginx.yaml

vim mynginx.yaml
```
```yaml
# mynginx.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: mynginx
  name: mynginx
spec:
  containers:
  - image: nginx
    name: mynginx
  restartPolicy: Never
```
- apiVersion : 리소스의 이름이 동일할 경우, 충돌을 피하기 위한 목적
- kind : 리소스 타입을 정의
- metadata : 리소스의 메타 정보를 나타냄
    - labels : 리소스의 라벨정보를 표기
    - name : 리소스의 이름을 표기(mynginx)
- spec : 리소스의 스펙을 정의
    - container : 1개 이상의 컨테이너를 정의
        - name : 컨테이너의 이름을 표기
        - image : 컨테이너의 이미지 주소를 지정

YAML 파일을 이요하여 Pod 생성
```bash
kubectl apply -f mynginx.yaml
# pod/mynginx created
```
## 5.2 라벨링 시스템
특정 리소스를 참조하거나 Pod에 트래픽을 전달할 때도 라벨링 시스템을 활용

### 5.2.1 라벨 정보 부여
#### label 명령을 이용하는 방법
```bash
kubectl label pod <NAME> <KEY>=<VALUE>
```
```bash
kubectl label pod mynginx hello=world
# pod/mynginx labeled
```
```bash
kubectl get pod mynginx -oyaml
# apiVersion: v1
# kind: Pod
# metadata:
#   creationTimestamp: "2020-06-21T06:54:52Z"
#   labels:
#     hello: world
#     run: mynginx
#   ...
```

#### 선언형 명령을 이용하는 방법
Pod YAML 정의서를 작성할 때 metadata property에 직접 레벨을 추가하여 리소스를 생성
```bash
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  # hello=world 라벨 지정
  labels:
    hello: world  
    run: mynginx
  name: mynginx
spec:
  containers:
  - image: nginx
    name: mynginx
EOF
```
  
### 5.2.2 라벨 정보 확인
Pod에 부여된 라벨을 확인하기 위해서 -L 옵션을 사용(키 : run)
```bash
# 키 run에 대한 값 표시
kubectl get pod mynginx -L run
# NAME      READY   STATUS    RESTARTS   AGE   RUN
# mynginx   1/1     Running   0          15m   mynginx
```
특정 라벨이 아닌 전체 라벨 --show-labels
```bash
# 모든 라벨 정보 표시
kubectl get pod mynginx --show-labels
# NAME      READY   STATUS    RESTARTS   AGE   LABELS
# mynginx   1/1     Running   0          83s   hello=world,run=mynginx
```

### 5.2.3 라벨을 이용한 조건 필터링
특정 라벨을 가진 Pod만 필터해서 보기 원한다면 -l 옵션
```bash
# 새로운 yournginx Pod 생성
kubectl run yournginx --image nginx
# pod/yournginx created

# key가 run인 Pod들 출력
kubectl get pod -l run
# NAME       READY   STATUS    RESTARTS   AGE
# mynginx    1/1     Running   0          19m
# yournginx  1/1     Running   0          20m

# key가 run이고 value가 mynginx인 Pod 출력
kubectl get pod -l run=mynginx
# NAME      READY   STATUS    RESTARTS   AGE
# mynginx   1/1     Running   0          19m

# key가 run이고 value가 yournginx Pod 출력
kubectl get pod -l run=yournginx
# NAME       READY   STATUS    RESTARTS   AGE
# yournginx  1/1     Running   0          20m
```

### 5.2.4 nodeSelector를 이용한 노드 선택
라벨링 시스템을 이용하여 Pod가 특정 노드에 할당되도록 스케줄링

별도의 선택없이 Pod를 생성하면 쿠버네티스 마스터가 어떤 노드위에서 실행할지 판단하여 스케줄링

쿠버네티스는 클러스터링 시스템이어서, Pod 스케줄링을 관리

특정 노드를 명시적으로 선택해서 실행, nodeSelector라는 property를 이용하여 노드를 선택
```bash
kubectl get node  --show-labels
# NAME     STATUS   ROLES    AGE   VERSION        LABELS
# master   Ready    master   14d   v1.18.6+k3s1   beta.kubernetes.io/...
# worker   Ready    <none>   14d   v1.18.6+k3s1   beta.kubernetes.io/...
```
노드에 라벨을 부여, Pod와 마찬가지로 노드에 label 명령을 사용
```bash
kubectl label node master disktype=ssd
# node/master labeled

kubectl label node worker disktype=hdd
# node/worker labeled
```
노드의 라벨을 확인
```bash
# disktype 라벨 확인
kubectl get node --show-labels | grep disktype
# NAME     STATUS   ROLES    AGE   VERSION        LABELS
# master   Ready    master   14d   v1.18.6+k3s1   ....disktype=ssd,....
# worker   Ready    <none>   14d   v1.18.6+k3s1   ....disktype=hdd,....
```
실행하고자 하는 Pod의 YAML 정의서에 nodeSelector property를 추가
```yaml
# node-selector.yaml
apiVersion: v1
kind: Pod
metadata:
  name: node-selector
spec:
  containers: 
  - name: nginx
    image: nginx
  # 특정 노드 라벨 선택
  nodeSelector:
    disktype: ssd
```
- nodeSelector : 선택하고자하는 노드의 라벨을 지정합니다
```bash
kubectl apply -f node-selector.yaml
# pod/node-selector created- 
```
-o wide라는 옵션을 이용하면 해당 Pod가 어느 노드에서 실행되고 있는지 확인
```bash
kubectl get pod node-selector -o wide
# NAME           READY   STATUS    RESTARTS   AGE   IP          NODE   
# node-selector  1/1     Running   0          19s   10.42.0.8   master
```

이제, 동일한 node-selector.yaml파일의 nodeSelector를 변경
```yaml
# node-selector.yaml
apiVersion: v1
kind: Pod
metadata:
  name: node-selector
spec:
  containers: 
  - name: nginx
    image: nginx
  # 특정 노드 라벨 선택
  nodeSelector:
    disktype: hdd     # 기존 ssd에서 hdd로 변경
```
기존 disktype=ssd에서 disktype=hdd로 변경 후, Pod를 다시 생성
```bash
# 기존의 Pod 삭제
kubectl delete pod node-selector
# pod/node-selector deleted

# 새로 라벨링한 Pod 생성
kubectl apply -f node-selector.yaml
# pod/node-selector created

kubectl get pod node-selector -o wide
# NAME            READY   STATUS    RESTARTS   AGE   IP          NODE  
# node-selector   1/1     Running   0          19s   10.42.0.6   worker 
```

## 5.3 실행 명령 및 파라미터 지정
Pod 생성 시, 실행 명령과 파라미터를 전달할 수 있음
```yaml
# cmd.yaml
apiVersion: v1
kind: Pod
metadata:
  name: cmd
spec:
  restartPolicy: OnFailure
  containers: 
  - name: nginx
    image: nginx
    # 실행명령
    command: ["/bin/echo"]
    # 파라미터
    args: ["hello"]
```
- command : 컨테이너의 시작 실행 명령을 지정, 도커의 ENTRYPOINT에 대응되는 property
- args : 실행 명령에 넘겨줄 파라미터를 지정, 도커의 CMD에 대응되는 property
- restartPolicy : Pod의 재시작 정책을 설정
    - Always : Pod 종료 시 항상 재시작을 시도(default)
    - Never : 재시작을 시도하지 않음
    - OnFailure : 실패 시에만 재시작을 시도
```bash
kubectl apply -f cmd.yaml
# pod/cmd created

kubectl logs -f cmd
# hello 
```
## 5.4 환경변수 설정
env property를 활용하여 간단히 환경변수를 설정
```yaml
# env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: env
spec:
  containers: 
  - name: nginx
    image: nginx
    env:
    - name: hello
      value: "world!"
```
- env : 환경변수를 설정하는 property를 선언
    - name : 환경변수의 key를 지정
    - value : 환경변수의 value를 지정
Pod를 생성한 이후 exec 명령으로 실행 중인 env Pod에 printenv 명령을 전달
```bash
# env.yaml 파일을 이용하여 Pod 생성
kubectl apply -f env.yaml
#pod/env created

# 환경변수 hello 확인
kubectl exec env -- printenv | grep hello
# hello=world! 
```

## 5.5 볼륨 연결
예제에서는 가장 기본이 되는 host Volume을 사용

host Volume은 도커 -v 옵션과 유사하게 host 서버의 볼륨 공간에 Pod가 데이터를 저장하는 것을 말함

```yaml
# volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume
spec:
  containers: 
  - name: nginx
    image: nginx

    # 컨테이너 내부의 연결 위치 지정
    volumeMounts:
    - mountPath: /container-volume
      name: my-volume
  
  # host 서버의 연결 위치 지정
  volumes:
  - name: my-volume
    hostPath:
      path: /home
```
- volumeMounts : 컨테이너 내부에 사용될 볼륨을 선언
    - mountPath : 컨테이너 내부에 볼륨이 연결될 위치를 지정(/container-volume)
    - name : volumeMount와 volumes을 연결하는 식별자로 사용(my-volume)
- volumes : Pod에서 사용할 volume을 지정
    - name : volumeMounts와 volumes을 연결하는 식별자로 사용(my-volume)
    - hostPath : 호스트 서버의 연결 위치를 지정(/home)

Volume Pod를 생성하여 /container-volume 내부를 살펴보면 호스트 서버의 /home과 동일
```bash
kubectl apply -f volume.yaml
# pod/volume created

kubectl exec volume -- ls /container-volume
# ubuntu

ls /home
# ubuntu
```

Pod 내에서 임시로 생성하는 emptyDir property, 주로 컨테이너끼리 파일 데이터를 주고 받을 때 사용
```yaml
# volume-empty.yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-empty
spec:
  containers: 
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /container-volume
      name: my-volume
  volumes:
  - name: my-volume
    emptyDir: {}
```
- emptyDir : Pod의 생명주기를 따라가는 임시 volume. Pod 생성 시 같이 생성되고, Pod 삭제 시 같이 사라짐. 컨테이너 내부의 데이터와 별반 다르지 않지만, 2개 이상의 컨테이너가 서로 디렉터리 공간을 공유 가능

## 5.6 리소스 관리
### 5.6.1 requests
Pod가 보장받을 수 있는 최소 리소스 사용량을 정의
```yaml
# requests.yaml
apiVersion: v1
kind: Pod
metadata:
  name: requests
spec:
  containers: 
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "250m"
        memory: "500Mi"
```
- request : 최소 리소스 사용량 정의
    - cpu : CPU 리소스 최소 사용량 정의
    - memory : 메모리 리소스 최소 사용량 정의

cpu에서 1000m은 1core를 뜻함. 예시의 250m은 0.25core를 의미, memory의 MI는 1MiB를 의미
### 5.6.2 limits
Pod가 최대로 사용할 수 있는 최대 리소스 사용량을 정의
```yaml
# limits.yaml
apiVersion: v1
kind: Pod
metadata:
  name: limits
spec:
  restartPolicy: OnFailure
  containers: 
  - name: mynginx
    image: python:3.7
    command: [ "python" ]
    args: [ "-c", "arr = []\nwhile True: arr.append(range(1000))" ]
    resources:
      limits:
        cpu: "500m"
        memory: "1Gi"
```
- limits : 최대 리소스 사용량 제한
    - cpu : CPU 리소스 최대 사용량 정의
    - memory : 메모리 리소스 최대 사용량 정의

무한히 메모리를 소비하는 스크립트가 최대 메모리 리소스 사용량을 넘으면 강제로 프로세스가 중단
```bash
kubectl apply -f limits.yaml
# pod/limits created

watch kubectl get pod limits
# NAME    READY   STATUS     RESTARTS   AGE
# limits  1/1     Running    0          1m
# ...
# limits  0/1     OOMKilled  1          1m
```

최소 최대 2개 리소스 관리 기능을 조합하여 사용
```yaml
# resources.yaml
apiVersion: v1
kind: Pod
metadata:
  name: resources
spec:
  containers: 
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "250m"
        memory: "500Mi"
      limits:
        cpu: "500m"
        memory: "1Gi"
```

## 5.7 상태 확인
### 5.7.1 livenessProbe
쿠버네티스에는 컨테이너가 정상적으로 살아있는지 확인하기 위해 livenessProbe property를 이용

Pod가 정상적으로 동작하는지 확인하며, 자가치유를 위한 판단 기준으로 활용
```yaml
# liveness.yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness
spec:
  containers: 
  - name: nginx
    image: nginx
    livenessProbe:
      httpGet:
        path: /live
        port: 80
```
- livenessProbe : Pod가 정상적으로 동작하고 있는지 확인하는 property
    - httpGet: HTTP GET method를 이용하여 상태 확인을 수행
        - path : HTTP PATH를 지정
        - port : HTTP 포트를 지정

예제에서 Pod의 상태를 확인하는 방법으로 HTTP프로토콜의 GET Method를 이용하여 /live 위치의 80포트를 지속적 호출

리턴코드가 200~300번대이면 정상으로 판단, 그 외는 비정상으로 판단하여 컨테이너 종료되고 재시작
```bash
kubectl apply -f liveness.yaml
# pod/liveness created

# <CTRL> + <C>로 watch를 종료할 수 있습니다.
watch kubectl get pod liveness
# NAME       READY   STATUS    RESTARTS   AGE
# liveness   1/1     Running   2          71s

# 기본적으로 nginx에는 /live 라는 API가 없습니다.
kubectl logs -f liveness
# ...
# 10.42.0.1 - - [13/Aug/2020:12:31:24 +0000] "GET /live HTTP/1.1" 404 153 "-" "kube-probe/1.18" "-"
# 10.42.0.1 - - [13/Aug/2020:12:31:34 +0000] "GET /live HTTP/1.1" 404 153 "-" "kube-probe/1.18" "-"
```
```bash
kubectl exec liveness -- touch /usr/share/nginx/html/live

kubectl logs liveness
# 10.42.0.1 - - [14/Aug/2020:08:50:08 +0000] "GET /live HTTP/1.1" 404 153 "-" "kube-probe/1.18" "-"
# 10.42.0.1 - - [14/Aug/2020:08:50:18 +0000] "GET /live HTTP/1.1" 200 0 "-" "kube-probe/1.18" "-"

kubectl get pod liveness
# NAME       READY   STATUS      RESTARTS   AGE
# liveness   1/1     Running     2          12m
```

### 5.7.2 readinessProbe
readinessProbe은 Pod가 생성 직후, 트래픽을 받을 준비가 완료되었는지 확인하는 property(ex, 오래걸리는 웹서버)
```yaml
# readiness.yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness
spec:
  containers: 
  - name: nginx
    image: nginx
    readinessProbe:
      httpGet:
        path: /ready
        port: 80
```
- readinessProbe : Pod의 준비 완료를 확인하는 property
    - httpGet : HTTP GET method를 이용
        - path : HTTP PATH(/ready)를 지정
        - port : HTTP 포트를 지정

```bash
kubectl apply -f readiness.yaml
# pod/readiness created

kubectl logs -f readiness
# 10.42.0.1 - - [28/Jun/2020:13:31:28 +0000] "GET /ready HTTP/1.1" 
# 404 153 "-" "kube-probe/1.18" "-"

# 기본적으로 nginx에는 /ready 라는 API가 없습니다.
# /ready 호출에 404 에러가 반환되어 준비 상태가 완료되지 않습니다.
kubectl get pod
# NAME        READY   STATUS      RESTARTS   AGE
# readiness   0/1     Running     0          2m

# /ready URL 생성
kubectl exec readiness -- touch /usr/share/nginx/html/ready

# READY 1/1로 준비 완료 상태로 변경되었습니다.
kubectl get pod
# NAME        READY   STATUS      RESTARTS   AGE
# readiness   1/1     Running     0          2m
```

livenessProbe, readinessProbe에서 HTTP 통신뿐만 아니라, 명령 실행을 통해서도 정상 여부 확인
```yaml
# readiness-cmd.yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-cmd
spec:
  containers: 
  - name: nginx
    image: nginx
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/ready
```
- readinessProbe : Pod의 준비 상태를 확인
    - exec : 다음 명령을 실행
      - command : 사용자가 실행할 명령을 지정

명령의 리턴값이 0이면 정상, 이외는 비정상

예제에서 /tmp/ready 파일이 존재하는 경우, cat 명령 실행 시 정상적으로 0이 리턴되어 Ready

```bash
kubectl apply -f readiness-cmd.yaml
# pod/readiness-cmd created

# /tmp/ready 라는 파일이 존재하지 않기 때문에
# READY 0/1로 준비가 완료 되지 않음
kubectl get pod
# NAME            READY   STATUS      RESTARTS   AGE
# readiness-cmd   0/1     Running     0          2m

# /tmp/ready 파일 생성
kubectl exec readiness-cmd -- touch /tmp/ready

# READY 1/1로 준비 완료 상태로 변경
kubectl get pod
# NAME            READY   STATUS      RESTARTS   AGE
# readiness-cmd   1/1     Running     0          2m
```
## 5.8 2개 컨테이너 실행
하나의 Pod 내에 2개의 서로 다른 컨테이너를 실행
```yaml
# second.yaml
apiVersion: v1
kind: Pod
metadata:
  name: second
spec:
  containers: 
  - name: nginx
    image: nginx
  - name: curl
    image: curlimages/curl
    command: ["/bin/sh"]
    args: ["-c", "while true; do sleep 5; curl -s localhost; done"]
```
첫 번째 컨테이너 : 간단한 NGINX 웹 서버

두 번째 컨테이너 : 루프를 돌며 5초간 대기 후 localhost로 호출

예제처럼 리스트에 여러 컨테이너를 넣을 수 있음
```bash
kubectl apply -f second.yaml
# pod/second created

kubectl logs second
# error: a container name must be specified for pod second, choose one of: [nginx curl]
```

기존과는 다르게 logs 명령에서 에러 발생, 어떤 컨테이너 인지 정확히 명시해야함 -c 옵션

```bash
# nginx 컨테이너 지정
kubectl logs second -c nginx
# 127.0.0.1 - - [22/Jun/2020:13:37:00 +0000] "GET / HTTP/1.1" 200 
# 612 "-" "curl/7.70.0-DEV" "-"
# 127.0.0.1 - - [22/Jun/2020:13:37:09 +0000] "GET / HTTP/1.1" 200 
# 612 "-" "curl/7.70.0-DEV" "-"

# curl 컨테이너 지정
kubectl logs second -c curl
# ...
```

## 5.9 초기화 컨테이너
initContainers property를 이용하여 먼저 초기화 작업을 수행
```yaml
# init-container.yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-container
spec:
  restartPolicy: OnFailure
  containers: 
  - name: busybox
    image: k8s.gcr.io/busybox
    command: [ "ls" ]
    args: [ "/tmp/moby" ]
    volumeMounts:
    - name: workdir
      mountPath: /tmp
  initContainers:
  - name: git
    image: alpine/git
    command: ["sh"]
    args:
    - "-c"
    - "git clone https://github.com/moby/moby.git /tmp/moby"
    volumeMounts:
    - name: workdir
      mountPath: "/tmp"
  volumes:
  - name: workdir
    emptyDir: {}
```
- initContainers : 메인 컨테이너 실행에 앞서 초기화를 위해 먼저 실행되는 컨테이너를 정의

```yaml
kubectl apply -f init-container.yaml
# pod/init-container created

watch kubectl get pod
# NAME            READY   STATUS      RESTARTS   AGE
# init-container  0/1     Init:0/1    0          2s

# initContainer log 확인
kubectl logs init-container -c git -f
# Cloning into '/tmp/moby'...

# 메인 컨테이너 (busybox) log 확인
kubectl logs init-container
# AUTHORS
# CHANGELOG.md
# CONTRIBUTING.md
# Dockerfile
# Dockerfile.buildx
# ...
```

## 5.10 Config 설정
설정값들을 따로 모아둘 수 있는 통을 쿠버네티스에서는 ConfigMap
### 5.10.1 ConfigMap 리소스 생성
ConfigMap 리소스는 메타데이터(설정값)를 저장하는 리소스
```bash
kubectl create configmap <key> <data-source>
```
```
# game.properties
weapon=gun
health=3
potion=5
```
--from-file 옵션을 사용하여 game.properties 파일을 game-cofing라는 이름의 ConfigMap 생성
```bash
kubectl create configmap game-config --from-file=game.properties
# configmap/game-config created
```

```bash
kubectl get configmap game-config -o yaml  # 축약하여 cm
# apiVersion: v1
# data:
#   game.properties: |
#     weapon=gun
#     health=3
#     potion=5
# kind: ConfigMap
# metadata:
#   name: game-config
#   namespace: default
```
- apiVersion : v1 core API
- kind : ConfingMap 리소스
- data : 설정값들이 저장된 데이터

game-config라는 ConfigMap에 설정값들이 담긴 것을 확인

--from-literal 옵션을 사용하여, ConfigMap 생성, 사용자가 직접 설정값 지정
```bash
kubectl create configmap special-config \
            --from-literal=special.power=10 \
            --from-literal=special.strength=20
# configmap/special-config created
```
--from-literal 옵션 파라미터로 직접 설정값을 저장
```bash
kubectl get cm special-config -o yaml
# apiVersion: v1
# kind: ConfigMap
# metadata:
#   name: special-config
#   namespace: default
# data:
#   special.power: 10
#   special.strength: 20
```
또는 사용자가 직접 ConfingMap 리소스를 YAML 정의서로 작성하여 생성
```yaml
# monster-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: monster-config
  namespace: default
data:
  monsterType: fire
  monsterNum: "5"
  monsterLife: "3"
```
```bash
kubectl apply -f monster-config.yaml
# configmap/monster-config created

kubectl get cm monster-config -o yaml
# apiVersion: v1
# kind: ConfigMap
# metadata:
#   name: monster-config
#   namespace: default
# data:
#   monsterType: fire
#   monsterNum: "5"
#   monsterLife: "3"
```

### 5.10.2 ConfigMap 활용
#### 볼륨 연결
ConfingMap 리소스 활용 방법으로 ConfigMap을 볼륨으로 마운트하여 파일처럼 사용
```yaml
# game-volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: game-volume
spec:
  restartPolicy: OnFailure
  containers:
  - name: game-volume
    image: k8s.gcr.io/busybox
    command: [ "/bin/sh", "-c", "cat /etc/config/game.properties" ]
    volumeMounts:
    - name: game-volume
      mountPath: /etc/config
  volumes:
  - name: game-volume
    configMap:
      name: game-config
```
- volumes : Pod에서 사용할 볼륨을 선언
  - configMap : 기존의 hostPath, emptyDir 외에 configMap이라는 볼륨을 사용
    - name : 볼륨으로 사용할 ConfigMap 이름을 지정

```bash
kubectl apply -f game-volume.yaml
# pod/game-volume created

# 로그를 확인합니다.
kubectl logs game-volume
# weapon=gun
# health=3
# potion=5
```

#### 환경변수 - valueFrom
ConfigMap을 Pod의 환경변수로 사용
```yaml
# special-env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: special-env
spec:
  restartPolicy: OnFailure
  containers:
  - name: special-env
    image: k8s.gcr.io/busybox
    command: [ "printenv" ]
    args: [ "special_env" ]
    env:
    - name: special_env
      valueFrom:
        configMapKeyRef:
          name: special-config
          key: special.power
```
- env: 환경변수 사용을 선언
- name : 환경 변수 key 지정
- valueFrom : 기존의 value property대신  valueFrom을 사용, 다른 리소스 정보를 참조
  - configMapKeyRef : ConfigMap의 키를 참조
    - name : ConfigMap의 이름을 설정
    - key : ConfigMap내에 포함된 설정값 중 특정 설정값을 명시적으로 선택

sepcial-config 라는 ConfigMap 중 special.power라는 설정값을 환경 변수 special-env로 활용

```bash
kubectl apply -f special-env.yaml
# pod/special-env created

kubectl logs special-env
# 10
```

#### 환경변수 - envFrom
1개의 설정값이 아니라, ConfigMap에 포함된 모든 설정값을 환경변수로 사용하는 envFrom이라는 property
```yaml
# monster-env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: monster-env
spec:
  restartPolicy: OnFailure
  containers:
  - name: monster-env
    image: k8s.gcr.io/busybox
    command: [ "printenv" ]
    # env 대신에 envFrom 사용
    envFrom:
    - configMapRef:
        name: monster-config
```
- envFrom : 기존 env 대신 envFrom을 사용함으로써 ConfigMap 설정값을 환경변수 전체로 사용
  - configMapRef : ConfigMap의 특정 키(configMapKeyRef)가 아닌 전체 ConfigMap(configMapRef)를 사용
    - name : 사용하려는 configMap 이름을 저장
    
```bash
kubectl apply -f monster-env.yaml
# pod/monster-env created

kubectl logs monster-env | grep monster
# HOSTNAME=monster-env
# monsterLife=3
# monsterNum=5
# monsterType=fire
```
## 5.11 민감 데이터 관리
Secret 리소스는 각 노드에서 사용될 때 디스크에 저장되지 않고, tmpfs라는 메모리 기반 파일시스템을 사용해서 보안에 강함
### 5.11.1 Secret 리소스 생성
계정 이름과 비밀번호를 base64로 인코딩
```bash
echo -ne admin | base64
# YWRtaW4=

echo -ne password123 | base64
# cGFzc3dvcmQxMjM=
```
해당 값을 이용하여 다음과 같은 Secret 리소스 만듦
```bash
# user-info.yaml
apiVersion: v1
kind: Secret
metadata:
  name: user-info
type: Opaque
data:
  username: YWRtaW4=            # admin
  password: cGFzc3dvcmQxMjM=    # password123
```
- type : Secret 리소스의 타입을 설정. 기본적으로 Opaque를 사용하지만, 경우에 따라서 https://kubernetes.io/service-account-token,%20kubernetes.io/tls
등을 사용
- data : 저장할 민감 데이터를 입력

```bash
kubectl apply -f user-info.yaml
# secret/user-info created

kubectl get secret user-info -o yaml
# apiVersion: v1
# data:
#   password: cGFzc3dvcmQxMjM=
#   username: YWRtaW4=
# kind: Secret
# metadata:
#   ...
#   name: user-info
#   namespace: default
#   resourceVersion: "6386"
#   selfLink: /api/v1/namespaces/default/secrets/user-info
#   uid: bd6ca3a6-17f1-4690-aefe-6200e402aa35
# type: Opaque 
```

data의 필드값을 직접 base64 인코딩하지 않고 쿠버네티스가 대신 처리해주길 원한다면 stringData property 사용
```yaml
# user-info-stringdata.yaml
apiVersion: v1
kind: Secret
metadata:
  name: user-info-stringdata
type: Opaque
stringData:
  username: admin
  password: password123
```
```bash
kubectl apply -f user-info-stringdata.yaml
# secret/user-info created

kubectl get secret user-info-stringdata -o yaml
# apiVersion: v1
# data:
#   password: cGFzc3dvcmQxMjM=
#   username: YWRtaW4=
# kind: Secret
# metadata:
#   ...
#   name: user-info-stringdata
#   namespace: default
#   resourceVersion: "6386"
#   selfLink: /api/v1/namespaces/default/secrets/user-info
#   uid: bd61b459-a0d4-4158-aefe-6200f202aa35
# type: Opaque
```
명령형 커맨드를 이용해서 Secret 리소스를 생성
```bash
# user-info.properties
username=admin
password=password123
```
--from-env-file 옵션을 이용하여 properties 파일로부터 Secret을 생성
```bash
kubectl create secret generic user-info-from-file \
                    --from-env-file=user-info.properties
# secret/user-info-from-file created

kubectl get secret user-info-from-file -oyaml
# apiVersion: v1
# data:
#   password: cGFzc3dvcmQxMjM=
#   username: YWRtaW4=
# kind: Secret
# metadata:
#   creationTimestamp: "2020-07-07T12:03:14Z"
#   name: user-info-from-file
#   namespace: default
#   resourceVersion: "3647019"
#   selfLink: /api/v1/namespaces/default/secrets/user-info-from-file2
#   uid: 57bc52e1-4158-4f13-a0d4-0581b45950db
# type: Opaque
```
--from-env-file 뿐만 아니라, ConfigMap과 마찬가지로 --from-file, --from-literal 옵션을 지원
### 5.11.2 Secret 활용
#### 볼륨 연결
Secret도 ConfigMap과 동일하게 볼륨 연결이 가능
```bash
# secret-volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume
spec:
  restartPolicy: OnFailure
  containers:
  - name: secret-volume
    image: k8s.gcr.io/busybox
    command: [ "sh" ]
    args: ["-c", "ls /secret; cat /secret/username"]
    volumeMounts:
    - name: secret
      mountPath: "/secret"
  volumes:
  - name: secret
    secret:
      secretName: user-info
```
볼륨을 연결하여 해당 위치에 디렉터리를 확인
```bash
kubectl apply -f secret-volume.yaml
# pod/secret-volume created

kubectl logs secret-volume
# password
# username
# admin
```

#### 환경변수 - env
Secret 리소스도 환경변수로 정보를 추출, 개별적인 환경변수를 지정할때 valueFrom.secretKeyRef property
```yaml
# secret-env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env
spec:
  restartPolicy: OnFailure
  containers:
  - name: secret-env
    image: k8s.gcr.io/busybox
    command: [ "printenv" ]
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: user-info
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: user-info
          key: password
```
```bash
kubectl apply -f secret-env.yaml
# pod/secret-env created

kubectl logs secret-env
# PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
# HOSTNAME=secret-envfrom
# USERNAME=admin
# PASSWORD=1f2d1e2e67df
# ...
```

#### 환경변수 - envFrom
전체 환경변수를 부르고자 할 때 envFrom.secretRef property를 사용
```yaml
# secret-envfrom.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-envfrom
spec:
  restartPolicy: OnFailure
  containers:
  - name: secret-envfrom
    image: k8s.gcr.io/busybox
    command: [ "printenv" ]
    envFrom:
    - secretRef:
        name: user-info
```
```bash
kubectl apply -f secret-envfrom.yaml
# pod/secret-envfrom created

kubectl logs secret-envfrom
# PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
# HOSTNAME=secret-envfrom
# username=admin
# password=1f2d1e2e67df
# ...
```

## 5.12 메타데이터 전달
Downward API : Pod의 메타데이터를 컨테이너에게 전달할 수 있는 메커니즘
### 5.12.1 볼륨 연결
```yaml
: downward-volume
  labels:
    zone: ap-north-east
    cluster: cluster1
spec:
  restartPolicy: OnFailure
  containers:
  - name: downward
    image: k8s.gcr.io/busybox
    command: ["sh", "-c"]
    args: ["cat /etc/podinfo/labels"]
    volumeMounts:
    - name: podinfo
      mountPath: /etc/podinfo
  volumes:
  - name: podinfo
    downwardAPI:
      items:
      - path: "labels"
        fieldRef:
          fieldPath: metadata.labels
```
- downwardAPI : DownwarAPI 볼륨 사용을 선언
  - items : 메타데이터로 사용할 아이템 리스트를 지정
    - path : 볼륨과 연결된 컨테이너 내부 path를 지정
    - fieldRef : 참조할 필드를 선언
      - filedPath : Pod의 메타데이터 필드를 지정

```bash
kubectl apply -f downward-volume.yaml
# pod/downward-volume created

# Pod의 라벨 정보와 비교해 보시기 바랍니다.
kubectl logs downward-volume
# cluster="cluster1"
# zone="ap-north-east"
```
### 5.12.2 환경변수 - env
Downward API도 마찬가지로 환경변수로 선언
```yaml
# downward-env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: downward-env
spec:
  restartPolicy: OnFailure
  containers:
  - name: downward
    image: k8s.gcr.io/busybox
    command: [ "printenv"]
    env:
    - name: NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
    - name: POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
```
- name : 환경변수의 키 값을 지정
- valueFrom : 환경변수값을 선언
  - fieldRef : 참조할 필드를 선언
    - fieldPath : Pod의 메타데이터 필드를 지정

```bash
kubectl apply -f downward-env.yaml
# pod/downward-env created

kubectl logs downward-env
# POD_NAME=downward-env
# POD_NAMESPACE=default
# POD_IP=10.42.0.219
# NODE_NAME=master
# ...
```
fieldPath의 필드에 지정해놓은 Pod의 이름, 네임스페이스, 아이피 등을 환경변수로 확인

## 5.13 마치며

### Clean up
```bash
kubectl delete pod --all
```