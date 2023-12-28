# Identity & Federation

- 이번 장에서는 **Solutions Architect Professional (SAP)** 을 준비하며 "Identity"와 "Federation"에 대해서 알아보도록 한다.

---

### IAM

- **사용자(User)**: 장기간 자격 증명(long term credentials)을 갖는다.
  - 사용자는 그룹화할 수 있다.
- **역할(Role)**: 단기적인 자격 증명(short term credentials)을 갖는다.
  - 우리는 STS 서비스를 이용해 이러한 역할들을 지원하고 자격 증명을 얻는다.
  - 권한이 있는 역할을 구행하기 위해 일시적인 자격 증명을 얻는다.
  - EC2 Instance Role: EC2 메타데이터 서비스를 사용해 EC2 인스턴스의 단기 자격 증명을 얻을 수 있고, 인스턴스에 따라 한 번에 한 역할만 할당할 수 있다. 
    이렇게 역할을 할당하는 방식으로 인스턴스는 S3 버킷이나 DynamoDB 테이블 등에 접근할 수 있는 권한을 가지게 된다.
  - Service Role: API Gateway, CodeDeploy와 같은 서비스가 있다면 Auto Scaling Group이나 Lambda Function 같은 서비스에서 작업을 해야 한다.
    이러한 서비스들은 역할이 있고 그 역할은 필요한 모든 작업을 할 수 있도록 프로비저닝 되어야 한다.
  - Cross Account Role: 다른 계정에서 작업을 수행하기 위해 다른 계정에 접근해야 하는 경우에 사용된다.
    사용자의 자격 증명을 공유하지 않는다.
- **정책(Policy)**: 역할이나 사용자가 할 수 있는 것을 정의한다.
  - AWS Managed: AWS에 의해 정의된 정책으로 시간이 지나면서 바뀔 수 있지만 아주 구체적인 작업을 한다.
  - Customer Managed: 사용자가 직접 정책을 만들고 다수의 사용자나 다수의 역할에 할당할 수 있다.
    시간이 지나면서 이전 정책을 발전시키거나 새로운 버전의 정책을 생성할 수 있다.
  - Inline: 특정 사용자나 역할에 할당되는 정책으로 시간이 흐르면서 발전시킬 수 있지만 사용자나 역할 간에는 공유할 수 없다.
- **Resource Based Policies**: S3 버킷 정책 또는 SQS Queue 정책을 가지고 여러 패턴의 작업을 수행할 수 있다.

#### IAM Policy

- IAM 정책은 Effect, Action, Resource, Condition, Policy 변수로 이루어진 JSON 파일로 구성되어 있다.
- 명시적 거부(Deny)가 허용(Allow)보다 우선되기 때문에 거부와 허용이 동시에 있다면 해당 작업은 결국 거부(Deny)된다.

![1-iam-policies-deep-dive.png](images%2F1-iam-policies-deep-dive.png)

- 첫 번째 Statement는 EC2 인스턴스 리소스에 정책을 부여하고 있다.
  - 인스턴스의 태그 중 `Department: Development`가 포함된 경우에만 `AttachVolume`, `DetachVolume` 작업을 허용하고 있다.
- 두 번째 Statement는 EC2 볼륨 리소스에 정책을 부여하고 있다.
  - 볼륨의 태그 중 `VolumeUser: "${aws:username}"`가 포함된 경우에만 `AttachVolume`, `DetachVolume` 작업을 허용하고 있다.

- 최대한의 보안을 위하여 최소한의 권한을 부여하는 것이 모범 사례다.
  - Access Advisor: 부여된 권한과 마지막 액세스를 확인한다.
  - Access Analyzer: 외부 엔티티와 공유하는 리소스를 분석한다.
- 접근 정책 예시는 [여기](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_examples.html)를 참고한다.

#### AWS Managed Policy

- AWS Managed Policy는 AWS에서 관리하는 정책이다.
- 아래는 `AdministratorAccess` 정책으로 모든 리소스에 대해서 모든 권한을 부여한다.

![2-managed-policies-administrator-access.png](images%2F2-managed-policies-administrator-access.png)

- 아래는 `PowerUserAccess` 정책이다.
- 좌측은 모든 리소스에 대해 `iam`, `organizations`, `account`를 제외한 액션을 허용한다.
- 우측은 `CreateServiceLinkedRole`, `DeleteServiceLinkedRole`, `ListRoles`, `DescribeOrganization`, `ListRegion` 액션을 허용하고 있다.

![3-managed-policies-poweruseraccess.png](images%2F3-managed-policies-poweruseraccess.png)

- 위의 예시에서 기억해야 하는 부분은 `Deny` 대신에 `NotAction`을 사용했다는 점이다.
- 만약 좌측의 정책에서 `NotAction` 대신 `Deny`를 사용했다면 우측의 정책에서 특정 액션을 허용하더라도 `Deny`의 우선순위가 높기 때문에 해당 액션이 허용되지 않는다.

#### IAM Policy Condition

- `"Condition": { "{condition-operator}" : { "{condition-key}" : "{condition-value}" }}`
- 아래와 같은 연산자(Operator)들이 있다.
  - String (StringEquals, StringNotEquals, StringLike 등..)
    - `"Condition": {"StringEquals": { "aws:PrincipalTag/job-category": "iamuser-admin" }}`
    - `"Condition": {"StringLinke": { "s3:prefix": [ "", "home/", "home/${aws:username}/" ]}}`
  - Numeric (NumericEquals, NumericNotEquals, NumericLessThan 등..)
  - Date (DateEquals, DateNotEquals, DateLessThan 등..)
  - Boolean (Bool):
    - `"Condition": {"Bool": {"aws:SecureTransport": "true"}}`
    - `"Condition": {"Bool": {"aws:MultiFactorAuthPresent": "true"}}`
  - (Not) IpAddress:
    - `"Condition": {"IpAddress": {"aws:SourceIp": "203.0.113.0/24"}}`
  - ArnEquals, ArnLike
  - Null: `"Condition": {"Null": {"aws:TokenIssueTime": "true"}}`

#### IAM Policy Variable & Tag

- `${aws:username}`: 흔하게 사용되는 변수로 AWS 사용자의 이름을 가져올 수 있다.
  - `"Resource":["arn:aws:s3:::mybucket/${aws:username}/*]`
- AWS Specific 변수
  - `aws:CurrentTime`, `aws:TokenIssueTime`, `aws:principaltype`, `aws:SecureTransport`, `aws:SourceIp`, `aws:userid`, `ec2:SourceInstanceARN` 등..
- Service Specific 변수
  - `s3:prefix`, `s3:max-keys`, `s3:x-amz-acl`, `sns:Endpoint`, `sns:Protocol` 등..
- Tag Based 변수
  - `iam/ResourceTag/key-name`, `aws:PrincipalTag/key-name` 등..

#### IAM Role vs Resource Based Policy

- 리소스에 정책을 연결(예: S3 버킷 정책)하는 방식과 프록시 역할을 사용하는 방법의 차이를 알아본다.

![4-iam-role-vs-resource-based-policy.png](images%2F4-iam-role-vs-resource-based-policy.png)

- 첫 번째는 계정 A가 계정 B의 역할을 가지고 계정 B의 S3 버킷에 접근한다.
- 두 번째는 S3 버킷 정책을 활용하여 계정 A가 계정 B의 S3 버킷에 접근한다.

- **두 방식에는 큰 차이점이 있다.**
- 역할(사용자, 애플리케이션, 서비스)을 맡으면 원래 사용 권한을 포기하고 역할에 할당된 사용 권한을 가져간다.
- 리소스 기반 정책을 사용하게 되면 원래 사용하던 권한을 포기할 필요가 없다.
- 예를 들어, 계정 A의 사용자는 계정 A의 DynamoDB 테이블을 스캔하여 계정 B의 S3 버킷에 덤프해야 한다.
  - 이런 경우 계정 A에 IAM 역할이 있어야 되고 S3 버킷에 리소스 정책과 계정 B가 있어야 된다.
- 많은 서비스들이 리소스 기반 정책을 지원하고 있다.
  - S3 버킷, SNS 토픽, SQS 큐, Lambda 함수, ECR, Backup EFS, Glacier, Cloud9, AWS Artifact, Secrets Manager, ACM, KMS, CloudWatch Logs, API Gateway, EventBridge 등..

#### IAM Permission Boundary

- IAM 권한 경계는 사용자 및 역할에 대해서 지원된다. 그룹에 대해서는 지원되지 않는다.
- 관리 정책을 사용하여 IAM 엔터티가 얻을 수 있는 최대 권한을 설정하는 고급 기능이다.

![5-iam-permission-boundary.png](images%2F5-iam-permission-boundary.png)

- 좌측은 IAM 권한 경계를 설정하고 있다. 모든 리소스에 대해서 `s3:*`, `cloudwatch:*`, `ec2:*` 작업을 허용하고 있다.
- 우측은 모든 리소스에 대해 `iam:CreateUser` 액션을 허용하고 있다.
- 결과적으로 교집합이 없기 때문에 사용자는 어떠한 작업도 수행할 수 없다.

- AWS Organizations SCP와 조합하여 사용할 수 있다.

![6-iam-permission-boundary-2.png](images%2F6-iam-permission-boundary-2.png)

- 대표적인 사용 사례는 아래와 같다.
  - 권한 범위 내에서 관리자가 아닌 사용자에게 책임을 위임한다.(예: 새로운 IAM 사용자 생성)
  - 개발자가 정책을 자체적으로 할당하고 자신의 권한을 관리하는 동시에 자신의 권한을 상승시키지 못하도록 보장한다. (= 스스로 관리)
  - 한 명의 특정 사용자를 제한하는 데 유용하다. (Organizations & SCP를 사용하는 전체 계정 대신)

- 권한 경계를 설정하는 자세한 방법은 [여기](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html)를 참고한다.

---

### IAM Access Analyzer

- IAM 콘솔 내의 서비스로 어떤 리소스가 외부에 공유될지 알아내는 데 사용된다.
  - S3 버킷
  - IAM 역할
  - KMS 키
  - Lambda 함수 & Layer
  - SQS Queue
  - Secrets Manager Secret
- 신뢰 영역 (Zone of Trust) 정의 = AWS 계정 또는 AWS 조직에 상응하는 영역이다.
- 신뢰 영역 외부에서 자원에 접근할 수 있는 리소스가 발견된다면 보고된다.

![7-iam-access-analyzer.png](images%2F7-iam-access-analyzer.png)

- **IAM Access Analyzer Policy Validation**
  - IAM 정책 문법 및 모범 사례에 대한 정책을 검증한다.
  - 일반 경고, 보안 경고, 오류, 제안 개선 방법에 대한 행동 가능한 권고 사항을 받게 된다.
  - 실행 가능한 권장 사항을 제공한다.
- **IAM Access Analyzer Policy Generation**
  - 액세스 활동을 기반으로 IAM 정책을 생성할 수 있다.
  - CloudTrail 로그를 검토하여 세분화된 권한과 적절한 조치 및 서비스를 통해 정책을 생성한다.
  - 최대 90일 동안 CLoudTrail 로그를 검토할 수 있다.

![8-iam-access-analyzer.png](images%2F8-iam-access-analyzer.png)

- 예시에서 S3 버킷이 있고 사용자나 계정 외부 클라이언트와 특정 역할과 공유할 수 있다. (IP 또는 VPC Endpoint 사용)
- 하지만 신뢰 영역을 계정으로 정의하고 역할만 계정 내에 사용자와 VPC Endpoint로 정의한다면 계정과 외부 클라이언트는 플래그로 표시되고 콘솔에서 볼 수 있다.
- 보안 위 요소라고 판단되면 조치를 취할 수 있다.

---

### STS (Security Token Service)

- STS는 같은 계정에서 다른 계정에서 역할을 수행할 수 있게 해주며, ID 페더레이션도 가능하게 지원해준다.
- 계정 또는 교차 계정 내에서 IAM 역할을 정의한다.
- 정의된 IAM 역할에 액세스할 수 있는 보안 주체를 정의한다.
- AWS STS를 사용하여 자격 증명을 검색하고 액세스할 수 있는 IAM 역할을 가장한다. (`AssumeRole` API)
- 임시 자격 증명의 유효 시간은 15분에서 12시간 사이다.

![9-sts-assume-role.png](images%2F9-sts-assume-role.png)

- 예시에서 동일한 계정 또는 다른 계정 내에 있는 역할에 액세스하기를 원하므로 이를 위해 STS `AssumeRole` API를 사용한다.
- STS는 권한을 확인하고 설정이 올바른지 확인하고 어떻게 보이는지 자세히 살펴본 다음 보안 자격 증명을 다시 얻는다.
- 일시적인 이 보안 자격 증명으로 타깃 역할에 접근할 수 있다.

#### Assuming a Role

- 자신이 소유한 AWS 계정의 IAM 사용자가 자신이 소유한 다른 계정의 리소스에 액세스할 수 있도록 액세스 권한을 제공한다.
- 타사 소유의 AWS 계정에서 IAM 사용자에 대한 액세스를 제공한다.
- AWS에서 제공하는 서비스에 대한 액세스 권한을 AWS 리소스에 제공한다.
- 외부에서 인증된 사용자(Identity Federation)에 대한 액세스를 제공한다.
- 역할에 대한 활성 세션 및 자격 증명을 취소할 수 있다.
  - 정책에 "Time Statement"를 추가하거나, `AWSRevokeOldSessions`로 정책을 관리하면 된다.
- **역할(사용자, 응용프로그램, 서비스)을 맡으면 원래 사용 권한을 포기하고 역할에 할당된 사용 권한을 가진다.**

#### 다른 AWS 계정의 IAM 사용자에 대한 액세스 제공

- IAM 사용자에게 AWS 계정 내의 역할 또는 소유한 다른 AWS 계정에 정의된 역할로 전환할 수 있는 권한을 부여할 수 있다.

![10-providing-access-to-iam-user-own-or-another.png](images%2F10-providing-access-to-iam-user-own-or-another.png)

- 이점 (Benefit)
  - 역할을 수행하려면 사용자에게 명시적으로 권한을 부여해야 한다.
  - 사용자는 AWS Management Console을 사용하여 적극적으로 역할로 전환하거나 AWS CLI 또는 AWS API를 사용하여 역할을 맡아야 한다.
  - MFA(Multi-Factor Authentication) 보호 기능을 역할에 추가하여 MFA 장치로 로그인한 사용자만 역할을 맡을 수 있다.
  - CloudTrail을 사용하여 최소한의 권한을 부여하고 감사할 수 있다.

#### Cross account access

![11-cross-account-access.png](images%2F11-cross-account-access.png)

- S3 버킷이 있는 프로덕션 계정이 있고 두 그룹(Tester, Developer)의 사용자로 된 개발 계정이 있다.
- 개발자에게 S3 버킷 프로덕션 앱에 액세스 권한을 제공한다.
  1. 프로덕션 계정에서 관리자가 역할을 생성해 개발 계정 읽기/쓰기 액세스 권한을 부여하는 역할을 생성한다.
  2. 관리자가 그룹 개발자 구성원에게 `UpdateApp` 역할을 맡을 수 있는 권한을 부여한다.
  3. 개발자 그룹의 사용자는 STS API를 사용하여 역할에 액세스 또는 요청 액세스를 할 수 있다.
  4. STS는 임시 자격 증명을 반환한다.
  5. 반환받은 자격 증명을 이용하여 사용자는 역할 자격 증명을 통해 S3 버킷에 액세스할 수 있다.

#### 타사 소유의 AWS 계정에 대한 액세스 제공

- 타사 소유의 AWS 계정에 액세스를 제공하려는 경우 "외부 ID"가 존재한다.
- 신뢰 영역(Zone of Trust): 우리가 소유한 모든 계정과 조직을 말한다. 
- 신뢰 영역 외부(Outside Zone of Trust): 3rd Party 조직
- IAM Access Analyzer를 사용하여 노출된 리소스를 확인한다.
- 제3자에 대한 접근 권한 부여
  - 제3자의 AWS 계정 ID를 확인한다.
  - **External ID (사용자와 제3자 간의 비밀)**
    - 사용자와 타사 간의 역할과 고유하게 연관시킨다.
    - 신뢰를 정의할 때 제공해야 하며 역할을 맡을 때 제공해야 한다.
    - 제3자가 선택해야 한다.
  - IAM 정책에서 사용 권한 정의

#### Confused deputy

![12-confused-deputy.png](images%2F12-confused-deputy.png)

- 좌측에는 AWS 계정이 있고 중앙에는 IAM 사용자가 있고 우측에는 다른 AWS 계정이 있다.
- 우측의 다른 AWS 계정이 우리의 계정을 공격할 것이고 중앙의 IAM 사용자를 이용하는 상황을 가정해본다.

- **External ID를 사용하지 않은 경우** 먼저 확인해 본다.
  - AWS 계정이 `AWS1:ExampleRole`을 생성하고 중앙의 계정에게 제공한다.
  - 중앙의 계정은 이 역할의 ARN을 활용하여 우리 서비스의 리소스에 접근한다.
  - 이번에는 우측의 AWS 계정이 중앙의 계정에게 동일한 역할인 `AWS1:ExampleRole`의 ARN을 제공한다.
  - 중앙의 계정은 이 역할이 좌측(우리) 계정의 역할인지 다른(우측) 계정의 역할인지 알지못한다.
  - 이러한 경우 우측 계정에서 작동한다고 생각할 수 있지만 실제로는 좌측 계정에서 작동한다.
- **External ID를 사용하여 공격으로부터 보호하는 방법**에 대해서 알아본다.
  - 위의 상황과 설정은 같지만 이번에는 `AWS1:ExampleRole`의 ARN을 제공할 때, External ID를 제공한다.
  - External ID는 비밀로 좌측(우리) 계정과 중앙 계정 사이에 공유된다.
  - 이번에는 우측의 AWS 계정이 중앙의 계정에게 동일한 역할인 `AWS1:ExampleRole`의 ARN을 제공한다. 하지만 External ID는 알지못한다.
  - 잘못된 External ID로 인해 우리 계정의 리소스에 접근할 수 없다.

#### Session Tag

- IAM 역할을 맡을 때, 사용자 페더레이션이 있어서 STS를 활용할 수 있다.
- STS에서 IAM 역할 또는 페더레이션 사용자를 가정할 때 전달하는 태그다.

![13-session-tags-in-sts.png](images%2F13-session-tags-in-sts.png)

- 사용자는 STS `AssumeRole` API를 호출하여 API 호출의 일부로 세션 태그를 전달한다. (예. `Department=HR`)
- STS는 세션 태그와 함께 해당 사용자에 대해 임시 보안 자격 증명을 반환한다.
- IAM 정책에서 `aws:PrincipalTag` 조건을 사용할 수 있다.
  - 요청을 하는 주체에 연결된 태그를 정책에서 지정한 경우 태그와 비교한다.
  - 예를 들어, 요청을 하는 주체가 지정된 태그를 가지고 있는 경우에만 주체가 세션 태그를 전달하도록 허용할 수 있다.

#### API

- STS에는 중요한 몇 가지 API가 있다.
- **AssumeRole**: 계정 또는 교차 계정 내의 역할에 액세스한다.
- **AssumeRoleWithSAML**: SAML로 로그인한 사용자에 대한 자격 증명을 반환한다.
- **AssumeRoleWithWebIdentity**: IDP로 로그인한 사용자의 자격 증명을 반환한다.
  - 예시 제공자에는 Amazon Cognito, Login with Amazon, Facebook, Google, OpenID Connect 호환 ID 제공자가 있다.
  - **AWS는 Cognito를 사용할 것을 권장한다.**
- **GetSessionToken**: 사용자 또는 AWS 계정의 루트 사용자가 MFA를 이용해 자격 증명을 얻을 때 사용한다.
- **GetFederationToken**: Federation 사용자(일반적으로 기업 네트워크 내의 분산 앱에 대한 크레딧을 제공하는 프록시 앱)의 임시 자격 증명을 가져온다.

---

### Identity Federation

- AWS 외부 사용자에게 계정의 AWS 리소스에 액세스할 수 있는 권한을 부여한다.
- IAM 사용자를 생성할 필요가 없다. (사용자 관리는 AWS 외부에 있음)
  - 사용자가 회사 디렉토리 내에 이미 존재하기 때문에 AWS 외부에서 사용자 관리를 수행하고 싶기 때문에 특정 IAM 사용자를 생성하지 않는다.
- 대표적은 사용 사례는 아래와 같다.
  - 기업은 고유한 ID 시스템(예: Active Directory)을 가지고 있다.
  - AWS 리소스에 액세스해야 하는 웹/모바일 애플리케이션을 위해 사용된다.
- Identity Federation은 다양하게 사용될 수 있다.
  - SAML 2.0
  - Custom Identity Broker
  - Web Identity Federation With(out) Amazon Cognito
  - Single Sign-On (SSO)

![14-identity-federation.png](images%2F14-identity-federation.png)

- 사용자가 AWS에 액세스하기를 원하고 ID 공급자가 있는 것처럼 보인다.
- AWS와 Identity Provider 사이에 "trust relationship"을 생성해야 한다.
- 공급자에게서 아이덴티티를 받아도 괜찮다고 AWS에 알려주어야 한다.
- 사용자는 아이디 공급자에 로그인을 하고 AWS에 대한 자격 증명을 다시 받고 임시 자격 증명을 이용해 AWS에 접근하게 된다.

#### SAML

- "SAML 2.0"은 "Security Assertion Markup Language 2.0"의 약자다.
- 다수의 ID 제공자(예: ADFS)가 사용하는 공개 표준이다.
  - 마이크로소프트 ADFS(Active Directory Federation Service)와의 통합을 지원한다.
  - AWS와 함께 SAML 2.0과 호환되는 IdPs와 통합을 지원한다.
- 임시 자격 증명을 사용하여 AWS 콘솔, AWS CLI 또는 AWS API에 엑세스할 수 있다.
  - 각 직원에 대해 IAM 사용자를 생성할 필요가 없다.
  - AWS IAM과 SAML 2.0 Identity Provider 간의 신뢰를 설정해야 한다. (양쪽 모두)
- 내부적으로 `AssumeRoleWithSAML` STS API를 사용한다.
- SAML 2.0 Federation은 "오래된 방식"이며, Amazon Single Sign-On (AWS SSO) Federation은 새로운 관리형 서비스다.
- ADFS & SAML 관련된 자세한 내용은 [여기](https://aws.amazon.com/blogs/security/enabling-federation-to-aws-using-windows-active-directory-adfs-and-saml-2-0/)를 참고한다.

#### SAML 2.0 Federation - AWS API Access

![15-saml-federation-aws-api-access.png](images%2F15-saml-federation-aws-api-access.png)

- 자격 증명 공급자가 있는 회사가 있고 AWS와 사용자가 S3 버킷에 액세스하기를 원한다.
- 회사 내부에 Portal 또는 Identity Provider가 있고 사용자는 IdP에 인증 요청을 한다.
- IdP는 ID 저장소(예: LDAP 기반)를 통해 요청을 확인한 다음 로그인에 성공하면 "SAML Assertion"을 반환한다.
- 이 "SAML Assertion"은 사용자가 요청이 가능한 사용자라는 증거이며 `AssumeRoleWithSAML` API를 사용하여 STS 서비스를 호출할 수 있다.
- STS 서비스는 임시 보안 자격 증명을 반환한다.
- 임시 자격 증명으로 AWS API를 사용하여 S3 버킷과 같은 서비스에 접근할 수 있다.

#### SAML 2.0 Federation - AWS Console Access

![16-saml-federation-aws-console-access.png](images%2F16-saml-federation-aws-console-access.png)

- IdP 및 LDAP ID 기반 저장소가 있다.
- AWS Sign-in Endpoint(`https://signin.aws.amazon.com/saml`)을 통해서 AWS 로그인에 접근한다.
- 반환되는 STS 서비스를 통해 보안 자격 증명을 획득하여 특수 URL, AWS 콘솔의 로그인 URL을 반환하고 사용자는 AWS Management로 리디렉션된다.
- 이외의 과정은 API를 통한 접근과 매우 유사하다.

#### SAML 2.0 Federation - Active Directory FS (ADFS)

![17-saml-federation-active-directory-fs.png](images%2F17-saml-federation-active-directory-fs.png)

- 이전의 과정들과 거의 동일하며 유일한 차이점은 좌측 상단에 Microsoft Active Directory Federation 서비스가 있다는 점이다.
- IdP와 ADFS와의 차이점을 이해하는 것이 중요하다.

#### Custom Identity Broker Application

![18-custom-identity-broker-application.png](images%2F18-custom-identity-broker-application.png)

- ID 공급자가 SAML 2.0과 호환되지 않는 경우 "Custom Identity Broker"를 사용해야 한다.
- Identity Broker에 로그인하여 로그인 자체를 확인하지만 Identity Broker는 AWS에서 임시 자격 증명을 요청하는 역할을 하며, 사용할 수 있는 AWS API가 없다.
- 따라서 "Custom Identity Broker"는 관리 권한이 있으며 이러한 자격 증명을 반환하는 STS 서비스에서 직접 임시 자격 증명을 요청할 수 있다.
- Identity Broker가 책임을 지고 해당 사용자에 대한 적절한 IAM 역할을 결정해야 하므로 사용자 관리를 Identity Broker 자체로 푸시한다는 의미다.
- `AssumeRole` 또는 `GetFederationToken` STS API를 호출하여 보안 자격 증명이 검색되면 사용자에게 전달되고 사용자는 API AWS에 액세스하거나 관리 콘솔을 통해 리디렉션될 수 있다.

#### Web Identity Federation - Without Cognito

![19-web-identity-federation-without-cognito.png](images%2F19-web-identity-federation-without-cognito.png)

- Cognito를 사용하지 않는 것을 AWS에서는 권장하지 않는다.
- 클라이언트에서 직접 서비스에 액세스하고 싶지만 타사 서비스를 사용하여 인증을 한다.
- 자격 증명 공급자는 Amazon, Google, Facebook 또는 모든 OpenID Connect 호환 IdP일 수 있다.
- AWS와 함께 트러스트 메커니즘으로 설정되며, 흐름은 클라이언트가 타사 IDP에 로그인하는 것이고 웹 ID 토큰은 클라이언트와 다시 공유되는 것이다.
- 클라이언트는 `AssumeRoleWithWebIdentity` API라는 특수 STS API를 사용한 다음 해당 토큰을 거래하여 AWS용 임시 보안 자격 증명을 받는다.
- 이러한 자격 증명은 AWS 리소스에 직접 액세스하는 데 사용된다.

#### Web Identity Federation - With Cognito

![20-web-identity-federation-with-cognito.png](images%2F20-web-identity-federation-with-cognito.png)

- Cognito에서 필요한 최소한의 권한으로 IAM 역할을 생성한 다음 OIDC IdP와 AWS 간의 신뢰를 구축하기만 하면 되므로 Web Identity Federation으로 선호된다.
  - 필요한 권한이 가장 적은 Cognito를 사용하여 IAM 역할을 생성한다.
  - OIDC IdP와 AWS 간 신뢰를 구축한다.
- 클라이언트는 여전히 제3자 Identity Provider에 로그인하여 토큰을 제공받는다.
- 클라이언트는 제공받은 토근을 Cognito와 교환하여, Cognito 토큰을 획득한다.
- 해당 Cognito 토큰을 STS와 교환하여 AWS용 임시 보안 자격 증명을 받을 수 있다.
- 이후 클라이언트는 AWS 리소스에 직접 액세스할 수 있다.
- Cognito는 아래와 같은 장점이 있다.
  - 익명의 사용자를 지원한다.
  - MFA를 지원한다.
  - 데이터 동기화를 지원한다.
- Cognito가 토큰 자판기(TVM, Token Vending Machine)을 대체한다.

#### Web Identity Federation - IAM Policy

![21-web-identity-federation-iam-policy.png](images%2F21-web-identity-federation-iam-policy.png)

- Web Identity Federation으로 인증된 후 IAM 정책 변수로 사용자를 식별할 수 있다.

![21-web-identity-federation-iam-policy.png](images%2F21-web-identity-federation-iam-policy.png)

- 예시는 버킷을 사용자 ID로 나열하고 해당 특정 접두사가 있는 개체를 업데이트하고 넣을 수 잇도록 허용하는 정책의 예시이다.
- Web Identity Federation이 있는 경우 몇 가지 특수 변수(예: `cognito-identity`)를 사용할 수 있는 아이디어다.
- 아래는 대표적으로 사용되는 몇 가지 변수 예시다.
  - `cognito-identity.amazonaws.com:sub`
  - `www.amazon.com:user_id`
  - `graph.facebook.com:id`
  - `accounts.google.com:sub`

---

### AWS Directory Service

#### Microsoft Active Directory (AD)

- AD 도메인 서비스라는 것이 있는 모든 Windows 서버에서 찾을 수 있다.
- 사용자 계정, 컴퓨터, 프린터, 파일 공유, 보안 그룹과 같은 객체의 데이터베이스로 이동한다.
- Microsoft 환경에서 보안 관리, 새 계정 생성, Windows의 모든 파티션 할당을 위한 중앙 집중식 방식이 된다.
- 개체, 사용자, 계정, 프린터 등이 함께 그룹화되어 트리라고 하는 것이 구성된다.
- 트리의 그룹을 숲(포레스트)라고 한다.

![22-microsoft-active-directory.png](images%2F22-microsoft-active-directory.png)

- John이 사용자 이름이고 이 도메인 컨트롤러에 연결된 다른 Microsoft 컴퓨터가 있다.
- 이러한 시스템 중 하나에서 John 암호를 사용하여 연결할 수 있고 활성 디렉토리가 로그인 자체를 확인한다.
- 도메인 컨트롤러에서 이러한 모든 로그인을 동기화할 수 있다.

#### Active Directory Federation Service (ADFS)

- ADFS는 응용 프로그램 전체에서 Single-Sign On을 제공하고 타사 전체에서 SAML을 제공한다.
- 제3자 SAML은 AWS 콘솔, Dropbox, Office365 등이 있다.

![23-active-directory-federation-service.png](images%2F23-active-directory-federation-service.png)

- 사용자는 Microsoft Active Directory에 대해 사용자를 인증하는 ADFS 서비스의 URL을 탐색한다.
- SAML 토큰은 사용자에게 반환되며 AWS와 교환하여 콘솔용 로그인 URL을 얻는다.

#### AWS Directory Service

![24-aws-directory-service.png](images%2F24-aws-directory-service.png)

- **AWS Managed Microsoft AD**
  - 클라우드의 Microsoft AD가 된다.
  - AWS에서 로컬로 사용자를 관리하는 자체 AD를 생성할 수 있으며 MFA를 지원한다.
  - 온프레미스 AD와 트러스트를 설정해야 한다.
  - AWS Manage AD가 트러스트를 통해 온프레미스 AD에 연결되어 있고 MFA를 지원하므로 사용자가 우측 또는 좌측에서 인증할 수 있다.
- **AD Connector**
  - 클라우드 AD에서 온프레미스 AD로 연결하는 프록시이며 MFA를 지원한다.
  - 사용자는 On-Premise AD라는 한 곳에서만 관리된다.
  - 인증은 프록시인 AD 커넥터로 이동하고 온프레미스 AD로 프록시를 다시 가져와 응답을 받는다.

- AWS Microsoft Manage AD의 경우 사용자가 정의된 두 곳이 있으며 둘 사이의 신뢰가 필요하다.
- AD Connector를 사용하면 온프레미스에서만 정의되며 클라우드에서 직접 해당 온프레미스 AD에 액세스하기 위한 프록시를 정의했다.

- **Simple AD**
  - AWS의 AD 호환 관리 디렉토리다.
  - AWS에서 관리되지면 독립 실행형이며 온프레미스 AD와 결합될 수 있다.
  - 지원되는 기능이 없지면(MFA 미지원) 저렴하게 사용할 수 있다.

#### AWS Managed Microsoft AD

- VPC내에 Microsoft Active Directory를 배포하게 된다.
- EC2 Windows Instance
  - EC2 Windows 인스턴스가 도메인에 가입하여 기존 AD 애플리케이션(셰어포인트 등)을 실행할 수 있다.
  - 다중 계정 및 VPC의 EC2 인스턴스에서 바로 원활한 도메인 조인이라는 작업을 수행할 수 있다.
- Integration
  - 클라우드에 관리형 Microsoft AD를 배포하면 SQL Server용 RDS, AWS WOrkspaces, Quicksight 등과 원활하게 통합되어 사용할 수 있다.
  - AWS SSO를 통해 타사 애플리케이션에 대한 액세스를 제공하고 이를 즉시 보여주는 또 다른 다이어그램을 만들 수 있다.
- 독립 실행형 저장소 AWS 또는 On-Premise AD에 가입할 수 있다.

![25-aws-managed-microsoft-ad.png](images%2F25-aws-managed-microsoft-ad.png)

- 다중 AZ 배포는 최소 2개의 AZ에 설치되며 스케일링과 사용 가능성을 증가시키고 싶다면 더 많은 도메인 컨트롤러를 추가할 수 있다.
- 자동화된 백업을 사용할 수 있다.
- 디렉토리의 자동화된 다중 지역 복제본을 얻을 수 있다.

![26-aws-microsoft-managed-ad-integrations.png](images%2F26-aws-microsoft-managed-ad-integrations.png)

- 관리형 Microsoft AD DC는 중앙에 있으며 양방향 포레스트 트러스트(AD two-way Forest trust)라는 것을 사용하여 온프레미스 활성 디렉토리와 통합될 수 있다.
- 여러 데이터베이스 서비스와 통합될 수 있다.
  - Sql Server용 RDS, AWS Workspace, Quicksight
- Single-Sign On을 통해 Github, Dropbox, Office 365 등과 같은 더 많은 SAML 비즈니스 애플리케이션에 액세스할 수 있다.
- EC2 인스턴스에 배포되는 SQL Server, SharePoint, .NET 애플리케이션과 같은 서비스와 연결될 수 있다.

#### Connect to on-premises AD

- On-Premise AD를 AWS Manage Microsoft AD에 연결할 수 있는 기능을 제공한다.
- 연결을 위해서 Direct Connect 또는 Site-to-Site VPN 설정이 필요하다.
- 포레스트 트러스트는 총 세 가지 종류의 설정을 할 수 있다.
  - One-way trust: AWS => On-Premise
  - One-way trust: On-Premise => AWS
  - Two-way forest trust: AWS <=> On-Premise
- 포레스트 트러스트는 동기화와는 다른 개념이다.
  - 복제는 AWS Managed Microsoft AD에서 지원하는 것이 아니다.
  - 사용자는 서로 다른 두 개의 Microsoft 활성 디렉토리에서 독립적으로 생활하고 있는 것이다.
  - 하지만 포레스트 트러스트 덕분에 한 사용자가 누락되었을 때, 서로 커뮤니케이션이 가능하다.

![27-connect-to-onpremise-ad.png](images%2F27-connect-to-onpremise-ad.png)

- 기존의 Active Directory 앱은 온프레미스에 연결할 수 있다.
- 간편한 인스턴스로 Microsoft Manage AD에 도모ㅔ인을 원활하게 연결할 수 있다.
- 양방향 포레스트 트러스트를 설정하여 신뢰(trust)를 생성하였기 때문에 기존 Active Directory 앱이 AWS에 속한 도메인을 요청하려는 경우에 대비한다.
- 끝까지 이동하여 우측의 사용자를 확인할 수 있다.

#### Active Directory Replication

- 클라우드의 EC2에 AD 복제본을 생성하여 DX 또는 VPN이 중단될 경우의 지연 시간을 최소화할 수 있다.
- Direct Connect나 Site-to-Site VPN이 다운되더라도 사용자가 연결되고 올바르게 작동하기를 원하는 경우 AD 복제를 사용한다.
- 이러한 AD 간에도 신뢰(trust)를 구축해야 한다.

![28-active-directory-replication.png](images%2F28-active-directory-replication.png)

- 한 도메인에는 On-Premise Microsoft AD가 있고 다른 도메인에는 VPC에서 AWS가 Microsoft AD를 관리하고 있다.
- 복제를 설정하는 유일한 방법은 두 개의 Windows 인스턴스에 Active Directory를 배포하는 것이며 복제를 설정해야 한다.
- 이러한 복제를 설정하면 대기 시간을 최소화하고 재해 복구 전략을 갖는 데 도움이 될 수 있는 On-Premise Microsoft AD 복제본이 생성된다.
- EC2 인스턴스와 AWS Managed Microsoft AD DC 간의 양방향 포레스트 트러스트를 설정할 수 있다.

#### AD Connector

- AD Connector는 디렉토리 요청을 사내 Microsoft Active Directory로 리디렉션하는 디렉토리 게이트웨이다.
- 캐싱 기능을 지원하지 않는다.
- 사용자를 On-Premise에서만 관리할 수 있으며, 설정되는 신뢰(trust)가 없고 Direct Connect나 Site-to-Site VPN이 필요하지 않다.
- SQL Server에서 작동하지 않으며, 공유 디렉토리에서 원활하게 결합할 수 없다.

![29-ad-connector.png](images%2F29-ad-connector.png)

- 회사 사무실과 데이터베이스 클라우드가 있고 둘 사이에 VPN이나 Direct Connect 연결이 있다.
- 회사 사무실에서 Active Directory를 직접 가질 수 있고 사용자들은 아래와 같은 플로우를 수행할 것이다.
  - 사용자 지정 서명 페이지에서 자격 증명을 사용한다.
  - 서명 페이지는 다중 AZ인 AD 커넥터에 연결된다. (하나의 AZ에서 AD 커넥터가 사용된다.)
  - 연결된 AD 커넥터는 회사 사무실의 활성 디렉토리까지 요청을 프록시하고 LDAP 인증을 수행한다.
  - AD 커넥터가 STS로부터 IAM 임시 자격증을 돌려받는다.
  - 사용자는 AWS에 인증된다.

#### Simple AD

- Simple AD는 저렴한 Active Directory 호환 서비스로 일반적인 디렉토리 기능을 갖추고 있다.
- EC2 인스턴스 조인을 지원하고, 사용자 및 그룹 관리를 지원한다.
- MFA, RDS SQL Server, AWS SSO를 지원하지 않는다.
- 500 ~ 5,000명의 소규모 사용자만 지원하며 계층에 따라 사용자가 달라진다.
- Samba 4로 구동되며 API 측면에서 Microsoft AD와 호환(또는 LDAP 호환)되지만 더 저렴하고 규모가 작다.
- 신뢰(trust)를 생성할 필요가 없다.

---

### AWS Organizations

![30-aws-organization.png](images%2F30-aws-organization.png)

- 조직 내에는 최상위 루트 조직 단위 또는 OU가 있으며 여기에 모든 관리 목적으로 사용될 계정인 마스터 계정이 포함된다.
- OU의 목적에 따라 여러개의 OU를 가질 수 있다.
  - 예를 들어, 개발 환경에 대한 OU를 가질 수 있으며 이러한 방식으로 여러 구성원 계정을 가질 수 있다.
  - 조직을 관리하는 데 사용되는 관리 계정과 일반 계정인 멤버 계정을 구분해서 사용해야 한다.
- Dev용 OU를 생성할 수 있고, Prod용 OU도 생성할 수 있으며 다른 멤버 계정과 함께 OU내에 속하는 OU를 생성할 수 있다.
  - 예시에서 Prod OU는 HR OU와 Finance OU를 가지고 있다.

#### OrganizationAccountAccessRole

- 조직과 관리 계정이 있을 때, 조직 서비스에서 API를 사용하여 회원 계정을 생성하면 회원 계정 내에서 자동으로 IAM 역할이 생성된다.
  - 이 IAM 역할을 `OrganizationAccountAccessRole`이라고 한다.
  - 마스터 계정은 회원 계정에 대한 관리 업무를 수행해야 할 때 API를 사용하여 해당 관리 역할을 맡게 된다.
  - 따라서 이 IAM 역할은 멤버 계정의 전체 관리 권한을 마스터 계정에 부여한다.
- 회원 계정의 전체 관리자 권한을 관리 계정에 부여하는 역할이다.
- 구성원 계정에서 관리자 작업을 수행하는 데 사용된다. (예를 들어, IAM 사용자 만들기)
- 권한이 있는 경우 마스터 계정의 IAM 사용자가 이 역할을 맡을 수도 있다.
  - AWS Organizations로 작성된 모든 새 구성원 계정에 자동으로 추가된다.
- 기존 구성원 계정을 초대하는 경우 수동으로 생성해야 한다.

![31-organization-account-access-role.png](images%2F31-organization-account-access-role.png)

#### Multi Account Strategy

- 규제 제한(SCP 사용)에 따라 부서별, 비용 센터별, 개발/테스트/프로덕트별 계정을 생성하고 보다 나은 리소스 격리(예: VPC), 별도의 계정별 서비스 제한, 로깅을 위한 격리된 계정을 생성한다. 
- 사용할 수 있는 여러가지 전략들이 있다.
  - 다중 계정 사용 vs 다중 VPC가 포함된 하나의 계정 사용
  - 청구를 목적으로 태그 지정 표준 사용
  - 모든 계정에서 CloudTrail 사용, 중앙 S3 계정으로 로그 전송
  - CloudWatch 로그를 중앙 로깅 계정으로 전송
  - 보안을 위한 중앙 계정을 생성
- 이러한 전략들은 단지 전략일 뿐이며 가이드라인은 존재하지 않는다.
  - 원하는 작업을 결정하는 것은 사용자의 아키텍처에 달려있다.

#### Organizational Unit (OU)

![31-organizational-units.png](images%2F31-organizational-units.png)

- 관리 계정이 있고 비즈니스 단위당 OU가 있고 각 비즈니스 단위 내에서 여러 계정이 있을 수 있다.
- 예를 들어, 환경에 따라 OU를 나눈다면 management 계정과 dev, test, prod 유형의 OU가 있을 수 있다.
- 마지막은 프로젝트 기반으로 OU를 나눈 예시다.

#### AWS Organization 기능

- 통합 청구 기능: 모든 계정에서 청구를 집계하고 관리 계정에서 직접 단일 결제 방법을 제공한다.
  - 모든 계정에 걸친 통합 청구 - 단일 지불 방법
  - 통합 사용량에 따른 가격 책정 이점 (EC2, S3에 대한 볼륨 할인 등..)
- 모든 기능(기본값)
  - SCP를 포함하여 통합 청구 기능을 제공한다.
  - 초대된 계정은 모든 기능 사용을 승인해야 한다.
  - 회원 계정이 기관을 떠나는 것을 방지하기 위해 SCP를 적용할 수 있는 기능을 통합 청구 기능으로만 다시 전환할 수 없다.
  - 통합 청구 기능으로만 다시 전환할 수 없다.

#### Reserved Instance

- 청구 목적으로 AWS Organization의 통합 청구 기능은 조직의 모든 계정을 하나의 계정으로 취급한다.
- 즉, 조직의 모든 계정은 다른 계정에서 구입한 예약 인스턴스의 시간당 비용 혜택을 받을 수 있다.
- 조직의 지급자 계정(Payer Account, Management Account)은 지급자 계정을 포함하여 해당 조직의 모든 계정에 대해 RI 할인 및 Saving Plans 할인 공유를 해제할 수 있다.
- 즉, RI 및 Saving Plans 할인은 공유가 해제된 계정 간에 공유되지 않는다.
- RI 또는 Saving Plans 할인을 계정과 공유하려면 두 계정 모두 공유가 켜져 있어야 한다.

#### Moving Account

- 만약 다른 AWS Organization이 있고 멤버 계정을 다른 Organization으로 마이그레이션하고 싶다면 아래의 단계를 따라야 한다.
  - 기존의 AWS Organization에서 멤버 계정을 제거한다.
  - 새로운 AWS Organization에서 멤버 계정으로 초대를 보낸다.
  - 구성원 계정에서 새로운 조직의 초대를 수락한다.

![32-moving-accounts.png](images%2F32-moving-accounts.png)

---

### Service Control Policy (SCP)

- 허용 목록(allowlist) 또는 차단 목록(blocklist) IAM 작업을 정의할 수 있도록 도와준다.
- OU 또는 계정 수준에서 적용되며 관리 계정에는 적용되지 않는다.
- 루트 사용자를 포함한 계정 내의 모든 사용자와 역할에 직접 적용된다.
- 서비스 연결 역할(Service-linked role)에는 영향을 미치지 않는다.
  - 서비스 연결 역할(Service-linked role)을 통해 다른 AWS 서비스를 AWS 조직과 통합할 수 있으며 SCP의 제한을 받을 수 없다.
- SCP는 기본적으로 아무것도 허용하지 않기 때문에, 명시적 허용이 있어야 한다.
- 대표적인 사용 사례는 아래와 같다.
  - 특정 서비스에 대한 액세스 제한(예: EMR 사용 제한)
  - 서비스를 명시적으로 비활성화하여 PCI 컴플라이언스 적용

#### SCP Hierarchy

![33-scp-hierarchy.png](images%2F33-scp-hierarchy.png)

- **Management Account**
  - 모든 작업을 할 수 있다.
  - 관리 계정이므로 SCP가 적용되지 않는다.
- **Account A**
  - 모든 작업을 할 수 있다.
  - Prod OU에 의해 제한되었기 때문에 Redshift에 대한 접근이 제한된다.
- **Account B**
  - 모든 작업을 할 수 있다.
  - Prod OU에 의해 제한되었기 때문에 Redshift에 대한 접근이 제한된다.
  - HR OU에 의해 제한되었기 때문에 Lambda에 대한 접근이 제한된다.
- **Account C**
  - 모든 작업을 할 수 있다.
  - Prod OU에 의해 제한되었기 때문에 Redshift에 대한 접근이 제한된다.

#### Blocklist & Allowlist Strategy

![34-scp-blocklist.png](images%2F34-scp-blocklist.png)

- 명시적으로 모든 것을 허용하지만 명시적으로 DynamoDB를 차단하고 있다.

![35-scp-allowlist.png](images%2F35-scp-allowlist.png)

- EC2와 CloudWatch에 대해서만 접근을 허용한다.
- SCP 전략에 대한 자세한 내용은 [여기](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_examples.html)에서 확인할 수 있다.

#### IAM Policy Evaluation Logic

![36-iam-policy-evaluation-logic.png](images%2F36-iam-policy-evaluation-logic.png)

- IAM 정책을 평가할 때, 여러 단계마다 평가할 수 있다.
- 모든 정책을 평가하고 마지막으로 특정 IAM 작업을 허용하거나 거부한다.
- IAM 정책 평가 로직에 대한 자세한 내용은 [여기](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)에서 확인할 수 있다.

#### SCP 사용 예시

- **예시 1**
- IAM SCP 정책을 사용하여 태그를 제한하는데 사용될 수 있다.

![37-restricting-tags-with-iam-policy.png](images%2F37-restricting-tags-with-iam-policy.png)

- `aws:TagKeys` Condition 키를 사용하면 된다.
  - IAM 정책의 태그 키를 기준으로 리소스에 연결된 태그 키의 유효성을 확인한다.
- 예시를 보면 "Env" 및 "CostCenter" 태그가 있는 경우에만 IAM 사용자가 EBS 볼륨을 만들 수 있도록 허용하고 있다.
- `ForAllValues`는 모든 키를 포함하고 있는 경우에만 허용되며, `ForAnyValue`는 여러 키 중에 하나만 일치하더라도 허용된다.

- **예시 2** 
- `aws:RequestRegion` Condition 키를 사용하여 전체 리전을 거부하는 SCP를 생성할 수 있다.

![38-deny-region-aws-requestregion.png](images%2F38-deny-region-aws-requestregion.png)

- 모든 것이 거부되는 리전은 `eu-central-1`, `eu-west-1`이 될 수 있고, `ec2:*`, `rds:*`, `dynamodb:*`와 같이 다양한 AWS 서비스를 지정할 수 있다.
- 특정 지역이나 특정 지역으로부터 멤버 계정을 제한하는 데 사용될 수 있다.

- **예시 3**
- 적절한 태그가 없는 리소스 생성을 제한하기 위해 SCP를 사용할 수 있다.

![39-restrict-creating-resources-without-appropriate-tags.png](images%2F39-restrict-creating-resources-without-appropriate-tags.png)

- 영향을 받는 구성원 계정의 IAM 사용자/역할이 특정 태그를 가지고 있지 않은 경우 리소스를 만들지 못하도록 제한할 수 있다.
- 예시에서는 EC2 인스턴스에 "Project" 및 "CostCenter" 태그가 없는 경우 리소스 시작을 제한하고 있다.

#### Tag Policy

![40-organization-tag-policy.png](images%2F40-organization-tag-policy.png)

- AWS Organization의 리소스 전반에 걸쳐 태그를 표준화할 수 있도록 지원한다.
- 일관된 태그 확인, 태그가 지정된 리소스 감사, 적절한 리소스 분류 유지 등을 지원한다.
- 태그 키 및 허용된 값을 정의한다.
- AWS 비용 할당 태그 및 속성 기반 액세스 제어를 지원한다.
- 지정된 서비스 및 리소스에 대해 규정을 준수하지 않는 태깅 작업을 방지한다.
- 모든 태그나 비준수 리소스를 모두 나열하는 보고서를 생성할 수 있다.
- Amazon EventBridge를 사용하여 비준수 태그를 모니터링할 수 있다.

#### AI Service Opt-out Policy

- 특정 AWS AI 서비스는 아마존 AI/ML 서비스의 지속적인 개선을 위해 사용자의 콘텐츠를 사용할 수 있다.
  - 이러한 AI 서비스는 Amazon Lex, Amazon Comprehend, Amazon Polly 등이 있다.
- AWS AI 서비스에서 콘텐츠를 저장하거나 사용하지 않도록 선택할 수 있다.
- 모든 구성원 계정 및 AWS 모든 서비스에 걸쳐 이 설정을 적용하는 Opt-out 정책을 성생한다.
- 모든 AI 서비스 또는 선택한 서비스를 선택 취소할 수 있다.
- Organization Root, 특정 OU, 또는 개별 구성원 계정에 연결할 수 있다.

![41-ai-service-optout-policy.png](images%2F41-ai-service-optout-policy.png)

#### Backup Policy

- AWS Backup을 사용하면 AWS 리소스를 백업하는 방법을 정의하는 백업 계획을 생성할 수 있다.
- AWS Organization 전체에 걸쳐 백업 계획을 정의하는 JSON 문서를 정의한다.
- 리소스 백업을 세부적으로 제어할 수 있다. 
  - 예를 들어, 백업 주기, 시간대, 백업 영역 등이 있다.
- Organization Root, 특정 OU 또는 개별 구성원 계정에 연결할 수 있다.
- 변경되지 않는 백업 계획이 구성원 계정에 나타난다. 구성원 계정은 계획을 수정할 수 없다. (ReadOnly)

![42-backup-policy.png](images%2F42-backup-policy.png)

---

### IAM Identity Center

- AWS Single Sign-On 서비스의 후속 서비스다.
- 한 번의 로그인(Single Sign-On)으로 모든 사용자를 지원한다.
  - AWS Organization의 AWS 계쩡
  - 비즈니스 클라우드 애플리케이션 (예: Salesforce, Box, Microsoft 365 등..)
  - SAML 2.0 지원 애플리케이션
  - EC2 Windows 인스턴스
- Identity Provider
  - IAM Identity Center에 내장된 ID 저장소
  - 제3자: Active Directory(AD), OneLogin, Okta 등..

#### Login Flow

![43-login-flow.png](images%2F43-login-flow.png)

- 로그인 페이지로 이동하여 사용자 이름과 암호를 입력한 다음 AWS IAM 자격 증명 센터로 바로 이동한다.
- AWS 계정에 대해 활성화되어 있다면 원하는 계정을 클릭하고 관리 콘솔에서 직접 연결할 수 있다.
- 이동된 관리 콘솔에는 이미 로그인이 되어 있으며 특정 콘솔에 로그인하는 방법을 알 필요가 없다.
- IAM ID 센터 포털에 로그인할 수 있고 SSO가 있다.
  - 따라서 Single Sign-On이 있으면 계정, 비즈니스 응용프로그램 등에 대한 SSO 액세스 권한이 있으면 암호를 입력할 필요가 없다.
- **여러 AWS 계정이 있는 경우 IAM Identity Center 서비스를 이용하는 것이 강력하게 권장**된다.

#### Work Flow

![44-iam-identity-center.png](images%2F44-iam-identity-center.png)

- 브라우저 인터페이스는 AWS IAM Identity Center의 로그인 페이지에 연결된다.
- 다른 사용자 저장소와 통합된다.
  - 다른 사용자 저장소는 Active Directory일 수도 있고 클라우드 또는 On-Premise일 수도 있다.
- 위의 예시에서는 Active Directory를 사용하여 사용자와 그룹을 관리하고 있으며, IAM Identity Center(Built-in)를 사용하고 있다.
  - IAM처럼 사용자와 그룹을 정의할 수 있다.
  - 이후 Organization, Windows EC2 인스턴스, 비즈니스 클라우드 애플리케이션, 사용자 지정 SAML 2.0 지원 SSO와 Identity Center를 통합한다.
- 모든 작업에 대해 하나의 로그인이 가능하고 흐름을 많이 단순화한다는 장점이 있다.
- 로그인을 해도 모든 항목에 액세스할 수는 없으며 Identity Center에서 권한 집합이라는 것을 정의하여 어떤 사용자가 무엇에 액세스할 수 있는지 정의해야 한다.

![45-iam-identity-center.png](images%2F45-iam-identity-center.png)

- Organization이 있고 관리 계정에서 IAM Identity Center를 설정한다.
- 각각 자체 계정이 있는 Development 및 Production OU가 있다.
- 회사의 개발자 그룹에는 Bob과 Alice 계정이 있다.
- Bob과 Alice가 모두 개발을 위한 OU의 전체 액세스 권한을 갖고 있는지 확인하려고 한다.
- 사용 권한 집합(Permission Set)을 생성하고 관리자 접근을 허용하고, 이 권한 집합을 특정 OU와 연결해야 한다.
- 따라서 구매를 Development OU와 연결한 다음 이러한 권한 집합을 개발자 그룹에 할당하여 Bob과 Alice가 해당 계정에 대한 전체 액세스 권한을 부여하는 모든 개발 계정에서 역할을 맡을 수 있도록 한다.
- 유사한 방식으로 `ReadOnlyAccess`라는 권한 집합(Permission Set)을 생성할 수 있다.
  - 이것과 연계시키고 이런 권한을 개발자 그룹에 할당한다.
- 사용자를 그룹, 권한 집합(Permission Set), IAM Identity Center의 특정 계정 할당에 연결하는 방법이다.

#### Fine-grained Permission and Assignment

- **Multi-Account Permissions**
  - AWS Organization 내 AWS 계정 간 액세스를 관리한다.
  - Permission Sets: AWS 액세스를 정의하기 위해 사용자 및 그룹에 할당된 하나 이상의 IAM 정책을 정의한다.
  - 예시에서 IAM Identity Center가 있으며 조직과 통합되어 데이터베이스 관리자에 대한 권한 집합을 정의한다.
  - 예를 들어, 개발 계정에서 RDS와 Aurora에 액세스할 수 있고 Prod 계정에서 RDS와 Aurora에 액세스할 수 있다.
  - 사용자에 대응되는 IAM 역할을 자동으로 생성한다.
  - 따라서, 데이터베이스 관리자가 있을 때 그룹이 될 수 있다.
    - 사용자가 이 그룹에 있다면 데이터베이스 관리자의 권한 모음을 할당해야 한다.
    - 사용자가 IAM Identity Center로 로그인하여 개발 계정이나 Prod 계정의 콘솔에 접근하면 특정 계정에서 자동으로 IAM 역할을 수행하게 된다.

![46-fine-grained-permissions-and-assignments.png](images%2F46-fine-grained-permissions-and-assignments.png)

- **Application Assignments**
  - 사용자나 그룹이 어떤 응용 프로그램에 액세스할 수 있는지 정의할 수 있다.
  - 많은 SAML 2.0 비즈니스 애플리케이션에 대한 SSO 액세스를 제공한다. (Salesforce, Box, Microsoft 365 등..)
  - 필요한 URL, 인증서 및 메타데이터를 제공한다.
- **Attribute-Based Access Control (ABAC)**
  - IAM Identity Center Identity Store에 저장된 사용자의 속성을 기반으로 세부적인 권한을 제공한다.
  - 예를 들어, 사용자를 코스트 센터에 할당할 수 있고 제목을 정하고, 로케일(Locale)을 정할 수 있다.
  - 사용 권한 설정을 한 번 정의하면 이런 특성을 활용할 수 있고 그룹 내 사용자의 AWS 액세스만 수정할 수 있다. 단, 기본적인 특성만 변경할 수 있다.

---

### AWS Control Tower

- 모범 사례를 기반으로 안전하고 규정을 준수하는 다중 계정 AWS 환경을 쉽게 설정하고 관리할 수 있는 방법이다.
- 아래와 같은 이점이 있다.
  - 클릭 몇 번으로 환경 설정을 자동화한다.
  - 가드레일을 사용하여 지속적인 정책 관리를 자동화한다.
  - 정책 위반 탐지 및 시정한다.
  - 대화형 대시보드를 통해 컴플라이언스 모니터링을 제공한다.
- AWS Control Tower는 AWS Organization 위에서 실행된다.
  - 자동으로 AWS Organization을 설정하여 계정을 구성하고 SCP(Service Control Policy)를 구현한다.

#### Account Factory

- 계정 프로비저닝 및 배포를 자동화한다.
- 조직의 AWS 계정에 대해 미리 승인된 기준선 및 구성 옵션(예: VPC 기본 구성, 서브넷, 리전 등..)을 생성할 수 있다.
- AWS Service Catalog 서비스를 사용하여 새로운 AWS 계정을 프로비저닝한다.

![47-account-factory.png](images%2F47-account-factory.png)

- 클라우드 컴퓨팅 환경이 있고 ADFS와 Active Directory가 있는 데이터 센터가 있다.
- 클라우드와 데이터 센터는 VPN 또는 Direct Connect 연결이 설정되어 있다.
- Control Tower와 Landing Zone을 사용하여 계정 팩토리를 만들 때 중심에는 IAM Identity Center가 있다.
- Microsoft Active Directory와 기업의 데이터 센터와 통합하고 싶다면 AWS Managed Microsoft AD를 생성하고 2-way Trust를 생성하면 된다.
- Landing Zone과 계정 팩토리를 통해 생성된 모든 계정은 IAM Identity Center를 통해 인증을 활용하도록 적절히 구성된다.

#### Guardrail

- Guardrail은 정책 위반 탐지 및 시정하며 Control Tower 환경에 대한 지속적인 거버넌스를 제공한다. (AWS Account)
- 두 종류의 Guardrail이 있다.
  - Preventive(예방): SCP를 사용한다. (예: 루트 사용자의 액세스 키 생성 거부)
  - Detective (검출): AWS Config를 사용한다. (예: 루트 사용자에 대한 MFA 활성화 여부 감지)
  - 예를 들어, 태그가 지정되지 않아 규정을 준수하지 않는 리소스가 있을 때 식별이 가능하다. 

![48-guardrail.png](images%2F48-guardrail.png)

- 태그가 없는 리소스를 검출하고 싶을 때, Guardrail은 AWS Config를 사용한다.
- Detective Guardrail이 Organization의 일부로서 멤버 계정에서 태그가 추가되지 않은 리소스를 모니터링한다.
- 규정 비준수 리소스가 검출되면 SNS를 통해 관리자에게 알림을 전송하고 Lambda Function을 실행하여 규정을 준수하도록 태그를 추가할 수 있다.

#### Guardrail Level

- **Mandatory(필수)**
  - AWS Control Tower에서 자동으로 활성화 및 적용된다.
  - 예를 들어, 로그 보관 계정에 대한 공용 액세스를 허용하지 않을 수 있다.
- **Strong Recommended(강력 추천)**
  - AWS 모범 사례 기반 (옵션)
  - 예를 들어, EC2 인스턴스에 연결된 EBS 볼륨에 대해 암호화를 사용할 수 있다.
- **Elective (선택 사항)**
  - 기업에서 일반적으로 사용 (옵션)
  - 예를 들어, S3 버킷에서 MFA가 없는 삭제 작업을 허용하지 않을 수 있다.

---

### AWS Resource Access Manager (RAM)

- 소유한 AWS 리소스를 다른 AWS 계정과 공유할 수 있다.
- 모든 계정 또는 조직(Organization) 내에서 공유할 수 있다.
- 공유를 통해서 리소스 중복을 피할 수 있다.
- VPC Subnet을 예시로 살펴본다.
  - 모든 리소스가 동일한 서브넷에서 시작되도록 허용할 수 있다.
  - 동일한 AWS Organization에 있어야 한다.
  - 보안 그룹 및 기본으로 생성되는 VPC는 공유할 수 없다.
  - 참가자는 자신의 리소스를 관리할 수 있다.
  - 참가자는 다른 참가자 또는 소유자의 리소스를 보거나 수정하거나 삭제할 수 없으므로 네트워킹만 공유한다.
- AWS Transit Gateway, Route 53 Resolver 규칙, DNS Firewall Rule Group, License Manager 구성을 공유할 수 있다.
- 이외에도 공유할 수 있는 많은 서비스들이 있다.
  - Aurora DB Cluster
  - ACM Private Certificate Authority
  - CodeBuild Project
  - EC2 (Dedicated Hosts, Capacity Reservation)
  - AWS Glue (Catalog, Database, Table)
  - AWS Network Firewall Policy
  - AWS Resource Group
  - Systems Manager Incident Manager (Contract, Response Plan)
  - AWS Outpost (Outpost, Site)

#### VPC 예시

![49-ram-vpc-example.png](images%2F49-ram-vpc-example.png)

- VPC와 VPC 소유자가 있고 Private Subnet은 "Account 1"과 "Account 2"가 공유하고 있다.
- 각 계정
  - 자신의 리소스에 대한 책임을 가진다.
  - 다른 계정의 다른 리소스를 보거나 수정하거나 삭제할 수 없다.
- 네트워크를 공유한다.
  - 예를 들어, "Account 1"이 EC2 인스턴스와 ALB를 배포하였을 때, "Account 2"는 통신은 할 수 있지만 서로의 리소스를 볼 수 없다.
  - 데이터베이스가 VPC 소유자의 개인 세그먼트에 있다.
  - 하지만, 네트워크는 공유되기 때문에 서로 통신할 수 있으며 Private IP를 사용하여 액세스할 수 있다. **VPC Peering이 필요하지 않다.**
  - 최고의 보안을 위해 다른 계정의 보안 그룹을 참조할 수 있다.
- 대표적인 사용 사례는 아래와 같다.
  - 동일한 신뢰(Trust) 범위 내의 애플리케이션을 동일한 VPC내에 배포하여 네트워크를 단순화할 수 있다..
  - VPC가 하나뿐이므로 간단한 네트워크 설정으로 높은 수준의 상호 연결성을 갖춘 애플리케이션을 구축할 수 있다.

#### Managed Prefix List

- 하나 이상의 CIDR 블록 집합이다.
- Security Group 및 Route Table 구성 및 유지 관리가 용이하다.

![50-ram-managed-prefix-list.png](images%2F50-ram-managed-prefix-list.png)

- "계정 A"에는 "Prefix List A"가 있고 CIDR1, CIDR2, CIDR3이 있다.
  - 예를 들어, 이러한 집합들은 회사의 내부 네트워크를 나타내는 규칙 집합일 수 있다.
- 보안 그룹을 생성할 때, 보안 그룹에 모든 CIDR를 추가하는 것이 아니라 "Prefix List A"를 참조하도록 할 수 있다.
- RAM을 사용하여 다른 계정과 공유하는 경우 보안 그룹의 Inbound 규칙은 간단하게 작성될 수 있다.
- **Customer-Managed Prefix List**
  - 사용자가 정의하고 관리하는 CIDR 집합이다.
  - 다른 AWS 계정 또는 AWS Organization과 공유할 수 있다.
  - 여러 보안 그룹을 한 번에 수정하도록 수정할 수 있다.
  - 즉, Prefix List를 중앙집중적으로 관리할 수 있다.
- **AWS-Managed Prefix List**
  - AWS 서비스에 대한 CIDR 집합이다.
  - 사용자는 생성, 수정, 공유 또는 삭제할 수 없다.

#### Route 53 Outbound Resolver

- 여러 계정과 VPC가 있는 경우 DNS로 전달 규칙을 확장할 수 있도록 지원한다.

![51-ram-route53-outbound-resolver.png](images%2F51-ram-route53-outbound-resolver.png)

- 메인 계정에서 Route 53에 대한 Resolver 규칙을 정의한다.
- 규칙에는 사용자가 원하는 도메인 이름과 대상 IP를 추가할 수 있다.
- 완성된 규칙을 다른 계정과 공유할 수 있고 공유한 후에는 다른 계정에서 수락해야 한다.
- 다른 계정에서 수락하는 경우 다른 VPC와 연결될 수 있다.
  - 규칙이 있는 다른 VPC는 사용자가 지정한 도메인 이름을 확인할 수 있다.
- DNS를 중앙 관리되는 Resolver 규칙을 갖게 해준다.

---

### Identity & Federation 요약

- AWS 내에서 관리할 수 있는 사용자와 계정이 있지만 AWS Organization을 사용하여 이러한 계정을 생성할 수도 있다.
- 보안 및 규정 준수를 위하여 다중 계정 AWS 환경 설정을 위한 AWS Control Tower 환경을 구축할 수 있다. (모범 사례)
- SAML을 통해 계정을 Federation할 수 있다.
- SAML을 사용하지 않고 커스텀 IdP(`GetFederationToken`)를 사용하여 Federation할 수 있다.
- 여러 AWS 계정(Organization) 및 SAML 앱에 연결하는 AWS Single Sign-On을 사용할 수 있다.
- Web Identity Federation을 사용할 수 있지만 권장되는 방법이 아니다.
- 대부분의 웹 및 모바일 애플리케이션에 대해 Cognito를 활용하여 "익명 모드" 또는 MFA를 활성화할 수 있다.
- AWS Directory Service
  - **Managed Microsoft AD**: 독립 실행형 또는 온프레미스와의 셋업 트러스트 AD, MFA, Seamless Join, RDS 통합 기능이 있다.
  - **AD Connector**: On-Premise에 대한 프록시 요청 기능이 있다.
  - **Simple AD**: MFA, 고급 기능은 없지만 독립되고 저렴한 AD 호환서비스다.
- AWS 리소스를 공유하기 위한 AWS RAM을 사용할 수 있다.

---

### 참고한 강의

- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)