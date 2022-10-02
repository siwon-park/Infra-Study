![Docker image](https://user-images.githubusercontent.com/93081720/174341063-d8894c50-7452-49b0-ae2f-7a4b019dc8a9.png)

# 03_Docker_Volume

## 01_데이터

데이터의 분류 => 어플리케이션, 임시 앱 데이터, 영구 앱 데이터

![image](https://user-images.githubusercontent.com/93081720/180634741-52a53d58-f5e3-4d89-9cec-92dd4379ee7c.png)

### 어플리케이션

- 이미지에 쓰는 코드들은 이미지가 일단 빌드되고 나면 컨테이너 실행 중에 수정을 해도 반영되지 않고 이미지를 다시 빌드해야 적용이 되므로 읽기 전용 데이터임



### 임시 앱 데이터

- 컨테이너 실행 도중에 생성되고 불러오는 데이터로, 메모리나 임시 파일에 잠시 저장되는 데이터
- 컨테이너를 삭제하면 동시에 삭제되는 데이터(컨테이너가 중지된다고 해서 무조건 임시 데이터가 삭제되는 것은 아님)

- 예) 유저 인풋



### 영구 앱 데이터

- 테이너 실행 도중에 생성되고 불러오는 데이터로, 파일이나 데이터베이스에 저장되는 데이터
- 컨테이너가 중지, 재시작되더라도 데이터가 삭제되어서는 안 됨

- 예) 유저 계정 정보

```
docker build -t feedback-node .

docker run -p 3000:80 -d --name feedback-app --rm feedback-node:latest
```

<br>

## 02_Volume(볼륨)

> 컨테이너와 맵핑된 호스트 머신(내 컴퓨터)의 폴더, 하드 드라이브 저장 공간
>
> 컨테이너 외부(호스트 머신)의 특정 폴더에 연결된 도커 컨테이너 내부의 폴더/파일

컨테이너를 중지하더라도 볼륨은 여전히 유지되며, 컨테이너를 (재)시작하면 해당 볼륨의 데이터를 컨테이너에서 사용 가능함

컨테이너는 볼륨에 있는 데이터를 읽고/쓸 수 있음

![image](https://user-images.githubusercontent.com/93081720/180636189-a5091868-6dd1-43de-86ec-aeb14c3cfd99.png)

### 기본 명령어

- `docker volume --help` : volume에 대한 명렁어를 알아볼 수 있음
- `docker volume ls` : (활성화된) 볼륨 리스트를 출력함



### 익명 볼륨(Anonymous Volume)

> 이름을 따로 명명하지 않은 볼륨

Dockerfiled에 작성하여 볼륨 생성 가능

`VOLUME ["경로"]`

![image](https://user-images.githubusercontent.com/93081720/183292460-aa00b626-9795-4fa9-b333-c02bdd418895.png)

- 경로
  - 여기서 말하는 "/app/feedback"과 같은 "경로"는 실제 우리 컴퓨터(호스트 머신)의 디렉토리 위치가 아니라, 내 컨테이너 내부의 위치임
  - 도커가 관리하며, 개발자는 실제 해당 위치가 호스트 머신 상 어디에 있는지 정확하게 모름(어디 있는지 반드시 알 필요 없음)

**익명 볼륨은 컨테이너가 종료되면 없어진다(단, 컨테이너가 --rm 명령어로 시작된 경우에만 해당) **

#### 익명 볼륨 제거하기

컨테이너가 제거되면, 익명 볼륨은 자동으로 제거됨. 이는 '`--rm`' 옵션으로 컨테이너를 시작/실행할 때 발생하게 됨

그러나 `--rm` 옵션 없이 컨테이너를 시작하면, 컨테이너를 (`docker rm ... `으로) 제거하더라도 **익명 볼륨은 제거되지 않음**

그래도 컨테이너를 다시 만들어, 다시 실행하면(즉, docker run ... 다시 실행), **새 익명 볼륨이 생성됨** 

즉, 익명 볼륨이 자동으로 제거되지 않았지만, 다음에 컨테이너가 시작될 때, 다른 익명 볼륨이 연결되기 때문에, 이전 컨테이너를 제거하고 새 컨테이너를 실행하는데 도움이 되지 않음

이제 **사용하지 않는 익명 볼륨이 쌓이기** 시작하는데,  '`docker volume rm <볼륨명>`' 또는 '`docker volume prune`'을 통해 **볼륨을 삭제** 가능



익명 볼륨은 외부 경로보다 컨테이너 내부 경로의 우선 순위를 높이는 데 사용 가능(바인딩 마운트 예시 참조)



### 기명 볼륨(Named Volume)

> 이름을 따로 정의해준 볼륨

익명 볼륨과 마찬가지로 도커에 의해 관리되며 컨테이너가 종료되어도 우리의 하드디스크 상에 남아있어 컨테이너를 다시 켜도 볼륨을 사용 가능함

- 기명 볼륨 생성하기
  - 컨테이너를 만들 때(run할 때) 지정해줌
  - `-v <볼륨명>:<주소>`
  - 예) `docker run -d -p 3000:80 --rm --name feedback-app -v feedback:/app/feedback feedback-node:volumes`



### 바인드 마운트(Bind Mount)

> 개발자에 의해 관리되며, 호스트 머신 상 맵핑된 폴더/주소를 지정 가능함 => 필요에 따라 우리가 직접 수정 가능

Docker file로 빌드하는 소스코드를 스냅샷으로 복사하지 않고 바인드 마운트에 복사하는 개념

일반적인 사용 목적 => 컨테이너에 이미지 리빌딩 필요없이 '라이브 데이터'를 제공하기 위함

컨테이너를 생성하면서 `-v <절대경로:맵핑할 주소>` 형태로 사용

- 예) `-v "/User/siwon-park:/app"` => 맵핑할 경로까지 포함하여 ""(따옴표)로 감싸주는 것이 좋다. 왜냐하면 폴더/주소명에 특수문자나 공백이 있을 경우에 따옴표 없이 사용할 경우 인식을 제대로 못해 에러가 발생하기 때문임
- 바로 가기(직접 절대 경로를 복사하지 않고 다음과 같이 바로 가기 형태로 사용 가능)
  - `-v "%cd%":/app` (Window)
  - `-v $(pwd):/app` (MacOs/Linux)



#### 바인드 마운트 이해하기

도커는 로컬에 있는 파일을 오버라이트하지 않고, 대신에 도커 컨테이너에 있는 파일을 오버라이트함

=> 경우에 따라 외부 파일로 덮어쓰지 말아야할 것들을 알려줘야하는 경우도 생김

![image](https://user-images.githubusercontent.com/93081720/183462229-e496b5f9-0b58-442c-8807-00b9f9fc8bca.png)

이게 무슨 말이냐하면 → 원래대로 이미지만 빌드한다면 컨테이너 내부에서 Node.js 환경을 구동하기 위해 /app이라는 컨테이너 내부 폴더에 package.json을 복사하고, npm install을 하여 `node_modules`폴더를 생하고, 로컬에 있는 내용의 스냅샷을 /app에 복사했었는데,

여기서 만약 바인딩 마운트를 생성한다면 로컬에 있는 내용을 컨테이너 내부의 /app에 오버라이트하는 것이기 때문에 이미지를 빌드하여 npm install 명령어를 통해 생성된 `node_modules`폴더가 사라지게 된다. 그래서 그냥 바인딩 마운트를 생성한다면 Node.js를 실행하기 위한 환경을 갖추지 못하게 되므로 에러가 발생하게 된다.

이 때, `익명 볼륨`을 통해서 이러한 문제를 해결할 수 있다 => 도커는 경로를 채택할 때, 더 길고 더 구체적인 경로를 우선으로 하여 채택하기 때문에 이것이 가능하다

예)

![image](https://user-images.githubusercontent.com/93081720/183463021-092d23b7-5f7c-400e-b657-5b6adeaaeb7c.png)

바운드 마운트를 생성할 폴더 주소인 /app보다 익명 볼륨이 생성될 위치인 /app/node_modules가 더 길고 더 구체적이므로 우선순위를 가져서 npm install을 통해 생성된 node_modules폴더가 살아남게 되고, /app에는 바인딩 마운트된 로컬 폴더의 내용으로 덮어씌어 진다

 

※ nodemon => 파일의 변경을 감지하여 Node.js와 같은 런타임 환경을 다시 켤 필요 없이(즉, 컨테이너를 중지했다가 다시 실행할 필요 없이) 변경 내용을 반영한 파일로 재실행 시켜주는 패키지

단, WSL2를 통해 도커를 실행하는 윈도우 환경의 경우에는 동작하지 않는다.

- package.json에 scripts와 devDependencies를 설정해준다

![image](https://user-images.githubusercontent.com/93081720/183463904-eb87e1dd-5204-433e-b475-3b4c969dfeae.png)

- 이미지 빌드를 위한 Dockerfile을 수정해준다

![image](https://user-images.githubusercontent.com/93081720/183464039-f7028eb7-a551-44ff-a73a-d01339ceb204.png)

<br>

#### ※ 바인드 마운트 명령어 쉽게 입력하기

바인드 마운트 경로를 직접 입력하기란 쉽지 않다. 이를 쉽게할 수  있는 팁이 있다.

- 바인드 마운트 연결을 하고 싶은 로컬 호스트 머신의 위치에 들어간다.
- 해당 위치에 존재하는 아무 파일을 선택하여 마우스 우클릭으로 경로를 복사한다.
- 복사한 경로에서 해당 파일 경로를 지우고, 그 뒤에 바로 `:/컨테이너 디렉토리 경로`로 바인드 마운트 맵핑시켜준다.

<br>

### 각 볼륨 구분법

- 콜론(:) 앞에 로컬 머신의 경로가 붙어있다 => 바인드 마운트
- 콜론(:) 앞에 경로가 아닌 명칭이 붙어있다(이름으로 취급됨) => 기명 볼륨
-  아무것도 없이 -v 다음에 바로 경로가 나온다 => 익명 볼륨

<br>

### 읽기 전용 볼륨(Read-Only Volume)

불륨의 default는 Read-Write이나 Read-Only모드로 생성할 수 있음

=> 바인드 마운트의 경로 끝에 `:ro`를 추가하면 됨

```
docker run -d 3000:80 --name feedback-app -v feedback:/app/feedback -v "Users/siwon-park/myfolder:/app:ro" -v /app/node_modules feedback-node:volumes
```

<br>

### 볼륨 관리하기

- `docker volume ls` : 활성화된 볼륨 조회
  - 바인드 마운트는 도커에 의해 관리되지 않기 때문에 ls 명령어로 조회되지 않음
- `docker volume create <볼륨명>` : 볼륨을 (수동으로) 생성함
  - 터미널에서 컨테이너 생성 시 -v로 볼륨을 생성한다면, 해당 컨테이너의 볼륨을 도커가 자동으로 생성해줌
  - 컨테이너 생성 후 볼륨이 필요할 경우 create 명령어를 사용하여 볼륨을 생성함
- `docker volume inspect <볼륨명>` : 볼륨에 대해서 검사하여, 볼륨에 대한 정보를 파악 가능함
  - 생성일, mountpoint 등에 대해 파악할 수 있음
    - mountPoint: 실제 데이터가 저장되는 호스트 머신상의 경로. 실제 경로는 아니며, 우리의 시스템에 도커가 설정한 가성 머신 내부에 존재하는 경로 => 사실상 찾기가 매우 어려움  

- `docker volume rm <볼륨명>` : 볼륨을 삭제함. 단, 실행 중인 컨테이너의 볼륨은 삭제가 불가능함. 컨테이너 중지 이후에 실행해야함

<br>

### COPY 명령어 vs. 바인드 마운트

개발 중에는 바인드 마운트를 사용해서 소스 코드의 변경 사항을 즉각적으로 반영할 수 있다. 그러나 실제 프로덕션 단계에서는 바인드 마운트를 사용하지 않는다. 왜냐하면 실제 개발이 끝나서 서버에서 컨테이너를 실행 시 바인드 마운트로 실행하지 않기 때문이다.

물론 볼륨을 사용할 수는 있지만, 바인드 마운트는 호스트 머신의 실제 주소이므로, 연결된 소스코드가 없어 사용하지 않는다.

실제 배포 단계에서는 코드의 스냅샷이 필요하기 때문에 COPY를 쓴다.

<br>

### .dockerignore

> COPY 명령어로 복사해서는 안 되는 폴더와 파일을 지정 가능함
>
> 어플리케이션이 실행되는데 필요없는 모든 것을 추가 가능함

- 예를 들어, .dockerignore 파일을 만들고 그 안에 node_modules라고 썼다면

```dockerfile
RUN npm install # 빌드 시, 이미지 내부에 node_modules 생성
COPY . . # 로컬에 있는 node_modules를 복사하여 위에서 npm install로 생성한 node_modules를 뒤엎음
```

- 그런데, 만약에 로컬에 있는 node_modules가 구버전이라면? 또는 굳이 이미지 내부에 복사한 node_modules와 완전히 동일한데 로컬에 있는 것을 복사하는 과정을 쓸 데 없이 할 필요가 있을까?
- `.dockerignore`를 사용하면 이미지 내부의 node_modules를 사용하기 때문에 이런 불필요한 행동을 사전에 방지할 수 있음

<br>

### 환경변수(.env)

> 환경 변수를 사용하여 docker run 명령어를 바꾸거나 Dockerfile을 하드 코딩하는 방식으로 값을 변경하지 않고 유연하게 원하는 변수 값을 변경 가능함

- Dockerfile

![image](https://user-images.githubusercontent.com/93081720/187053054-02f73998-8850-4722-a937-b3ffd14d5867.png)

- PORT값을 사용하는 소스코드(server.js)

![image](https://user-images.githubusercontent.com/93081720/187053060-fa303f4c-f471-44af-bbd2-6fb8768c4471.png)

#### 방법1) docker run 명령어에서 설정

`--env key=value` 형태로 지정함

- 예) PORT를 8000으로 변경하고 싶을 때 `--env PORT=8000` 또는 `-e PORT=8000`

```
docker run -d --rm -p 3000:80 --env PORT=8000 --name feedback-app -v feedback:/app/feedback -v "/Users/siwon-park/my-folder:/app:ro" -v /app/node_modules -v /app/temp feedback-node:env
```

#### 방법2) .env 파일 생성

![image](https://user-images.githubusercontent.com/93081720/187053184-bcaa2d58-95e3-4092-9ee8-193eb1de970f.png)

`--env-file ./.env`를 통해서 .env 파일에서 정의한 환경 변수를 불러 올 수 있음

- 예)

```
docker run -d --rm -p 3000:80 --env-file ./.env --name feedback-app -v feedback:/app/feedback -v "/Users/siwon-park/my-folder:/app:ro" -v /app/node_modules -v /app/temp feedback-node:env
```

<br>

### 빌드변수(ARG)

> 변경사항이 있을 때, Dockerfile의 값을 변경하지 않고 보다 유연성있게 이미지를 빌드 가능함

다음과 같이 Dockerfile에 설정하여 사용 가능함

![image](https://user-images.githubusercontent.com/93081720/187053253-3986a25b-e9c7-46db-822b-642f3ba30dac.png)

단, 해당 명령어는 RUN npm install과 같이 실행 프로세스가 오래 걸리는 명령어 보다 나중에 쓰는 것이 바람직하다.

=> Why? ARG 명령어의 값이 변경 시, 후속 레이어 또한 다시 빌드/재실행됨

=> 포트 값만 변경된 것인데 굳이 npm install을 다시 할 필요는 없음

- 빌드 시 사용 예시

```
docker build -t feedback-node:dev --build-arg DEFAULT_PORT=8000
```

Dockerfile에 DEFAULT_PORT가 80으로 지정되어 있어도 8000으로 바꿔 빌드함

