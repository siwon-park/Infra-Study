![image](https://user-images.githubusercontent.com/93081720/212346428-0b1fadf6-f630-4107-b9ae-9b81057b1d4c.png)

# 03_Jenkins Pipeline

> Jenkins Pipeline Flow

![image](https://user-images.githubusercontent.com/93081720/215792879-3c68b4ba-e061-4de7-9617-30c1751406c8.png)

<br>

## 1. 파이프라인 문법 종류

젠킨스 파이프라인 스크립트 작성 방법은 크게 2가지로 나뉜다.

두 가지 문법을 번갈아가면서 사용 가능하지만, 동시에 사용하는 것은 불가능하다.

### Declarative Pipeline

> 선언적 파이프라인

`pipeline` 블럭으로 감싸져 있음.

유저 정의하는 CD pipeline 모델. 빌드나 테스트, 배포를 포함한 전체적인 빌드 프로세스를 정의함 

Groovy-Syntax 기반이며, 보다 쉽게 작성할 수 있도록 커스텀되어 있음. Groovy 문법을 몰라도 작성이 가능함.

![image](https://user-images.githubusercontent.com/93081720/215798135-54bb3f51-4956-4445-8294-9ec39710b13f.png)

<br>

### Scripted Pipeline

> 스크립트화된 파이프라인

`node` 블럭으로 감싸져 있음.

Groovy 기반, Declarative보다 효과적으로 많은 기능을 포함하여 작성 가능하지만 작성 난이도가 높음.

![image](https://user-images.githubusercontent.com/93081720/215798303-bbc52fa5-c702-481b-a14e-a8e666d00949.png)

<br>