![Docker image](https://user-images.githubusercontent.com/93081720/174341063-d8894c50-7452-49b0-ae2f-7a4b019dc8a9.png)

# 04_Docker_Network

도커 네트워크

## 1. 컨테이너의 통신

>  ※ 엔드 포인트: 요청을 받아 응답을 제공하는 서비스를 사용할 수 있는 지점

컨테이너와 타겟 간의 통신

### 1) HTTP (API 요청)

컨테이너에서 외부 API나 http 웹 주소로 언제든지 통신 가능하며, 이 때는 따로 별 다른 옵션을 정의할 필요가 없다.

<br>

### 2) 로컬 호스트 머신

컨테이너 내부에서 로컬 호스트 머신과 통신하는 것.

#### (1) host.docker.internal

도커 컨테이너에서 호스트 머신(로컬)에 실행 중인 서비스로 접근할 때 사용하는 특별한 DNS.

`host.docker.internal` 이라는 주소를 사용해야 호스트 머신의 로컬에서 실행 중인 서비스와 통신이 가능하다.

컨테이너 내부에서 로컬 호스트 머신의 IP주소로 자동 변환되어 로컬의 app을 listening 가능해진다.

- 예) 스프링부트 프로젝트를 호스트 머신에 실행 중인 상황: localhost:8080
  - 컨테이너로 실행 중인 서비스에서 스프링부트 프로젝트에 요청을 보내려면 `http://host.docker.internal:8080/~`과 같은 형태로 요청을 보내야 함.

- 예) Mongo DB를 로컬에서 실행 중일 때, 컨테이너의 서비스가 해당 DB에 붙고자 하는 경우
  - `mongodb://host.docker.internal:27017/swfavorites`로 요청을 보내야 한다.
    - `mogodb://`: 프로토콜 구분자(스킴, scheme), 몽고 DB에 붙겠다는 의미로 웹 요청 시 http://, https://를 붙이는 것과 같은 원리.
      - MySQL일 경우 mysql://, PostgreSQL일 경우 postgresql://로 시작함.
    - swfavorites: 데이터베이스명

<br>

### 3) 컨테이너 간 통신 (Network)

도커 컨테이너 간에도 서로 통신이 가능하다. 이 때, 컨테이너 - 로컬 호스트 머신 간 연결처럼 설정이 필요하며, 여러 가지 방법이 존재한다.

#### (1) 컨테이너 내부의 IP주소를 이용하여 컨테이너 간 연결

- 도커 허브에서 mogoDB의 이미지를 가져와서 mongodb라는 이름으로 컨테이너를 실행함

```bash
docker run -d --name mongodb mongo
```

- 컨테이너에 대한 상세 정보를 조회함
  - `docker container inspect [컨테이너 명]`

```bash
docker container inspect mongodb
```

- 컨테이너의 IP 주소를 찾아서 컨테이너 간 연결하는 데 사용 가능

```javascript
mongoose.connect(
  'mongodb://172.17.0.2:27017/swfavorites',
  { useNewUrlParser: true },
  (err) => {
    if (err) {
      console.log(err);
    } else {
      app.listen(3000);
    }
  }
);
```

##### 1 - 예시

grafana(3030포트)와 prometheus(9090 포트)를 도커 컨테이너로 실행하고, 스프링부트를 로컬(localhost:8080)에 실행하여 acutator로 통해 헬스 체크를 하고자 한다면, prometheus가 actuator 헬스 체크한 데이터를 grafana에서 가져가야 한다.

이 때 grafana에서 prometheus로 연결을 해야 하는데, prometheus의 주소는 localhost가 아니라  `http://prometheus:9090/`으로 작성해야 한다. 왜냐하면 도커 컨테이너로 실행 중이기 때문에 컨테이너 명으로 호출해야 하기 때문이다.

- 호스트 머신에서 스프링부트, grafana, prometheus 등의 서비스에 접근할 때는 `http://localhost:[포트번호]`로 접속하면 접근이 가능하지만, 컨테이너 간 통신에 있어서는 localhost가 아닌 컨테이너의 이름을 사용해야 한다.
  - 그라파나에서 입력하는 localhost:9090에서 localhost는 그라파나의 컨테이너 자기 자신을 의미한다. 프로메테우스 컨테이너가 아니다.
  - prometheus:9090은 도커 네트워크에서 prometheus라는 이름의 컨테이너를 의미한다.

<br>

#### (2) 컨테이너들을 네트워크화하여 연결

##### 1- 네트워크 생성 후 컨테이너 실행

- docker network ls

  - 네트워크 목록들을 조회할 수 있는 명령어

- `--network` 옵션 (컨테이너 실행 시)

  - 컨테이너 실행 시 해당 컨테이너를 지정한 네트워크로 밀어 넣을 수 있음.

  - `docker run --network [네트워크 명] ...`
    - 예) docker run -d --name mongodb `--network favorites-net` mongo
  - ※ 그러나 바로 위의 예시처럼 사용하면 에러가 발생한다. => 왜냐하면 네트워크가 없기 때문.
  - ☆★ 볼륨(volume)과 달리 네트워크의 경우 없을 경우에 자동 생성해주지 않는다! => 따라서 직접 만들어야 한다.


- `docker network create [네트워크 명]`
  - 예) docker network create favorites-net
  - 네트워크를 생성했다면 도커 실행 시 해당 네트워크를 사용 가능하며 에러가 발생하지 않는다.



##### 2 - 컨테이너 실행 후 네트워크 연결

컨테이너를 실행한 뒤에도 네트워크를 구성할 수 있다. 단, 역시 네트워크가 생성되어 있어야만 한다.

- `docker network connect [네트워크 명] [컨테이너 명]`
  - 예) docker network connect my_network springbootback


컨테이너 실행 시 정의했던 컨테이너 이름을 IP처럼 적용하여 사용 가능해진다.

- mongodb://mongodb:27017/swfavorites에서 `mogodb:27017`이 `[컨테이너 이름]:[포트]`에 해당한다.

```javascript
mongoose.connect(
  'mongodb://mongodb:27017/swfavorites', // mongodb라는 이름의 컨테이너가 있으면 사용 가능
  { useNewUrlParser: true },
  (err) => {
    if (err) {
      console.log(err);
    } else {
      app.listen(3000);
    }
  }
);
```

☆★ 컨테이너 간에 같은 네트워크로 구성을 하면 연결되는 컨테이너는 포트(-p 옵션)를 게시할 필요가 없다.

메인 앱의 경우 `docker run --rm -d -p 3000:3000 --name favorites favorites-node`과 같이 실제 포트를 연결해주었지만, db로 사용할 연결될 컨테이너는 `docker run -d --name mongodb --network favorites-net mongo`과 같이 따로 포트를 열어주지 않아도 된다

=> 같은 네트워크에서 컨테이너 간 자유롭게 통신이 가능하기 때문

<br>

## 2. 네트워크의 종류

네트워크는 네트워크 드라이브에 따라 종류가 다르다.

### 1) 네트워크 드라이브

- `bridge` : 하나의 호스트 컴퓨터 내에서 여러 컨테이너들이 서로 소통할 수 있음
- `host` : 컨테이너를 호스트 컴퓨터와 동일한 네트워크에서 실행하기 위해 사용 
- `overlay` : 여러 호스트에 분산되어 돌아가는 컨테이너들 간에 네트워킹을 위해 사용함

#### (1) 브릿지(bridge)

기본 네트워크 타입 (도커가 자동 생성)

- 같은 브릿지 네트워크에 있는 컨테이너 간에는 컨테이너 이름(혹은 ip)으로 통신 가능
- `호스트 머신과는 별개의 네트워크 공간`(호스트 머신과 네트워크 격리)
  - localhost는 호스트 머신 혹은 컨테이너 내부의 자신을 가리킴
  - 컨테이너 → 호스트 머신 간 통신은 host.docker.internal을 사용해야 하며 포트 바인딩이 필수임
  - 컨테이너 - 컨테이너 간 통신은 컨테이너 이름으로 접근

- 주로 단일 호스트 내 여러 컨테이너를 하나의 네트워크로 묶을 때 사용
- 호스트 머신과 네트워크가 격리되어 있기 때문에 `포트의 중복은 허용`됨
  - ※ 포트의 중복 허용 != 포트 바인딩 시 중복 포트 허용
  - 포트의 중복을 허용 한다는 것은 여러 컨테이너가 중복된 포트로 서비스가 실행 가능하다는 의미이다.
    - 스프링부트 프로젝트를 다중으로 실행하고자 할 때, 컨테이너 내부에서는 같은 포트(예: 8080)으로 서비스를 실행할 수 있다.
    - 각 컨테이너 안에서는 포트가 자기 자신만의 것이기 때문에 가능한 것임

```bash
docker run -p 8080:8080 컨테이너A
docker run -p 8081:8080 컨테이너B
```

- 컨테이너 A의 8080 포트를 호스트 머신의 8080 포트에 바인딩
- 컨테이너 B의 8080 포트를 호스트 머신의 8081 포트에 바인딩

호스트 포트의 바인딩 중복과 컨테이너의 포트 중복은 다른 개념이다.

#### (2) 호스트(host)

컨테이너가 호스트 머신의 네트워크 스택을 그대로 사용하는 네트워크 구성

- 호스트 머신과 네트워크 격리가 없음 (호스트의 네트워크를 공유)
- 컨테이너 - 컨테이너, 컨테이너 - 호스트 머신 간 통신 시 localhost로 통신하면 됨
  - 모든 컨테이너가 호스트 네트워크를 공유하기 때문에 `포트 중복이 허용되지 않음`
    - 같은 네트워크를 공유하기 때문에 포트 바인딩이 필요 없는 대신에 포트가 중복되면 충돌이 발생함.
    - 즉 스프링부트 프로젝트 2개를 컨테이너로 실행하고자 할 때, A가 8080을 썼다면 B는 8081로 서비스를 실행해야 함.

#### (3) 오버레이(overlay)

여러 호스트 서버를 하나의 가상 네트워크로 묶어서 컨테이너 간에 통신을 하도록 구성하는 방식

컨테이너들이 서로 다른 호스트에 있어도 같은 네트워크에 있는 것처럼 통신할 수 있는 방식을 의미

- 컨테이너들이 실제 물리적으로는 각각 서로 다른 서버에 있지만, 오버레이 네트워크를 통해서 서로 DNS나 ip로 통신이 가능하다.
- 내부적으로는 VXLAN(Virtual Extensible LAN)과 같은 기술을 사용하여 통신 간에 패킷 손실 없이 안전하게 통신할 수 있도록 구성한다.
- 주로 Kubernetes와 같이 클러스터 환경에서 사용하는 네트워크 구성 방식이다.

<br>

### 2) 네트워크 연결 해제

하나의 컨테이너는 다른 여러 네트워크와 동시에 연결이 가능하다.

- 기본적으로 실행 시 도커의 기본 네트워크인 bridge와 연결됨(--network를 붙여서 컨테이너를 실행할 경우에는  커스텀 네트워크와 바로 연결)
- `docker network disconnect [네트워크 명] [컨테이너 명]`
  - 예) docker network disconnect my_network springbootback

### 3) 네트워크 제거

#### (1) 특정 네트워크만 제거

- `docker network rm [네트워크 명]` : 특정 네트워크만 제거
  - 단, 이 때 네트워크에 연결된 실행 중인 컨테이너가 없어야 함 => 연결된 모든 컨테이너를 중지시키고 나서 네트워크 제거 가능

#### (2) 사용하지 않는 네트워크 전부 제거

- `docker network prune`: 사용하지 않는 불필요한 네트워크 제거

