![kubernets](https://user-images.githubusercontent.com/93081720/174333422-4e2f7a03-f585-4edf-884c-0af7fea7ac5d.png)

# 02_Kubernates 핵심 개념

## 01_Kubernates가 하는 일과 우리가 하는 일

### Kubernates가 하는 일

Kubernates는 인프라 생성 도구가 아님 => 컨테이너화된 App의 배포를 설정할 수 있는 도구들의 집합

Pod와 컨테이너 모니터링, 관리, 스케일링 등

### 우리가 하는 일

클러스터 생성, 마스터/워커 노드 생성, API 셋업, kubelet, 로드밸런서, EC2, ...

### Kubectl

deploy 생성/변경과 같은 명령을 클러스터에 보내는 역할

※ 굳이 비유하자면, kubectl은 총군수권자, cluster는 군대, master node는 장군, worker node는 군인

![image](https://user-images.githubusercontent.com/93081720/198072726-7ad1e054-5264-4998-b107-7a495ae670a0.png)

<br>

## 03_Kubernates 오브젝트

![image-20221027004859283](C:\Users\SIWON\AppData\Roaming\Typora\typora-user-images\image-20221027004859283.png)

### 파드(Pod)

> 쿠버네티스가 상호작용하는 가장 작은 단위

하나 또는 그 이상의 컨테이너를 가짐 but 일반적으로는 1pod 1container

컨테이너들의 공유 자원인 볼륨을 포함

다른 pod나 외부로 소통이 가능하지만, 디폴트로 클러스터 내부 IP주소가 있어서 pod 또는 해당 pod 내의 컨테이너에 요청을 보낼 때 사용할 수 있음

#### ※ Pod에서 유의해야할 점

- Pod는 임시적(ephemeral)이다
  - 지속적이지 않음(not permenant)
  - pod가 쿠버네티스에 의해 교체/제거되면 pod의 리소스는 손실됨 => 도커 컨테이너와 유사
- 사용자가 직접 pod를 강제로 생성/수정/삭제할 수 있지만 쿠버네티스가 자동적으로 해야하는 일임

<br>

### 레플리카 셋(Replica Set)

> 파드 수를 관리/모니터링함

일반적으로 레플리카 셋 단독으로 사용되는 경우는 거의 없다. 디플로이먼트와 함께 사용



### 디플로이먼트(Deployment)

> 파드의 배포를 관리, 일반적으로 레플리카 셋과 함께 사용



### 서비스(Service)

![image](https://user-images.githubusercontent.com/93081720/198083922-5dc140f3-2c87-4798-a478-4d65e1fe6fe7.png)
