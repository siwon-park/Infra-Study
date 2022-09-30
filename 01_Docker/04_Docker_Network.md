![Docker image](https://user-images.githubusercontent.com/93081720/174341063-d8894c50-7452-49b0-ae2f-7a4b019dc8a9.png)

# 04_Docker_Network

## 01_컨테이너와 통신

>  ※ 엔드 포인트: 요청을 받아 응답을 제공하는 서비스를 사용할 수 있는 지점

### case1) Http(WWW, API 요청)

컨테이너에서 외부 API나 http 웹 주소로 언제든지 통신 가능하며, 따로 별 다른 옵션을 정의할 필요가 없다

<br>

### case2) 로컬 호스트 머신

컨테이너 내부에서 로컬 호스트 머신과 통신하는 것

`host.docker.internal` 이라는 도커에 의해 지정된 주소로 로컬 호스트 머신과 통신이 가능함

- 컨테이너 내부에서 로컬 호스트 머신의 IP주소로 자동 변환되어 로컬의 app을 listening 가능해진다.
- 예) mongodb://**host.docker.internal**:27017/swfavorites

<br>

### case3) 컨테이너

컨테이너 간에도 통신이 가능하다. 이 때, 컨테이너 - 로컬 호스트 머신 간 연결처럼 여러 설정이 필요하고 방법이 몇 개 있다.



#### 방법1) 컨테이너 내부의 IP주소를 이용하여 컨테이너 간 연결

- 도커 허브에서 mogoDB의 이미지를 가져와서 mongodb라는 이름으로 컨테이너를 실행함

```bash
docker run -d --name mongodb mongo
```

- 컨테이너에 대한 상세 정보를 조회함
  - `docker container inspect [컨테이너 이름]`

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

<br>

#### 방법2) 컨테이너들을 네트워크화하여 연결

--network 옵션

- 컨테이너 실행 시 해당 컨테이너를 지정한 네트워크로 밀어 넣을 수 있음 
- `docker run --network [네트워크 이름] ...`
  - 예) docker run -d --name mongodb **--network favorites-net** mongo

그러나 바로 위의 예시처럼 사용하면 에러가 발생한다

왜냐하면 네트워크가 없기 때문

☆★ 도커는 볼륨과 달리 네트워크의 경우 옵션으로만 자동 생성해주지 않는다! => 따라서 직접 만들어야함

- `docker network create [네트워크명]`
  - 예) docker network create favorites-net
- `docker network ls`
  - 네트워크 목록들을 조회 가능

네트워크를 create했다면 해당 네트워크 이름으로 docker run을 할 때 사용 가능하며 더 이상 에러가 발생하지 않는다.

<br>

컨테이너 실행 시 정의했던 컨테이너 이름을 IP처럼 적용하여 사용 가능해진다.

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

<br>

☆★ 특이점: 컨테이너 간에 연결이 있다면 연결되는 컨테이너는 포트(-p 옵션)를 게시할 필요가 없다.

메인 앱의 경우 `docker run --rm -d -p 3000:3000 --name favorites favorites-node`과 같이 실제 포트를 연결해주었지만, db로 사용할 연결될 컨테이너는 `docker run -d --name mongodb --network favorites-net mongo`과 같이 따로 포트를 열어주지 않아도 된다

=> 컨테이너 내부에서 모든 컨테이너가 자유롭게 통신이 가능하기 때문

<br>