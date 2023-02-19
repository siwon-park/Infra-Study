# DinD vs. DooD

![image](https://user-images.githubusercontent.com/93081720/211159053-d97811b7-98b8-4193-8217-64e6560d0b9b.png)

Docker 시스템 유닛은 그림과 같이 크게 `Client`, `Host(Daemon)`, `Registry` 3개로 분리된다.

대부분의 현대 CI 도구들(travis, circle, gocd, jenkins)등이 agent를 통해 docker관련 Task를 수행을 하기 때문에 docker daemon은 호스트 머신에서 동작하면서 컨테이너로 동작하는 agent들이 docker-client역할을 하는 경우가 많다. 

그래서 데브옵스 개발자들은 쉽게 daemon과 client의 분리를 고려하며 docker container에서 agent가 호스트 머신에 위치한 docker daemon에게 어떻게 도커 명령을 전달해야할지 고민하게된다.

## 1. Docker-in-Docker

> 도커 컨테이너 내에서 도커 데몬을 이용해서 도커 컨테이너를 추가 생성(실행)하는 방법

호스트 머신의 도커 데몬을 이용하는 것이 아니라 컨테이너 내부에 다른 격리된 공간을 만드는 것이므로 도커 데몬이 2개 실행되는 것임

![image](https://user-images.githubusercontent.com/93081720/211158873-6af8062f-5f66-411a-a4cb-d750a7478889.png)

### DinD 방법의 문제점?

도커에서는 DinD 방식을 권장하지 않는데, 그 이유는 보안상의 이유 때문이다.

그 이유는 DinD 환경을 구성하기 위해 호스트 도커 컨테이너 실행 시 `--privileged`옵션을 사용하기 때문이다.

```bash
docker run --privileged --name DinD_test -d docker:1.8-dind
```

이 옵션은 호스트 컨테이너가 호스트 머신 전체에 대한 엑세스를 허용하기 때문에 호스트 컨테이너가 호스트 머신에서 할 수 있는 거의 모든 작업을 할 수 있게 된다. 따라서 DinD 방식은 보안상 치명적인 결함이 존재하는 것이다.

※ 기본적으로 모든 컨테이너는 unprivileged 상태로 호스트 시스템의 자원에 접근하거나 사용하는 것이 불가능하다.

<br>

### Jenkins Docker In Docker

젠킨스 컨테이너를 Docker In Docker 방식으로 실행시키고 컨테이너를 생성하는 과정

젠킨스 컨테이너 내부에 도커를 설치하고, 내부의 도커 데몬을 이용해서 호스트 머신에 컨테이너를 생성하고 실행시킴![image](https://user-images.githubusercontent.com/93081720/219939079-e9e0fccf-e04c-4768-aff1-32130edb4792.png)

<br>

## 2. Docker-out-of-Docker

> 호스트 머신에 설치되어 있는 도커를 이용해서 도커 컨테이너를 추가 생성(실행)하는 방법

호스트 머신의 docker socket을 에이전트(agent) 컨테이너에 볼륨 세팅을 통해서 공유하여 호스트 머신의 도커 데몬을 이용해서 도커 명령을 실행하는 방법

![image](https://user-images.githubusercontent.com/93081720/211158889-8a5a6cc1-f5ec-4ec7-9fcb-5c4253ba1bf2.png)

`-v`옵션으로 볼륨을 만들고 호스트 머신의 docker socket을 빌려서 사용하고, 소켓 통신을 통해 에이전트 컨테이너가 호스트 머신의 도커 데몬에 도커 명령어를 전달함

```bash
docker run -it -v /var/run/docker.sock:/var/run/docker.sock docker
```

### DooD의 문제점?

DinD를 권장하지 않을 뿐이지, DooD 역시 문제점이 존재한다.

만약 `-v`를 통해 tmp와 crontab 파일을 공유하고 tmp 디렉토리에 백도어 파일을 하나 배치하고 crontab을 수정해 실행 예약을 걸어둔다면 호스트 머신의 통제권이 공격자에게 넘어가는 사태가 발생하게 된다.

```bash
docker run -it -v /tmp:/tmp -v /etc/crontab:/etc/crontab --rm busybox sh
```

또한 docker.sock을 공유하기 위해 그만큼의 권한을 도커 컨테이너에 제공했으니, 그만큼의 취약점은 생길 수 밖에 없다.

정리하자면 DooD 방식은 DinD 방식보다 조금 더 보안상 안전하다는 것이지 완벽하게 안전한 방법은 아니다.

<br>

### Jenkins Docker Out of Docker

젠킨스 컨테이너를 실행시키고 Docker Out of Docker 방식으로 컨테이너를 생성하는 과정

젠킨스 컨테이너를 호스트 머신의 도커 클라이언트와 도커 소켓을 볼륨 마운트 시켜서 실행시키고, 호스트 머신의 도커 데몬을 이용해서 컨테이너를 생성하고 실행함

![image](https://user-images.githubusercontent.com/93081720/219939587-38c40716-f787-494e-93e6-98eec36398c7.png)

<br>

## 3. 결론

DinD 방식과 DooD 방식은 장단점이 있고, 보안상 단점이 존재한다.

어떤 것을 하고 싶은지에 따라 적절한 방법을 선택하는 것이 좋다.

만약, 컨테이너 안에 새로운 격리 환경을 만들고 싶다면 DinD 방식을,

컨테이너가 자신이 속한 도커 서비스를 다룰 수 있도록 하고 싶으면 DooD 방식을 선택하면 된다.

<br>

## 4. CI/CD

### Jenkins

그러면 Jenkins와 같은 CI 툴을 사용할 때 어떤 방법을 사용하는 것이 적절할까? 다음과 같은 선택권이 존재한다.

- DinD로 구축

- DooD로 구축

- Jenkins 내의 Docker API 이용