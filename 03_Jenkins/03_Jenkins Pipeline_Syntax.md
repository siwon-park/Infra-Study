![image](https://user-images.githubusercontent.com/93081720/212346428-0b1fadf6-f630-4107-b9ae-9b81057b1d4c.png)

# 03_Jenkins Pipeline

> Jenkins Pipeline Flow

![image](https://user-images.githubusercontent.com/93081720/215792879-3c68b4ba-e061-4de7-9617-30c1751406c8.png)

<br>

## 1. 파이프라인 문법 종류

젠킨스 파이프라인 스크립트 작성 방법은 크게 2가지로 나뉜다.

두 가지 문법을 번갈아가면서 사용 가능하지만, 동시에 사용하는 것은 불가능하다.

### Declarative Pipeline

> 선언적 파이프라인

`pipeline` 블럭으로 감싸져 있음.

유저 정의하는 CD pipeline 모델. 빌드나 테스트, 배포를 포함한 전체적인 빌드 프로세스를 정의함 

Groovy-Syntax 기반이며, 보다 쉽게 작성할 수 있도록 커스텀되어 있음. Groovy 문법을 몰라도 작성이 가능함.

Scripted Pipeline보다 더 풍부한 기능을 제공하며, 파이프라인 코드를 더 읽기 쉽고 사용하기 쉽게 설계되었음.

![image](https://user-images.githubusercontent.com/93081720/215798135-54bb3f51-4956-4445-8294-9ec39710b13f.png)

<br>

### Scripted Pipeline

> 스크립트화된 파이프라인

`node` 블럭으로 감싸져 있음.

![image](https://user-images.githubusercontent.com/93081720/215798303-bbc52fa5-c702-481b-a14e-a8e666d00949.png)

<br>

## 2. Sections

> 선언적 파이프라인의 섹션 블럭은 하나 이상의 명령적 구문(Directives) 또는 단계(Steps)들을 포함하고 있음

### agent

> 전체 파이프라인, 특정 stage에 Jenkins가 실행자(executor)나 workspace를 할당하도록 지시하는 구문

#### 매개변수

- `any`: 사용 가능한 모든 agent 아래에서 pipeline이나 stage를 실행
- `none`: global agent가 전체 pipeline에 적용되지 않고, 각 개별 stage별로 agent가 필요함
- `label`: 특정 label로 명시된 Jenkins environment를 agent로 설정하여 적용함
- `node`: label과 동일하지만, customWorkspace와 같이 추가적인 옵션을 사용할 수 있음
- `docker`: 특정 도커 이미지로 파이프라인이 작업을 수행하도록 정의함

![image](https://user-images.githubusercontent.com/93081720/215929104-47e27e35-441a-4a50-9dd3-a36a066285ee.png)

- `dockerfile`: 특정 도커 파일을 이미지로 빌드한 다음에 해당 이미지로 파이프라인이 작업을 수행하도록 정의

![image](https://user-images.githubusercontent.com/93081720/215929133-d63a5f0b-1df3-42f5-b7ff-4b0a28147765.png)

- `kubernates`: 파이프라인이나 스테이지를 k8s 클러스터 내 파드에서 실행하도록 정의

#### 사용 위치

- pipeline 블럭 내 최상단: 필수
- stage 블럭 내: 선택(agent가 none으로 작성되었을 때는 각 stage별로 agent가 필요함)









### stage(s)

파이프라인의 진행 단계를 나타내는 문법 블록.

 `stages`섹션 내에는 여러 `stage`들이 있으며, 각 `stage`로 나누어 파이프라인의 진행 `단계`를 정의함.

파이프라인에는 반드시 최소 한 개 이상의 stage가 있어야 함.

![image](https://user-images.githubusercontent.com/93081720/215924598-549802e1-eaa5-4427-a28e-3ad00cc0e3b8.png)

### steps

이번 `stage` 내에서 실행되어야 할 `행동`들을 단계적으로 표현한 문법

![image](https://user-images.githubusercontent.com/93081720/215925078-3a083a46-ad22-40f9-ab3a-a6f939af1b98.png)