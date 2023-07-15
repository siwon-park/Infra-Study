![image](https://user-images.githubusercontent.com/93081720/212346428-0b1fadf6-f630-4107-b9ae-9b81057b1d4c.png)

# 02_Jenkins CI/CD 환경 구성 - DooD

> Jenkins 내부에 도커를 설치하지 않고 호스트 머신의 도커 데몬을 활용하여 CI/CD 환경 구성(Docker out of Docker)

Jenkins 컨테이너화 이후 과정은 DinD에서의 진행 방식과 달리 GitHub 배포로 진행한다.

<br>

## 1. 개요

환경 구성 개요는 아래 그림과 같다.

<img src="https://user-images.githubusercontent.com/93081720/219939587-38c40716-f787-494e-93e6-98eec36398c7.png" referrerpolicy="no-referrer" alt="image" height="500px">

<br>

## 2. 환경 구성

### 1) Docker 설치

> EC2에 Docker Engine을 설치한다. 자세한 사항은 [공식문서](https://docs.docker.com/engine/install/ubuntu/)를 참조할 것

#### set up repository & gpg key 설치

```bash
# 업데이트
sudo apt-get update

sudo apt-get install ca-certificates curl gnupg

# 도커 공식 gpg키 설치
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 레포지토리 셋업
echo \
"deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
"$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### Docker Engine 설치

```bash
# 업데이트
sudo apt-get update

# 최신 버전의 Docker 설치
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 추가 docker-compose 설치(선택)
sudo apt-get install docker-compose
```

#### 설치 확인

```bash
# 도커 버전 확인
sudo docker -v

# 도커 컴포즈 버전 확인(선택)
sudo docker-compose version
```

<br>

### 2) Jenkins 설치

docker-compose.yml 파일 작성 후 진행. 실수했을 경우 `docker run` 명령어에 함께 작성해야할 볼륨 옵션이나 명령어들이 많아서 docker-compose로 실행하는 것이 편하다.

#### docker-compose.yml 파일 작성

```bash
sudo vim docker-compose.yml
```

#### 컨테이너 스펙 정의

> ☆★ docker socket과 docker cli를 볼륨 마운트★☆

호스트 머신의 도커 데몬과 소통하기 위해 docker socket을 볼륨 마운트한다. 그러나 여기까지만 하면 Jenkins 내부에서 호스트 머신의 도커 데몬과 소통 가능한 환경은 구성했지만, docker-cli 환경이 없기 때문에 명령어를 작성할 수 없다. 따라서 docker-cli를 Jenkins 컨테이너가 사용할 수 있도록 docker-cli도 볼륨 마운트 해준다.

- `/var/run/docker.sock:/var/run/docker.sock`: docker socket 볼륨 마운트
- `/usr/bin/docker:/usr/bin/docker`: docker-cli 볼륨 마운트

```yml
version: '3'

services:
    jenkins:
        image: jenkins/jenkins:lts
        container_name: jenkins
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
            - /usr/bin/docker:/usr/bin/docker
            - /jenkins:/var/jenkins_home
        ports:
            - "9090:8080"
        user: root
```

#### 컨테이너 실행

```bash
sudo docker-compose up -d
```

#### Docker 설치 확인

```bash
# jenkins라는 이름의 컨테이너에 bash 환경 접속
sudo docker exec -it jenkins /bin/bash

# docker 버전 확인(컨테이너 내부이기 때문에 sudo를 입력하지 않아도 됨)
docker -v 

# 터미널 상에서 직접 확인
sudo docker exec jenkins docker ps
```

<br>

## 3. Pipeline 프로젝트 생성

DinD 방식과 달리 Pipeline 방식으로 배포한다.

※ 플러그인 설치까지의 과정은 동일하므로 생략함

### 1) 프로젝트 생성

새로운 Item에서 `Pipeline`으로 프로젝트를 생성한다.

![image](https://github.com/siwon-park/Infra_Study/assets/93081720/7c4898a8-2e55-43d5-ac39-00337268c741)

<br>

### 2) 프로젝트 내용 작성

#### Github 연동 레포지토리 입력

Github를 통해 배포할 예정이기 때문에 Github project에 체크한 뒤 연동할 프로젝트의 repository url을 작성한다.

![image](https://github.com/siwon-park/Infra_Study/assets/93081720/de145ca4-307c-4a63-a6fe-a1492ff9ea44)

<br>

#### Builde Trigger 설정

`Github hook trigger for GITScm polling`에 체크한다.

![image](https://github.com/siwon-park/Infra_Study/assets/93081720/5a97cff3-81e1-492c-aa02-1ea871815416)

#### Pipeline Script 작성

Jenkins가 실행할 Job들을 Pipeline 문법으로 Scirpt로 작성한다.

다음은 Springboot 컨테이너 이미지를 빌드하고 실행하는 Pipeline Script 예시이다.

```groovy
pipeline {
 agent any
 
 environment {
   GIT_URL = "[https://깃허브 레포지토리 ]"
 }
 
 stages {
   stage('Pull') {
     steps {
       script {
         git url: "${GIT_URL}", branch: "master", credentialsId: "jenkins_token", poll: true, changelog: true
       }
     }
   }
   
  stage('Stop and Remove Container') {
     steps {
      script {
          try {
              sh 'docker stop springboot && docker rm springboot'
          } catch(e) {
              echo 'fail to stop and remove container'
          }
        }
     }
  }
   
   stage('Build Images') {
      steps {
       script {
           dir('testapp'){
               sh 'docker build -t springboot:lts .'
          }
       }
      }
   }
   
    stage('Deploy') {
     steps {
       script {
           sh 'docker run -d --name springboot -p 8080:8080 springboot:lts'
       }
     }
   }
   
   stage('Finish') {
     steps {
       script {
           // 사용하지 않는 이미지 제거
           sh 'docker images -qf dangling=true | xargs -I{} docker rmi {}'
       }
     }
   }
 }
}
```

<br>

### 3) 인증 정보 등록

Pipeline Script 중 `Pull`이라는 stage에 자세히 보면 `credentialsId`라는 부분이 보일 것이다.

GitHub 레포지토리 정보만 입력한다고 해서 해당 레포지토리로 부터 정보를 아무나 받아올 수 있는 것은 아니다. 따라서 해당 레포지토리의 내용에 접근할 수 있는 인증된 사용자라는 것을 증명해야 한다.

#### Credentials 등록

Jenkins 관리 >> Crendentials에 들어간다.

![image](https://github.com/siwon-park/Infra_Study/assets/93081720/a47176ca-c89b-4910-ac1d-1069e567152c)

그 후 `Stored scoped to Jenkins`라는 항목에서 `(global)`을 클릭한다.

![image](https://github.com/siwon-park/Infra_Study/assets/93081720/02d7dc99-59f6-4974-a169-301dfa4afcde)

좌측 상단에 있는 `Add Credentials`를 누른다.

![image](https://github.com/siwon-park/Infra_Study/assets/93081720/724f5dea-7848-4229-b1ba-ac723dd2af7b)

#### 인증 정보 입력

`Username with password` 방식으로 인증한다.

- Username에는 `Github 아이디`를 입력한다.
- Password에는 `Github Personal Access Token`을 입력한다.
- ID에는 Jenkins credential의 이름을 마음에 드는 것으로 작성한다.

![image](https://github.com/siwon-park/Infra_Study/assets/93081720/65423c71-16da-4294-9f85-f8f036a54cb1)

### 4) Git hub Personal Access Token 발급

난데없이 Password에는 `Github Personal Access Token`을 입력하라고 해서 당황했을 것이다. Token을 발급받는 방법은 아래와 같다.

좌측 상단 프로필 이미지를 누르고 `Settings`메뉴를 선택한다. 그후 우측 메뉴 최하단에서 `Developer settings`를 클릭한다.

#### Settings - Develop settings

| Settings                                                     | Develop settings                                             |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![image](https://github.com/siwon-park/Infra_Study/assets/93081720/6fb7a1bb-d529-4854-b02c-b387b5c0f6f9) | ![image-20230715193209624](C:\Users\SIWON\AppData\Roaming\Typora\typora-user-images\image-20230715193209624.png) |

#### Personal access tokens - Tokens (classic)

`Personal access tokens` - `Tokens (classic)`로 접속하여 `Generate new token`을 선택한다.

![image](https://github.com/siwon-park/Infra_Study/assets/93081720/c39068fd-4bd7-4df1-be0e-09d0d4b195b5)

![image](https://github.com/siwon-park/Infra_Study/assets/93081720/93046255-d6e9-4921-9807-13049d2cd4f1)

#### 토큰 등록

토큰 이름, 만료일, 토큰 권한 범위를 지정해서 생성한다.

![image](https://github.com/siwon-park/Infra_Study/assets/93081720/9fefa57c-5384-4dfe-8173-4cc3e16a1357)

그 후 다음과 같이 생성된 토큰을 볼 수 있고, 토큰값을 복사하여 사용한다.

- 한 번 보고 난 이후부터는 다시 볼 수 없기 때문에 따로 복사하여 저장해서 관리하는 것이 좋다.
- 만약 잊어버려서 도저히 생각이 나지 않는다면 토큰을 삭제 후 재발급 받으면 된다.

![image](https://github.com/siwon-park/Infra_Study/assets/93081720/688561bd-774a-41ce-bd04-b2626e85ee81)

<br>

### 5) Webhook 등록

인증 정보를 등록하고 나서 끝나는 게 아니다. Jenkins가 레포지토리의 변화를 감지하고 작업을 끌어오기 위해서는  `Webhook`이라는 것을 등록해야 한다.

#### Repository Webhook 설정

Webhook을 등록할 Repository에 상단의 `Settings`에 들어간 뒤 `Webhooks`에 들어간다.

![image](https://github.com/siwon-park/Infra_Study/assets/93081720/e01892f8-ba86-4188-8288-52dbce374c40)

#### Webhook url 등록

- `Payload URL`: `https://젠킨스url:포트번호/github-webhook/`와 같이 jenkins url 뒤에 `/github-webhook/`을 추가하여 입력한다.
  - `https://[젠킨스 url]/[젠킨스 프로젝트]/github-webhook/`과 같은 형식으로 작성해야 할 수도 있다.
- `Content type`: `application/json`으로 설정한다.

![image](https://github.com/siwon-park/Infra_Study/assets/93081720/4bea6c82-d2ed-4756-b7cc-16c7c5474ee0)

<br>

## 4. 결과 확인

GitHub 레포지토리에 소스 코드를 업데이트해서 push해서 테스트해보자.

정상적으로 환경 설정 및 파이프라인을 작성했다면 다음과 같이 성공적인 빌드 진행 및 결과 화면을 볼 수 있다.

### Stage View 확인

![image](https://github.com/siwon-park/Infra_Study/assets/93081720/a98b63ef-cb99-45c1-8fc2-1f13ff51f43d)

<br>

### 빌드 실패

빌드가 실패했을 경우 여러 가지 요인이 있을 수 있다.

- 일단 빌드 트리거를 유발해서 빌드를 진행했다면 기본적인 Webhook과 같은 환경 설정은 성공한 상태임을 알 수 있다.
  - 빌드 트리거링 조차 되지 않는다면 연결 설정을 잘못한 것이다.
- 각 stage 별로 log를 확인할 수 있며 log를 확인해서 에러를 잡으면 된다.
  - Pipeline Script에 문제가 있을 수도 있다.
  - 소스 코드 파일명이 잘못되었을 수도 있다.