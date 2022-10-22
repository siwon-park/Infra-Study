# 00_Kubernates_Objects

쿠버네티스 오브젝트들

## 1. Pod (파드)

> 쿠버네티스에서 관리하는 가장 작은 배포 단위, 쿠버네티스와 상호작용하는 가장 작은 단위

### kubectl run (Pod 생성하기)

```
 kubectl run [이름] --image [이미지 주소]
```

- 예시

```bash
kubectl run echo --image ghcr.io/subicura/echo:v1
```

![image](https://user-images.githubusercontent.com/93081720/197321703-4ef7f541-8094-4e24-a3d2-2f9f3ca12283.png)

- 생성 과정

![image](https://user-images.githubusercontent.com/93081720/197321783-6b74cf27-b794-418a-a7e6-20c98067f726.png)

1. `Scheduler`는 API서버를 감시하면서 할당되지 않은unassigned `Pod`이 있는지 체크
2. `Scheduler`는 할당되지 않은 `Pod`을 감지하고 적절한 `노드`node에 할당 (minikube는 단일 노드)
3. 노드에 설치된 `kubelet`은 자신의 노드에 할당된 `Pod`이 있는지 체크
4. `kubelet`은 `Scheduler`에 의해 자신에게 할당된 `Pod`의 정보를 확인하고 컨테이너 생성
5. `kubelet`은 자신에게 할당된 `Pod`의 상태를 `API 서버`에 전달

### kubectl apply

- yml 파일 작성

```yml
# echo-pod.yml
apiVerison: v1
kind: Pod
metadata:
  name: echo
  labels:
    app: echo
spec:
  containers:
    - name: app
      image: ghcr.io/subicura/echo:v1
```

- pod 생성(yml이 존재하는 폴더에서 실행)

```bash
kubectl apply -f echo-pod.yml
```

```bash
# Pod 목록 조회
kubectl get pod

# Pod 로그 확인
kubectl logs echo
kubectl logs -f echo

# Pod 컨테이너 접속
kubectl exec -it echo -- sh
# ls
# ps
# exit

# Pod 제거
kubectl delete -f echo-pod.yml
```

![image](https://user-images.githubusercontent.com/93081720/197324634-9dfcf079-5d47-4d39-b026-b6ef85727e3f.png)

### 컨테이너 상태 모니터링

`컨테이너 생성`과 실제 `서비스 준비`는 약간의 차이가 있음. 서버를 실행하더라도 바로 접속할 수 없고 초기화까지 대기 시간이 필요하고, 이후에 실제 접속이 가능할 때 `서비스가 준비되었다`고 말할 수 있음

![image](https://user-images.githubusercontent.com/93081720/197322138-15d36d81-c0a1-443f-87e4-dc541fe6fc2f.png)

#### livenessProbe

컨테이너가 정상적으로 동작하는지 체크하고 정상적으로 동작하지 않는다면 **컨테이너를 재시작**하여 문제를 해결

#### readinessProbe

컨테이너가 준비되었는지 체크하고 정상적으로 준비되지 않았다면 **Pod으로 들어오는 요청을 제외**합니다

#### livenessProbe + readinessProbe

보통은 `livenessProbe`와 `readinessProbe`를 같이 적용합니다.

```yml
apiVerison: v1
kind: Pod
metadata:
  name: echo
  labels:
    app: echo
spec:
  containers:
    - name: app
      image: ghcr.io/subicura/echo:v1
      livenessProbe:
        httpGet:
          path: /
          port: 3000
        initialDelaySeconds: 5
        timeoutSeconds: 2 # Default 1
        periodSeconds: 5 # Defaults 10
        failureThreshold: 1 # Defaults 3
      readinessProbe:
        httpGet:
          path: /
          port: 3000
        initialDelaySeconds: 5
        timeoutSeconds: 2 # Default 1
        periodSeconds: 5 # Defaults 10
        failureThreshold: 1 # Defaults 3
```



### 다중 컨테이너

대부분 `1Pod 1Container`이지만, 1Pod내에 여러 개의 컨테이너를 가진 경우도 있음

하나의 Pod에 속한 컨테이너들은 서로 localhost로 네트워크를 공유하고 동일한 디렉토리를 공유할 수 있음

```yml
# counter pod 생성
apiVersion: v1
kind: Pod
metadata:
  name: counter
  labels:
    app: counter
spec:
# 컨테이너가 app, db로 2개의 멀티 컨테이너로 이루어져있음
  containers:
    - name: app
      image: ghcr.io/subicura/counter:latest
      env:
        - name: REDIS_HOST
          value: "localhost"
    - name: db
      image: redis
```

![image](https://user-images.githubusercontent.com/93081720/197324612-d339e7df-170d-4bce-afc5-5e4e52ea2a5a.png)

<br>

## 2. ReplicaSet (레플리카 셋)

> Pod를 정해진 수만큼 복제/생성하고 관리하는 도구. pod를 유지하는 역할을 담당

레플리카 셋은 단독으로 쓰는 경우는 거의 없고, `Deployment`가 레플리카 셋을 이용하기 때문에 주로 `Deployment`를 사용함  

### 레플리카 셋 생성

![image](https://user-images.githubusercontent.com/93081720/197324978-3622eeb4-f7ac-46ba-9e87-928265c3e4e0.png)

```bash
# ReplicaSet 생성
kubectl apply -f echo-rs.yml

# 리소스 확인(pod, replicaset)
kubectl get po,rs
```

![image](https://user-images.githubusercontent.com/93081720/197324601-027d8487-f78a-418b-a565-fd85144d4e97.png)

> ReplicaSet은 `label을 체크`해서 `원하는 수`의 Pod이 없으면 `새로운 Pod`을 생성 => label이 겹치지 않게 신경써서 정의해야함

- `spec.selector`: label 체크 조건(매칭 라벨)
- `spec.replicas`: 원하는 pod의 개수
- `spec.template`: 생성할 pod의 명세



- label 확인

```bash
kubectl get pod --show-labels
```

![image](https://user-images.githubusercontent.com/93081720/197325209-d95d4c84-4f4c-4c3b-be8b-f8a90833a637.png)

- label 제거

```bash
# app- 를 지정하면 app label을 제거
kubectl label pod/echo-rs-9qsrf app-

# 다시 Pod 확인
kubectl get pod --show-labels
```

![image](https://user-images.githubusercontent.com/93081720/197325267-d202257a-6510-4d1d-a186-10adb333d0bc.png)

기존에 생성된 Pod의 `app` label이 사라지면서 `selector`에 정의한 `app=echo,tier=app` 조건을 만족하는 Pod의 개수가 0이 되어 새로운 Pod가 만들어짐

![image](https://user-images.githubusercontent.com/93081720/197325428-f8458536-4b3f-4d41-a822-0350e5c3d2ce.png)

- label 다시 추가하기

```bash
kubectl label pod/echo-rs-9qsrf app=echo
```

![image](https://user-images.githubusercontent.com/93081720/197325573-24e78513-2c26-41ee-98eb-492f29fe97a7.png)

pod 수가 2개에서 1개가 제거되어 1개로 줄어듬

### 레플리카 셋 동작 과정

![image](https://user-images.githubusercontent.com/93081720/197325640-1f6b4fc4-05b1-4e42-aed3-13a0371e0004.png)

1. `ReplicaSet Controller`는 ReplicaSet조건을 감시하면서 현재 상태와 원하는 상태가 다른 것을 체크
2. `ReplicaSet Controller`가 원하는 상태가 되도록 `Pod`을 생성하거나 제거
3. `Scheduler`는 API서버를 감시하면서 할당되지 않은unassigned `Pod`이 있는지 체크
4. `Scheduler`는 할당되지 않은 새로운 `Pod`을 감지하고 적절한 `노드`node에 배치
5. 이후 노드는 기존대로 동작



- 레플리카 셋 삭제
  - pod도 함께 제거된다

```bash
kubectl delete replicaset.apps/echo-rs
```



### 스케일 아웃

ReplicaSet을 이용하면 손쉽게 Pod을 여러개로 복제 가능

![image](https://user-images.githubusercontent.com/93081720/197325722-aca96824-2050-414c-8038-cce47697396c.png)

```bash
# 레플리카 셋 생성
kubectl apply -f echo-rs-scaled.yml

# Pod 확인
kubectl get pod,rs
```

![image](https://user-images.githubusercontent.com/93081720/197325871-66f223c3-d268-4637-8140-6337889b68ca.png)

<br>

## 3. Deployment (디플로이먼트)

> 쿠버네티스에서 가장 널리 사용되는 오브젝트로 ReplicaSet을 이용해 Pod를 업데이트하고 이력을 관리하거나 롤백(rollback) 또는 특정 버전으로 돌아갈 수 있음(revision) 



<br>

## 4. Service (서비스)

> Pod를 외부로 노출시키므로써 클러스터 외부에서  Pod로 접근할 수 있게 해주는 도구

로드밸런서의 개념

 