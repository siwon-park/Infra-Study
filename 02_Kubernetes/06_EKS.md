![image](https://user-images.githubusercontent.com/93081720/221838771-dc557d37-fd2a-496c-9a55-9e007084e06b.png)

# 06_EKS

> AWS EKS(Elastic Kubernates Service)

아마존(AWS)에서 제공하는 완전관리형 쿠버네티스(kubernates) 서비스

자체적으로 쿠버네티스 클러스터를 구성하고 유지/보수할 필요 없이 AWS에서 쿠버네티스를 쉽게 실행할 수 있도록 지원하는 관리형 쿠버네티스 서비스이다.

- 비교 요약

| 항목         | K8S                  | EKS                                                      |
| ------------ | -------------------- | -------------------------------------------------------- |
| 마스터 노드  | 직접 설치, 직접 관리 | AWS에서 관리                                             |
| 인증         | 직접 구성            | 자동 구성                                                |
| 파드 수 제한 | 없음                 | 있음(인스턴스 타입에 따른 ENI당 IP 부여량 제한으로 발생) |

## 1. EKS를 사용하는 이유

> 구성 및 관리의 편의성

### 만약 EKS를 사용하지 않는다면?

> 인증 및 쿠버네티스 클러스터를 직접 구성해야 한다

쿠버네티스 클러스터의 복잡한 동작 과정에 대한 이해가 바탕이 되어야 직접적인 설치 및 운용이 가능하다.

인증, 네트워크, 볼륨, 모니터링 등 신경써야 하는 요소가 한 두가지가 아니기 때문에 학습과 이해가 바탕이 되어야 한다.

예를 들어, API 서버에서 장애가 발생할 경우를 대비해서 2개의 API 서버를 가지도록 설계해야하고

etcd에서 장애가 발생할 경우 3개 이상의 etcd를 도입하여 고가용성을 유지해야함

- etcd를 최소 3개를 두는 이유는 쿠버네티스에서 뭔가 결정을 하게 될 때, 투표를 진행하는데 짝수면 과반수가 아니고, 1개면 의사결정을 할 수 없기 때문

<br>

### 쿠버네티스 클러스터 내부 동작 과정

#### Pod 생성

nginx 이미지로 컨테이너를 실행하는 파드를 생성

```bash
kubectl run nginx --image=nginx
```

#### ※ Pod 생성 순서

1. Deployment 생성
2. Deployment로 ReplicaSet을 생성
3. ReplicaSet에 의해 Pod 생성

![image](https://user-images.githubusercontent.com/93081720/222948455-791098fd-e71b-4c36-a4de-5763663d7bbf.png)

<br>

#### Watch 매커니즘

> 쿠버네티스 클러스터 내부 동작 구조

각 컴포넌트들은 API 서버를 감시하고, API 서버는 etcd를 감시하여, 변경 사항에 대해 인지함

API 서버에서 변경 사항이 발생하면 etcd에 업데이트 요청을 보내고, etcd는 업데이트 이후 변경 사항이 발생했으니, 이 변경 사항을 API 서버에 다시 알린다. 이후 API 서버는 해당 변경 사항 내용을 받아 컴포넌트에게 알린다.

- 컴포넌트: API Server, Controller Manager, Scheduler, etcd, Kublet, Kube proxy

![image](https://user-images.githubusercontent.com/93081720/222949289-0113e57b-2f61-4b4b-a6d4-281ac78af546.png)

<br>

![image](https://user-images.githubusercontent.com/93081720/222951405-7f6119a8-1ed5-4bc3-84db-8258f23e8a87.png)

<br>

### EKS를 사용하면

하지만, EKS를 사용하게 되면 쿠버네티스 클러스터를 구축하는데 어려운 과정들을 EKS가 대신 해주기 때문에 훨씬 편리하다.

![image](https://user-images.githubusercontent.com/93081720/222951769-9ead0826-b25b-4370-a1ff-e26ac1e1fa23.png)

