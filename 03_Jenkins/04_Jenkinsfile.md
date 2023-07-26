# 04_Jenkinsfile

> Jenkinsfile로 Pipeline Script를 관리하는 방법

## 1. 개요

Jenkins GUI 환경에서 직접적으로 Pipeline 스크립트를 관리해도 되지만 `Jenkinsfile`을 통해 Pipeline 스크립트를 관리할 수 있다.

|                         Jenkins GUI                          |                         Jenkinsfile                          |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![50](https://github.com/siwon-park/Problem_Solving/assets/93081720/ec2bb4b0-236b-45f7-be87-f031f65f186b) | ![51](https://github.com/siwon-park/Problem_Solving/assets/93081720/87a0ce63-3219-473c-9ba7-450fea5480df) |
| Pipeline 탭 아래 Pipeline script를 통해 Jenkins GUI 상에서 스크립트를 관리 | Pipeline 탭 아래 Pipeline script from SCM을 통해 연동된 Git Repository 상에서 Jenkinsfile로 관리 |

Jenkinsfile을 사용한다면 아래와 같이 프로젝트 디렉토리에 Jenkinsfile이라는 이름의 파일이 있어야 한다.

![image](https://github.com/siwon-park/Problem_Solving/assets/93081720/77e53dd3-3888-4357-a198-15f22a8934eb)

파일 내용은 Pipeline 스크립트와 동일하다.

<br>

## 2. 왜 쓸까?

굳이 왜 Jenkinsfile로 따로 관리해서 사용하는 걸까?

사실 Jenkinsfile로 pipeline 스크립트를 관리하면 GUI 상에서 직접 관리하는 것보다 여러 가지 이점이 있다.

### Jenkinsfile 사용 시 이점들

#### 1. 보안적으로 안전성

GUI 상에서 직접 pipeline 스크립트를 작성하여 관리하는 것보다 보안적으로 더 안전하다. 왜냐하면 다른 누군가가 jenkins url을 알고 jenkins 계정을 알면 해당 pipeline 내용이나 환경 변수, job 등 다양한 내용에 접근할 수도 있다.

하지만 jenkinsfile로 관리한다면 Git Repository는 public이 아닌 이상 접근 가능한 사용자만 볼 수 있다.

#### 2. 코드 리뷰를 통한 안정성

Jenkinsfile을 통해 pipeline 스크립트를 관리하면 Repository를 통해 관리해야 하기 때문에 스크립트의 변화된 내용을 적용시키기 위해선 `merge`과정을 거쳐야 한다.

이 과정에서 우리는 코드 리뷰를 통해서 pipeline 스크립트 상에 잘못된 부분은 없는지 사전에 점검하고 넘어갈 수 있다. 따라서 에러 발견에 용이하며, 시행 착오도 덜 격게 된다.

#### 3. 변경 이력 추적 가능

 2번과 마찬가지로 Repository를 통해 관리해야 하기 때문에 스크립트의 변화된 내용을 `Git History` 및 `merge` 내용 등 변경 이력을 추적하여 관리 가능하다. 또한 누가 수정했는지도 알기 때문에 책임 소재를 분명하게 물을 수 있다.

#### 4. 단일 진실 공급원(Single Source Of Truth)

하나의 진실된 출처라는 의미로, Jenkins GUI 상에서 pipeline 스크립트를 관리하면 어떤 누가 수정했는지 아니면 심지어 해커가 와서 수정했는지 알 방법이 없지만 Jenkinsfile을 통해 관리하면 해당 pipeline 스크립트에 대한 수정이나 보기 권한은 프로젝트 관계자에게 있기 때문에 프로젝트 관계자에 의해서만 관리된다. 따라서 신뢰성을 보장할 수 있다.