![Docker image](https://user-images.githubusercontent.com/93081720/174341063-d8894c50-7452-49b0-ae2f-7a4b019dc8a9.png)

# 05_Docker-Compose

> 다중 컨테이너를 쉽게 다룰 수 있게 해주는 툴
>
> 하나의 config파일 + orchestration 명령어들(build, start, stop...)

![image](https://github.com/siwon-park/Problem_Solving/assets/93081720/1448691b-4df1-4ea0-9cfd-09c60f7e5a9e)

### ※ docker-compose의 주의점

- docker-compose는 Dockerfile을 대체하는 도구가 아니다 => Dockerfile과 함께 일한다.
- docker-compose는 이미지나 컨테이너를 대체하는 개념이 아니다 => 여전히 이미지나 컨테이너가 필요하다
- docker-compose는 다중 호스트 상황에서는 적합하지 않다 => 싱글 호스트 상황에서 적합함

### 1. Docker vs. Docker-Compose

| 항목                          | Docker                        | Docker-Compose                  |
| ----------------------------- | ----------------------------- | ------------------------------- |
| 이미지 빌드                   | 한 프로젝트                   | 다중 프로젝트                   |
| 컨테이너 실행                 | 한 프로젝트                   | 다중 프로젝트                   |
| 명령어 실행                   | 한 명령어 당 한 프로젝트      | 한 명령어 다중 프로젝트         |
| 실행 옵션                     | 실행 명령어에 추가(하드 코딩) | yml에 스펙 기입                 |
| 컨테이너 간 실행 순서(의존성) | 수동, 직접 순서대로 작업      | 자동, 지정된 순서대로 작업 가능 |
| 호스트                        | 싱글 호스트, 다중 호스트 적합 | 싱글 호스트 적합                |

<br>

### 2. 버전

#### docker compose v1

`docker-compose`로 명명하며 반드시 `하이픈(-)`을 같이 붙여줘야 한다.

#### docker compose v2

`docker compose`로 `하이픈(-)`이 빠진 명령어로도 동작이 가능하다.

`docker-cli`에 통합되어 있기 때문에 기본적으로 docker를 설치하고 나면 docker help 입력 시 management command에 `compose*`로 표시되어 있는 것을 확인할 수 있으며 `docker compose version` 명령어로 버전을 확인할 수도 있다.

`docker compose version`이 가능한 이유가 기본 Docker 명령어 옵션 순서가 `Docker [OPTIONS] COMMAND`인데, command로 compose가 들어가 있기 때문이다.

<br>

### 3. 설치

※ Linux는 docker-compose를 따로 설치해줘야한다. (Mac OS나 Windows는 도커 설치 시 자동으로 설치된다.)

다음은 Jenkins 컨테이너 내에 docker-compose를 설치하는 예시이다.

#### Jenkins 컨테이너 접속

컨테이너 이름: jenkins

```bash
sudo docker exec -it jenkins /bin/bash
```

#### docker-compose 다운로드

Github에서 최신 버전의 docker-compose 바이너리 파일을 다운받는다.

```bash
curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
```

#### 실행 권한 부여

다운로드한 파일에 실행 권한을 부여한다.

```bash
chmod +x /usr/local/bin/docker-compose
```

#### 버전 확인

제대로 설치되었는지 버전을 확인한다.

```bash
docker-compose version
```

※ `docker-compose` 버전을 확인해보면 `V2`인 것을 확인할 수 있지만, `docker compose XXX`로는 명령어를 날릴 수 없다.

그 이유는 컨테이너 내부에서 `docker help`를 처보면 알겠지만, compose가 명령어로서 보이지 않는 것을 확인할 수 있다. 따라서 설치한 이름 그대로 `docker-compose`로 사용하는 것이다.

<br>

## 01_Getting Started

>  docker-compose 파일 생성

`docker-compose.yml` 또는 `docker-compose.yaml` 파일을 생성



![image](https://user-images.githubusercontent.com/93081720/193403937-dfb16a44-d100-4696-9a02-93a882de38dd.png)

※ yml 파일

- 구성 옵션 간 종속성을 표현하는 특정한 텍스트 포멧
- 엔터 + 2칸의 들여쓰기(공백)으로 하위 구성을 표현할 수 있음

### 1. version

[docker-compose 버전](https://docs.docker.com/compose/compose-file/compose-versioning/)

![image](https://user-images.githubusercontent.com/93081720/193404017-1fa65c1b-ab73-4f07-b4ed-1f3bc65c7bcd.png)

docker-compose.yml 파일의 최상단에 써줘야 하는 것으로 docker-compose의 버전을 말함

우리가 만든 앱의 버전이 아니라 docker-compose에서 지원하는 버전을 의미함

<br>

### 2. services

> 다중 컨테이너 어플리케이션을 구성하는 컨테이너

다루고자하는 컨테이너들에 대한 정보, 실행 옵션 등을 명시함

![image](https://user-images.githubusercontent.com/93081720/193404086-947e1583-0425-4160-ab7b-7fed3589bcda.png)

<br>

![image](https://user-images.githubusercontent.com/93081720/193404216-45606688-d494-4572-91e8-915e37a9e680.png)

<br>

### 3. image & build

![image](https://user-images.githubusercontent.com/93081720/193404281-da0b6a04-4cc9-4485-97e5-8752e65fc7cd.png)

#### image

이미지를 `DockerHub`로부터 받아와야한다면 image 속성에 공식 이미지 이름을 적는다

이 때 이미지명 또는 전체 url, 저장소명 등을 쓸 수 있다.

#### build

이미지를 로컬에서 빌드를 해야한다면 빌드할 파일의 **상대 경로**를 작성해준다.

이 때, build 속성 바로 옆에 상대 경로를 작성해도 되지만, 하위에 context, dockerfile이라는 옵션을 주어 더 정확하고 자세하게 작성할 수도 있다. 

이렇게 하위로 나눠서 더 작성하는 이유는 기본적으로 Dockerfile이라는 이름의 파일을 찾지만, 파일명이 Dockerfile-dev처럼 개발 단계에서 테스트 중인 도커 파일로 나눠져 있을 수도 있기 때문에 빌드해야 할 특정 도커 파일을 사용하기 위해서 지정할 수도 있다.

<br>

### 4. volumes

볼륨에 대한 정보는 volumes로 표시한다.

![image](https://user-images.githubusercontent.com/93081720/193404699-6eae685d-6ddc-4e07-bc72-b38951a6475f.png)

#### 기명 볼륨(named volume)

기명 볼륨의 경우 `볼륨명:컨테이너 경로`로 작성한다.

단, 여기서 기명 볼륨은 상위 레벨의 volumes 항목에 볼륨명을 작성한 다음에 컨테이너 하위의 volumes 항목에 적는다.

※ 다른 서비스에서 동일한 볼륨명을 사용하면 해당 볼륨은 공유된다.

#### 바인드 마운트(bind mount)

docker-compose에서 바인드 마운트는 **상대 경로로 작성**한다.

#### 익명 볼륨(anonymous volume)

익명 볼륨의 경우 원하는 가상의 경로를 작성 해주면 된다.

<br>

### 5. container_name

컨테이너 이름을 명명할 수 있다.

#### ※ 주의

`services` 바로 아래 단계에 작성하는 이름이 컨테이너 이름이 된다고 착각할 수 있지만, `container_name으로 따로 명명하지 않는다면 도커에 정의된 컨테이너 명명 규칙에 따라 컨테이너 이름이 정해진다.`

- 예) docker_container_mongodb

![image](https://user-images.githubusercontent.com/93081720/193404820-22670821-1266-4547-82b2-b533ea1d86b9.png)

<br>

### 6. environment & env_file

환경 변수에 대한 값을 작성할 수 있다.

![image](https://user-images.githubusercontent.com/93081720/193405113-c5416413-1143-4eb9-a6ac-0048e189ac1d.png)

#### environment

직접 환경 변수에 대한 키와 값을 작성할 수 있다.

이 때, 작성 법은 위의 예시처럼 `키: 값`으로 작성할 수도 있지만, 아래와 같이 `- 키=값` 형식으로도 작성 가능하다.

※ `키 = 값`으로 쓸 때는 -(dash)가 필요 하지만, `키:값`으로 작성할 때는 필요 없다.

![image](https://user-images.githubusercontent.com/93081720/193405151-8c9bd41e-74fa-46c5-958e-8569e83c4077.png)

<br>

#### env_file

환경 변수에 대한 키와 값을 직접 작성할 수도 있지만, 너무 많을 경우 일일이 다 지정해주는 것은 많은 공수가 소모된다. 따라서 환경 변수 파일에 대한 상대 경로를 작성하여 환경 변수에 대한 값을 사용할 수도 있다.

<br>

### 7. args

Dockerfile에서 `ARG` 옵션을 사용한다면, 아래와 같이 args에 키=값 형태로 사용하여 ARG를 사용할 수 있다.

![image](https://user-images.githubusercontent.com/93081720/193405386-438187cd-46d3-4567-96a6-7cf1008181d4.png)

<br>

### 8. networks

services 아래의 다중 컨테이너는 모두 같은 네트워크 안에 속하지만 볼륨에서와 마찬가지로 따로 네트워크 명을 명명해주지 않으면 도커에서 기본적으로 작성해주는 네트워크명을 사용하게 된다.

networks옵션으로 네트워크 이름을 따로 명명할 수 있다.

![image](https://user-images.githubusercontent.com/93081720/193405260-ea4fd296-62bb-4860-9906-9d5362e41456.png)

<br>

### 9. ports

![image](https://user-images.githubusercontent.com/93081720/193405468-4466a620-18b7-4c7f-96cd-32fecee58903.png)

docker run -p에서 포트를 지정해주 듯이 ports 속성에 포트 번호를 지정해준다. 단일 포트 뿐만 아니라 다중 포트도 사용할 수 있다.

<br>

### 10. depends_on

`docker-compose는 컨테이너 실행 작업을 동기적으로 처리하지 않는다.(비동기적 실행 작업)`

컨테이너에 따라 선행 컨테이너가 먼저 실행되어야만 에러 없이 정상적으로 실행 가능한 컨테이너들이 있다. 이를 위해서 depends_on 옵션을 사용하여 선행적으로 실행되어야할 컨테이너에 대해 정의할 수 있다.

예를 들어, Jenkins와 같은 자동화 툴을 사용할 때 FreeStyle로 환경 구성 시 일반 Docker로 여러 개의 컨테이너를 실행시킨다면 작업의 순서가 보장되지 않을 수 있다. (단, Pipeline으로 환경을 구현하면 작업의 실행 순서가 보장되기 때문에 컨테이너의 실행 순서도 보장된다.)

이 때, FreeStyle + docker-compose 조합이라면 동기적으로 컨테이너 실행 작업을 수행할 수 있다.

![image](https://user-images.githubusercontent.com/93081720/193405584-414c26c7-f985-4cfb-bff0-f18fb6064042.png)

<br>

### 11. stdin_open & tty

docker run 시, `-it` 옵션으로 인터렉티브 모드로 동작하고자 할 때는 아래와 같이 사용하여서 적용할 수 있다.

![image](https://user-images.githubusercontent.com/93081720/193405685-cea1515c-336e-4411-a11c-ed8b009b83e5.png)

<br>

## 02_실행

docker-compose는 기본적으로 `--rm`이 default이다.

### 1. docker-compose up

> docker-compose를 실행하는 명령어

#### docker-compose up -d

detach모드로 실행하게 함

#### docker-compose up --build

이미지 빌드를 강제하면서 docker-compose를 실행함

이렇게 하는 이유는 기본적으로 docker-compose는 처음 빌드 이후 이미 빌드된 이미지를 사용하고, 필요할 때만 rebuild를 진행하는데, 경우에 따라 빌드를 강제해야할 때가 있는데 이 때 사용한다.

<br>

### 2. docker-compose down

> docker-compose를 종료하는 명령어

#### docker-compose down -v

볼륨을 삭제하면서 docker-compose를 제거함

