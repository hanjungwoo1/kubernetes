# Chapter 06. 쿠버네티스 네트워킹

## 6.1 Service 소개
쿠버네티스에는 Pod 자체에도 IP가 부여
```bash
kubectl run mynginx --image nginx
# pod/mynginx created

# Pod IP는 사용자마다 다릅니다.
kubectl get pod -owide
# NAME     READY   STATUS     RESTARTS   AGE   IP           NODE    ...
# mynginx  1/1     Running    0          12d   10.42.0.26   master  ...

kubectl exec mynginx -- curl -s 10.42.0.26
# <html>
# <head>
# <title>Welcome to nginx!</title>
# <style>
#     body {
# ...
```
### 6.1.1 불안정한 Pod vs 안정적인 Service
불안정한 Pod의 생명주기와는 상관없이 안정적인 서비스 끝점을 제공하는 Service

Service 리소스는 Pod의 앞단에 위치, 리버스 프록시와 같은 역할(트래픽 전달)을 수행

### 6.1.2 서비스 탐색(Service Discovery)
Sevice 리소스는 안정적인 IP 제공, 서비스 탐색 기능을 수행하는 도메인 이름 기반의 서비스 끝점을 제공

쿠버네티스 클러스터 내에서 Service 리소스의 이름을 기반으로 DNS 참조가 가능
### 6.1.3 Service 첫 만남
```yaml
# myservice.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    hello: world
  name: myservice
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 80
  selector:
    run: mynginx
```
- apiVersion : Pod와 마찬가지로 v1 API 버전
- kind : Service 리소스를 선언
- metadata : 리소스의 메타 정보를 나타냄
  - labels : Service에도 라벨을 부여
  - name : 이름을 지정, 해당 이름이 도메인 주소로 활용
- spec : Service의 스펙을 정의
  - ports : Service의 포트들을 정의
    - port : Service로 들어오는 포트를 지정
    - protocol : 사용하는 프로토콜을 지정. TCP, UDP, HTTP 등
    - targerPort : 트래픽을 전달할 컨테이너의 포트를 지정
  - selector : 트래픽을 전달할 컨테이너의 라벨을 선택. run=mynginx 라벨을 가진 Pod에 Service 트래픽을 전달

#### 라벨 셀렉터를 이용하여 Pod을 선택하는 이유
라벨링 시스템 : 느슨한 연결 관계(loosely coupled)로 표현

Service에서 바라보는 특정 라벨을 가지고 있는 어떠한 Pod에도 트래픽을 전달
```bash
# 앞에서 살펴 본 Service 리소스를 생성합니다.
kubectl apply -f myservice.yaml
# service/myservice created

# 생성된 Service 리소스를 조회합니다.
# Service IP (CLUSTER-IP)를 확인합니다. (예제에서는 10.43.152.73)
kubectl get service  # 축약 시, svc
# NAME          TYPE        CLUSTER-IP     EXTERNAL-IP  PORT(S)    AGE
# kubernetes    ClusterIP   10.43.0.1      <none>       443/TCP    24d
# myservice     ClusterIP   10.43.152.73   <none>       8080/TCP   6s

# Pod IP를 확인합니다. (예제에서는 10.42.0.226)
kubectl get pod -owide
# NAME     READY  STATUS    RESTARTS   AGE   IP            NODE   ...
# mynginx  1/1    Running   0          6s    10.42.0.226   master ...
```
get 명령을 이용하여 리소스를 나열

myservice라는 이름을 가진 service 있음
```bash
# 트래픽 전달
# curl 요청할 client Pod 생성
kubectl run client --image nginx
# pod/client created

# Pod IP로 접근
kubectl exec client -- curl -s 10.42.0.226

# Service IP로 접근 (CLUSTER-IP)
kubectl exec client -- curl -s 10.43.152.73:8080

# Service 이름 (DNS 주소)로 접근
kubectl exec client -- curl -s myservice:8080
```
- Pod IP, Service IP, Service 이름으로 요청하면 전부 동일하게 mynginx 서버의 결과가 반환
- Service 끝점을 요청하는 경우, Service에서 사용하는 포트(port property), 8080을 사용
myservice의 IP주소를 확인하기 위해  DNS lookup을 수행
```bash
# DNS lookup을 수행하기 위해 nslookup 명령을 설치합니다.
kubectl exec client -- sh -c "apt update && apt install -y dnsutils"
# Hit:1 http://deb.debian.org/debian buster InRelease
# Hit:2 http://security.debian.org/debian-security buster/updates 
# Reading package lists...
# ...

# myservice의 DNS를 조회합니다.
kubectl exec client -- nslookup myservice
# Server:         10.43.0.10
# Address:        10.43.0.10#53
# 
# Name:   myservice.default.svc.cluster.local
# Address: 10.43.152.73
```
Service의 이름이 도메인 주소 역할을 수행하는 것

### 6.1.4 Service 도메인 주소 법칙
Service 리소스의 전체 도메인 이름 법칙
```bash
<서비스이름>.<네임스페이스>.svc.cluster.local
```
- <서비스 이름>.<네임 스페이스> == <서비스 이름>.<네임스페이스>.svc.cluster.local
- <서비스 이름> == <서비스 이름>.<네임스페이스>.svc.cluster.local
```bash
# Service의 전체 도메인 주소를 조회합니다.
kubectl exec client -- nslookup myservice.default.svc.cluster.local
# Server:         10.43.0.10
# Address:        10.43.0.10#53
# 
# Name:   myservice.default.svc.cluster.local
# Address: 10.43.152.73

# Service의 도메인 주소 일부를 생략하여 조회합니다.
kubectl exec client -- nslookup myservice.default
# Server:         10.43.0.10
# Address:        10.43.0.10#53
# 
# Name:   myservice.default.svc.cluster.local
# Address: 10.43.152.73

# Service 이름만 사용해도 참조가 가능합니다.
kubectl exec client -- nslookup myservice
# Server:         10.43.0.10
# Address:        10.43.0.10#53
# 
# Name:   myservice.default.svc.cluster.local
# Address: 10.43.152.73 
```

### 6.1.5 클러스터 DNS 서버

Service 이름은 도메인 주소로 사용이 가능한 이유는 쿠버네티스에서 제공하는 DNS 서버가 있기 때문
```bash
# 로컬 호스트 서버의 DNS 설정이 아닌 Pod의 DNS 설정을 확인합니다.
kubectl exec client -- cat /etc/resolv.conf
# nameserver 10.43.0.10
# search default.svc.cluster.local svc.cluster.local cluster.local ap-northeast-2.compute.internal
# options ndots:5
``` 
nameserver에 10.43.0.10라는 IP를 확인. 쿠버네티스의 모든 Pod들은 바로 이 IP로 DNS 조회

```bash
kubectl get svc -n kube-system
# NAME      TYPE       CLUSTER-IP   EXTERNAL-IP  PORT(S) 
# kube-dns  ClusterIP  10.43.0.10   <none>       53/UDP,53/TCP,9153/TCP 
```
kube-dns라는 Service가 10.42.0.10의 주인, --show-labels 옵션으로 어떤 라벨을 가지고 있는지 확인
```bash
kubectl get svc kube-dns -nkube-system --show-labels
# NAME       TYPE         CLUSTER-IP  ...    AGE   LABELS
# kube-dns   ClusterIP    10.43.0.10  ...    46h   k8s-app=kube-dns,...
```
k8s-app=kube-dns 라벨 정보를 이용하여 매핑되는 Pod가 어떤 것이 있는지 확인
```bash
kubectl get pod -n kube-system -l k8s-app=kube-dns
# NAME              READY   STATUS    RESTARTS   AGE
# coredns-6c6bb68   1/1     Running   0          46h
```

## 6.2 Service 종류
### 6.2.1 ClusterIP
가장 기본이 되는 타입, 내부에서만 접근 가능, 외부에서 접근 불가능
- 네트워크 보안
- 기본 빌딩블록으로 사용

직접 ClusterIP 타입의 Service를 생성, Pod template 파일을 생성할 때, --expose, --port 80 옵션을 추가
```bash
kubectl run cluster-ip --image nginx --expose --port 80 \
    --dry-run=client -o yaml > cluster-ip.yaml

vim cluster-ip.yaml
```
```yaml
# cluster-ip.yaml
apiVersion: v1
kind: Service
metadata:
  name: cluster-ip
spec:
  # type: ClusterIP  # 생략되어 있음
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 80
  selector:
    run: cluster-ip
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: cluster-ip
  name: cluster-ip
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
```
Pod와 Service를 생성하고 Service의 타입을 조회
```bash
ubectl apply -f cluster-ip.yaml
# service/cluster-ip created
# pod/cluster-ip created

kubectl get svc cluster-ip -oyaml | grep type
#  type: ClusterIP

kubectl exec client -- curl -s cluster-ip
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...
```
###6.2.2 NodePort
로컬 호스트의 특정 포트를 Service의 특정 포트와 연결시켜 외부 트래픽을 Service까지 전달

```yaml
# node-port.yaml
apiVersion: v1
kind: Service
metadata:
  name: node-port
spec:
  type: NodePort     # type 추가
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 80
    nodePort: 30080  # 호스트(노드)의 포트 지정
  selector:
    run: node-port
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: node-port
  name: node-port
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
```
- type : 타입을 NodePort로 지정
- nodeport : 호스트 서버에서 사용할 포트 번호를 정의, 쿠버네티스에서 제공하는 30000~32767

```bash
kubectl apply -f node-port.yaml
# service/node-port created
# pod/node-port created

kubectl get svc
# NAME         TYPE        CLUSTER-IP     EXTERNAL-IP  PORT(S)         AGE
# kubernetes   ClusterIP   10.43.0.1      <none>       443/TCP         26d
# myservice    ClusterIP   10.43.152.73   <none>       8080/TCP        2d
# cluster-ip   ClusterIP   10.43.9.166    <none>       8080/TCP        42h
# node-port    NodePort    10.43.94.27    <none>       8080:30080/TCP  42h
```
NoderPort는 단지 Pod가 위치한 노드 뿐만 아니라, 모든 노드에서 동일하게 서비스 끝점을 제공

예를 들어, 방금 생성한 node-port Pod가 마스터 노드에 위치해있어도 마스터 노드, 워커 노드 모두 동일한 NodePort로 서비스에 접근

```bash
# 트래픽을 전달 받을 Pod가 마스터 노드에 위치합니다.
kubectl get pod node-port -owide
# NAME         READY   STATUS    RESTARTS   AGE   IP           NODE   
# node-port    1/1     Running   0          14m   10.42.0.28   master

MASTER_IP=$(kubectl get node master -ojsonpath="{.status.addresses[0].address}")
curl $MASTER_IP:30080
# <!DOCTYPE html>
# <html>
# <head>
# ...

WORKER_IP=$(kubectl get node worker -ojsonpath="{.status.addresses[0].address}")
curl $WORKER_IP:30080
# <!DOCTYPE html>
# <html>
# <head>
# ...
```
외부에서 접근 가능한 공인 IP가 있다면 공인 IP로도 연결 테스트
```bash
curl <공인IP>:30080

# 웹 브라우저에서 <공인IP>:30080으로도 확인할 수 있습니다.
```

### 6.2.3 LoadBalancer
### 6.2.4 ExternalName

## 6.3 네트워크 모델

## 6.4 마치며