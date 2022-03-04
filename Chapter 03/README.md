# Chapter 03. 쿠버네티스 설치

- k3s : IoT 디바이스에서 사용 가능한 경량 k8s 버전

- EKS : AWS에서 제공하는 k8s managed 서비스
- GKE : GCP에서 제공하는 k8s managed 서비스
- AKS : Azure에서 제공하는 k8s managed 서비스
- docker : 윈도우 및 mac OS용 도커 설치 시 쿠버네티스가 같이 설치
- minikube : hypervisor를 이용하여 단일 서버에서 클러스터 효과
- microk8s : canocical에서 제공하는 소형 k8s
- kubeadm : 온프레미스 서버에서 k8s를 구축할때 많이 사용하는 툴


## 3.1 k3s 소개
1. 설치가 쉽다
2. 가볍다
3. 대부분의 기능이 다 있다

## 3.2 k3s 설치하기

### 3.2.1 MobaXterm 설치

### 3.2.2 마스터 노드 설치
```bash
sudo apt update
sudo apt install -y docker.io nfs-common dnsutils curl

# k3s 마스터 설치
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="\
    --disable traefik \
    --disable metrics-server \
    --node-name master --docker" \
    INSTALL_K3S_VERSION="v1.18.6+k3s1" sh -s -

# 마스터 통신을 위한 설정
mkdir ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown -R $(id -u):$(id -g) ~/.kube
echo "export KUBECONFIG=~/.kube/config" >> ~/.bashrc
source ~/.bashrc

# 설치 확인
kubectl cluster-info
# Kubernetes master is running at https://127.0.0.1:6443
# CoreDNS is running at https://127.0.0.1:6443/api/v1/namespaces...
# 
# To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

kubectl get node -o wide
# NAME     STATUS   ROLES    AGE   VERSION        INTERNAL-IP    ...
# master   Ready    master   27m   v1.18.6+k3s1   10.0.1.1       ...
```

```bash
# 마스터 노드 토큰 확인
NODE_TOKEN=$(sudo cat /var/lib/rancher/k3s/server/node-token)
echo $NODE_TOKEN
# K10e6f5a983710a836b9ad21ca4a99fcxx::server:c8ae61726384c19726022879xx

MASTER_IP=$(kubectl get node master -ojsonpath="{.status.addresses[0].address}")
echo $MASTER_IP
# 10.0.1.1
```

### 3.2.3 워커 노드 추가

```bash
NODE_TOKEN=<마스터에서 확인한 토큰 입력>
MASTER_IP=<마스터에서 얻은 내부IP 입력>

sudo apt update
sudo apt install -y docker.io nfs-common curl

# k3s 워커 노드 설치
curl -sfL https://get.k3s.io | K3S_URL=https://$MASTER_IP:6443 \
    K3S_TOKEN=$NODE_TOKEN \
    INSTALL_K3S_EXEC="--node-name worker --docker" \
    INSTALL_K3S_VERSION="v1.18.6+k3s1" sh -s -
```

### 3.2.4 설치문제 해결 방법