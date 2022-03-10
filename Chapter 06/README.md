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

### 6.1.4 Service 도메인 주소 법칙
### 6.1.5 클러스터 DNS 서버








## 6.2 Service 종류
### 6.2.1 ClusterIP
### 6.2.2 NodePort
### 6.2.3 LoadBalancer
### 6.2.4 ExternalName

## 6.3 네트워크 모델

## 6.4 마치며