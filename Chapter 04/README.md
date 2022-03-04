# Chapter 04. 쿠버네티스 첫 만남

## 4.1 기본 명령
쿠버네티스는 여러 컨테이너를 관리해주는 컨테이너 오케스트레이션 플랫폼

기본적으로 도커 컨테이너와 마찬가지로 컨테이너의 실행과 삭제, 조회

### 4.1.1 컨테이너 실행
kubectl을 이용하여 첫 컨테이너 실행
```bash
kubectl run <NAME> --image <IMAGE>
```
mynginx라는 이름의 컨테이너를 nginx라는 이미지를 이용하여 생성
```bash
kubectl run mynginx --image nginx
# pod/mynginx created
```

도커 명령 비교 : 
```bash
docker run
```


### 4.1.2 컨테이너 조회
방금 실행한 컨테이너를 확인하기 위해 kubectl get명령
```bash
kubectl get pod
```

STATUS
 - Pending : 쿠버네티스 마스터에 생성 명령은 전달 되었지만 아직 실행되지 않음
 - ContainerCreating : 특정 노드에 스케줄링되어 컨테이너를 생성 중인 단계
 - Running : Pod가 정상적으로 실행되고 있는 상태
 - Completed : 계속 실행되고 있는 프로세스가 아닌 한 번 실행되고 완료되는 배치작업 Pod에서 작업이 완료
 - Error : Pod에 문제가 생겨 에러가 발생한 상태
 - CrashLoopBackOff : 지속적으로 에러 상태로 있어 crash가 반복되는 상태

특정 Pod의 상태 정보를 더 자세히 보고 싶다면 Pod의 이름과 함께 -o yaml 옵션
```bash
kubectl get pod mynginx -o yaml
# apiVersion: v1
# kind: Pod
# metadata:
#   creationTimestamp: "2020-06-21T06:54:52Z"
#   labels:
#     run: mynginx
#   managedFields:
#   - apiVersion: v1
#   ...
#   ...
#   name: mynginx
#   namespace: default
#   resourceVersion: "11160054"
#   ...
```

Pod의 IP를 확인하려면 -o wide 옵션
```bash
kubectl get pod -o wide
# NAME     READY   STATUS    RESTARTS   AGE  IP            NODE    ...
# mynginx  1/1     Running   0          6s   10.42.0.226   worker  ...
```
도커 명령 비교 :
```bash
docker ps
```

### 4.1.3  컨테이너 상세정보 확인
describe 명령도 get 명령과 유사하게 Pod 상태 정보, Events 기록까지
```bash
kubectl describe pod <NAME>
```

```bash
kubectl describe pod mynginx
# Name:         mynginx
# Namespace:    default
# Priority:     0
# Node:         worker/10.0.1.2
# Start Time:   Sun, 21 Jun 2020 06:54:52 +0000
# Labels:       run=mynginx
# Annotations:  <none>
# Status:       Running
# IP:           10.42.0.155
# IPs:
#   IP:  10.42.0.155
# ....
```

도커 명령 비교 : 
```bash
docker inspect
```

### 4.1.4 컨테이너 로깅
컨테이너의 로그 정보 확인, -f --follow 지속적 로그 출력
```bash
kubectl logs <NAME>
```

```bash
kubectl logs -f mynginx
# /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will ...
# /docker-entrypoint.sh: Looking for shell scripts in /docker-...
# ...
```

도커 명령 비교 : 
```bash
docker logs
```


### 4.1.5 컨테이너 명령 전달
실행 중인 컨테이너에 명령을 전달할 때 사용, 구분자(--)로 전달할 명령을 구분
```bash
kubectl exec <NAME> -- <CMD>
```

```bash
# mynginx에 apt-update 명령을 전달합니다.
kubectl exec mynginx -- apt-get update
# Get:1 http://security.debian.org/debian-security buster/updates ...
# Get:2 http://deb.debian.org/debian buster InRelease [121 kB]
# Get:3 http://deb.debian.org/debian buster-updates InRelease [51.9 kB]
# ...
```

도커와 마찬가지로 -it 옵션을 이용하여 컨테이너 내부 접속
```bash
kubectl exec -it mynginx -- bash

$root@mynginx:/# hostname
# mynginx

# 컨테이너 exit
$root@mynginx:/# exit
```

도커 명령 비교 : 
```bash
docker exec
```

### 4.1.6 컨테이너 / 호스트간 파일 복사
컨테이너에서 호스트로 또는 호스트에서 컨테이너로 파일을 복사할 때
```bash
kubectl cp <TARGET> <SOURCE>
```
```bash
# <HOST> --> <CONTAINER>
kubectl cp /etc/passwd mynginx:/tmp/passwd

# exec 명령으로 복사가 되었는지 확인합니다.
kubectl exec mynginx -- ls /tmp/passwd
# /tmp/passwd

kubectl exec mynginx -- cat /tmp/passwd
# root:x:0:0:root:/root:/bin/bash
# daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
# bin:x:2:2:bin:/bin:/usr/sbin/nologin
# ...
```
도커 명령 비교 :
```bash
docker cp
```

### 4.1.7 컨테이너 정보 수정
실행된 컨테이너 정보를 수정, edit 명령을 실행하면 vim과 같은 텍스트 에디터가 열림
```bash
kubectl edit pod <NAME>
```

```bash
kubectl edit pod mynginx
# apiVersion: v1
# kind: Pod
# metadata:
#   creationTimestamp: "..."
#   labels:
#     hello: world        # <-- 라벨 추가
#     run: mynginx
#   managedFields:
#   ...
```

```bash
# mynginx 상세 정보 조회
kubectl get pod mynginx -oyaml
# apiVersion: v1
# kind: Pod
# metadata:
#   creationTimestamp: "..."
#   labels:
#     hello: world        # <-- 추가된 라벨 학인
#     run: mynginx
#   ...
```

도커 명령 비교 : 

```bash
docker update
```

### 4.1.8 컨테이너 삭제
생성한 컨테이너를 삭제하기 위해 delete 명령
```bash
kubectl deleted pod <NAME>
```
```bash
kubectl delete pod mynginx
# pod mynginx deleted

kubectl get pod
# No resources found ..
```
도커 명령 비교 : 
```bash
docker rm
```

### 4.1.9 선언형 명령 정의서(YAML) 기반의 컨테이너 생성
쿠버네티스는 선언형 명령을 지향
```bash
kubectl apply -f <FILE_NAME>
```
```yaml
# mynginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mynginx
spec:
  containers: 
  - name: mynginx
    image: nginx
```
```bash
ls
# mynginx.yaml

kubectl apply -f mynginx.yaml
# pod/mynginx created

kubectl get pod 
# NAME      READY   STATUS      RESTARTS   AGE
# mynginx   1/1     Running     1          6s

kubectl get pod mynginx -oyaml
# apiVersion: v1
# kind: Pod
# metadata:
#   ...
#   name: mynginx
#   ...
# spec:
#   containers:
#   - image: nginx
#     name: mynginx
#   ...
```
apply 명령의 장점은 로컬 파일시스템 YAML 정의서 뿐만 아니라, 인터넷 상에 위치한 YAML 사용 가능
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/pods/simple-pod.yaml
# pod/nginx created

kubectl delete -f https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/pods/simple-pod.yaml
# pod/nginx deleted
```
yaml을 수정하여 적용하면 새로운 컨테이너가 아닌, 기존 컨테이너 설정 값이 수정
```yaml
# mynginx.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    hello: world          # <-- 라벨 추가
  name: mynginx
spec:
  containers: 
  - name: mynginx
    image: nginx:1.17.2   # <-- 기존 latest에서 1.17.2로 변경
```
```bash
kubectl apply -f mynginx.yaml
# pod/mynginx configured

kubectl get pod 
# NAME      READY   STATUS      RESTARTS   AGE
# mynginx   1/1     Running     1          3m48s


kubectl get pod mynginx -oyaml
# apiVersion: v1
# kind: Pod
# metadata:
#   labels:
#     hello: world
#   ...
#   name: mynginx
#   ...
# spec:
#   containers:
#   - image: nginx:1.17.2
#     name: mynginx
#   ...


kubectl apply -f mynginx.yaml
# pod/mynginx unchanged

kubectl get pod 
# NAME      READY   STATUS      RESTARTS   AGE
# mynginx   1/1     Running     1          3m48s

kubectl get pod mynginx -oyaml
# apiVersion: v1
# kind: Pod
# metadata:
#   labels:
#     hello: world
#   ...
#   name: mynginx
#   ...
# spec:
#   containers:
#   - image: nginx:1.17.2
#     name: mynginx
#   ...
```
## 4.2 고급 명령

### 4.2.1 리소스별 명령
쿠버네티스는 모든 것이 리소스로 표현

쿠버네티스는 Pod 리소스 외에 Service, ReplicaSet, Deployment 등 다양한 리소스를 포함

Pod라고 적은 부분을 다른 리소스로 변경하여 명령 실행

```bash
kubectl get service
# NAME         TYPE       CLUSTER-IP   EXTERNAL-IP   PORT(S)  AGE
# kubernetes   ClusterIP  10.43.0.1    <none>        443/TCP  13d

# describe 명령도 동일하게 작동합니다.
kubectl describe service kubernetes
# Name:              kubernetes
# Namespace:         default
# Labels:            component=apiserver
#                    provider=kubernetes
#  ...
```

쿠버네티스 클러스터를 구축하기 위해 사용한 Node 또한 쿠버네티스의 리소스 중 하나로 표현

```bash
kubectl get node
# NAME     STATUS   ROLES    AGE   VERSION
# master   Ready    master   13d   v1.18.6+k3s1
# worker   Ready             13d   v1.18.6+k3s1

kubectl describe node master
# Name:               master
# Roles:              master
# Labels:             beta.kubernetes.io/arch=amd64
#                     beta.kubernetes.io/instance-type=k3s
# ...
```

### 4.2.2 네임스페이스(Namespace)

쿠버네티스 클러스터를 논리적으로 나누는 역할. Pod, Service와 같은 리소스가 네임스페이스별로 생성 되고 사용자 접근제어, Network 접근제어 정책을 다르게 함

```bash
kubectl get namespace
# NAME             STATUS   AGE
# default          Active   12m
# kube-system      Active   12m
# kube-public      Active   12m
# kube-node-lease  Active   12m 

kubectl describe namespace kube-system
# Name:         kube-system
# Labels:       <none>
# Annotations:  <none>
# ...
```
- default : 기본 네임스페이스, 아무런 옵션 없이 컨테이너 만들 시 
- kube-system : 쿠버네티스의 핵심 컴포넌트들이 들어있음. 네트워크 설정, DNS 서버 등 중요한 역할을 담당
- kube-public : 외부로 공개 가능한 리소스를 담고 있는 네임스페이스
- kube-node-lease : 노드가 살아있는지 마스터에 알리는 용도로 존재하는 네임 스페이스

명령을 실행할 때, --namespace 옵션(-n)을 이용하여 특정 네임스페이스에 리소스를 생성
```bash
kubectl run mynginx-ns --image nginx --namespace kube-system
# pod/mynginx-ns created

# kube-system 네임스페이스에서 Pod 확인
kubectl get pod mynginx-ns -n kube-system  # --namespace를 -n로 축약 가능
# NAME        READY   STATUS    RESTARTS   AGE
# mynginx-ns  1/1     Running   0          13s

kubectl delete pod mynginx-ns -n kube-system
# pod/mynginx-ns deleted
```


### 4.2.3 자동완성 기능

kubectl 명령을 매번 일일이 입력하는것이 귀찮을 때, 자동완성 스크립트를 제공

https://kubernetes.io/docs/tasks/tools/#enabling-shell-autocompletion

```bash
echo 'source <(kubectl completion bash)' >> ~/.bashrc
source ~/.bashrc
```

```bash
kubectl run yournginx --image nginx
# pod/yournginx created

kubectl get pod <TAB>
# mynginx     yournginx
```

### 4.2.4 즉석 리소스 생성

cat & here document 명령 조합을 활용하여 즉석으로 리소스를 생성

```bash
# 즉석에서 YALM 문서를 생성하여 쿠버네티스에게 전달
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: cat-nginx
spec:
  containers:
  - image: nginx
    name: cat-nginx
EOF
# pod/cat-nginx created
```

### 4.2.5 리소스 특정 정보 추출

--jsonpath라는 옵션을 이용하여 리소스의 특정 정보만을 골라서 추출
```bash
kubectl get node master -o yaml
# apiVersion: v1
# kind: Node
# metadata:
#   annotations:
#   ...
# spec:
#   podCIDR: 10.42.0.0/24
#   podCIDRs:
#   - 10.42.0.0/24
#   providerID: k3s://master
# status:
#   addresses:
#   - address: 10.0.1.1
#     type: InternalIP
#   - address: master
#     type: Hostname
```
```bash
kubectl get node master -o wide
# NAME     STATUS   ROLES    AGE   VERSION        INTERNAL-IP     ...
# master   Ready    master   27m   v1.18.6+k3s1   10.0.1.1        ...
```

```bash
kubectl get node master -o jsonpath="{.status.addresses[0].address}"
# 10.0.1.1
```

https://kubernetes.io/docs/reference/kubectl/jsonpath

### 4.2.6 모든 리소스 조회
Pod 리소스 외에 쿠버네티스에 다양한 리소스를 확인
```bash
kubectl api-resources
# NAME        SHORTNAMES  APIGROUP  NAMESPACED   KIND
# namespaces  ns                    false        Namespace
# nodes       no                    false        Node
# pods        po                    true         Pod
# ...
```
- 쿠버네티스 리소스는 크게 네임스페이스 레벨 / 클러스터 레벨 리소스로 구분
- 네임스페이스 레벨의 리소스 : 해당 리소스가 반드시 특정 네임스페이스에 속해야 하는 리소스
- 클러스터 레벨의 리소스 : 네임스페이스와 상관없이 클러스터 레벨에 존재하는 리소스
- 네임스페이스 레벨의 대표적인 리소스 Pod, 클러스터 레벨의 대표적인 리소스 Node

네임스페이스 레벨의 API 리소스만 탐색하기 위한 명령어
```bash
kubectl api-resources --namespaced=true 
# NAME          SHORTNAMES   APIGROUP  NAMESPACED   KIND
# bindings                             true         Binding
# configmaps    cm                     true         ConfigMap
# endpoints     ep                     true         Endpoints
# events        ev                     true         Event
```

### 4.2.7 리소스의 정의 설명
리소스의 간단한 정의를 살펴보려면 다음과 같은 명령
```bash
# Pod에 대한 정의를 확인할 수 있습니다.
kubectl explain pods
# KIND:     Pod
# VERSION:  v1
# 
# DESCRIPTION:
#      Pod is a collection of containers that can run on a host. 
#      This resource is created by clients and scheduled onto hosts.
# 
# FIELDS:
#    apiVersion   <string>
#      APIVersion defines the versioned schema of this representation 
# ....
```

### 4.2.8 클러스터 상태 확인
전반적인 클러스터의 health check을 확인
```bash
# 쿠버네티스 API서버 작동 여부 확인
kubectl cluster-info

# 전체 노드 상태 확인
kubectl get node

# 쿠버네티스 핵심 컴포넌트의 Pod 상태 확인
kubectl get pod -n kube-system
```


### 4.2.9 클라이언트 설정 파일
kubectl 툴은 내부적으로 KUBECONFIG ($HOME/.kube/config)설정 파일을 참조하여 마스터 주소, 인증 정보를 관리
```bash
kubectl confing <SUMCOMMAND>
```

```bash
kubectl config view
# apiVersion: v1
# clusters:
# - cluster:
#     certificate-authority-data: DATA+OMITTED
#     server: https://127.0.0.1:6443
#   name: default
# ...
```

KUBECONFIG 설정 파일을 직접 출력
```bash
cat $HOME/.kube/config
# apiVersion: v1
# clusters:
# - cluster:
#     certificate-authority-data: ....
#     server: https://127.0.0.1:6443
#   name: default
# contexts:
# - context:
#     cluster: default
#     user: default
#   name: default
# current-context: default
# kind: Config
# preferences: {}
# users:
# - name: default
#   user:
#     password: ...
#     username: admin
```
- clusters : kubectl 툴이 바라보는 클러스터 정보를 입력, 원격 주소지를 입력
- users : 클러스터에 접속하는 사용자를 정의, Chapter 13 접근 제어
- contexts : cluster와 user를 연결해주는 것을 context


### 4.2.10 kubectl 명령 치트 시트

https://kubernetes.io/docs/reference/kubectl/cheatsheet/


### Clean up
```bash
kubectl delete pod --all
```