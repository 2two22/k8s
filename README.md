
## 1. 서비스 소개

### 1) 서비스 주제

- 개발자를 위한 동기 부여 플랫폼 및 커뮤니티
- 주니어 개발자, 현직 개발자, 개발을 공부하는 학생 등 개발을 꿈꾸는 사람들을 위한 깃허브 커밋 기록을 연동한 플랫폼
- 서비스 기대 효과
    - 커밋 기록에 따라 레벨을 쌓아나가며 개발 공부에 대한 동기 부여를 제공한다.
    - 어떠한 개발 관련 주제에 대해 심도있게 소통할 수 있는 게시판 기능을 제공한다.
    - 최신 개발 뉴스를 실시간으로 볼 수 있도록 서비스를 제공한다.
- 다른 서비스와의 차별점
    - 개발자와 떼어 놓을 수 없는 깃허브에서 커밋 기록을 가져와 ‘개발 동기 부여 플랫폼’ 면에서 사용자 유용성을 향상시킨다.
    - 개발 트렌드 및 최신 IT뉴스와 정보를 키워드 별로 검색하고 찾아보는 등 개발 관련 정보를 한 눈에 모아볼 수 있다.

### 2) 서비스 핵심 기능

**1) 나의 깃허브 커밋 히스토리 조회 기능**

<img width="326" alt="Untitled (1)" src="https://github.com/user-attachments/assets/1f0ab5c0-093f-48e7-935b-398252a41c97">


커밋 기록에 따라 레벨을 쌓아나가며 
개발 공부에 대해 동기 부여

**2) 커뮤니티 기능** 


<img width="325" alt="Untitled (2)" src="https://github.com/user-attachments/assets/b3e65aa2-85c5-4c5d-8ee7-6d66f1607acc">

개발자끼리 어떠한 주제에 대해 
심도있게 소통할 수 있는 게시판 제공

**3) 네이버 최신 개발 뉴스 조회 기능**

<img width="326" alt="Untitled (3)" src="https://github.com/user-attachments/assets/9791a930-c09d-406c-8300-146e5bf3de7f">

최신 개발 뉴스에 대한 정보를 
실시간으로 제공

### 3) 기술 스택

- 개발 도구 -  `Spring Boot` `JPA` `QueryDSL` `Cloud Open Feign`
- 데이터 베이스 - `MongoDB` `MySQL`
- 배포 도구 - `Kubernetes` `Docker` `Git Action`

## 2. 서비스 구성

### 1) 서비스 전체 아키텍처

- 서비스 아키텍처 도식화
![22-Page-1의 복사본의 복사본 drawio (5)](https://github.com/user-attachments/assets/8f67335f-ca7f-4cd5-ba6d-d9de943ef333)

    - CI/CD
        - github develop 브랜치에 git action 트리거를 걸어, 코드 변경 시 docker hub 에 도커 이미지를 빌드하고 도커 허브에 push 되게 설정하였다.
        - GCP 내 위치한 쿠버네티스 호스트 서버에 접속하여 이미지를 업데이트한다.
                
    - front-end
        - 리액트 형태의 도커 컨테이너로 서버를 띄웠다. 프론트엔드는 백엔드 쿠버네티스 클러스터와 분리된 외부 서버로 가정했다.
    - Kubernetes
        - Ingress - 대표 도메인으로 요청을 받고, 해당 요청을 path에 따라 적절한 서비스로 분배한다.
        - Service (ClusterIP) - 파드들은 클러스터 내부에서 통신할 수 있다. 통신할 때 Cluster 내 도메인을 이용해 통신한다.            
        - Deployment
        - Pod
        - CronJob - 특정 주기로 정해진 Job을 수행한다.
            - 12시간 주기로 naver 개발 뉴스 크롤링 코드가 수행되도록 설정하였다.

### 2) 마이크로 서비스 아키텍처 구성요소 설명 및 통신 방법

- 마이크로 서비스 아키텍처 도식화
    - User 서버 중심의 MSA 간 통신이 이루어진다.
    <img width="1111" alt="Untitled (4)" src="https://github.com/user-attachments/assets/4440b2f3-a4b4-40f6-95fa-bd470a5f9b2d">

    MSA 아키텍처 도식화
    
- 마이크로 서비스 간의 통신
    - 모든 통신은 직접 통신으로 이루어진다.
        - **User ↔ Community** : User 서버는 Community 서버에게 user의 포스팅 개수를 요청한다.
        - **User ↔ News** : User 서버는 News 서버에게 user의 북마크 개수를 요청한다.
        - **User ↔ Github** : User 서버는 소셜 로그인 시 깃허브 서버에게 유저의 깃허브 토큰과 아이디를 저장해달라고 요청한다.
        - **Community ↔ User** : Community 서버는 User 서버에게 글 쓴 유저 프로필 데이터를 요청한다.
- 마이크로 서비스 별 기능
    - **User** : 유저의 회원가입, 로그인, 토큰 생성 및 갱신 등 인증에 관한 부분 및 유저 팔로우, 프로필 조회
    - **Github** : 커밋 정보(한 주간의 커밋 수, 연속 커밋 수), 자신의 커밋 기록에 따라 업그레이드 되는 레벨
    - **News** : 크롤링한 IT 관련 naver 뉴스 조회, 검색 및 스크랩 기능
    - **Community** : QnA 및 자유글 등의 포스팅 및 댓글, 좋아요 기능

### 3) 주요 변경점

- **News** - batch 서버 분리
    - 기존에는 뉴스 관련 api 와 뉴스를 12시간 간격으로 크롤링하는 서비스 코드가 News 서버에 함께 있었다. 크롤링 시 다량의 News 데이터를 DB I/O 하기 때문에, News 서버에 부하가 매우 심해진다. 이 부분이 News 서버의 조회 성능에 영향을 줄 수 있다.
    - 크롤링을 하는 코드를 news-api 와 별도의 서버인 batch 서버로 분리하였다.
    - 뉴스 크롤링 job은 서버가  띄워지자마자 수행되고 바로 shutdown 되게 개발하였다.
    - 쿠버네티스의 CronJob 을 활용해 batch 서버가 12시간 간격으로 가동되게 설정해 두었다.
 
        <img width="534" alt="스크린샷 2024-06-14 오후 12 32 11" src="https://github.com/user-attachments/assets/c01584da-f6db-476d-bed8-c5e962061cca">

        cronjob 의 설정
        
- **Github** - level 저장 방법
    - 기존의 모놀리식 아키텍처에서는 유저의 level 이 member 테이블에 저장되어 있었다.
    - member 와의 의존성을 제거하기 위해 user_level 테이블을 별도로 만들어, Github 서버가 독립된 마이크로 서비스로 동작할 수 있게 수정하였다.

- **Community** - MySQL에서 mongoDB로 전환
    - 조회 성능이 중요한 커뮤니티 서버는 조회에 더 유리한 mongoDB로 전환하였다.
    - 기존의 MySQL db 에서는 좋아요와 이미지가 게시물과 별도의 테이블로 분리되어 있었고, 조회 시 이를 join 연산해서 데이터를 가져왔다.
    - Key Value Store에서 join 연산시 rdb보다 오히려 성능이 안좋아져, 조회 성능의 장점을 잃는다는 문제가 있었다. 따라서 좋아요와 이미지를 게시물 내부 array 필드로 변경하였다.
      
    <img width="638" alt="스크린샷 2024-09-09 오전 11 27 18" src="https://github.com/user-attachments/assets/c933352d-d827-4c02-9dd2-20847cfe16ce">

- News - 스크랩 기능 추가
    - User 서버와의 MSA 통신을 위해 뉴스 스크랩 기능을 추가하였다.
    - 뉴스 스크랩, 유저의 뉴스 스크랩 개수 조회, 유저가 스크랩한 뉴스 조회 엔드포인트를 추가했다.
    - 뉴스 정보 조회 시 해당 뉴스에 대한 유저의 스크랩 여부를 필드에 추가했다.
    <img width="648" alt="스크린샷 2024-09-09 오전 11 27 40" src="https://github.com/user-attachments/assets/55c56de2-d0b0-4951-9a49-6a4e9042b54f">
    

### 4) 특별히 강조하고 싶은 부분

- 토큰 기반 인증
    - 모든 마이크로 서버는 JWT 키를 공유해 유저의 id 정보를 decode 할 수 있도록 설계하였다. 토큰 기반 인증을 통해 stateless 한 환경에서 보안상 안전한 인증을 할 수 있다.
- Git Action 을 활용한 CI/CD
    - Git Action 을 활용해 CI/CD 파이프라인을 구축했다. 따라서 빠르게 코드의 업데이트와 변경 사항을 서버에 통합할 수 있다.
    - Git Action 에 롤링 업데이트를 명시 해 무중단 배포라는 목표를 이룰 수 있게 하였다.
- Ingress 를 활용해 특정 엔드포인트만 개방한다. 파드의 ip가 클러스터 외부로 노출되지 않게 격리하여 노드의 리소스를 최적화하였다.

## 3. 프로젝트 진행 간 문제점 및 해결 방안

### 1) k8s 배포 이슈

- CORS policy
    - 문제점 : FE에서 대표 도메인으로 요청 시 CORS policy 에 막혀 요청이 제대로 전달되지 않았다. 또한 Authorization 헤더가 무시되어, 토큰 기반 인증이 제대로 이뤄지지 않는 현상이 있었다.
        - CORS policy에 위반하는 경우
            - 프로토콜이 다르다
            - 도메인이 다르다 → **우리 서비스에 해당되는 문제 상황**
            - 포트 번호가 다르다
    - 해결 방법
        - ingress 설정 파일에 CORS 관련 설정들을 명시해 CORS 에 동의해주었다.
        <img width="683" alt="Untitled (7)" src="https://github.com/user-attachments/assets/da664578-ace1-43ed-8b31-895d61b58ebb">
        ingress 설정 파일에 CORS 관련 설정 명시 부분
        

- Rewrite Target 설정
    - 문제점 : user 서비스의 경우 oauth2 설정으로 인해 URI prefix가 여러개이다.
    - 해결 방법
        - 대표 path를 각각의 서비스와 맵핑하고, 각각의 서비스에 실제로 요청을 전달하는 시점에는 대표 path가 제외되도록 설정했다.
<img width="659" alt="스크린샷 2024-09-09 오전 11 28 29" src="https://github.com/user-attachments/assets/533f68ef-ee92-4651-94a1-1d33ba7e6ea9">

### 2) User 서비스 이슈

- OAuth2 오류
    - 문제점 : OAuth2 통신 로직의 이상으로 인증 서버에 요청이 전달되지 않았다.
    - 해결 방법
        - 통신 로직을 다음과 같이 수정하여 문제를 해결하였다.
        1. 인증 코드를 요청한다.
        2. redirect uri를 통해 인증 코드를 부여한다.
        3. 인증 코드를 전달한다.
        4. 인증 코드를 보내서 액세스 토큰을 요청한다.
        5. 액세스 토큰을 부여한다.
        6. 액세스 토큰을 전달한다.

![Untitled (8)](https://github.com/user-attachments/assets/ba6d71c1-2894-439f-8cf4-9f30afa0e780)


### 3) Github 서비스 이슈

- Duplicate Key Error
    - 문제점 : 유저의 github 정보를 저장하는 과정에서 이미 회원가입이 된 기존 유저의 경우 중복 키 오류인 duplicate key 오류가 발생하였다.
    - 해결방법
        - 회원가입이 되어있는 기존 유저의 경우에는 github 정보를 업데이트하는 update() 메소드를 적용하였다.
        - 새로 가입하는 유저인 경우에는 github 정보를 새로 저장하도록 적용하였다.


### 4) News 서비스 이슈

- MySQL Driver 호환성 문제
    - 문제점 : IntelliJ에서는 DB 문제 없이 잘 연결되었으나, jar 파일로 빌드되었을 때 MySQL 드라이버를 찾을 수 없다는 no suitable driver found 오류가 발생했다.
    - 해결방법
        - 공식 문서를 참고해 spring boot 버전과 적합한 MySQL 드라이버로 변경하였다.
        
        <img width="550" alt="스크린샷 2024-09-09 오전 11 29 28" src="https://github.com/user-attachments/assets/f4575eda-a0ac-4f19-a226-5f72d78b83d2">

- No Manifest 이슈
    - 문제점 :  jar파일로 빌드 시, 실행할 메인 파일을 찾을 수 없다는 오류가 발생하였다. 
    이유는 기존 프로젝트를 클론하면서 구조를 그대로 유지하여 settings.gradle의 위치가 루트에 존재하지 않았고, settings.gradle의 rootProject.name을 변경해주지 않았기 때문이었다.
    - 해결방법
        - settings.gradle 위치를 루트로 옮기고, settings.gradle의 rootProject.name을 프로젝트 이름으로 변경하여 해결했다.
        
        ---
        
        <img width="624" alt="스크린샷 2024-09-09 오전 11 30 06" src="https://github.com/user-attachments/assets/ee270755-6f41-4a3d-9d43-023a58df5e1b">


## 4. Lessons Learned

### 1) 쿠버네티스 기반 무중단 배포

- 도커만을 활용한 배포는 컨테이너를 terminating 이후 새로운 컨테이너를 띄우기 때문에 꽤 긴 시간 서비스가 중단된다.
- 반면에 롤링 업데이트는 파드를 점진적으로 업데이트 하기 때문에, 새로운 파드가 Running 상태가 되기 전까지 기존 파드가 살아있다.
→ 클라이언트 입장에서 서비스가 중단되는 시간이 매우 짧게 느껴진다.
    <img width="734" alt="스크린샷 2024-06-11 오후 7 01 15 (1)" src="https://github.com/user-attachments/assets/0f046da0-6751-486b-93ea-81002e61f448">

    쿠버네티스 롤링 업데이트 시 파드의 상태
    

### 2) 기존 모놀리스 서비스를 MSA로의 전환

- 코드를 분리하는 과정에서 타인의 코드와 서비스 로직을 이해하고 수정하는 경험을 해보았고, 왜 가독성이 좋은 코드가 중요한지 느꼈다.
- 코드 분리뿐만 아니라 서버를 잘 동작시키고, 도커로 빌드하는 과정에서 트러블 슈팅에 대한 경험을 쌓을 수 있었다.
    - 공식 커뮤니티나 공식 레포지토리의 github issue를 활용하면 유사한 issue 를 찾을 수 있다.
 

<img width="656" alt="스크린샷 2024-09-09 오전 11 30 59" src="https://github.com/user-attachments/assets/cf9c2b63-dd5f-457f-aa99-594c6cee96a7">
  

## 5. 링크 자료

- 깃허브 링크
    
    https://github.com/orgs/2two22/repositories
    
- 데모 영상 링크
    
    https://drive.google.com/file/d/1XTolFP6lmqjLRVKlctRIoJXl5wjyGGHj/view
