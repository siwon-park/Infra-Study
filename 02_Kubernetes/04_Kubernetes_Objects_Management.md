# 04_Kubernetes 오브젝트 관리

`kubectl` 커맨드라인을 통해서 kubernetes 오브젝트들을 관리할 수 있는데,

오브젝트 관리는 크게 3가지로 나뉜다.

<br>

## 1. 관리 기법

> 쿠버네티스 오브젝트는 하나의 기법만 사용하여 관리해야 한다. 동일한 오브젝트에 여러 기법을 혼용하는 것은 예상치 못한 동작을 초래한다

| 관리 기법            | 대상                      | 권장 환경     | 비고 |
| -------------------- | ------------------------- | ------------- | ---- |
| 명령형 커맨드        | 활성화된 오브젝트         | 개발 환경     |      |
| 명령형 오브젝트 구성 | 개별 구성 파일 (yml)      | 프로덕션 환경 |      |
| 선언형 오브젝트 구성 | 구성 파일이 있는 디렉토리 | 프로덕션 환경 |      |

<br>

## 2. 명령형 커맨드(Imperative Command)

직접 명령 커맨드를 통해 쿠버네티스 오브젝트를 실행하거나 추가 작업을 수행함

### 예시

직접 디플로이먼트 오브젝트를 생성하여 nginx 컨테이너를 구동시킴

```bash
$ sudo kubectl create deployment nginx --image nginx
```

혹은 이미 작동 중인 쿠버네티스 오브젝트를 삭제함(디플로이먼트 오브젝트를 제거)

```bash
$ sudo kubectl delete deploy/[Deployment 이름]
```

<br>

### 장점

<br>

### 단점

<br>

## 3. 명령형 오브젝트 구성(Imperative Object Configuration)

구성 파일(`yml`)에 정의된 내용의 오브젝트에 대한 작업을 수행한다.

### 예시

디플로이먼트 구성 파일을 바탕으로 디플로이먼트 오브젝트를 생성함

```bash
$ sudo kubectl apply -f deployment-test.yml # create도 가능
```

구성 파일에 정의된 오브젝트를 삭제한다. (2개의 오브젝트를 제거하는 경우)

```bash
$ sudo kubectl delete -f nginx.yml -f redis.yml
```

<br>

### 장점

<br>

### 단점

<br>

## 4. 선언형 오브젝트 구성(Declarative Object Configuration)

디렉토리 내 모든 오브젝트 구성 파일을 처리하고 오브젝트를 생성하거나 추가 작업(패치 등)을 수행한다.

선언형 오브젝트 구성 방식으로 쿠버네티스 오브젝트를 관리할 때, `diff` 명령을 수행하여 먼저 어떤 변경이 이루어질지 확인하고 적용할 수 있다.

### 예시

```bash
$ sudo kubectl diff -f configs/
$ sudo kubectl apply -f configs/
```

<br>

### 장점

<br>

### 단점

<br>



## 5. 구성 파일 작성 및 이해

`yml`파일에 작성하는 각 항목에 대해 정리





TEMP

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: be42-deploy
spec:
  selector:
    matchLabels:
      app: be42
  template:
    metadata:
      labels:
        app: be42
    spec:
      dnsPolicy: Default
      dnsConfig:
        nameservers:
          - 8.8.8.8
      containers:
        - name: be42
          image: 15.164.102.222:1588/return/be
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
```

