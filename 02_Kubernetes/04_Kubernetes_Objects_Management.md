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

- 직접 디플로이먼트 오브젝트를 생성하여 nginx 컨테이너를 구동시킴

```bash
$ sudo kubectl create deployment nginx --image nginx
```

- 혹은 이미 작동 중인 쿠버네티스 오브젝트를 삭제함(디플로이먼트 오브젝트를 제거)

```bash
$ sudo kubectl delete deploy/[Deployment 이름]
```

<br>

### 장점

- 커맨드로 하나의 동작에 대해서만 수행할 수 있어 어떤 행동을 하는지 명확하게 알 수 있다.

<br>

### 단점

- 커맨드는 변경 검토 프로세스와 통합되지 않는다.
- 커맨드는 변경에 대한 감사 추적(audit trail)을 제공하지 않는다.
- 커맨드는 활성 동작 중인 경우를 제외하고는 레코드의 소스를 제공하지 않는다.
- 커맨드는 새로운 오브젝트 생성을 위한 템플릿을 제공하지 않는다.

<br>

## 3. 명령형 오브젝트 구성(Imperative Object Configuration)

구성 파일(`yml`)에 정의된 내용의 오브젝트에 대한 작업을 수행한다.

### 예시

- 디플로이먼트 구성 파일을 바탕으로 디플로이먼트 오브젝트를 생성함

```bash
$ sudo kubectl apply -f deployment-test.yml # create도 가능
```

- 구성 파일에 정의된 오브젝트를 삭제한다. (2개의 오브젝트를 제거하는 경우)

```bash
$ sudo kubectl delete -f nginx.yml -f redis.yml
```

- 활성 동작 중인 구성을 덮어씀으로써 구성 파일에 정의된 오브젝트를 업데이트 함

```bash
$ sudo kubectl replace -f nginx.yml
```

<br>

### 장점

#### 명령형 커맨드 구성에 비해

- 오브젝트를 구성할 수 있는 내용을 파일로 보관할 수 있다
  - Git과 같은 형상관리 시스템에 업로드하여 버전 관리를 할 수 있다.
  - 재사용성, 템플릿화

#### 선언형 오브젝트 구성에 비해

- 동작 이해에 있어 보다 간결하고 이해하기 쉽다.

<br>

### 단점

#### 명령형 커맨드 구성에 비해

- 구성 파일의 오브젝트 스키마에 대한 기본적인 이해가 선행되어야 한다.
- yml 파일을 작성해서 오브젝트를 관리해야 한다는 번거로움이 있다.

#### 선언형 오브젝트 구성에 비해

- 활성화된 오브젝트에 대한 업데이트는 구성 파일에 반영시켜줘야 한다. 그렇지 않으면 다음 교체 작업 시 변경한 내용이 손실된다.

<br>

## 4. 선언형 오브젝트 구성(Declarative Object Configuration)

디렉토리 내 모든 오브젝트 구성 파일을 처리하고 오브젝트를 생성하거나 추가 작업(패치 등)을 수행한다.

선언형 오브젝트 구성 방식으로 쿠버네티스 오브젝트를 관리할 때, `diff` 명령을 수행하여 먼저 어떤 변경이 이루어질지 확인하고 적용할 수 있다.

### 예시

- config라는 디렉토리에 접근하여 구성 파일을 실행했을 경우 어떠한 변경이 있는지 우선 확인 한 다음, 디렉토리 내의 구성 파일을 적용한다.

```bash
$ sudo kubectl diff -f configs/
$ sudo kubectl apply -f configs/
```

<br>

### 장점

#### 명령형 오브젝트 구성에 비해

- 활성화된 오브젝트에 직접 작성된 변경 사항은 구성 파일로 다시 병합하지 않더라도 유지된다.
- 선언형 오브젝트 구성은 디렉토리에서의 작업 및 오브젝트 별 작업 유형 (생성, 패치, 삭제)의 자동 감지에 더 나은 자원을 제공한다.

<br>

### 단점

#### 명령형 오브젝트 구성에 비해

- 선언형 오브젝트 구성은 예상치 못한 결과를 디버깅하고 이해하기에 더 어려울 수도 있다.
- `diff`를 사용한 부분 업데이트는 복잡한 병합 및 패치 작업을 일으킨다.

<br>

## 5. 구성 파일 작성 및 이해

다음은 쿠버네티스 구성 `yml`파일에 대한 예시이다.

![image](https://github.com/siwon-park/Problem_Solving/assets/93081720/9aad51b5-8544-4793-a414-92300227d9fb)

#### apiVersion (Kubernetes API)

생성, 수정, 삭제든 쿠버네티스 오브젝트를 동작시키려면 `Kubernetes API`를 이용해야 한다.

우리가 사용하는 `kubectl`도 실제로는 kubernetes API를 호출하여 동작하는 것이다.

따라서 yml 구성 파일을 작성할 때도 호출할 kubernetes API를 지정해줘야 한다.

- 각 오브젝트별로 호출할 수 있는 api 버전이 [공식문서](https://kubernetes.io/docs/reference/kubernetes-api/)에 잘 정리되어 있다.
  - 따라서 자기 마음대로 버전관리 한답시고`v1`, `v2`를 붙일 수 없다. 물론 우연히 얻어 걸려서 해당 오브젝트에 해당하는 apiVersion이 있다면 괜찮겠지만, 그게 아니라면 에러가 난다.
