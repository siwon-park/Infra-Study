# 08_Ingress

> 인그레스(Ingress)

인그레스(Ingress)의 의미는 일반적으로 `외부에서 내부로 향하는 것`을 의미하는 단어이다. 즉, 인그레스 트래픽이라고 하면, 외부에서 내부로 유입되는 트래픽이라는 뜻이다.

인그레스 네트워크는 인그레스 트래픽을 처리하기 위한 네트워크이다.

<br>

## 1. 개념

>  서비스(Service)가 외부의 요청을 받아들이기 위한 것(accept)이었다면, 인그레스는 외부의 요청을 어떻게 처리할 것(how to handle)인지의 개념이다.

인그레스는 `요청을 어떻게 처리할 것`인지 `네트워크 7계층 레벨`에서 정의하는 쿠버네티스 오브젝트이다.

인그레스 오브젝트가 처리할 수 있는, 담당하는 기능은 크게 다음과 같다

- 외부 요청 라우팅
  - `/apple`, `/apple/red` 등과 같이 특정 경로로 들어온 요청을 어떤 서비스로 전달할지 정의하는 라우팅 규칙을 설정할 수 있음
- 가상 호스트 기반 요청 처리
  - 같은 IP에 대해 다른 도메인 이름으로 요청이 도착했을 때, 어떻게 처리할 것인지 정의 가능
- SSL/TLS 보안 연결 처리
  - 여러 개의 서비스로 요청을 라우팅할 때, 보안 연결을 위한 인증서를 쉽게 적용할 수 있음

<br>

## 2. 인그레스를 사용하는 이유

서비스의 NodePort, LoadBalancer만으로도 인그레스에서 제공하는 기능들을 구현할 수는 있다. 불가능하지 않다. 그럼에도 불구하고 인그레스를 사용하는 이유는 다음과 같다.

### 인그레스를 사용하지 않았을 때(Without Ingress)

크게 문제가 없는 구조이지만, 사실 서비스마다 새로운 세부 설정을 한다고 했을 때 작업이 복잡해질 수도 있다. 각 서비스마다 매번 엔드포인트, 라우팅, SSL 인증 등의 작업이 필요하다.

![image](https://github.com/siwon-park/Problem_Solving/assets/93081720/44837875-beb3-4c79-95a9-eaad03621d04)

<br>

### 인스레스를 사용했을 때(With Ingress)

그러나 인그레스를 사용하면 각 서비스마다 일일이 이러한 설정을 할 번거로움이 없어지고, 인그레스에만 SSL 보안 인증이나 엔드 포인트 설정을 추가하면 된다.

인그레스가 gateway 역할을 하게 되는 셈이다.

![image](https://github.com/siwon-park/Problem_Solving/assets/93081720/45b295d9-5c27-4d30-9f66-b5250c18b0ef)

<br>

## 3. 인그레스 컨트롤러 (Ingress Controller)

Ingress를 선언하기 위해 yml파일을 작성하고 적용시켜도 아무 일이 발생하지 않는다.

Ingress라는 오브젝트 리소스가 동작하기 위해서는 `인그레스 컨트롤러(Ingress Controller)`가 필요하다.

인그레스 컨트롤러는 특수한 서버이며, 인그레스 컨트롤러가 인그레스 규칙을 로드하여 사용하는 것이고 실제로 외부의 요청을 받아들이는 것은 인그레스 컨트롤러이다.

`kube-controller-manager`와 함께 동작하는 다른 타입의 컨트롤러와는 다르게, 인그레스 컨트롤러는 클러스터와 함께 자동적으로 실행되지 않는다.

따라서 정상적으로 인그레스에 정의한 규칙들을 적용하기 위해서는 본인의 인프라(클러스터)에 맞는 적합한 인그레스 컨트롤러를 선택해서 적용해야 한다.

### 종류

인그레스 컨트롤러의 종류는 다양하며, 쿠버네티스에서 공식적으로 관리되는 것이 아니라 제 3자에 의해 관리되는 서드 파티 라이브러리의 개념이다.

NGINX Ingress Controller, ngrok Ingress Controller, Istio Ingress Controller, Kong Ingress Controller, HAProxy Ingress Controller, Traefik Ingress Controller 등 다양한 인그레스 컨트롤러가 존재한다.

자세한 내용은 [쿠버네티스 공식 문서](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)에서 확인할 수 있다.

