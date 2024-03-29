# 02_MSA with DB

*"마이크로 서비스 아키텍처(MSA)를 적용하기 위해서는 반드시 서비스 별로 DB를 나눠야 하는가?"*

결론부터 이야기하자면, MSA 도입의 목적과 의미를 생각해 보았을 때 DB도 역시 서비스 별로 나누는 게 맞다.

그러나 중요한 것은 무작정 서비스 별로 나누는 것이 아니라 비용, 가능성, 환경, 비즈니스 등을 종합적으로 고려해서 올바른 MSA 이행 로드맵을 구축하고 서비스 별로 점진적으로 확대해 나가는 것이 적절하다.

빅뱅 방식의 MSA 도입은 정말 많은 노력과 비용, 아키텍처/서비스 설계 등이 필요하다.

![image](https://user-images.githubusercontent.com/93081720/214557451-b67cfd76-df01-4ce3-a16a-7b305f1f0b40.png)

<br>

## 1. Database per Service의 장/단점

### 장점

- 서비스의 결합도를 보다 낮출 수 있어 MSA 도입 목적에 부합할 수 있으며, 한 서비스의 데이터베이스 변경이 다른 서비스에 영향을 미치지 않음
- 각 서비스 별로 가장 적합한 데이터베이스 유형을 사용할 수 있음
  - 예) 검색 서비스는 ElasticSearch를 사용 가능, 소셜 그래프를 조작하는 서비스는 Neo4j를 사용 가능

<br>

### 단점

- 여러 데이터베이스에 있는 데이터를 join하는 쿼리를 구현하는 것이 쉽지 않다.
- 여러 SQL 및 NoSQL 데이터베이스 관리의 복잡성이 증가한다.
  - DB 간 동기화 문제 발생

<br>

## 2. Database per Service 구현 시 고려할 점

가장 중요한 것은 서비스 간 데이터 동기화 측면

![image](https://user-images.githubusercontent.com/93081720/214561571-af1c2432-8430-48ac-9552-19b9a5fb7325.png)

- IPC(Inter-Process Communication)
  - 모놀리식에서는 하나의 프로세스 내에서 메서드 콜이 발생했지만, MSA에서는 각 서비스가 별도로 프로세스를 실행하므로 프로세스 간 소통에 대한 고민이 필요
- 조인 쿼리
  - 모놀리식 어플리케이션 내 조인 쿼리에 대응하기 위한 동기/비동기 처리 호출 설계 필요
  - API Composition Patter, CQRS Pattern 등
- 트랜잭션 관리
  - 분산 트랜잭션 관리를 위한 이벤트 처리 설계
- 보상 트랜잭션
  - 데이터 복구를 위한 패턴 설계
- API 호출 증가
  - 데이터 호출 증가에 따른 API 설계 고려
- 배치
  - 배치 아키텍처 설계를 위한 데이터 통합 방안 설계 필요