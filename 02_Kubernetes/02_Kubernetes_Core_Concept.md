![kubernets](https://user-images.githubusercontent.com/93081720/174333422-4e2f7a03-f585-4edf-884c-0af7fea7ac5d.png)

# 02_Kubernetes 핵심 개념

## 1. Kubernetes란 무엇인가?

> 컨테이너화된 워크로드와 서비스를 관리하기 위한 이식성이 있고, 확장가능한 오픈소스 플랫폼
> open-source system for automating deployment, scaling, and management of containerized applications
>
> 컨테이너 오케스트레이션 도구

쿠버네티스(kubernetes)라는 명칭은 키잡이(helmsman)나 파일럿을 뜻하는 그리스어에서 유래했다.

### k8s

k8s라는 표기는 "k"와 "s"와 그 사이에 있는 8글자를 나타내는 약식 표기이다.

<br>

### Kubernetes의 필요성

#### 컨테이너를 배포할 때...

만약 자고 있을 때 이러한 문제가 발생한다면...?

- 수동 배포 => 유지 보수에 많은 노력이 들어감
- 컨테이너에서 동작 중인 어플리케이션 간 충돌
- 컨테이너의 교체 필요
- 트래픽 급증 시, 또는 많은 부하로 인해 더 많은 컨테이너가 필요할 경우
  - 스케일링을 통한 로드 분산 필요
- 또는 트래픽 감소 시 컨테이너 수를 감소시키고자 할 때
- 인스턴스를 여러 개 나눠서 분산시키고자 할 때

#### Kubernetes가 하는 일

대규모 배포, 컨테이너 스케일링, 모니터링에 도움이 되는 도구들의 집합, 시스템

Pod와 컨테이너 모니터링, 관리, 스케일링 등

#### 우리가 하는 일

클러스터 생성, 마스터/워커 노드 생성, API 셋업, kubelet, 로드밸런서, EC2, ...

<br>

### Kubernetes의 기본 동작 개념

> 쿠버네티스에서 가장 중요한 것은 **바람직한 상태(Desired State)**이다.

여기서 말하는 바람직한 상태는 개발자가 원하는 환경(웹 서버는 몇 개, 포트는 어디로 연결...)을 말한다.

즉, 쿠버네티스는 현재 상태를 모니터링하면서 관리자가 설정한 바람직한 상태로 유지하려고 내부적으로 동작하는 로직을 갖고 있다.

![image](https://user-images.githubusercontent.com/93081720/226630421-31d8d657-bc21-4e57-8329-939ce46e479c.png)

이러한 동작을 하기 위해서 직접적인 동작을 명령하지 않고, `상태를 선언하는 방식`을 사용한다.

#### 명령적(imperative)

어떠한 동작을 해달라고 명령을 내리는 것

- "nginx 컨테이너를 80번 포트로 연결해서 실행시켜라"
- `docker run ~` (명령)

#### 선언적(declarative)

어떠한 동작을 한다는 것을 명시하는 것

- "nginx 컨테이너를 80번 포트로 연결해서 실행한다"
- `kubectl create ~` (선언)

<br>

### Kubernetes에 대한 오해

- 쿠버네티스는 도커의 대안이 아니라, 도커와 함께 사용되는 것이다.
- 쿠버네티스는 소프트웨어가 아니며, 컨테이너화된 App의 배포를 설정할 수 있는 툴/도구의 집합이다.
- 쿠버네티스는 PaaS(Platform as a Service)가 아니며, 클라우드 프로바이더에 의한 클라우드 서비스가 아니다.
- 쿠버네티스는 인프라 생성 도구가 아니다.

<br>

### Kubectl

> 쿠버네티스 API를 사용하여 쿠버네티스 클러스터의 컨트롤 플레인과 통신하기 위한 커맨드라인 툴

deploy 생성/변경과 같은 명령을 클러스터에 보내는 역할

```bash
kubectl [command] [TYPE] [NAME] [flags]
```

※ 굳이 비유하자면, kubectl은 총군수권자, cluster는 군대, master node는 장군, worker node는 군인

![image](https://user-images.githubusercontent.com/93081720/198072726-7ad1e054-5264-4998-b107-7a495ae670a0.png)

<br>

## 2. Kubernetes 아키텍처

쿠버네티스는 항상 `바람직한 상태(desired state)`를 유지하려한다 => 현재의 상태와 바람직한 상태를 비교하여 바람직한 상태로 유지한다.

### 클러스터(Cluster)

> 마스터 노드(Master Node)와 워커 노드(Worker Node)로 구성된 집합

컨테이너화된 어플리케이션을 구동 중인 '워커 노드'들과 이들을 관리하기 위한 '마스터 노드'의 집합

모든 클러스터는 최소 한 개의 마스터 노드와 워커 노드를 가진다.

![image](https://user-images.githubusercontent.com/93081720/198068743-e2c790c2-f55e-43a6-8094-456546b21835.png)

<br>

### 노드(Node)

>  한 개 이상의 포드를 호스팅하고 있는 물리적 또는 가상 머신으로 클러스터와 의사소통 함

#### 마스터 노드(Master Node)

컨트롤 플레인을 통해서 워커 노드들을 관리하는 노드로, 클러스터에 관한 전반적인 결정 및 이벤트 감지와 반응하는 역할을 함

##### 컨트롤 플레인(control plane)

![image](https://user-images.githubusercontent.com/93081720/221198482-f63aa645-a262-4d73-a6c3-b2161181f51e.png)

##### kube-api server

외부와 통신하는 프로세스(서버)

-  `kubectl`로부터 명령을 전달받고 실행함
- api 통신을 통해 etcd 정보 조회 및 업데이트
- 권한 체크 및 차단 기능 제공 

##### kube-controller-manager

`컨트롤러`를 통합/관리/실행함

- 논리적으로 다양한 컨트롤러가 존재함
- 컨트롤러에서 지속적으로 상태를 체크하여 바람직한 상태를 유지할 수 있도록 함

##### kube-scheduler

새롭게 생성된 파드(Pod)를 감지하고 워커 노드에 할당하는 역할을 수행

- 파드(Pod) 생성을 위해 요청한 리소스, 우선순위 및 기타 제약조건에 따라서 노드에 바인드하는 역할을 수행함

##### cloud-controller-manager

클라우드 서비스와 연동해 서비스(Service)를 생성함

##### etcd

클러스터 관련 정보 전반을 관리하는 DB

- 키(key)와 값(value) 쌍의 DB로 모든 상태와 데이터를 저장함
- 쿠버네티스는 etcd에 등록된 내용에 따라 실제 파드나 서비스를 생성한다. 이렇게 생성된 오브젝트들을 인스턴스라고 부름
- .yaml(yml) 파일(매니페스트 파일)의 내용이 DB에 저장되고, 쿠버네티스가 해당 파일의 내용을 읽어옴
- 만약 직접 커맨드로 컨테이너에 어떤 작업을 하도록 명령하게 되면 etcd에 있는 내용과 불일치가 발생

<br>

#### 워커 노드(Worker Node)

할당된 업무(Task)를 요청대로 수행하는 시스템으로 컨테이너들 간의 네트워킹 등 서비스 동작에 필요한 전반적인 일들을 마스터 노드와 통신하며 수행하는 역할을 함

파드(Pod)를 호스팅하며, 자원을 가지고 앱 컨테이너를 실행 중인 노드

`kubelet`, `Kube-proxy`, `Docker(컨테이너 런타임 환경)`와 `여러 개의 파드(Pod)`로 구성되어 있음

![image](https://user-images.githubusercontent.com/93081720/198069521-9586fb2d-0892-4ac6-affb-1e2b8572fb15.png)

##### Kubelet

- 마스터 노드에 있는 `kube-scheduler`와 연동하여 워커 노드에 파드를 배치, 실행하고 컨테이너가 동작하도록하는 역할을 담당
  - 일종의 에이전트(agent) 역할
  - 파드(Pod)를 실행 및 중지하고 상태를 체크한 뒤에 `api-server`에 보고
  - `Docker API`를 이용해 도커 데몬과 통신하여 컨테이너를 실행함
    - Kubelet은 쿠버네티스를 통해 생성되지 않은 컨테이너는 관리하지 않는다

- 실행 중인 파드의 상태를 정기적으로 모니터링하여 `kube-scheduler`에 알려주는 역할을 담당

##### Kube-proxy

- 각 노드에서 실행되는 네트워크 프록시로, 워커 노드의 네트워크 통신 라우팅 역할을 담당
  - 워커 노드 내부의 로드 밸런서 역할

- 노드의 네트워크 규칙을 유지/관리하며, 네트워크 규칙이 내부 네트워크 세션이나 클러스터 바깥에서 파드로 네트워크 통신을 할 수 있게 해준다
- 파드 간 통신(Overlay Network), 노드 간 통신(Service)

##### Container Runtime Interface (CRI)

- 컨테이너의 실행을 담당하는 역할 (컨테이너 런타임 환경)
- containerd

<br>

## 3. Kubernetes 오브젝트

![image](https://user-images.githubusercontent.com/93081720/221162521-c39a9763-1769-476b-8627-8ef26e25276d.png)

### 파드(Pod)

> 쿠버네티스가 상호작용하는 가장 작은 단위로, 컨테이너화된 어플리케이션을 실행 중인 작업 실행 단위

![image](https://user-images.githubusercontent.com/93081720/226638238-d7781f3c-ea64-4bb3-87d8-36d4b9ec1cd3.png)

하나 또는 그 이상의 컨테이너를 가지나, 보통 `하나의 파드에는 1개의 컨테이너와 1개의 볼륨(1 Pod per 1 Container 1 Volume)`으로 구성된다. (멀티 컨테이너로 구성될 수도 있다.)

컨테이너 구동을 위한 자원(예- 볼륨)을 요구한다.

다른 파드나 외부로 소통이 가능하지만, 디폴트로 클러스터 내부 IP주소가 있어서 파드 또는 해당 파드 내의 컨테이너에 요청을 보낼 때 사용할 수 있음

#### ※ Pod 유의점

- Pod는 임시적(ephemeral)이다
  - 지속적이지 않음(not permenant)
  - pod가 쿠버네티스에 의해 교체/제거되면 pod의 리소스는 손실됨 => 도커 컨테이너와 유사
- 사용자가 직접 pod를 강제로 생성/수정/삭제할 수 있지만 이는 쿠버네티스가 자동으로 해야 하는 작업임

<br>

### 레플리카 셋(Replica Set)

> 파드 수를 생성/복제하고 관리/모니터링하는 오브젝트

![image](https://user-images.githubusercontent.com/93081720/226637982-957a9c1c-a1b0-4323-80ca-3cd2c14d7af0.png)

복제할 파드의 개수, 개수를 모니터링할 라벨 선택자, 생성할 파드의 설정값(템플릿) 등을 갖고 있음.

일반적으로 레플리카 셋 단독으로 사용되는 경우는 거의 없다. 디플로이먼트와 함께 사용됨.

<br>

### 디플로이먼트(Deployment)

![image](https://user-images.githubusercontent.com/93081720/226640529-d5b5a485-4176-4f59-9c77-f4ed7848ea04.png)

> 파드의 배포를 관리하며, 일반적으로 레플리카 셋과 함께 사용되는 오브젝트

쿠버네티스에서 가장 널리 사용되는 오브젝트로 레플리카셋을 이용해서 파드를 업데이트하고 이력을 관리하거나 롤백(Rollback) 또는 특정 버전으로 되돌아갈 수 있게함(Revision)

<br>

### 서비스(Service)

![image](https://user-images.githubusercontent.com/93081720/226642465-5d60fd24-2238-45f4-90be-1173b4f62935.png)

> 네트워크와 관련된 오브젝트로, 파드를 외부 네트워크와 연결시켜주고, 워커 노드 내 로드 밸런서 역할을 하는 오브젝트

Load Balancer의 역할을 함 (※ 실제 흔히 말하는 로드 밸런서의 개념과는 다름)

서비스가 분배하는 통신은 해당 워커 노드에만 국한된다.

- 워커 노드 간 분배는 `인그레스(Ingress)`와 실제 `로드 밸런서(Load Balancer)`가 담당한다.
  - 이들은 별도의 노드에서 동작하거나 물리적인 하드웨어를 통해 구성된다.

![image](https://user-images.githubusercontent.com/93081720/198083922-5dc140f3-2c87-4798-a478-4d65e1fe6fe7.png)

<br>