# API Gateway

이번 장에서는 SAA를 준비하며 **API Gateway**에 대해서 알아보도록 한다.

---

### Overview

- AWS Lambda + API Gateway: 관리할 인프라스트럭쳐가 없다.
- WebSocket 프로토콜을 지원한다.
- API 버전 관리를 할 수 있다.
- 여러가지 다양한 환경(Dev, Test, Prod)를 처리할 수 있다.
- 보안(Authentication, Authorization)을 처리할 수 있다.
- API 키를 생성하거나, 요청을 조절하여 처리할 수 있다.
- Swagger / Open API를 가져와서 신속하게 API를 정의할 수 있다.
- 요청 및 응답을 변환하거나 검증할 수 있다.
- SDK 및 API 규격을 생성한다.
- API 응답 캐시를 사용할 수 있다.

#### Integrations High Level

- **Lambda Function**:
  - Lambda 함수 호출
  - AWS Lambda에서 지원하는 REST API를 쉽게 노출할 수 있는 방법이다.
- **HTTP**:
  - 백엔드에서 HTTP 엔드포인트를 노출할 수 있다.
  - 예: 내부 HTTP API 전체, Application Load Balancer
  - 속도 제한, 캐싱, 사용자 인증, API 키 등을 추가할 수 있다.
- **AWS Service**:
  - API Gateway를 통해서 AWS API를 노출할 수 있다.
  - 예: AWS Step Function 워크플로우를 시작하고, SQS에 메시지를 게시한다.
  - 인증 추가, 공개 배포, 요금 통제 등을 추가할 수 있다.

- 아래와 같이 API Gateway와 Kineses 및 S3를 통합할 수 있다.

![aws-service-integration-kinesis-data-streams.png](images%2FAPIGateway%2Faws-service-integration-kinesis-data-streams.png)

#### Endpoint Types

- **Edge-Optimized (default)**: 글로벌 클라이언트용
  - CloudFront Edge Location을 통해 요청을 라우팅하여 지연 시간을 개선한다.
  - API Gateway가 여전히 한 Region에서만 작동한다.
- **Region**:
  - 동일한 지역 내 클라이언트의 경우
  - CloudFront와 수동으로 결합이 가능하다.(캐싱 전략 및 배포에 대한 통제력을 강화한다.)
- **Private**:
  - 인터페이스 VPC Endpoint를 사용하여 VPC에서만 액세스할 수 있다.
  - 리소스 정책을 사용하여 액세스를 정의할 수 있다.

#### Security

- 아래의 항목을 통해서 사용자 인증(Authentication)을 진행할 수 있다.
  - IAM 역할(내부 응용프로그램에 유용)
  - Cognito(외부 사용자를 위한 ID - 예를 들어, 모바일 사용자)
  - Custom Authorizer (직접 개발한 로직)
- AWS ACM(Certificate Manager)과의 통합을 통한 **Custom Domain Name HTTPS** 보안
  - **Edge-Optimized Endpoint**를 사용하는 경우 인증서가 "us-east-1"에 있어야 한다.
  - **Regional Endpoint**를 사용하는 경우 인증서가 API Gateway 영역에 있어야 한다.
  - "Route 53"에서 CNAME 또는 A-Alias 레코드를 설정해야 한다.

---

### AWS Step Functions

- Lambda Function을 조정하기 위한 Serverless 시각적 워크플로우를 구축할 수 있다.
- **특징**: 시퀀스(Sequence), 병렬(Parallel), 조건(Conditions), 타임아웃(Timeouts), 오류 처리(Error Handling)
- EC2, ECS, On-Premises Server, API Gateway, SQS 대기열 등과 통합될 수 있다.
- 사람이 직접 승인할 수 있는 기능을 구현할 수 있는 가능성이 있다.
- **사용 사례**: 주문 이행, 데이터 처리, 웹 애플리케이션, 모든 워크플로우

![aws-step-functions.png](images%2FAPIGateway%2Faws-step-functions.png)

---

### Amazon Cognito

- 사용자에게 웹 또는 모바일 애플리케이션과 상호 작용할 수 있는 ID를 제공한다.
- **Cognito 사용자 Pool**:
  - 앱 사용자 로그인 기능
  - API Gateway 및 Application Load Balancer와 통합
- **Cognito Identity Pools(Federated ID)**:
  - 사용자가 AWS 리소스에 직접 액세스할 수 있도록 AWS 자격 증명을 제공
  - Cognito 사용자 Pool과 ID Provider로 통합
- **Cognito vs IAM**: "수백 명의 사용자", "모바일 사용자", "SAML로 인증"

#### Cognito User Pools(CUP) - User Features

- 웹 및 모바일 앱을 위한 Serverless 사용자 데이터베이스 만들기
- 간편 로그인: 사용자 이름(또는 이메일) / 비밀번호 조합
- 비밀번호 재설정
- 이메일 및 전화번호 확인
- Multi-Factor 인증(MFA)
- Federated Identities: 페이스북, 구글, SAML의 사용자

#### Cognito User Pools(CUP) - Integrations

- CUP는 아래의 이미지와 같이 API Gateway와 Application Load Balancer와 통합된다.

![cup-integrations.png](images%2FAPIGateway%2Fcup-integrations.png)

#### Cognito Identity Pools (Federated Identities)

- 임시 AWS 자격 증명을 얻을 수 있도록 "사용자"에 대한 ID를 가져온다.
- 사용자 소스는 Cognito User Pools나 3rd Party 로그인 등일 수 있다.
- 사용자가 직접 또는 API Gateway를 통해 AWS 서비스에 액세스할 수 있다.
- 자격 증명에 적용되는 IAM 정책은 Cognito에 정의되어 있다.
- `user_id`를 기반으로 사용자 지정하여 세분화된 제어를 할 수 있다.
- 인증된 사용자 및 게스트 사용자를 위한 기본 IAM 역할을 한다.

![cognito-identity-pools-diagram.png](images%2FAPIGateway%2Fcognito-identity-pools-diagram.png)

- 아래의 구성파일과 같이 Cognito Identity Pools은 DynamoDB의 Row 단위로 보안 레벨을 설정할 수 있다.

![row-level-security-dynamodb.png](images%2FAPIGateway%2Frow-level-security-dynamodb.png)

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03