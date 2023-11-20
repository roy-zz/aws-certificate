# Identity

이번 장에서는 **SysOps Administrator**를 준비하며 **Identity**에 대해서 알아보도록 한다.

---

### IAM Security Tools

- **IAM Credentials Reports (accounts-level)**
  - 계정의 모든 사용자와 다양한 자격 증명 상태를 나열하는 보고서다.
- **IAM Access Advisor (user-level)**
  - 액세스 어드바이저는 사용자에게 부여된 서비스 권한과 해당 서비스에 마지막으로 액세스한 시간을 보여준다.
  - 이 정보를 사용하여 정책을 수정할 수 있다.

#### IAM Access Analyzer

- 어떤 리소스가 외부에 공유되고 있는지 확인할 수 있다.
  - S3 버킷
  - IAM 역할
  - KMS Keys
  - Lambda Function 및 레이어
  - SQS Queue
  - Secret Manager Secret
- 신뢰 영역(Zone of Trust) 정의 = AWS Account 또는 AWS Organization
- 신뢰 영역(Zone of Trust) 외부에서의 접근을 탐색한다.

![1-iam-access-analyzer.png](images%2F1-iam-access-analyzer.png)

#### Identity Federation

- 페더레이션을 통해 AWS 외부 사용자는 AWS 리소스에 액세스하기 위한 임시 역할을 맡을 수 있다.
- 이러한 사용자는 ID 제공 액세스 역할을 맡는다.
- 페더레이션은 제3자 인증 형식을 취한다.
  - LDAP
  - Microsoft Active Directory (~= SAML)
  - Single Sign On
  - Open ID
  - Cognito
- 페더레이션을 사용하면 IAM 사용자를 생성할 필요가 없다.
- 사용자 관리는 AWS 외부에서 이루어진다.

![2-identity-federation.png](images%2F2-identity-federation.png)

#### SAML Federation (Enterprise)

- Active Directory/ADFS를 AWS(또는 모든 SAML 2.0)와 통합할 수 있다.
- AWS 콘솔 또는 CLI에 대한 액세스를 제공한다.(임시 자격 증명을 통해)
- 각각의 사용자에 대해 IAM 사용자를 생성할 필요가 없다.

![3-saml-federation-for-enterprise.png](images%2F3-saml-federation-for-enterprise.png)

#### Customer Identity Broker Application (Enterprise)

- ID 공급자가 SAML 2.0과 호환되지 않는 경우에 사용하는 것이 권장된다.
- ID 브로커는 적절한 IAM 정책을 결정해야 한다.

![4-custome-identity-broker-application.png](images%2F4-custome-identity-broker-application.png)

---

### AWS Cognito - Federated Identity Pools (Public Application)

- 목표
  - 클라이언트 측에서 AWS 리소스에 대한 직접 액세스를 제공한다.
- 방식
  - 통합 ID 제공 업체에 로그인 하거나 익명으로 접근한다.
  - Federated Identity Pool에서 임시 AWS 자격 증명을 다시 가져온다.
  - 이러한 자격 증명에는 권한을 명시하는 사전 정의된 IAM 정책이 함께 제공된다.
- 예시
  - Facebook 로그인을 사용하여 S3 버킷에 쓸 수 있는 임시 액세스를 제공한다.
- 참고
  - 웹 자격 증명 연동은 Cognito의 대안이지만 AWS는 권장하지 않는다.

![5-cognito-federated-identity-pools.png](images%2F5-cognito-federated-identity-pools.png)

---

### AWS STS (Security Token Service)

- **AWS 리소스에 대한 제한적이고 임시적인 액세스 권한을 부여**할 수 있다.
- 토큰은 최대 1시간 동안 유효하다. (갱신 필요)
- **AssumeRole**
  - 자신의 계정 내에서: 보안을 강화한다.
  - 교차 계정 액세스: 대상 계정의 역할을 맡아 그곳에서 작업을 수행한다.
- **AssumeRoleWithSAML**
  - SAML로 로그인한 사용자에 댛나 자격 증명을 반환한다.
- **AssumeRoleWithWebIdentity**
  - IdP로 로그인한 사용자에 대한 자격 증명을 반환한다.(Facebook Login, Google Login, OIDC 호환 등..)
  - AWS는 이를 사용하는 것 대신 Cognito를 사용할 것을 권장한다.
- **GetSessionToken**
  - MFA의 경우 사용자 또는 AWS 계정 루트 사용자.

#### Assume Role

- 계정 또는 교차 계정 내에서 IAM 역할을 정의한다.
- IAM 역할에 액세스할 수 있는 주체를 정의한다.
- AWS STS를 사용하여 자격 증명을 검색하고 액세스 권한이 있는 IAM 역할을 가장한다. (AssumeRole API)
- 임시 자격 증명은 15 ~ 60분 동안 유효할 수 있다

![6-sts-assume-role.png](images%2F6-sts-assume-role.png)

#### Cross-Account Access 

- AWS STS를 사용하여 다른 계정에 있는 AWS 리소스에 접근할 수 있다.
- 특정 IAM 그룹만 접근하도록 허용하여 다른 그룹의 사용자는 접근하지 못하도록 제한할 수 있다.

![7-cross-account-access-sts.png](images%2F7-cross-account-access-sts.png)

---

### Cognito User Pool (CUP) - User Feature

- **웹 및 모바일 앱을 위한 서버리스 사용자 데이터베이스를 생성**한다.
- 사용자 이름 또는 이메일 / 비밀번호 조합으로 간단한 로그인을 제공한다.
- 이메일과 휴대전화 검증을 통해 비밀번호 초기화를 할 수 있다.
- MFA(Multi-factor authentication)을 지원한다.
- Federated Identity: Facebook, Google, SAML 사용자 등..
- Feature: 자격 증명이 다른 곳에서 손상된 경우 사용자를 차단한다.
- 로그인하면 이후 요청부터 JWT(Json Web Token)을 전송한다.

![8-cognito-user-pool-diagram.png](images%2F8-cognito-user-pool-diagram.png)

#### 통합

- Cognito User Pool은 API Gateway와 Application Load Balancer와 통합된다.

![9-cognito-user-pool-integration.png](images%2F9-cognito-user-pool-integration.png)

---

### Cognito Identity Pool (Federated Identity)

- 임시 AWS 자격 증명을 얻을 수 있도록 "사용자" ID를 얻는다.
- 자격 증명 풀(예. 자격 증명 소스)에는 아래의 항목이 포함될 수 있다.
  - 공개 공급자 (Amazon, Facebook, Google, Apple 로그인)
  - Amazon Cognito 공급자 및 SAML ID 공급자
  - OpenID Connect 공급자 및 SAML ID 공급자
  - 개발자 인증 ID (맞춤형 로그인 서버)
  - **Cognito 자격 증명 풀은 인증되지 않은 게스트의 액세스를 허용**한다.
- 사용자는 직접 또는 API Gateway를 통해 AWS 서비스에 액세스할 수 있다.
  - 자격 증명에 적용되는 IAM 정책은 Cognito에서 정의된다.
  - 세밀한 제어를 위해 `user_id`를 기반으로 맞춤설정할 수 있다.

![10-cognito-identity-pool-diagram.png](images%2F10-cognito-identity-pool-diagram.png)

#### CUP 통합

- Cognito User Pool과 통합될 수 있다.

![11-cognito-identity-pool-cup.png](images%2F11-cognito-identity-pool-cup.png)

#### IAM Role

- 인증된 사용자 및 게스트 사용자를 위한 기본 IAM 역할을 제공한다.
- 사용자 ID를 기반으로 각 사용자의 역할을 선택하는 규칙을 정의한다.
- 정책 변수를 사용하여 사용자의 액세스를 분할할 수 있다.
- IAM 자격 증명은 STS를 통해 Cognito 자격 증명 풀에서 획득된다.
- 역할에는 Cognito 자격 증명 풀의 "Trust" 정책이 있어야 한다.

#### S3 예시

- 아래는 S3에 업로드되어 있는 특정 파일을 읽을 수 있는 권한을 부여하는 예시다.

![12-cognito-identity-pool-guest.png](images%2F12-cognito-identity-pool-guest.png)

- 아래는 정책을 생성할 때, "Cognito Identity Pool"의 변수를 사용하는 예시다.

![13-cognito-identity-pool-variable.png](images%2F13-cognito-identity-pool-variable.png)

- 아래는 변수를 사용하여 특정 Key값으로 요청하는 경우 허용하는 예시다.

![14-cognito-identity-pool-dynamodb.png](images%2F14-cognito-identity-pool-dynamodb.png)

---

### Cognito User Pool vs Identity Pool

- **Cognito User Pool (인증(Authentication), 신원 확인)**
  - 웹 및 모바일 애플리케이션용 사용자 데이터베이스다.
  - 소셜, OIDC, SAML을 통해 로그인을 통합할 수 있다.
  - 인증을 위한 호스팅 UI 사용자 정의가 가능하다. 
  - 인증 흐름 중에 AWS Lambda를 통한 트리거가 있다.
  - 다양한 위험 수준(MFA, Adaptive Authentication 등..)에 맞게 로그인 환경을 조정한다.
- **Cognito Identity Pool (인가(Authorization), 접근 제어)**
  - 사용자를 위한 AWS 자격 증명 얻기
  - 사용자는 공개 소셜, OIDC, SAML 및 Cognito 사용자 풀을 통해 로그인할 수 있다.
  - 사용자가 게스트 사용자라면 인증되지 않을 수 있다.
  - 사용자는 IAM 역할 및 정책에 매핑되어 정책 변수를 활용할 수 있다.

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [ID Role Provider SAML](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_saml.html)
- [ID Role Provider Enable Console SAML](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_enable-console-saml.html)
- [ID Role Common Scenario Federate User](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_common-scenarios_federated-users.html)