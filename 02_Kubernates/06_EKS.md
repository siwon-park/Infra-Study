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

> 직접 쿠버네티스 클러스터를 구축해야함

다음과 같은 과정을 직접 해야 한다.

#### Pod 생성

nginx 이미지로 컨테이너를 실행하는 파드를 생성

```bash
kubectl run nginx --image=nginx
```

#### ※ Pod 생성 순서

1. Deployment 생성
2. Deployment로 ReplicaSet을 생성
3. ReplicaSet에 의해 Pod 생성
