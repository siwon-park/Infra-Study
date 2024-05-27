# 08_Docker Architecture

> 도커의 아키텍처(구조)

도커의 아키텍처는 크게 `클라이언트로서의 도커(도커 클라이언트)`와 `서버로서의 도커(도커 호스트)`로 나뉜다.

- 도커 명령어 API를 사용할 수 있게 해주는 Docker Cli (도커 클라이언트)
- 도커 데몬과 도커 클라이언트 간 상호작용 할 수 있게 도와주는 Docker Socket (도커 소켓)
- 실제로 도커 컨테이너를 생성/실행/관리하는 주체인 Docker Daemon (도커 데몬, 도커 호스트)
- 도커 이미지들을 저장하고 있는 저장소인 Docker Registry (도커 레지스트리)

![image](https://user-images.githubusercontent.com/93081720/211159053-d97811b7-98b8-4193-8217-64e6560d0b9b.png)

![image](https://user-images.githubusercontent.com/93081720/219390654-2cf9c1a5-3d26-4339-bd2d-43249784bd77.png)

## 1. 서버로서의 도커(Docker Host)

> 실제로 컨테이너를 생성하고 실행하며, 이미지를 관리하는 주체

`dockerd` 프로세스로 동작

### 1) 도커 데몬(Docker Daemon)

`/usr/bin/dockerd`

도커 엔진(Docker Engine)은 외부에서 API 입력을 받아 도커 엔진의 기능을 수행하는데, 도커 프로세스가 실행되어 서버로서 입력을 받을 준비가 된 상태를 도커 데몬이라고 한다.

`docker daemon` →  `dockerd`

<br>

## 2. 클라이언트로서의 도커(Docker Client)

> 도커 데몬이 API 입력을 통해 동작할 수 있도록 CLI를 제공하는 주체. 도커 데몬에게 도커 명령어를 전달할 수 있도록 하는 역할

### 1) 도커 클라이언트(Docker CLI)

`/usr/bin/docker`

사용자가 `docker`로 시작하는 명령어를 입력하면 도커 클라이언트를 사용하는 것임.

도커 클라이언트는 이러한 명령어를 API로서 dockerd(도커 데몬)으로 보내고, 도커 데몬이 명령어를 수행하게 됨.

보다 상세히 말하자면, 도커 클라이언트는 `/var/run/docker.sock`에 있는 도커 소켓(유닉스 소켓)을 통해 도커 API를 호출하고, API는 도커 데몬에 전달되어 명령어를 수행하는 것임.

결국 컨테이너나 이미지를 다루는 명령어는 `/usr/bin/docker`에서 실행되지만, 도커 엔진 프로세스는 `/usr/bin/dockerd` 로 실행되고 있음. 이는 도커 명령어가 실제 도커 엔진이 아니라 클라이언트로서의 도커라는 것을 알 수 있음.

<br>

## 3. Docker Socket(docker.sock)

> 도커 소켓

`docker.sock`은 도커 컨테이너 내부에서 데몬과 상호 작용할 수 있게 해주는 Unix 소켓이다.

`/var/run/docker.sock`에 위치하고 있으며, 기본적으로 메인 도커 데몬(Docker Daemon)과 통신하기 위해서 사용된다.

즉, Docker Socket은 Docker API의 진입점이며, 도커 소켓은 기본적으로 Docker CLI에 의해 도커 커맨드를 실행하기 위해 사용된다.

<br>

## 4. Docker Registries

> Docker Hub

Docker 이미지들이 저장되는 registries. Docker Hub는 누구나 사용할 수 있는 공용 레지스트리이며, Docker는 기본적으로 `FROM`커맨드를 통해서 이미지를 불러올 때 기본적으로 Docker Hub에서 이미지를 찾는다.