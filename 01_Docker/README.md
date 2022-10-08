![Docker image](https://user-images.githubusercontent.com/93081720/175770437-f0af4ada-49cc-4469-9db6-78446df47f35.png)

# 01_Docker

## 목차

- [01_Docker_Intro](https://github.com/siwon-park/Docker_Kubernates/blob/master/01_Docker/01_Docker_Intro.md)
- [02_Docker_Image & Container](https://github.com/siwon-park/Docker_Kubernates/blob/master/01_Docker/02_Docker_Image%20%26%20Container.md)
- [03_Docker_Volume](https://github.com/siwon-park/Docker_Kubernates/blob/master/01_Docker/03_Docker_Volume.md)
- [04_Docker_Network](https://github.com/siwon-park/Docker_Kubernates/blob/master/01_Docker/04_Docker_Network.md)
- [05_Docker_Compose](https://github.com/siwon-park/Docker_Kubernates/blob/master/01_Docker/05_Docker_Compose.md)
- [06_Utility_Container](https://github.com/siwon-park/Docker_Kubernates/blob/master/01_Docker/06_Utility_Container.md)

<br>

## Docker 요약

### Docker Core Concept

#### 컨테이너

> 코드와 그 코드를 실행하는데 필요한 환경을 포함하는 격리된 박스

특징

- 격리(isolated)
- 한 가지 일에만 집중(single-task focused)
  - 하나의 컨테이너에서는 하나의 일(task)만 집중함
  - 예) 백엔드 컨테이너는 백엔드 서버만 실행, 마찬가지로 프론트엔드 컨테이너는 프론트엔드 서버만 실행
- 공유 가능, 재생산성(sharable, reproducible)
- 무상태(statless)
  - 컨테이너가 종료될 때마다 데이터는 손실됨
  - 단, 볼륨을 설정할 경우 예외

#### 이미지

> Dockerfile로 만들어진 Snap Shot 또는 Docekr Hub로부터 가져온 이미지
>
> 컨테이너에 대한 블루프린트(blueprint)

이미지는 공유 가능하며, 이미지를 기반으로 다중 컨테이너를 생성 가능함

<br>

### Key Command

#### 이미지 빌드

.은 경로를 의미함 => 보통 일반적으로 dockerfile이 있는 곳에서 해당 명령어를 사용하므로 .을 주로 사용함

`docker build -t [이미지명]:[태그명] .`

#### 컨테이너 실행

`docker run --name [컨테이너명] --rm -d [이미지ID/이미지명]`

#### Docker hub

※ 도커 허브에 push/pull 하기 위해서는 이미지 이름은 반드시 도커허브의 repository이름과 일치해야함

- push

`docker push [레포지토리/이름:태그]`

- pull

`docker pull [레포지토리/이름:태그]`

<br>

### Volume

> 익명 볼륨, 명명된 볼륨, 바인드 마운트

정확한 표현은 아니지만, 볼륨(익명/명명)은 임시 저장소, 바인드마운트는 영구 저장소로 이해하면 편함

#### 볼륨

- 익명 볼륨
  - 컨테이너가 종료되면 익명 볼륨은 삭제됨(단, --rm명령어로 실행 시에만 해당)
  - 따라서 데이터도 삭제됨

- 명명된 볼륨
  - 컨테이너가 종료되어도 데이터가 유지됨
  - 컨테이너를 다시 실행할 경우 해당 볼륨을 사용 가능하므로 데이터도 살아 있음



#### 바인드마운트

호스트 머신의 특정 경로와 컨테이너를 미러링

<br>

### Network

>  컨테이너는 서로 연결되어 컨테이너 간 요청을 보낼 수 있고, 외부로도 요청을 보낼 수 있음

<br>

### Docker Compose

> 다중 컨테이너를 실행하기에 유용한 도구

컨테이너 시작 구성 파일을 작성하여 다중 컨테이너를 쉽게 실행 가능

기존 명령어 기반으로 도커 컨테이너를 실행한다면 반복적이며, 긴 명령어를 계속 입력해야하지만, docker-compose 구성 파일을 통해 이를 해결 가능

<br>

### Local & Remote

#### Local Host(Development)

도커는 개발을 매우 편리하게 만들어줌 => 상호 종속성이 없어서 시스템의 다른 환경과 충돌하지 않음

다중 프로젝트에서 작업하며, 버전과 런타임 간의 충돌을 피하기 위해서 로컬에서 도커를 사용 가능함

#### Remote Host(Production)

로컬에서 실행된 컨테이너는 원격 환경에서도 동일하게 실행 가능함 => 이미지에 모든 내용이 다 담겨져 있기 때문

업데이트가 간단함 => 업데이트된 소스 코드가 포함된 컨테이너로 바꾸기만 하면 됨

<br>

### Deployment

컨테이너 배포 시 유의할 점

- 바인드 마운트는 프로덕션 단계에서 사용하지 말 것
  - 바인드 마운트의 용도는 개발 단계에서 변경된 소스 코드 또는 일부 구성 및 데이터를 미러링하기 위한 용도임
- 다중 호스트 머신이 필요한 경우에는 도커 명령어 및 도커 컴포즈만으로 간단하게 해결이 불가능할 수도 있음
- 멀티 스테이지 빌드
  - 이미지 내에서 빌드를 진행하고 컨테이너를 실행할 때 완료된 결과물이 있는 컨테이너로 실행 가능함
  - `as`키워드를 통해 사용



