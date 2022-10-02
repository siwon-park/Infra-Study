# 07_Utility_Container

유틸리티 컨테이너

> 특정 환경만 포함하는 컨테이너
>
> 특정 작업을 실행하기 위해 지정한 명령과 함께 실행되는 컨테이너

<br>

## 왜 사용하는가?

실행 중인 컨테이너에 특정 작업을 수행하도록 명령하게 하거나

전역에 직접 환경을 설치하지 않고 환경을 구성하고 싶을 때 사용 가능

### docker exec

> 실행 중인 컨테이너에 특정 명령을 실행하게 함

`docker exec [컨테이너 명] [특정 명령어...]`

- 예) docker exec -it node npm init
  - (공식 node 이미지를 가져온 다음에) node 컨테이너를 실행하면서 npm init으로 Node.js 프로젝트 환경을 구성함

<br>

### docker-compose run

여러 services가 있더라도, 서비스명을 지정하여 단일 서비스를 실행할 수 있다.

- 예) docker-compose run --rm npm init
  - 여기서 npm은 docker-compose.yml에 작성한 서비스명임(npm명령어가 아니다)
  - init은 Dockerfile에 `ENTRYPOINT ["npm"]`으로 작성하여 default 수행 명령어를 지정해줬기 때문에 가능함

<br>

### ※ RUN, CMD, ENTRYPOINT 명령어의 차이

#### RUN

> 이미지를 빌드하는 시점에 실행되는 명령어

보통 컨테이너 실행 환경을 구축하기 위해 필요한 dependencies나 라이브러리들을 설치하기 위해 사용됨

#### CMD

> 컨테이너 실행 시 수행되는 명령어

Dockerfile에 한번만 등장해야 하는 명령어로, 컨테이너를 실행하면서 수행되어야하는 명령어를 지정하여 수행하게 한다.

단, ENTRYPOINT와 차이가 있다면, CMD 명령어는 default 수행 명령어가 아니기 때문에, docker run 명령어 작성 시 다른 수행 명령어를 입력한다면 오버라이딩 되어 오버라이딩 된 명령어가 수행된다.

#### ENTRYPOINT

> 컨테이너 실행 시 수행되는 명령어

컨테이너를 실행하면서 수행되어야하는 명령어를 지정하여 수행하게 한다.

단, CMD와 차이가 있다면, ENTRYPOINT는 default 명령어로써 항상 수행된다. 그래서 보통 컨테이너 실행 후 변경되지 않을 명령어를 지정하여 사용한다.