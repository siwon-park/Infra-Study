![Docker image](https://user-images.githubusercontent.com/93081720/174341063-d8894c50-7452-49b0-ae2f-7a4b019dc8a9.png)![kubernets](https://user-images.githubusercontent.com/93081720/174333422-4e2f7a03-f585-4edf-884c-0af7fea7ac5d.png)![image](https://user-images.githubusercontent.com/93081720/212346428-0b1fadf6-f630-4107-b9ae-9b81057b1d4c.png)

# Infrastructure

## 1. Index

인프라 학습에 대한 내용을 정리하는 레포지토리

[01_도커(Docker)](https://github.com/siwon-park/Infra_Study/tree/master/01_Docker)

[02_쿠버네티스(Kubernates)](https://github.com/siwon-park/Infra_Study/tree/master/02_Kubernates)

[03_젠킨스(Jenkins)](https://github.com/siwon-park/Infra_Study/tree/master/03_Jenkins)

[04_클라우드(Cloud)](https://github.com/siwon-park/Infra_Study/tree/master/04_Cloud)

<br>

## 2. 인프라에 대한 기본적인 이해의 필요성

개발자는 왜 인프라에 대해 알아야 하는가?

`서버 개발자`라는 말이 있다. 여기서 말하는 `서버`란 결국 하나의 컴퓨터이다.

CPU, 메모리, 디스크가 있고 OS(운영체제)가 갖춰진 컴퓨터가 서버 역할을 하는 것이고, 서버 개발자는 결국에는 이런 환경에서 동작할 수 있는프로그램, 로직 등을 개발하는 사람이다.

이렇게 개발한 것을 외부로 노출시키는 것을 `배포`라고 한다. 개발자는 개발에 대해 잘 아는 것도 중요하지만, 개발의 최종 목표가 배포이기 때문에 기본적인 인프라에 대한 개념이나 흐름은 알아두는 것이 좋다.

결국 본인이 개발하고 있는 환경과 자원에 따라서 사용자에게 전달되는 방식이 달라지기 때문에, 개발과 인프라는 뗄래야 뗄 수 없는 관계이다.