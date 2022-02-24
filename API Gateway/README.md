###### 김지호

# API Gateway

>   Amazon Web Services API Gateway에 대해 정리한 문서입니다.

## Overview

>   모든 API의 관문

-   모든 MicroServices가 API Gateway를 통하게 된다.
-   여러 서비스를 마치 하나의 서비스처럼 사용할 수 있다.
-   AWS에서 제공하는 다양한 서비스들과 쉽게 병합할 수 있다.

## Settings

### Index

1.  인스턴스 생성
2.  리소스 및 메서드 생성
3.  스테이지와 배포

### 1. 인스턴스 생성

-   REST API 구축
    -   프로토콜 선택: REST
    -   새 API 생성
        -   새 API / 기존 API에서 복제 / Swagger에서 가져오기 또는 API 3 열기 / 예제 API
    -   설정
        -   API 이름
        -   설명
        -   엔드포인트 유형: 지역

### 2.  리소스 및 메서드 생성

-   `리소스`
    -   `https://www.naver.com/login`에서 `/login`을 의미
    -   Root는 `/`이며, 계층을 가질 수 있다. (`/grandmother/father/child`)
    -   필요시 CORS 설정을 추가할 수 있으며, 나중에도 설정 가능
-   `메서드`
    -   실제 동작하는 API를 연결할 인스턴스
    -   `통합 유형`
        -   Lambda: AWS Lambda Serverless API
        -   HTTP: 구축된 서버와 연결
        -   Mock: 테스트용
        -   AWS 서비스: AWS 서비스에 연결하여 사용
        -   VPC 링크: VPC 보안 그룹 접근(LB 필수)
    -   `엔드포인트`: 해당 메서드를 실행할 주체
    -   세부 설정은 통합 유형에 따라 다름

#### 2.1. 메서드 상세 설정

메서드의 스펙은 기본적으로 본래 설계된 API Server 스펙을 따른다.

```
클라이언트 - 메서드 요청 - 통합 요청 - Endpoint(Server) - 통합 응답 - 메서드 응답 - 클라이언트
```

위와 같은 순서로 메서드가 진행된다.
클라이언트 부분을 제외한 각 단계에는 여러 설정들이 있으며 필요에 따라 알맞게 설정한다.

***Request/Reponse Header/Body***에 대한 설정으로 볼 수 있다.

##### 메서드 요청 & 통합 요청

-   메서드 요청
    -   설정
        -   승인
        -   요청 검사기 : URL 쿼리 문자열 파라미터 및 헤더 등의 값을 검사한다.
        -   API 키 필요 여부(이 문서에서 다루지 않음)
    -   URL 쿼리 문자열 파라미터
        -   `https://www.example.com/search?keyword="hello"`에서 `keyword`를 의미
        -   필요한 쿼리 문자열이 있으면 등록하며, 요청 검사기를 등록한다.
    -   HTTP 요청 헤더
        -   Request Header에 값이 메서드 동작의 필요한 값일 경우 등록한다.
        -   예를 들어 JWT와 같이 인증키 등을 엔드포인트에 넘겨야 할 경우 사용한다.
    -   요청 본문
        -   엄격한 방식으로 요청 본문을 관리할 경우 사용하며,
            자세한 설정은 이 문서에서 다루지 않는다.
    -   SDK Settings(이 문서에서 다루지 않음)
-   통합 요청
    -   메서드 요청 설정에 포함된 값들이 제대로 매핑되었는지 확인할 수 있다.
    -   제대로 할당되지 않았을 경우 AWS 문서에 따라 알맞게 설정한다.
    -   또한, 메서드 유형을 변경할 수 있다.
        -   > ex) HTTP 방식에서 Lambda로 변경

##### 메서드 응답 & 통합 응답

메서드 요청이 Client로부터 온 요청을 필터링해서 Endpoint로 넘기는 작업이라면,
메서드 응답은 Endpoint로부터 받은 응답을 필터링해서 Client로 넘기는 작업이다.

-   메서드 응답 (HTTP 상태 코드에 따른 응답을 분류하여 관리)
    -   응답 헤더: 엔드포인트로부터 받은 응답 헤더에서 클라이언트로 넘길 헤더 목록
    -   응답 본문: 엔드포인트로부터 받은 응답 본문에 대한 설정
-   통합 응답
    -   요청과 마찬가지로 메서드 응답에 따라 값이 자동으로 할당된다.
    -   필요 시 더 자세하게 처리할 수 있다.

##### 주의할 점

-   엔드포인트에서 응답하는 헤더값은 API Gateway가 전부 필터링되어 API Gateway 형식에 맞게 변환한다.
-   따라서 위 메서드 상세 설정에 따라 `query string`, `path`, `HTTP header` 등
    필요에 따라 알맞게 설정해야 정상적으로 동작한다.
-   또한 지정하더라도 API Gateway에서 자동으로 Key에 Prefix를 추가하므로 Client의 수정이 발생한다.

### 3.  스테이지와 배포

-   리소스 패널의 작업 목록 중에서 API 배포 항목
-   기본적으로 stage를 가지며 실제 API의 Root URI가 된다.
    -   Stage명이 `test`일 때, `https://xxx.excute-api.region.amazonaws.com/test`로 자동 설정
-   따라서 Stage 명명 또한 생각하고 API 설계를 진행해야 한다.

---

## Examples

>   Normal Pattern

```
                         Reverse
+--------+     +-------+  Proxy  +-------------+     +-----+
| Client |-----| Nginx |---------| API Gateway |-----| WAS |
+--------+     +-------+         +-------------+     +-----+
```

>   Layered API Gateway for Cognito

```
                         Reverse
+--------+     +-------+  Proxy +---------+   +----------+
| Client |-----| Nginx |--------|   API   |---|   API    |--- WAS
+--------+     +-------+        | Gateway |   | Gateway  |
                                +---------+   | (Public) |
                                    |         +----------+
                                    |
                          +--------------+   +-----------+
                          |   Cognito    |---|    API    |--- WAS
                          | (Authorizer) |   |  Gateway  |
                          +--------------+   | (Private) |
                                             +-----------+
```

>   VPC Link - 안타깝게도 VPC로 안전하게 사용하기 위해서는 LB를 필수적으로 적용해야 한다.

```
                         Reverse                          
+--------+     +-------+  Proxy  +-------------+
| Client |-----| Nginx |---------| API Gateway |
+--------+     +-------+         +-------------+          
                                        |                 
                                    VPC Link
                                        |
                              +- VPC -------------+
                              |+-----+     +-----+|
                              || NLB |-----| WAS ||
                              |+-----+     +-----+|
                              +-------------------+
```