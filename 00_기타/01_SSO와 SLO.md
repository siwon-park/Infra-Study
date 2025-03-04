# 01_SSO와 SLO

> SSO(Single Sign On)과 SLO(Single Log On)

여러 시스템을 운영 중일 때, 각 시스템별로 계정/인증 정보를 따로 가지고 서비스를 하는 경우가 많다.

이 때 각 시스템에 접근을 하려면 모든 서비스별로 계정을 다 기억하고 있어야 하며, 별도로 로그인을 해야 한다는 불편함이 있다.

또한 때로는 관리 측면에서도 계정 정보가 분산되어 관리가 어렵다는 단점도 있다.

이를 해결하기 위해 도입된 아키텍처가 SSO와 SLO이다.

- 공통점: 두 방법 모두 한 번의 인증으로 여러 서비스에 접근할 수 있다.
- 차이점: 아키텍처 및 동작에 있어 차이가 존재한다.
  - SSO: 공통 인증 서비스(통합 인증 서비스) 혹은 모듈이 하나 존재하여 이를 통해 인증하고 서비스를 이용하는 방식
  - SLO: 각 시스템별로 계정 정보와 인증 서비스가 존재하는데 서로 다른 시스템 간 인증을 할 수 있도록 쿠키나 세션 정보 등을 활용해 인증하는 방식 (토큰 정보에 대해 암호화 알고리즘 활용 등)

<br>

## 1. SSO

> Single Sign On

통합 인증을 통해 한 번의 인증 과정으로 여러 시스템, 서비스를 사용할 수 있게 해주는 인증 방식.

즉, 어느 시스템이나 서비스를 접근하더라도 공통된 하나의 인증 방식 / 계정을 통해 로그인하여 서비스를 사용하는 방식이다.

### 1) SSO 아키텍처 종류

#### (1) 인증 대행 모델 (delegation, agent)

- 각 시스템의 인증 방식을 변경하기 어려울 경우 많이 사용하는 방식
- 통합 agent가 인증 작업을 대신 수행

![image](https://github.com/user-attachments/assets/c888c1c4-21a1-48c6-b9a2-95193c60fc72)

#### (2) 인증 정보 전달 모델 (propagation)

- 웹 기반의 시스템에서 주로 사용
- 미리 인증된 토큰을 받아서 각 시스템 접근 시 토큰을 통해 인증

![image](https://github.com/user-attachments/assets/b9a4c8ce-1ebd-4c0a-8c91-4157ce96d0c6)

<br>

## 2. SLO

> Single Log On

각 시스템 별로 계정 정보가 존재하고 인증 서비스도 각 시스템별로 존재한다.

하지만 SSO와 달리 하나의 통합 인증 서비스가 존재하는 것이 아니라 각 시스템별 개별 인증 서비스로 인증을 하되, 다른 시스템으로 이동 시 현재 로그인된 시스템의 인증 정보를 이동할 서비스가 인증할 수 있도록 암호화하여 쿠키나 세션 정보에 담아서 인증을 요청한 다음에 시스템에 접근할 수 있도록 하는 방식이다.

- 시스템#1에 로그인하여 인증한 뒤, 시스템#2로 이동 시 시스템#1의 인증 정보를 암호화하여 쿠키나 세션에 담아서 시스템#2의 인증 서비스를 호출하고 인증 과정을 통과하면 시스템#2를 사용할 수 있는 방식

![image](https://github.com/user-attachments/assets/bc947648-04b6-4398-9762-6b90acbad65e)