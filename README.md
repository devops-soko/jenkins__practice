# jenkins


## 1. Related Concept 
### 1) Issues before ci/cd
불규칙한 커밋 -> integration(통합) 문제 -> 테스트 지연 -> 버그 발생 빈도 높아짐

### 2) CI/CD 개념
Continuous Integration(지속적인 통합) : 코드, 설정 등을 통합하는 것
Continuous Delivery(지속적인 서비스 제공) : 합쳐진 코드가 항상 배포 가능 상태 유지
Continuous Deployment (지속적인 배포) : 합쳐진 코드를 사용자/내부 사용자가 사용가능한 환경에 배포 자동화 

### 3) CI/CD 파이프라인
개발자 -> 코드작성 -> 빌드 -> 테스트 -> 배포 -> 사용자


## 2. Basic Concept
### 1) What is Jenkins
Jenkins란?
CI/CD 파이프라인을 자동으로 해주는 툴
Jenkins 특징 : 다양한 플러그인들을 활용해 각족 자동화 작업 처리가능
Ex. credential plugin(권한부여, username, 토큰 등 저장 플러그인), git plugin(깃에서 코드 합치거나 불러드리는 플러그인), pipeline(jenkins의 핵심 기능이지만 이도 플러그인)
Jenkins pipeline : 플러그인의 집합이자 일종의 작업 명세서로 
Declarative syntax또는 script syntax(jenkins의 2가지 문법)를 통해
해야하는 작업을 순서대로 하도록 자동화 

### 2) install Jenkins
Jenkins 설치 파일 종류
LTS(Long Term Support) Release : 12주 마다 출시되는 버전
Weekly Release : 매주 출시 되는 버전
설치 방법
