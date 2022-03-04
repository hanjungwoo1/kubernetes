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
### 5.2.3 라벨을 이용한 조건 필터링
### 5.2.4 nodeSelector를 이용한 노드 선택

## 5.3 실행 명령 및 파라미터 지정

## 5.4 환경변수 설정

## 5.5 볼륨 연결

## 5.6 리소스 관리
### 5.6.1 requests
### 5.6.2 limits

## 5.7 상태 확인
### 5.7.1 livenessProbe
### 5.7.2 readinessProbe

## 5.8 2개 컨테이너 실행

## 5.9 초기화 컨테이너

## 5.10 Config 설정
### 5.10.1 ConfigMap 리소스 생성
### 5.10.2 ConfigMap 활용

## 5.11 민감 데이터 관리
### 5.11.1 Secret 리소스 생성
### 5.11.2 Secret 활용

## 5.12 메타데이터 전달
### 5.12.1 볼륨 연결
### 5.12.2 환경변수 - env

## 5.13 마치며