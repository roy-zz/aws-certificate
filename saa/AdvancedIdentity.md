# Advanced Identity

이번 장에서는 SAA를 준비하며 **심화 Identity**에 대해서 알아보도록 한다.

---

### AWS Organizations

![organizational-unit.png](images%2FAdvencedIdentity%2Forganizational-unit.png)

- 글로벌 서비스다.
- 여러 AWS 계정을 관리할 수 있다.
- 주 계정은 관리 계정(Management Account)이다.
- 기타 계정은 구성원 계정(Member Account)이다.
- 구성원 계정은 하나의 조직에만 속할 수 있다.
- 모든 계정에 걸친 통합 청구가 이루어 진다. - 단일 결제 방식
- Aggregated 사용량을 통한 가격 책정 혜택(EC2, S3의 볼륨 할인 등)을 받을 수 있다.
- 계정 간 공유 예약 인스턴스 및 절감 계획 할인을 받을 수 있다.
- API를 사용하여 AWS 계정 생성을 자동화할 수 있다.

![organizations.png](images%2FAdvencedIdentity%2Forganizations.png)

- **장점(Advantage)**
  - 다중 계정 vs 단일 계정 다중 VPC
  - 청구 목적으로 태그 지정 표준을 사용할 수 있다.
  - 모든 계정에서 "CloudTrail"을 사용하여 중앙 "S3" 계정으로 로그를 전송할 수 있다.
  - 중앙 로깅 계정으로 "CloudWatch Logs"로 전송할 수 있다.
  - 관리 목적으로 계정 간 역할을 설정할 수 있다.
- **보안: Service Control Policies(SCP)**
  - 사용자 및 역할을 제한하기 위해 OU 또는 Accounts에 적용되는 IAM 정책을 생성할 수 있다.
  - 관리 계정(전체 관리 권한)에는 적용되지 않는다.
  - 명시적 허용(IAM과 같이 기본적으로 어떤 것도 허용하지 않음)이 있어야 한다.

#### SCP Hierarchy

![scp-hierarchy.png](images%2FAdvencedIdentity%2Fscp-hierarchy.png)

- Management Account
  - 무엇이든 작업할 수 있다.
  - "SCP"가 적용되지 않는다.
- Account A
  - 무엇이든 작업할 수 있다.
  - "OU"에서 명시적으로 거부했기 떄문에 "Redshift"는 액세스할 수 없다.
- Account B
  - 무엇이든 작업할 수 있다.
  - "Prod OU"에서 명시적으로 거부했기 때문에 "Redshift"는 액세스할 수 없다.
  - "HR OU"에서 명시적으로 거부했기 때문에 "Lambda"에는 액세스할 수 없다.
- Account C
  - 무엇이든 작업할 수 있다.
  - "Prod OU"에서 명시적으로 거부했기 때문에 "Redshift"는 액세스할 수 없다.
  
- 아래의 이미지는 거절 목록(Blocklist)과 허용 목록(Allowlist)을 설정하는 예시이다.

![scp-blocklist-allowlist.png](images%2FAdvencedIdentity%2Fscp-blocklist-allowlist.png)

---

### IAM Conditions

- 지금부터 IAM "Condition" 블록을 활용하는 몇가지 방법에 대해서 알아본다.
- 아래에서 좌측 이미지는 특정 IP의 요청을 거부(Deny)하는 예시이며, 우측 이미지는 특정 리전에서 발생하는 요청을 거부하는 예시이다.

![iam-conditions-denyip-denyregion.png](images%2FAdvencedIdentity%2Fiam-conditions-denyip-denyregion.png)

- 아래에서 좌측 이미지는 특정 태그(`DataAnalytics`, `Data`)가 포함된 리소스는 허용(Allow)하는 예시이며, 우측 이미지는 "MFA"가 적용되지 않은 경우 요청을 거부하는 예시이다.

![iam-conditions-allowtag-forcemfa.png](images%2FAdvencedIdentity%2Fiam-conditions-allowtag-forcemfa.png)

#### IAM for S3

![iam-s3.png](images%2FAdvencedIdentity%2Fiam-s3.png)

- `s3:ListBucket` 권한은 `arn:aws:s3::test`에 적용된다.
  - => 버킷 수준의 허가
- `s3:GetObject`, `s3:PutObject`, `s3:DeleteObject`는 `arn:aws:s3:::test/*`에 적용된다.
  - => 객체 수준의 권한

#### Resource Policies & aws:PrincipalOrgID

- `aws:PrincipalOrgID`는 모든 리소스 정책에서 AWS 조직의 구성원인 계정에 대한 액세스를 제한하는 데 사용할 수 있다.
- 아래의 이미지에서 `arn:aws:s3:::2022-financial-data/*` 버킷에는 "o-yyyyyyyyyy" ID를 가진 "Organization"의 계정만 접근할 수 있다.

![resource-policies-organitaions-id.png](images%2FAdvencedIdentity%2Fresource-policies-organitaions-id.png)

#### IAM Roles vs Resource Based Policies

- 교차 계정(Cross Account)
  - 리소스에 리소스 기반 정책 연결(예: S3 버킷 정책)
  - 계정의 "Role"을 프록시로 사용한다.

![iam-role-resource-based-policies.png](images%2FAdvencedIdentity%2Fiam-role-resource-based-policies.png)

- 역할(사용자, 애플리케이션 또는 서비스)을 맡으면 원래 사용 권한(Permissions)을 포기하고 역할(Role)에 할당된 사용 권한을 가진다.
- 자원 기반 정책을 사용하는 경우, 사용자는 자신의 권한(Permissions)을 포기할 필요가 없다.
- 예를 들어, 계정 A의 사용자는 계정 A의 DynamoDB 테이블을 스캔하여 계정 B의 S3 버킷에 덤프해야 한다.
- 지원 대상: "Amazon S3 Buckets", "SNS Topics", "SQS Queues" 등이 있다.

#### Amazon EventBridge - Security

- 규칙을 실행할 때 대상에 대한 사용 권한이 필요하다.
- 리소스 기반의 정책: Lambda, SNS, SQS, CloudWatch Logs, API Gateway 등..
- IAM Role: Kinesis Stream, System Manager Run Command, ECS Task 등..

![eventbridge-security.png](images%2FAdvencedIdentity%2Feventbridge-security.png)

#### IAM Permission Boundaries

- IAM 권한 경계는 그룹이 아닌 사용자 및 역할에 대해 지원된다.
- 관리되는 정책을 사용하여 IAM 엔티티가 얻을 수 있는 최대 권한을 설정하는 고급 기능이다.

![iam-permission-boundaries-1.png](images%2FAdvencedIdentity%2Fiam-permission-boundaries.png)

- "AWS Organizations SCP"와 조합되어 사용될 수 있다.

![iam-permission-boundaries-2.png](images%2FAdvencedIdentity%2Fiam-permission-boundaries-2.png)

- 대표적인 사용 사례는 아래와 같다.
  - 권한 범위 내에서 비관리자에게 책임을 위임(예: 새 IAM 사용자 생성)할 수 있다.
  - 개발자가 정책을 자체 할당하고 자체 권한을 관리할 수 있도록 허용하는 동시에 권한을 단계적으로 확대할 수 없도록 한다.(스스로 관리하도록 유도)
  - 조직 및 SCP를 사용하는 전체 계정이 아닌 특정 사용자 한 명ㅇ르 제한하는 데 유용하다.

#### IAM Policy Evaluation Logic

![iam-policy-evaluation-logic.png](images%2FAdvencedIdentity%2Fiam-policy-evaluation-logic.png)

---

### AWS IAM Identity Center (AWS Single Sign-On의 후속작)

- 한 번의 로그인(single sign-on)으로 모든 작업을 수행한다.
  - "AWS Organizations"의 AWS 계정
  - 비즈니스 클라우드 애플리케이션(예: Salesforce, Box, Microsoft 365 등)
  - SAML2.0 지원 애플리케이션
  - EC2 Windows 인스턴스
- Identity 제공자
  - "IAM Identity Center"에 ID 스토어가 내장되어 있다.
  - 3rd Party: Active Directory(AD), OneLogin, Okta 등...

- 아래는 "IAM Identity Center"의 로그인 흐름이다.

![iam-identity-center-login-flow.png](images%2FAdvencedIdentity%2Fiam-identity-center-login-flow.png)

- 아래는 "IAM Identity Center"의 대략적인 예시이다.

![iam-identity-center-1.png](images%2FAdvencedIdentity%2Fiam-identity-center-1.png)

![iam-identity-center-2.png](images%2FAdvencedIdentity%2Fiam-identity-center-2.png)

#### 세분화된 권한 및 할당(Fine-grained Permission and Assignments)

- **Multi-Account Permissions**
  - "AWS Organizations"의 AWS 계정 간 액세스 관리
  - 권한 세트 - AWS 액세스를 정의하기 위해 사용자 및 그룹에 할당된 하나 이상의 IAM 정책 모음
- **Application Assignments**
  - 많은 SAML 2.0 비즈니스 애플리케이션에 SSO 액세스
  - 필요한 URL, 인증서 및 메타데이터 제공
- **속성 기반 액세스 제어(ABAC)**
  - "IAM Identity Center Identity Store"에 저장된 사용자의 속성에 따라 세분화된 권한
  - 예를 들어: 비용 센터, 타이틀, Locale
  - 사용 사례: 권한을 한 번 정의한 다음 속성을 변경하여 AWS 액세스를 수정한다.

![iam-identity-center-fine-grained-permissions.png](images%2FAdvencedIdentity%2Fiam-identity-center-fine-grained-permissions.png)

---

### Microsoft Active Directory (AD)

![what-is-ad.png](images%2FAdvencedIdentity%2Fwhat-is-ad.png)

- AD 도메인 서비스가 있는 모든 Windows Server에서 찾을 수 있다.
- 객체 데이터베이스: 사용자 계정, 컴퓨터, 프린터, 파일 공유, 보안 그룹
- 중앙 집중식 보안 관리, 계정 생성, 권한 할당
- 객체들이 트리 형태로 정리되어 있다.
- 하나의 그룹은 트리들의 숲과 같은 형태로 되어있다.

#### AWS Directory Services

- **AWS Managed Microsoft AD**
  - AWS에서 자신만의 AD를 생성하고, 로컬에서 사용자를 관리하며, MFA를 지원한다.
  - 사내 AD와 신뢰(trust) 연결을 설정한다.
- **AD Connector**
  - 사내 AD로 리디렉션할 디렉토리 게이트웨이(프록시), MFA 지원
  - 사용자는 사내 AD에서 관리된다.
- **Simple AD**
  - AWS의 AD 호환 관리 디렉토리
  - 사내 AD와 함께 가입할 수 없다.

![aws-directory-services.png](images%2FAdvencedIdentity%2Faws-directory-services.png)

#### Active Directory Setup

- "AWS Managed Microsoft AD(Directory Service)"에 연결한다.
  - 통합은 불가능하다.

![active-directory-setup-1.png](images%2FAdvencedIdentity%2Factive-directory-setup-1.png)

- 자체 관리 디렉토리 연결
  - "AWS Managed Microsoft AD"를 사용하여 양방향 신뢰 관계를 생성한다.
  - "AD Connector"를 생성한다.

![active-directory-setup-2.png](images%2FAdvencedIdentity%2Factive-directory-setup-2.png)

---

### AWS Control Tower

- Best Practice를 기반으로 안전하고 규정을 준수하는 다중 계정을 쉽게 관리할 수 있는 방법이다.
- "AWS Control Tower"는 "AWS Organizations"를 사용하여 계정을 생성한다.
- 대표적인 이점은 아래와 같다.
  - 클릭 몇 번으로 환경 설정을 자동화한다.
  - 가드레일을 사용하여 지속적인 정책 관리를 자동화한다.
  - 정책 위반 탐지 및 업데이트를 적용한다.
  - 대화형 대시보드를 통해 컴플라이언스를 모니터링한다.

#### Guardrails

- Control Tower 환경(AWS Accounts)에 대한 지속적인 거버넌스를 제공한다.
- 예방적(Preventive) 가드레일 - SCP를 사용한다.(예: 모든 계정에서 지역 제한)
- 검출(Detective) 가드레일 - AWS Config를 사용한다. (예: 태그가 지정되지 않은 리소스 식별)

![control-tower-guardrails.png](images%2FAdvencedIdentity%2Fcontrol-tower-guardrails.png)

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03