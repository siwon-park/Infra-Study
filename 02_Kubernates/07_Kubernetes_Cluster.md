![kubernets](https://user-images.githubusercontent.com/93081720/174333422-4e2f7a03-f585-4edf-884c-0af7fea7ac5d.png)



# 07_쿠버네티스 클러스터 구축하기

실제 서버를 여러 대 사용해서 클러스터를 구축하는 방법

- 물리적 머신이나 가상 머신을 필요한 만큼 준비(마스터 노드 포함)
- 마스터 노드 역할을 할 머신에 k8s, etcd, CNI를 설치함(kubeadm 사용)
- 마스터 노드에서 kubeadm init으로 클러스터 초기화
- 워커 노드에서 kubeadm join으로 마스터와 연결

## 1. 용어 정리

### Kubeadm

쿠버네티스에서 제공하는 기본적인 도구로 쿠버네티스 클러스터 구축을 위한 다양한 기능을 제공한다.

<img src="https://user-images.githubusercontent.com/93081720/221210314-9f2c21c4-9a2f-421b-8bf2-217fcf5a1bc7.png" referrerpolicy="no-referrer" alt="image" height="200px">

※ 부트스트랩핑(bootstrapping) : 부팅의 현재 진행형(부팅이라고 이해하는 게 더 빠름)

- `kubeadm init` : 쿠버네티스 컨트롤 플레인 노드를 부트스트랩한다.
- `kubeadm join` : 쿠버네티스 워커 노드를 부트스트랩하고 클러스터에 연결시킨다.

<br>

### CNI (Container Network Interface)

컨테이너 네트워크 인터페이스



<br>

## 2. 사전 준비 및 쿠버네티스 설치

### 마스터-워커 노드 IP 주소 확인

미리 사전에 마스터 노드와 워커 노드로 쓰일 서버의 public IP 주소를 정리해두면 구축 시 용이하다.

<br>

## 3. 클러스터 구성