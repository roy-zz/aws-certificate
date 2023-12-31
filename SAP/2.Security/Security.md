# Security

- 이번 장에서는 **Solutions Architect Professional (SAP)** 을 준비하며 "보안(Security)"에 대해서 알아보도록 한다.

---

### AWS CloudTrail

- AWS 계정에 대한 거버넌스, 컴플라이언스 및 감사(Audit)를 제공한다.
- CloudTrail은 기본적으로 활성화되어 있다.
- 아래의 항목들을 통해서 AWS 계정 내에서 발생한 이벤트 및 API 호출 기록을 확인할 수 있다.
  - Console, SDK, CLI, AWS Service
- CloudTrail의 로그를 CloudWatch Logs 또는 S3에 저장할 수 있다.
- 모든 리전(기본값) 또는 단일 리전에 적용될 수 있다.
- AWS에서 의도치 않게 리소스가 삭제된 경우 CloudTrail을 확인하면 빠르게 원인을 파악할 수 있다.

![1-cloudtrail-diagram.png](images%2F1-cloudtrail-diagram.png)

- CloudTrail이 중앙에 있고, SDK, CLI, Consol, IAM 유저, IAM 역할 혹은 다른 서비스의 동작들이 CloudTrail Console에 표시된다.
- 표시되는 정보를 기반으로 어떤 작업이 있었는지 조사하거나 감사할 수 있다.
- 저장되는 데이터는 최대 90일 저장이 가능하므로 90일 이상의 이벤트를 저장하고 싶다면 CloudWatch Logs 또는 S3 버킷에 저장해야 한다.

#### Event

- **Management Events**
  - AWS 계정의 리소스에 대해 수행되는 작업이다.
  - 예를 들어, 아래와 같은 작업이 있다.
    - 보안 구성 (IAM `AttachRolePolicy`)
    - 라우팅 데이터에 대한 규칙 구성 (EC2 `CreateSubnet`)
    - 로깅 설정 (CloudTrail `CreateTrail`)
  - 기본적으로 CloudTrail은 Management Event를 추적하도록 구성된다.
  - 읽기 이벤트(리소스 수정 X)와 쓰기 이벤트(리소스 수정 O)를 구분할 수 있다.
- **Data Events**
  - 기본적으로 Data Event는 데이터의 양이 많기 때문에 기록되지 않는다.
  - S3 객체 수준의 활동(예: `GetObject`, `DeleteObject`, `PutObject`)을 저장하며, 읽기와 쓰기 이벤트를 분리할 수 있다.
  - AWS Lambda Function 실행 활동(`Invoke` API)을 기록할 수 있다.
- **CloudTrail Insights Event**

#### CloudTrail Insights Event

- CloudTrail Insights를 사용하여 계정의 비정상적인 작업을 감지할 수 있다.
  - 부정확한 리소스 프로비저닝
  - 서비스 한계 도달
  - IAM 작업의 버스트
  - 주기적인 유지보수 활동의 공백
- CloudTrail Insights는 일반적인 관리 이벤트를 분석하여 기준선(baseline)을 생성한다.
- 생성된 기준선으로 쓰기 이벤트를 지속적으로 분석하여 비정상적인 패턴을 탐지한다.
  - CloudTrail 콘솔에 이상 징후가 표시된다.
  - 이벤트가 S3로 전송된다.
  - 자동화 필요에 따라 EventBridge 이벤트가 생성된다.

![2-cloudtrail-insights.png](images%2F2-cloudtrail-insights.png)

#### Event Retention

- 이벤트는 CloudTrail에 90일 동안 저장된다.
- 이 기간 이후의 이벤트를 유지하려면 S3에 저장하고 Athena를 사용하여 검색해야 한다.

![3-cloudtrail-event-retention.png](images%2F3-cloudtrail-event-retention.png)

#### EventBridge와의 통합

![4-eventbridge-intercept-api-call.png](images%2F4-eventbridge-intercept-api-call.png)

- 사용자는 `DeleteTable` API를 호출하여 DynamoDB의 테이블을 삭제할 수 있다.
- 이러한 API 호출은 발생할 때마다 기록이 CloudTrail에 저장된다.
- CloudTrail의 이벤트가 저장될 때, EventBridge에서 동일한 이벤트를 획득할 수 있다.
- EventBridge는 이벤트가 발생하였을 때, SNS를 통해서 경고를 생성하여 사용자에게 전달할 수 있다.

![5-eventbridge-cloudtrail.png](images%2F5-eventbridge-cloudtrail.png)

- 사용자가 역할을 수행할 때마다 알림을 요청하도록 아키텍처를 구성한다.
  - 사용자는 `AssumeRole`이라는 IAM 서비스의 API를 호출하고 기록은 CloudTrail에 저장된다.
  - EventBridge와의 통합을 이용해서 SNS 토픽에 메시지를 트리거할 수 있다.
  - 유사하게 API 호출도 가로챌 수 있다.
- 사용자가 보안 그룹의 인바운드 규칙을 변경할 때마다 알림을 요청하도록 아키텍처를 구성한다.
  - 사용자는 인바운드 규칙을 변경하기 위해서 `AuthorizeSecurityGroupIngress` API를 호출한다.
  - 호출된 기록은 CloudTrail에 저장되고 EventBridge에 나타난다.
  - EventBridge는 SNS에서 알림을 발생시킬 수 있다.

#### 설계 예시 - Delivery to S3

![6-delivery-to-s3.png](images%2F6-delivery-to-s3.png)

- CloudTrail이 있고 5분마다 CloudTrail 파일을 S3로 보낼 수 있다.
- 데이터는 기본값으로 SSE-S3 암호화되며 직접 SSE-KMS를 사용하도록 CloudTrail 파일을 설정할 수 있다.
- 일반적으로 CloudTrail 데이터는 자주 접근하지 않으므로 비용 절감을 위해 Glacier에 저장하고 검색이 필요한 경우에만 저장한다.
- 파일이 S3에 전달되면 SQS 큐, SNS 토픽, Lambda Function에게 알리게 하거나, 직접 SNS로 알림을 전달할 수 있다.

- S3의 향상된 기능에는 아래와 같은 항목들이 있다.
  - 버전 관리 활성화
  - MFA 삭제 보호
  - S3 라이프사이클 정책 (S3 IA, Glacier 등..)
  - S3 객체 잠금
  - SSE-S3 또는 SSE-KMS 암호화
  - CloudTrail 로그 파일 무결성 검증을 수행하는 기능 (해싱 및 서명을 위한 SHA-256)

#### 설계 예시 - Multi Account & Multi Region 로깅

![7-multi-account-multi-region-logging.png](images%2F7-multi-account-multi-region-logging.png)

- "계정 A"와 "계정 B"가 있고 로그를 수집하는 "보안 계정"이 있다.
- CloudTrail은 "계정 A"와 "계정 B"에 있고 로그를 수집하는 "보안 계정"에 S3 버킷이 있다.
- CloudTrail에서 S3 버킷에 데이터를 전달하는 유일한 방법은 S3 버킷 정책(Bucket Policy)를 사용하는 것이다.
- 만약 "계정 A"에서 CloudTrail 파일에 접근하고 싶다면 아래의 옵션 중에 선택할 수 있다.
  - 옵션 1: 교차 계정 역할을 생성하고 역할을 맡는다.
  - 옵션 2: 버킷 정책을 편집한다.

#### 설계 예시 - API 호출을 위한 알람

![8-alert-api-call.png](images%2F8-alert-api-call.png)

- 특정 API 호출이 완료되면 경고를 생성한다.
- CloudWatch Logs에서는 이벤트를 스트리밍할 수 있으며 Metric Filter를 생성하고 CloudWatch 알람을 생성할 수 있다.
- Metric Filter에서 기본적으로 사용자가 원하는 API 호출을 필터링한다.
- 예를 들어, 아래와 같은 항목들에 필터를 생성할 수 있다.
  - EC2 `TerminateInstance` API 발생 횟수
  - 사용자당 API 호출 횟수
  - 높은 수준의 거부된 API 호출 탐지

#### 설계 예시 - Organizational Trail

![9-organizational-trail.png](images%2F9-organizational-trail.png)

- AWS Organization이 있고 관리 계정, 멤버 계정과 함께 여러 개의 OU가 있다면 관리 계정에서 직접 Organizational Trail을 설정할 수 있다.
- **멤버 계정이 아닌 관리 계정에서 CloudTrail을 생성한다는 것을 기억**해야 한다.
- 이러한 설정으로 모든 계정에서의 모든 이벤트는 관리 계정에서 확인할 수 있다.

#### 즉각적인 이벤트 호출

- 전체적으로 CloudTrail은 이벤트를 제공하는 데 최대 15분이 소요될 수 있다.
- **EventBridge**
  - CloudTrail에 저장된 모든 API 호출에 트리거가 될 수 있다.
  - CloudTrail의 이벤트에 반응하는 데 가장 빠르고 즉각적인 방법을 제공한다.
- **CloudTrail Delivery in CloudWatch Logs**
  - CloudTrail의 모든 이벤트가 CloudWatch Logs에 스트리밍될 수 있다.
  - 메트릭 필터를 만들어 발생 빈도를 분석하고 이상 현상을 감지할 수 있다.
- **CloudTrail Delivery in S3**
  - 이벤트는 5분마다 전달된다.
  - 로그 무결성 분석, 교차 계정 제공, 장기 저장 스토리지를 제공한다.

---

### AWS KMS (Key Management Service)

- AWS 서비스에 대한 "암호화"를 들으면 AWS KMS를 연상시킬 수 있어야한다.
- 데이터에 대한 액세스를 쉽게 제어할 수 있는 AWS가 키를 관리한다.
- 인증을 위해 IAM과 완벽하게 통합된다.
- 아래와 같은 AWS 서비스들과 원활하게 통합된다.
  - Amazon EBS: 볼륨 암호화
  - Amazon S3: 서버측 객체 암호화
  - Amazon Redshift: 데이터 암호화
  - Amazon RDS: 데이터 암호화
  - Amazon SSM: 매개변수 저장소
  - 등등..
- CLI 및 SDK를 사용할 수도 있다.

#### Key 유형

- **대칭(Symmetric, AES-256)**
  - 암호화 및 복호화에 사용되는 단일 암호화 키인 KMS의 제품이다.
  - KMS와 통합된 AWS 서비스는 대칭형 KMS 키를 사용한다.
  - 엔벨로프(envelope) 암호화에 필요하다.
  - 암호화되지 않은 KMS 키에 액세스할 수 없다. (사용하려면 KMS API를 호출)
- **비대칭(Asymmetric, RSA & ECC key pairs)**
  - 공개 키(암호화)와 개인 키(복호화) 쌍이 있다.
  - 암호화/복호화 또는 서명(Sign)/확인(Verify) 작업에 사용된다.
  - 공개 키는 다운로드할 수 있지만 암호화되지 않은 개인 키에는 액세스할 수 없다.
  - 활용 사례: KMS API 호출이 불가능한 사용자가 AWS 외부에서 암호화

- **Customer Managed Key**
  - 생성, 관리 및 사용, 활성화 또는 비활성화가 가능하다.
  - 순환 정책(매년 새로운 키 생성, 이전 키 보존)을 설정할 수 있다.
  - CloudTrail에서 키 정책(리소스 정책) 및 감사 기능을 추가할 수 있다.
  - 엔벨로프(envelope) 암호화를 활용할 수 있다.
- **AWS Managed Key**
  - AWS 서비스에서 사용한다. (S3, EBS, Redshift 등..)
  - AWS에서 관리하고 1년마다 자동으로 갱신된다.
  - CloudTrail에서 주요 정책 및 감사 기능을 제공한다.
- **AWS Owned Key**
  - AWS에서 생성 및 관리, 일부 AWS 서비스에서 리소스를 보호하는 데 사용된다.
  - 여러 AWS 계정에서 사용되지만 AWS가 내부적으로 사용하기 때문에 사용자는 키를 보거나 사용하거나 추적하거나 감사할 수 없다.

![10-type-of-kms-keys.png](images%2F10-type-of-kms-keys.png)

#### Material Origin

- KMS 키의 핵심 재료의 출처를 식별한다.
- 키를 생성한 후에는 변경할 수 없다.

- **KMS(AWS_KMS) - 기본값**
  - AWS KMS는 자체 키스토어에서 키 재료를 생성하고 관리한다.
- **External**
  - 키 재료를 KMS 키로 가져온다.
  - AWS 외부에서 이 핵심 재료의 보안 및 관리 책임이 있다.
- **Custom Key Store (AWS_CLOUDHSM)**
  - AWS KMS는 사용자 정의 키 저장소(CloudHSM Cluster)에 핵심 재료를 생성한다.

#### Custom Key Store (CloudHSM)

- KMS와 CloudHSM 클러스터를 Custom Key Store로 통합한다.
- 주요 재료는 사용자가 소유하고 관리하는 CloudHSM 클러스터에 저장된다.
- 암호화 작업은 CloudHSM에서 수행된다.
- 아래와 같은 사용 사례에 사용된다.
  - 높은 보안을 위해 HSM에 대한 직접적인 제어가 필요한 경우
  - KMS키를 전용 HSM에 저장해야 하는 경우

![11-custom-key-store-cloudhsm.png](images%2F11-custom-key-store-cloudhsm.png)

- AZ가 2개인 CloudHSM 클러스터는 KMS에 직접 연결되어 있다.
- 고가용성을 위해 최소한 2개의 HSM을 실행시켜야 하며, KMS와 통합할 수 있다.
- 사용자는 여전히 KMS API를 이용해 키를 확인하고 생성하고 관리할 수 있다.
- KMS 내부적으로는 암호화 연산, 저장, 검색을 위해 CloudHSM 클러스터를 사용한다.

#### Key Source - External

- 자신의 키 재료를 KMS키로 가져오거나 자신의 키를 가져올 수 있다. (BYOK, Bring Your Own Key)
- AWS 이외의 사용자는 주요 재료의 보안, 가용성 및 내구성에 대한 책임이 있다.
- 256-bit 대칭키만 지원하며, 비대칭키는 지원하지 않는다.
- 사용자 지정 키 저장소(CloudHSM)와 함께 사용할 수 없다.
- KMS 키를 수동으로 갱신해야 하며, 자동 갱신은 지원되지 않는다.

![12-key-source-external.png](images%2F12-key-source-external.png)

- 사용자는 AWS 외부에 있는 키 재료를 통해서 KMS에서 KMS 키를 생성하고 키를 대칭(Symmetric)으로 지정한다.
- 공개 키와 임포트 토큰을 다운로드하고 공용키를 사용한다.
- 외부에서 생성된 키 재료로 공개 키를 사용하여 암호화를 생성한다.
- 공개 키, 암호화된 개인 키도 임포트 토큰을 이용해서 KMS 서비스로 보낼 수 있다.
- KMS는 복호화 메커니즘을 이용해 핵심 재료를 KMS 키에 저장한다.

#### Multi Region Key

![13-multi-region-key.png](images%2F13-multi-region-key.png)

- 예를 들어, 키를 `us-east-1`에서 생성하고 다양한 리전에 걸쳐 복제할 수 있다.
- 같은 키 재료와 같은 키 ID를 갖게 되지만 다른 리전에 위치한다.
- 같은 키를 가지고 있기 때문에 하나의 리전에서 암호화하고 다른 리전에서 복호화할 수 있다.
- 하지만 KMS 키는 글로벌 서비스가 아니며 단순히 복제본을 다른 리전에 복제하는 것이다.
- 각각의 리전에서 키는 독립적으로 관리된다.
- 한 번에 하나의 기본 키만 복제본을 자신의 기본 키로 승격할 수 있다.
- 대표적인 사용 사례는 아래와 같다.
  - 재해 복구
  - 글로벌 데이터 관리 (예: DynamoDB Global Table)
  - 다중 리전의 Active-Active 애플리케이션
  - 분산 서명 애플리케이션

---

### SSM Parameter Store

- 구성 및 비밀을 위한 보안 저장소다.
- 선택적으로 KMS를 통해서 구성들을 암호화할 수 있다.
- 서버리스에 확장성과 내구성이 높으며 SDK를 통해서 간편하게 사용할 수 있다.
- 매개변수를 업데이트하면 해당 매개변수의 버전을 추적할 수 있다.
- 보안은 IAM을 통해서 제공된다.
- Amazon EventBridge를 통해서 알림을 받을 수 있다.
- CloudFormation과 완벽하게 통합될 수 있다.
  - Parameter Store의 매개변수를 스택의 입력 매개변수로 활용할 수 있다.

![14-ssm-parameter-store.png](images%2F14-ssm-parameter-store.png)

- 애플리케이션과 SSM Parameter Store가 있다.
- 일반 텍스트 구성을 저장하고 애플리케이션의 IAM 사용 권한을 확인한다.
- 예를 들어, EC2 인스턴스 역할이나 암호화된 구성을 가질 수 있다.
- 암호화가 필요한 경우 AWS KMS를 통해 암호화되고 복호화 또한 KMS 서비스를 통해서 이루어진다.
  - 암호화를 위해서는 애플리케이션이 KMS 키에 액세스할 수 있어야 한다.

#### Hierarchy

![15-ssm-parameter-store-hierarchy.png](images%2F15-ssm-parameter-store-hierarchy.png)

- 계층 구조와 함께 매개변수를 Parameter Store에 저장할 수 있다.
- `/aws/reference/secretsmanager/secret_ID_in_Secrets_Manager` 경로를 통해서 Secret Manager 서비스에 접근할 수 있다.
- `/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2` 경로를 통해서 AWS가 발행하는 Public 파라미터를 사용할 수 있다.

#### Standard vs Advanced

- Standard와 Advanced 티어의 Parameter Store가 있다.
- 가장 큰 차이는 매개변수의 최대 크기이며, Advanced 티어는 무료로 사용할 수 없다.

![16-ssm-parameter-store-standard-advanced.png](images%2F16-ssm-parameter-store-standard-advanced.png)

#### Parameter Policy

- Advanced 티어에만 제공되는 기능이다.
- 비밀번호와 같은 중요한 데이터를 강제로 업데이트하거나 삭제하기 위해 매개변수(만료일)에 TTL을 할당할 수 있다.
- 한 번에 여러 정책을 할당할 수 있다.

- **만료되면 매개변수를 제거**

```yaml
{
  "Type":"Expiration",
  "Version":"1.0",
  "Attributes":{
     "Timestamp":"2023-12-31T23:59:59.000Z"
  }
} 
```

- **만료되기 15일 전에 EventBridge를 통해 알림을 수신**

```yaml
{
  "Type":"ExpirationNotification",
  "Version":"1.0",
  "Attributes":{
     "Before":"15",
     "Unit":"Days"
  }
}
```

- **20일 동안 수정되지 않은 경우 알림을 수신**

```yaml
{
  "Type":"NoChangeNotification",
  "Version":"1.0",
  "Attributes":{
     "After":"20",
     "Unit":"Days"
  }
}
```

---

### AWS Secrets Manager

- 비밀(예: 비밀번호, API 키)를 저장하는 용도로 사용된다.
- X일마다 비밀을 강제로 갱신하는 기능을 제공한다.
  - Lambda Function을 사용하여 갱신 시 비밀 생성을 자동화한다.
  - 기본적으로 Amazon RDS(지원되는 모든 DB 엔진), Redshift, DocumentDB를 지원한다.
  - 다른 데이터베이스 및 서비스를 지원한다. (Custom Lambda Function)
- 리소스 기반(Resource-based) 정책을 사용하여 비밀에 대한 접근 제어를 활성화할 수 있다.
- 다른 AWS 서비스와의 통합을 통해 Secrets Manager로부터 기본적으로 비밀을 추출할 수 있다.
  - CloudFormation, CodeBuild, ECS, EMR, Fargate, EKS, Parameter Store 등..

![17-aws-secrets-manager.png](images%2F17-aws-secrets-manager.png)

- ECS Task는 RDS 데이터베이스에 접속해야 한다.
- ECS는 Secrets Manager와 통합되어 있다.
- 부팅될 때, ECS Task가 자동으로 비밀을 가져와 작업 내에 환경 변수로 삽입한다.
- 환경 변수로 사용 가능하기 때문에 ECS Task 안에서 실행되는 소프트웨어가 무엇이든 RDS 데이터베이스에 안전하게 액세스할 수 있다.

#### CloudFormation 통합

- CloudFormation 내에서 Secret Manager와 함께 비밀을 생성하는 템플릿이다.

![18-secret-manager-cloudformation.png](images%2F18-secret-manager-cloudformation.png)

- `MyRDSDBInstanceRotationSection`을 통해 비밀을 생성한다.
- `MyRDSDBInstance`를 통해 RDS 인스턴스를 생성할 때, 생성된 비밀을 참조한다.
- `SecretRDSInstanceAttachement`에서는 RDS DB 인스턴스에 비밀을 연결한다.

#### 계정 간 공유

![19-sharing-accross-accounts.png](images%2F19-sharing-accross-accounts.png)

- 비밀을 가지고 있는 "Security Account"가 있고 비밀을 사용해야 하는 "Dev Account"가 있다.
- Secret Manager는 "Security Account" 계정에 있고, KMS 키로 보호되고 암호화되어 있다.
- **Resource Access Manager(RAM)으로는 비밀을 공유할 수 없다**
- KMS 키에 KMS 정책을 첨부하여 사용자가 `kms:Decrypt` API를 호출할 수 있게 한다.
- Secret Manager의 비밀을 기반으로 리소스 기반 정책을 생성하여 사용자가 `secretsmanager:GetSecretValue` API를 호출할 수 있게 한다.
- "Dev Account"에 포함되어 있는 사용자는 누구라도 비밀에 접근하고 복호화할 수 있다.

#### SSM Parameter Store vs Secrets Manager

- **Secrets Manager($$$)**
  - Lambda를 통한 비밀 자동 갱신을 제공한다.
  - RDS, Redshift, DocumentDB에 대해 Lambda Function이 제공된다.
  - KMS 암호화는 필수 사항이다.
  - CloudFormation과 통합될 수 있다.
- **SSM Parameter Store($)**
  - 간단한 API를 제공한다.
  - 비밀을 자동으로 갱신하는 기능을 제공하지 않는다.
    - 자동 갱신을 원하는 경우 EventBridge에 의해 트리거된 Lambda를 사용하여 갱신되도록 설정해야 한다.
  - KMS 암호화는 선택사항이다.
  - CloudFormation과 통합될 수 있다.
  - SSM Parameter Store API를 사용하여 Secrets Manager 비밀을 가져올 수 있다.

![20-ssm-parameter-store-secrets-manager-rotation.png](images%2F20-ssm-parameter-store-secrets-manager-rotation.png)

- Secret Manager의 경우 30일마다 비밀이 갱신되고 Lambda Function을 호출하여 RDS에서 사용하고 있는 비밀을 변경할 수 있다.
- SSM Parameter Store의 경우 30일마다 실행되는 EventBridge 규칙을 생성하고 Lambda Function을 실행시켜 비밀을 변경하고 RDS에서 사용하고 있는 비밀도 변경해 주어야 한다.

---

### RDS - Security

- 기본 EBS 볼륨 및 스냅샷에 대해 저장될 때 KMS 암호화가 가능하다.
- Oracle 및 SQL Server용 TDE(Transparent Data Encryption)을 사용할 수 있는 옵션이 제공된다.
- 모든 DB 엔진에 대해서 SSL 암호화를 통한 전송 중 암호화를 제공한다.
- MySQL, PostgreSQL, MariaDB는 IAM을 통한 인증을 제공한다.
- RDS 내의 인가(Authorization)은 여전이 제공되며 IAM을 통해서 이루어지지 않는다.
- 암호화되지 않은 RDS 스냅샷을 암호화된 스냅샷으로 복사할 수 있다.
- CloudTrail을 사용하여 RDS 내에서 이루어진 쿼리를 추적할 수 있다.

---

### SSL/TLS

- SSL은 "Secure Sockets Layer"의 약자로 연결을 암호화하는 데 사용된다.
- TLS는 "Transport Layer Security"의 약자로 최신 버전이다.
- 요즘은 주로 TLS 인증서를 사용하지만 여전히 사람들은 SSL이라고 부른다.
- 공인 SSL 인증서는 CA(Certificate Authority)에서 발급된다.
  - CA는 대표적으로 Comodo, Symantec, GoDaddy, Global Sign, Digicert, Letsencrypt가 있다.
- SSL 인증서는 만료 날짜가 있으므로 주기적으로 갱신해야 한다.

#### 작동 방식

- 클라이언트와 서버가 있고 둘은 보안 채널을 통해 서로 통신해야 한다.
- SSL 암호화를 사용해야 하지만 SSL의 기본인 비대칭(Asymmetric) 암호화는 비용이 매우 비싸다.
    - 여기서 비용이 비싸다는 것은 CPU 리소스를 많이 사용한다는 것을 의미한다.
- 가장 저렴하게 사용할 수 있는 방법은 대칭(Symmetric) 암호화다.
- 처음으로 클라이언트는 서버와 비대칭 Handshake를 진행하고, 진행하는 동안 교환 키를 사용한다.
    - 이러한 통신을 위해 일반적으로 공유되는 대칭 키다.
    - 이 시점부터는 대칭 키를 사용하여 서로 통신할 때 사용된다.
- SSL의 비대칭 암호화가 Handshake할 때만 사용되고 전부 대칭을 사용한다.

![21-ssl-encryption-how-it-works.png](images%2F21-ssl-encryption-how-it-works.png)

- 클라이언트는 cipher suit와 랜덤한 값을 서버로 전송한다.
- 랜덤한 서버는 SSL 인증서 공개키와 함께 응답한다.
- 클라이언트는 SSL 인증서가 정상적으로 저장되었는지 확인한다.
- 인증서가 정상이라면 전송된 마스터 대칭 키를 전송하고 해당 공개 키로 암호화된다.
  - 비밀이 서버에 의해 검색되고 암호화될 수 있다.
  - 클라이언트도 SSL 인증서가 있을 경우 이 시점에 전송한다.
  - 양방향 인증서를 가질 수 있고 선택적으로 서버는 클라이언트의 SSL 인증서가 유효한지 확인한다.
- 서버의 마스터 키는 비공개(개인) 키를 통해서 복호화된다.
- 이후 클라이언트와 서버는 대칭 통신을 할 수 있게 된다.

#### Server Name Indication

- SNI는 하나의 웹 서버에 여러 SSL 인증서를 로드하는 문제를 해결한다. (여러 웹 사이트를 해결하기 위해)
- 이 프로토콜은 "새로운" 프로토콜이며 클라이언트가 초기 SSL Handshake에서 대상 서버의 호스트 이름을 표시해야 한다.
- 이후 서버가 올바른 인증서를 찾거나 기본 인증서를 반환한다.
- ALB, NLB, CloudFront에서만 사용 가능하며, CLB에서는 작동하지 않는다.
  - 만약 CLB를 사용하는데 여러 호스트 이름을 제공해야 한다면 필요한 호스트 이름의 수에 맞는 CLB를 생성해야 한다. 

![22-server-name-indication.png](images%2F22-server-name-indication.png)

- 하나의 ALB가 있고 서로 다른 호스트 이름에 연결된 두 개의 대상 그룹에 연결되어 있다.
- 하나는 `www.mycorp.com`이고 다른 하나는 `Domain1.example.com`이다.
- ALB는 각 대상 그룹의 호스트 이름에 맞는 SSL 인증서 2개를 가지고 있으며 더 많은 인증서를 가지고 있을 수 있다.
- ALB는 SNI가 할성화되어 있기 때문에 올바른 SSL 인증서를 선택하여 올바른 대상 그룹으로 리디렉션할 수 있다.
- 클라이언트가 다른 호스트 이름에 대한 요청을 하더라도 적절한 SSL 인증서를 찾아서 알맞는 대상으로 리디렉션한다.

#### Man in the Middle Attack

![23-man-in-the-middle-attack.png](images%2F23-man-in-the-middle-attack.png)

- 사용자는 "Good Server"와 HTTP 통신하고 싶지만 중간에 "Pirate Server"가 있고 해당 서버는 패킷을 가로챌 수 있다.
  - 중간 지점이기 때문에 민감한 정보도 모두 열람할 수 있다.
- 사용자가 "Good Server"와 HTTPS 통신하고 싶지만 이전과 동일하게 중간에 "Pirate Server"가 있다.
  - "Pirate Server"는 가짜 SSL 인증서를 사용자에게 보내려고한다.
  - "Good Server"에서 패킷을 해독하고 다시 암호화하고 바꾼다.
- 사용자의 디바이스가 감염되지 않았다면 가짜 SSL 인증서를 감지하고 "Pirate Server"와 통신하지 않는다.
  - 하지만 사용자의 디바이스가 감염되고 "Pirate Server"의 가짜 SSL 인증서를 신뢰하게 된다면 사용자는 HTTP를 사용할 때처럼 민감한 정보를 노출하게 된다.

- 아래와 같은 방식으로 중간자 공격(Man in the Middle Attack)을 예방할 수 있다.
- public-facing HTTP를 사용하지 않고, HTTPS(SSL/TLS 인증서)를 사용한다.
- DNSSEC가 활성화된 DNS를 사용한다.
  - 클라이언트의 요청을 "Pirate Server"로 전달하려면 DNS 응답을 가로채는 서버가 DNS 응답을 위조해야 한다.
  - DNSSEC를 구성하여 도메인 이름을 보호할 수 있다.
  - Route 53은 도메인 등록을 위해 DNSSEC를 지원한다.
  - Route 53은 2020년 12월 기준 DNS 서비스를 위한 DNSSEC를 지원하며 KMS를 사용한다.
  - 예를 들어, EC2에서 사용자 지정 DNS 서버를 실행할 수 있다. (Bind가 가장 일반적인 dnsmasq, KnotDNS, PowerDNS)

---

### AWS Certificate Manager (ACM)

- AWS에서 공용 SSL 인증서를 호스트하려면 다음을 수행해야 한다.
  - CLI를 사용하여 직접 구입하여 업로드한다.
  - ACM 프로비저닝 및 공용 SSL 인증서 갱신을 사용한다. (무료)
- ACM은 다음의 항목들과 통합되어 SSL 인증서를 로드할 수 있다.
  - Elastic Load Balancer
  - CloudFront
  - API Gateway
- SSL 인증서는 전반적으로 수동으로 관리해야 하는 어려움이 있으므로 ACM은 AWS 인프라를 활용하는 데 탁월하게 사용된다.

![24-aws-certificate-manager.png](images%2F24-aws-certificate-manager.png)

- 사용자는 EC2 인스턴스와 통신하기 위해 ALB를 경유하게 된다.
- 사용자는 HTTPS로 ALB와 통신하고 ALB에서 SSL이 종료되고, ALB는 인스턴스와 HTTP 통신을 한다.
  - EC2 인스턴스는 SSL 암호화나 복호화를 수행할 필요가 없기 때문에 CPU 비용이 적게 발생한다.
- ACM은 ALB의 좌측에서 SSL 인증서를 프로비저닝하고 관리한다.

#### Good to know

- 공인(Public) 인증서를 생성할 수 있다.
  - 공용 DNS를 확인해야 한다.
  - 신뢰할 수 있는 공인 인증 기관(CA)에서 발급해야 한다.
- 개인(Private) 인증서를 생성할 수 있다.
  - 사용자는 자신만의 개인 CA를 생성할 수 있다.
  - 내부 애플리케이션을 위해 사용된다.
  - 내부 애플리케이션은 사용자의 개인 CA를 신뢰해야 한다.
- 인증서를 갱신할 수 있다.
  - ACM에서 프로비저닝되고 생성된 경우 자동으로 수행된다.
  - 수동으로 업로드한 인증서는 수동으로 갱신하고 다시 업로드해야 한다.
- ACM은 리전 서비스다.
  - 글로벌 애플리케이션(예: 여러 ALB)과 함께 사용하려면 애플리케이션이 배포된 각 지역에서 SSL 인증서를 발급해야 한다.
  - 여러 리전에 걸쳐 인증서를 복사할 수 없다.
  - CloudFront는 글로벌 배포이기 때문에 사용자가 신경 쓸 필요가 없지만 ALB & ACM은 리전 서비스기 때문에 여러 SSL 인증서를 사용해야 한다.

---

### CloudHSM

- KMS는 AWS에서 암호화된 소프트웨어를 관리한다.
- CloudHSM은 AWS의 암호화 하드웨어를 프로비저닝한다.
- 전용 하드웨어를 사용하며, HSM은 "Hardware Security Module"의 약자다.
- AWS가 아니라 사용자가 암호화 키를 관리하는 책임을 가지며 암호화 키의 보안과 유지보수, 백업을 전적으로 책임져야 한다.
- HSM 장치는 변조 방지, FIPS 140-2 레벨 3를 준수한다.
- 대칭 및 비대칭 암호화(SSL/TLS 키)를 모두 지원한다.
- 프리 티어에서는 사용할 수 없다.
- CloudHSM을 사용하려면 CloudHSM Client 소프트웨어가 필요하며, API로 호출되지 않는다.
- Redshift는 데이터베이스 암호화 및 키 관리를 위한 CloudHSM을 지원한다.
- S3 객체에 대한 SSE-C 유형 암호화를 사용하고 싶다면 S3에 자신의 암호화 키를 제공하면 서버 측 암호화가 발생한다.
  - 암호화 키를 생성하는 좋은 방법은 CloudHSM을 사용하는 것이며 흔하게 사용되는 패턴이다.

#### Diagram

![25-cloudhsm-diagram.png](images%2F25-cloudhsm-diagram.png)

- AWS가 하드웨어를 관리해 주지만 CloudHSM 장치를 사용하는 것은 사용자의 책임이다.
- 예를 들어, 암호화 키를 분실하게 되는 경우 AWS가 장치를 복구할 수 없다.
- 이러한 암호화 키를 관리하려면 CloudHSM 클라이언트가 필요하다.
  - CloudHSM 장치에 액세스하고 키를 관리한다.
  - CloudHSM 클라이언트와 CloudHSM 장치 사이의 암호화된 연결을 확보한다.
  - 보안 정보를 암호화하고 공유하기 위해서다.
  - 이렇게 장치 내의 모든 것을 관리하는 것은 사용자의 책임이다.
- IAM에서 AWS가 할 수 있는 건 CloudHSM 클러스터를 생성하고 설명하고 삭제하는 것 뿐이며 CloudHSM 클러스터 내에서는 모두 사용자의 책임이다.
- CloudHSM 소프트웨어를 이용해 키를 관리할 수 있다.
  - 키는 생성, 읽기, 수정, 삭제를 할 때 사용한다.
  - 또한 키에 대한 액세스 권한이 있는 사용자도 관리할 수 있다.
  - 예를 들어, AWS가 데이터베이스를 제공하고 그 안의 사용자를 관리하는 방식과 유사하다.
- 장치 내의 모든 보안을 관리해야 하는데 AWS는 그것을 도와줄 수 없다.

#### High Availability

- CloudHSM 클러스터가 여러 개의 HA(Multi AZ)로 분산된다.
- 가용성과 내구성에 탁월한 선택이다.

![26-cloudhsm-high-availability.png](images%2F26-cloudhsm-high-availability.png)

#### CloudHSM vs KMS

- CloudHSM은 KMS와 아래와 같은 차이점이 있다.

![27-cloudhsm-kms.png](images%2F27-cloudhsm-kms.png)

![28-cloudhsm-kms.png](images%2F28-cloudhsm-kms.png)

---

### ALB & SSL

![29-ssl-on-alb.png](images%2F29-ssl-on-alb.png)

- ACM을 통해 SSL 인증서를 로드하고 있는 ALB가 있다.
- 사용자는 HTTPS를 이용해 ALB에 액세스할 수 있다.
- ALB는 HTTP를 이용하는 EC2 인스턴스와 통신하도록 설정되어 있다.

#### SSL 인증서가 탑재된 EC2 인스턴스

![30-ssl-on-web-server-instance.png](images%2F30-ssl-on-web-server-instance.png)

- NLB가 있고 클라이언트와 TCP 연결이 되어 있다.
- NLB는 ASG의 EC2 인스턴스와 HTTPS 통신을 하고 있다.
- EC2 인스턴스는 HTTPS 통신을 하기 위해 SSL 인증서가 필요하다.
- EC2가 부팅되는 시점에 사용자 데이터(user data) 스크립트를 이용해 SSM Parameter Store에서 인증서를 검색한다.
- EC2 인스턴스가 SSL 인증서를 검색하고 로드하기 위해서는 적절한 IAM 역할 설정이 필요하다.
- 만약 EC2 인스턴스에서 SSL 인증서를 로드하고 직접 SSL 암호화와 복호화를 수행하면 EC2 인스턴스의 CPU 리소스를 사용하게 된다.
- 또한, EC2 인스턴스에 누군가 SSH 접속을 할 수 있다면 SSL 인증서가 외부에 유출될 수도 있다.

#### CloudHSM - SSL Offloading

- SSL을 CloudHSM(SSL Acceleration)으로 오프로드할 수 있다.
- 백엔드의 CloudHSM 장치가 EC2 인스턴스를 위한 SSL Acceleration이나 오프로딩을 수행하게 되고 EC2 인스턴스를 위한 민감한 리소스를 저장할 수 있다.

![31-cloudhsm-ssl-offloading.png](images%2F31-cloudhsm-ssl-offloading.png)

- NGINX, Apache Web Server, IIS for Windows Server에서 지원된다.
- SSL 비공개(Private) 키가 HSM 장치를 떠나지 않기 때문에 보안에 유리하다.
- CloudHSM 장치에서 암호화 사용자(CU, Cryptographic User)를 설정해야 하며 EC2 인스턴스는 해당 사용자를 사용할 수 있다.

---

### S3 Encryption

- **SSE-S3**: AWS에서 처리 및 관리하는 키를 사용하여 S3 객체를 암호화한다.
- **SSE-KMS**: KMS를 활용하여 암호화 키를 관리한다.
  - 주요 사용량이 CloudTrail에 표시되어 감사 목적에 유용하게 사용된다.
  - 버킷이 Public으로 공개되더라도 객체 복호화를 위해서는 KMS 키에 접근할 수 있어야 하기 때문에 객체를 읽을 수 없다.
  - `s3:PutObject`를 통해 객체를 업로드할 경우 IAM 권한 중에 `kms:GenerateDataKey`가 허용되어 있어야 한다.
- **SSE-C**: 자신의 암호화 키를 관리하고자 할 때 사용된다.
- 만약 사용자 고유의 키를 사용하여 클라이언트에서 암호화하고 싶다면 "Client-Side Encryption"을 사용해야 한다.
- **Glacier**: 모든 데이터는 "AES-256" 암호화되며 암호화 키는 AWS에 의해 통제된다.

#### 전송 중 암호화 (SSL/TLS)

- Amazon S3는 아래의 항목을 노출한다.
  - HTTP Endpoint: 전송 중 암호화 되지 않음.
  - HTTPS Endpoint: 전송 중 암호화 됨.
- 사용자는 원하는 Endpoint를 자유롭게 사용할 수 있지만 HTTPS를 사용하는 것이 권장된다.
- SSE-C 암호화의 경우 HTTPS를 필수로 사용해야 한다.
- HTTPS 사용을 강제하고 싶다면 `aws:SecureTransport` 버킷 정책을 활성화해야 한다.

#### S3 버킷 이벤트

- **S3 Access Logs**
  - 버킷에 대한 요청에 대한 자세한 기록을 제공한다.
  - 제공하는 데까지 몇 시간이 소요될 수 있다.
  - 불완전할 수 있다.
- **S3 Event Notification**
  - 버킷에서 특정 이벤트가 발생할 때 알림을 수신한다.
  - 예를 들어, 객체 생성, 객체 제거, 객체 복원, 객체 복제 등의 이벤트가 있다.
  - 목적지는 SNS, SQS Queue, Lambda Function이 될 수 있다.
  - 일반적으로 몇 초안에 전달되지만 몇 분 정도 소요될 수 있으며, 버전 설정이 사용되도록 설정된 경우 모든 객체에 대한 알림이 발생할 수 있다.
  - 버전 관리를 활성화하지 않은 경우 아주 드물게 같은 객체에서 동시에 두 개의 쓰기 작업을 한다면 두 개가 아니라 한 개의 알림만 받을 수 있다.
- **Trusted Advisor**
  - 버킷의 권한을 확인할 수 있다.
  - 예를 들어, 버킷이 Public인지 확인할 수 있다.
- **Amazon EventBridge**
  - 먼저 S3에서 CloudTrail 객체 레벨 로깅을 사용하도록 설정해야 한다.
  - 대상은 Lambda Function, SQS Queue, SNS 등이 될 수 있다.

#### S3 Security

- **User Based**
  - IAM 정책: IAM 콘솔에서 특정 사용자에게 API 요청을 허용할 수 있다.
- **Resource Based**
  - 버킷 정책: S3 콘솔에 대한 버킷의 광범위한 규칙으로 계정간 액세스할 때 유용하게 사용된다.
  - S3 버킷에 액세스하려는 계정이 역할을 맡아서 권한을 넘기지 않도록 도움을 준다.
  - Object Access Control List (ACL): 미세한 정보
  - Bucket Access Control List (ACL): 일반적으로 사용되지 않음

#### Bucket Policy

- S3 버킷 정책을 활용하여 아래의 항목들을 수행할 수 있다.
  - 버킷에 대한 Public 액세스 권한을 부여한다.
  - 업로드 시 객체를 암호화하도록 강제한다.
  - 다른 계정(Cross Account)에 대한 액세스 권한을 부여한다.
- 선택적인 조건을 부여할 수 있다.
  - SourceIP: Public IP, Elastic IP, VpcSourceIP (Private IP, VPC Endpoint)
  - Source VPC, Source VPC Endpoint는 VPC Endpoint를 사용할 때 조건을 부여할 수 있다.
  - CloudFront Origin Identity
  - MFA

- S3 버킷 정책에 대한 자세한 내용은 [여기](https://docs.aws.amazon.com/AmazonS3/latest/dev/example-bucket-policies.html)를 확인한다.

#### Pre-signed URL

- SDK 또는 CLI를 사용하여 사전 서명된 URL을 생성할 수 있다.
  - 다운로드의 경우 CLI를 통해 간편하게 사용할 수 있다.
  - 업로드의 경우 복잡하지만 반드시 SDK를 사용해야 한다.
- 기본값으로 3,600초 동안 유효하며 `--expires-in {second}` 옵션으로 시간 초과를 변경할 수 있다.
- 미리 서명된 URL이 부여된 사용자는 GET/PUT에 대해 URL을 생성한 사용자의 권한을 상속한다.
- 대표적으로 아래와 같은 작업을 진행할 때 사용된다.
  - 로그인한 사용자만 S3 버킷에 프리미엄 비디오 다운로드를 허용할 수 있다.
  - URL을 동적으로 생성하여 파일을 다운로드할 수 있도록 변경되는 사용자 목록을 허용할 수 있다.
  - 사용자가 버킷의 정확한 위치에 파일을 업로드할 수 있도록 일시적으로 허용된다.

#### VPC Endpoint Gateway

![32-vpc-endpoint-gateway.png](images%2F32-vpc-endpoint-gateway.png)

- VPC가 있고 Public, Private 서브넷이 있으며 각각 EC2 인스턴스가 실행되고 있다.
- Public 서브넷의 경우 S3 버킷에 Public 액세스를 얻기 위해 Internet Gateway를 사용한다.
  - S3 버킷 정책에는 `AWS:SourceIP`를 사용하여 인스턴스의 IP를 허용해야 한다.
  - 이때 허용하는 IP는 반드시 Public IP 또는 Elastic IP가 되야 한다.
- Private 서브넷의 경우 S3 버킷에 액세스하기 위해 Internet을 통할 수 없다.
  - VPC Endpoint Gateway를 사용하여 AWS 네트워크를 사용하여 S3 접근할 수 있다.
  - S3 버킷 정책에는 `AWS:SourceVpce`를 사용하여 VPC Endpoint Gateway를 지정해야 한다. 
  - 또는 `AWS:SourceVpc`를 사용하여 VPC에 속한 모든 Endpoint를 허용할 수도 있다.

#### S3 Object Lock & Glacier Vault Lock

- **S3 Object Lock**
  - WORM (Write Once Read Many) 모델을 채택한다.
  - 지정된 시간 동안 객체의 버전 삭제를 차단할 수 있다.
  - 보호되는 객체는 해당 시간동안 절대로 지워지지 않는다.
- **Glacier Vault Lock**
  - WORM (Write Once Read Many) 모델을 채택한다.
  - 향후 편집을 위해 정책 잠금을 사용한다.
  - 설정된 객체는 절대로 수정되지 않기 때문에 규정 준수와 데이터 유지 요구가 있는 경우 감사관에게 증명하는데 큰 도움이 된다.
- **"S3 Object Lock"이나 "Glacier Vault Lock"이 설정되면 해당 객체는 절대로 삭제되지 않는다.**

![33-s3-object-lock-glacier-vault-lock.png](images%2F33-s3-object-lock-glacier-vault-lock.png)

---

### S3 Access Point

- Finance 데이터에 연결된 Finance Access Point를 생성할 수 있다.
- 만약 분석을 위한 사용자들이 분석을 위해 Finance 데이터와 Sales 데이터 모두에 접근이 필요한 경우 Analytics Access Point는 두 버킷 모드에 접근할 수 있도록 권한을 설정할 수 있다.
- 각각의 Access Point는 고유한 보안을 가진다.
- 그러므로 적절한 IAM 권한이 있다면 사용자는 Finance Access Point에 접근할 수 있고 Finance Access Point에만 접근할 수 있다.

![34-s3-access-point.png](images%2F34-s3-access-point.png)

- S3 버킷이 많이 생성되고 접근하려는 사용자가 많아지면 버킷 정책이 계속 생성되어 관리가 복잡해질 수 있다.
- Access Point 생성을 통해 S3 버킷에 대한 보안 관리를 간소화할 수 있다.
- 각 액세스 포인트는 다음과 같다.
    - 자체 DNS 이름 (인터넷 오리진 또는 VPC 오리진)
    - Access Point 정책(버킷 정책과 유사) - 보안을 규모에 맞게 관리

#### VPC Origin

- Access Point를 VPC 내에서만 액세스할 수 있도록 정의할 수 있다.
- Access Point(Gateway or Interface Endpoint)에 액세스하려면 VPC Endpoint를 생성해야 한다.
- VPC Endpoint 정책에서 대상 버킷 및 Access Point에 대한 액세스를 허용해야 한다.

![35-s3-access-point-vpc-origin.png](images%2F35-s3-access-point-vpc-origin.png)

#### Multi-Region Access Point

![36-multi-region-access-point.png](images%2F36-multi-region-access-point.png)

- 여러 AWS 리전의 S3 버킷에 걸쳐 있는 글로벌 엔드포인트를 제공할 수 있다.
- 가장 가까운 S3 버킷으로 요청을 동적으로 라우팅한다. (가장 낮은 지연 시간)
- 양방향 S3 버킷 복제 규칙을 만들어 여러 리전에서 데이터를 동기화 상태로 유지화할 수 있다.
- Failover Control: 서로 다른 AWS 리전의 S3 버킷 간에 몇 분 내에 요청을 전환할 수 있다. (Active-Active or Active-Passive)

![37-multi-region-access-point.png](images%2F37-multi-region-access-point.png)

- `us-east-1`, `eu-west-1`, `ap-southeast-1` 총 3개 리전에 있는 S3 버킷에 데이터가 복제된다.
  - 모든 버킷에 걸쳐 복제 규칙이 있어야 한다.
- S3 Multi-Region Access Point를 생성한다.
- 애플리케이션이 S3 버킷 객체를 요청하면 대기 시간이 가장 낮은 리전으로 자동으로 라우팅된다.
- 해당 지역이 다운되면 다른 리전으로 자동으로 라우팅된다.

#### Multi-Region Access Point - Failover Control

![38-multi-region-access-point-failover-control.png](images%2F38-multi-region-access-point-failover-control.png)

- 두 리전(`us-east-1`, `eu-west-1`)에 걸쳐서 S3 버킷이 복제되고 있다.
- S3 Multi-Region Access Point가 생성되어 있다.
- 위의 예시에서는 `us-east-1`의 버킷이 Active로 지정되어 있으며 `eu-west-1`의 버킷이 Passive로 지정되어 있다.
- Active-Passive로 설정되어 있는 경우 애플리케이션의 요청은 지연시간이 짧은 곳이 아니라 Active되어 있는 곳으로 라우팅된다.
  - Active된 곳이 여러 개인 경우 Active된 리전 중에서 가장 지연시간이 짧은 곳으로 라우팅된다.
- 예시처럼 `us-east-1`에 장애가 발생한 경우 `eu-west-1`로 장애 조치가 자동으로 시작된다.

---

### S3 Object Lambda

- Lambda Function을 사용하여 호출하는 애플리케이션에서 객체를 검색하기 직전에 객체를 변경할 때 사용된다.
  - 객체를 복제하여 각 객체의 다른 버전을 생성하는 대신에 S3 Object Lambda를 사용할 수 있다.
- S3 버킷은 하나만 필요하며, 그 위에 "S3 Access Point"와 "S3 Object Lambda Access Point"를 생성한다.
- 대표적인 사용 사례는 아래와 같다.
  - 분석 또는 비운영 환경을 위해 개인 식별 정보를 수정한다.
  - XML을 JSON으로 변환하는 등의 데이터 형식 간 변환이 필요할 때 사용한다.
  - 객체를 요청한 사용자와 같은 발신자별 세부 정보를 사용하여 이미지 크기를 조정하고 이미지에 워터마킹을 생성하는 데 사용한다.

![39-s3-object-lambda.png](images%2F39-s3-object-lambda.png)

- AWS 클라우드가 있고 그 안에 S3 버킷이 생성되어 있다.
- "E-Commerce" 애플리케이션은 S3 버킷에 데이터를 소유하고 있기 떄문에 직접 S3 버킷에 접근해 원본 객체를 꺼낼 수 있다.
- "Analytics" 애플리케이션은 수정된 객체에만 접근하도록 설정할 수 있다.
  - S3 버킷을 새로 만드는 대신 S3 버킷 위애 "S3 Access Point"를 만들고 "Lambda Function"을 연결할 수 있다.
- "Marketing" 애플리케이션은 향상된 객체에 대한 액세스 권한을 가지도록 설정할 수 있다.
  - 고객은 데이터를 강화하기 위한 "Customer Loyalty Database"를 가진다.
  - 새로운 S3 버킷을 만들어 모든 객체를 강화하는 대신 Lambda Function을 사용하여 데이터를 강화할 수 있다.

---

### DDoS

- DDoS는 "Distributed Denial-of-Service"의 약자다.

![40-ddos.png](images%2F40-ddos.png)

- 공격자는 다양한 방법을 통해 봇을 생성할 수 있는 마스터를 생성하고, 생성된 마스터는 수많은 봇을 생성한다.
- 봇들은 애플리케이션 서버에 수많은 요청을 하게 되고, 애플리케이션 서버는 부하가 높아지며 일반 사용자의 요청을 처리하지 못하게 된다.

#### Infrastructure 공격 종류

- Distributed Denial of Service (DDoS)
  - 요청이 너무 많아서 서비스를 사용할 수 없는 경우
  - SYN Flood (Layer 4): 너무 많은 TCP 연결 요청을 전송
  - UDP Reflection (Layer 4): 다른 서버가 많은 UDP 요청을 보내도록 설정
  - DNS Flood Attack: DNS를 압도하여 합법적인 사용자가 사이트를 찾을 수 없도록 한다.
  - Slow Loris Attach: 많은 HTTP 연결이 생성되고 유지된다.
- Application Level Attach
  - 더 복잡하고 더 구체적인 공격을 한다. (HTTP 수준)
  - 캐시 버스트 전략: 캐시를 비활성화하여 백엔드 데이터베이스를 오버로드한다.

#### DDoS Protection

- AWS Shield Standard: 웹 사이트 및 애플리케이션에 대한 DDoS 공격으로부터 추가 비용 없이 보호한다.
- AWS Shield Advanced: 연중무휴 프리미엄으로 DDoS 공격으로부터 보호한다.
- AWS WAF: 규칙에 따라 특정 요청만 필터링한다.
- CloudFront & Route 53
  - 글로벌 엣지 네트워크를 사용하여 가용성을 보호한다.
  - AWS Shield와 통합하여 엣지에서 DDoS 공격 완화 기능을 제공한다.
- Be read to scale: AWS Auto Scaling을 사용할 수 있다.
- 정적 리소스(S3/CloudFront)와 동적 리소스(EC2/ALB)를 분리한다.
- 자세한 내용은 [DDoS 백서](https://d1.awsstatic.com/whitepapers/Security/DDoS_White_Paper.pdf)를 확인한다.

#### Sample 아키텍처

![41-sample-reference-architecture.png](images%2F41-sample-reference-architecture.png)

- DNS 서비스인 Route 53은 기본적으로 AWS Shield가 보호한다.
- CloudFront로 배포된 애플리케이션에 대해서도 AWS Shield가 기본적으로 보호한다.
  - 추가로 애플리케이션에 방화벽(AWS WAF)를 설치할 수 있다.
- 마지막으로 트래픽이 통과하면 로드밸런서에 요청을 하게 되고 로드밸런서에도 AWS Shield를 적용할 수 있다.

#### AWS Shield

- **AWS Shield Standard**
  - 모든 AWS 고객에게 활성화되는 무료 서비스다.
  - SYN/UDP Floods, Reflection 공격 및 기타 Layer3/Layer4 공격과 같은 공격으로부터 보호한다.
- **AWS Shield Advanced**
  - DDoS 완화 서비스로 조직당 월 3,000달러를 지불하고 선택적으로 사용할 수 있다.
  - EC2, ELB, CloudFront, Global Accelerator, Route 53에 대한 보다 정교한 공격으로부터 보호할 수 있다.
  - 24시간 연중무휴 서비스로 AWS DDoS 대응 팀(DRP)의 지원을 받을 수 있다.
  - DDoS로 인한 사용량 급증 시 발생할 수 있는 높은 클라우드 사용료로부터 보호받을 수 있다.

---

### AWS WAF - Web Application Firewall

- 일반적인 웹 공격으로부터 웹 애플리케이션을 보호한다. (Layer 7)
- 애플리케이션 로드 밸런서에 배포한다. (현지화된 규칙)
- API Gateway에 배포한다. (리전 또는 엣지 로케이션에서 실행되는 규칙)
- CloudFront에 배포한다. (전세계 엣지 로케이션에서 실행되는 규칙)
  - CLB, EC2 인스턴스, Custom Origin, S3 Website 등 다른 솔루션을 소개하는 데 사용된다.
- AppSync에 배포한다. (GraphQL API 보호)
- WAF는 DDoS 보호를 위해 사용되지 않는다.
- Web ACL을 정의한다. (Web Access Control List)
  - 규칙에는 IP 주소, HTTP 헤더, HTTP 본문 또는 URI 문자열이 포함될 수 있다.
  - 일반적인 공격으로부터 보호한다. (SQL Injection, Cross-SIte Scripting (XSS))
  - 크기 제약을 두거나 특정 국가의 요청을 제한할 수 있다.
  - 속도 기반 규칙을 사용할 수 있다. (이벤트 발생 횟수)
- 규칙 동작으로 트래픽을 허용 또는 차단할 수 있다.
  - Clout, Allow, Block Captcha

#### Managed Rules

- 190개 이상의 관리되는 규칙이 있는 라이브러리가 있다.
- AWS 및 AWS 마켓플레이스 판매자가 관리하는 즉시 사용 가능한 규칙이다.
- **Baseline Rule Groups(기본 규칙 그룹)**: 일반적인 위협으로부터 일반적인 보호를 제공한다.
  - `AWSManagedRulesCommonRuleSet`, `AWSManagedRulesAdminProtectionRuleSet` 등..
- **Use-case Specific Rule Groups(사용 사례별 규칙 그룹)**: 많은 AWS WAF 사용 사례에 대한 보호를 제공한다.
  - `AWSManagedRulesSQLiRuleSet`, `AWSManagedRulesWindowsRuleSet`, `AWSManagedRulesPHPRuleSet`, `AWSManagedRulesWordPressRuleSet` 등..
- **IP Reputation Rule Groups(IP 평판 규칙 그룹)**: 소스(예: 악의적인 IP)를 기반으로 요청을 차단한다.
  - `AWSManagedRulesAmazonIpReputationList`, `AWSManagedRulesAnonymousIpList`
- **Bot Control Managed Rule Group(봇 컨트롤 관리 규칙 그룹)**: 봇의 요청을 차단 및 관리한다.
  - `AWSManagedRulesBotControlRuleSet`

#### Web ACL - Logging

- 로그를 다음 위치로 보낼 수 있다.
  - Amazon CloudWatch Log Group - 초당 5MB
  - Amazon S3 Bucket - 5분 간격
  - Amazon Kinesis Data Firehose - Firehose 할당량에 따라 제한

![42-webacl-logging.png](images%2F42-webacl-logging.png)

#### Solution Architecture - CloudFront Origin 보안 강화

![43-ehance-cloudfront-origin-security.png](images%2F43-ehance-cloudfront-origin-security.png)

- EC2 인스턴스에서 실행되는 애플리케이션의 로드 밸런서 앞에 CloudFront가 위치하고 있다.
- CloudFront에서 요청하는 트래픽만 애플리케이션 로드 밸런서에 들어갈 수 있고 직접 ALB를 호출하는 요청은 허용해서는 안된다.
- WAF를 사용하면 CloudFront 수준에서 Web ACL을 설정할 수 있다.
  - 애플리케이션의 로드 밸런서에 도움이 되지 않지만 최소한 CloudFront의 보안은 제어할 수 있다.
- CloudFront에서 사용자 지정 HTTP 헤더를 생성한다.
  - 예를 들어, `X-Origin-Verify`라는 헤더를 추가하고 비밀 문자열을 추가한다.
  - 해당 헤더는 CloudFront를 통과하는 모든 요청에 추가된다.
  - 로드 밸런서의 WAF는 필터링 규칙을 통해 해당 헤더가 포함된 요청만 허용하도록 설정할 수 있다.
- AWS Secret Manager를 통해서 일정한 주기로 헤더에 추가된 비밀 문자열을 갱신되도록 설정할 수 있다.
  - 설정되면 Lambda Function에 의해 CloudFront에서 전송하는 헤더의 값과, WAF에서 허용하는 헤더의 값이 변경된다.

---

### AWS Firewall Manager

- **AWS Organization에 포함되어 있는 모든 계정에서 규칙을 관리한다.**
- 보안 정책: 공통 보안 규칙 집합
  - WAF 규칙 (ALB, API Gateway, CloudFront)
  - AWS Shield Advanced (ALB, CLB, NLB, EIP, CloudFront)
  - EC2 인스턴스 보안 그룹, VPC의 ALB와 ENI 리소스
  - AWS Network Firewall (VPC 레벨)
  - Route 53 Resolver DNS Firewall
  - 정책은 리전 수준에서 생성된다.
- **규칙은 Organization의 모든 계정과 향후 계정이 생성될 때 새 리소스에 적용된다.**

#### WAF, Firewall Manager, Shield

- 포괄적인 보호를 위해 WAF, Shield 및 Firewall Manager가 함께 사용된다.
- WAF에서 Web ACL 규칙을 정의한다.
- 리소스를 세부적으로 보호하려면 WAF만 사용하는 것이 올바른 선택이다.
- 여러 계정에서 AWS WAF를 사용하려면 "WAF 구성 가속화", 새 리소스 보호 자동화, Firewall Manager와 AWS WAF를 사용한다.
- Shield Advanced는 SRT(Shield Response Team)의 전용 지원 및 고급 보고와 같은 AWS WAF 기능을 추가한다.
- DDoS 공격이 잦은 경우 Shield Advanced 구입을 고려할 필요가 있다.

---

### IP 주소 차단

![44-blocking-ip-address.png](images%2F44-blocking-ip-address.png)

- VPC에 EC2 인스턴스가 실행되고 있고 보안 그룹을 가지고 있다.
- 인스턴스는 Public IP를 가지고 있으므로 공개적으로 액세스가 가능하다.
- NACL에서 클라이언트의 IP 주소를 위한 거부 규칙을 생성할 수 있다.
- 만약 EC2 인스턴스에 접근할 수 있는 승인된 클라이언트 하위 집합만 있다면 보안 그룹에서 EC2 인스턴스에 들어갈 수 있는 IP 하위 집합을 정의하는 것이 좋다.
  - 애플리케이션이 글로벌 서비스라면 모든 IP 주소를 알 수 없기 때문에 보안 그룹은 도움이 되지 않는다.
- 이미 클라이언트의 요청이 EC2 인스턴스에 도달했다면 처리를 해야 하고 요청을 처리하려면 CPU 비용이 많이 소비된다.

#### ALB & NLB

![45-blocking-ip-address-alb.png](images%2F45-blocking-ip-address-alb.png)

- ALB는 VPC내에 정의되어 있고 동일한 VPC에 EC2 인스턴스도 실행되고 있다.
  - ALB의 보안 그룹과 EC2 인스턴스의 보안 그룹이 있다.
- ALB는 클라이언트와 EC2 인스턴스 사이에서 있으므로 클라이언트가 ALB에 연결을 하면 연결이 종료되고 ALB로부터 EC2 인스턴스로 새로운 연결이 생성된다.
- EC2 인스턴스의 보안 그룹은 ALB의 보안 그룹이 되도록 구성해야 한다.
- EC2 인스턴스는 Private 서브넷에 Private IP와 함께 배포되어 클라이언트에게 공개되지 않도록 구성되어 있다.
- 이전과 동일하게 NACL을 통해서 클라이언트의 요청을 제한할 수 있다.

![46-blocking-ip-address-nlb.png](images%2F46-blocking-ip-address-nlb.png)

- ALB 대신 NLB가 사용된 경우에도 동일한 방식으로 클라이언트의 요청을 제한한다.

#### ALB + WAF

![47-blocking-ip-address-alb-waf.png](images%2F47-blocking-ip-address-alb-waf.png)

- 특정 IP를 거부하기 위해 WAF나 "Web Application Firewall"을 설치할 수 있다.
  - WAF의 경우 부가 서비스이며 방화벽 서비스이기 때문에 비용이 조금 더 발생한다.
- WAF를 통해서 복잡한 필터링을 할 수 있다.
- 요청에 대한 규칙을 설정하여 클라이언트가 많은 요청을 동시에 하는 것을 방지할 수 있다.
- WAF는 클라이언트와 ALB 사이의 서비스가 아니라 ALB에 설치한 서비스로 여러 규칙을 정의할 수 있다.

#### ALB + CloudFront + WAF

![48-blocking-ip-address-alb-cloudfront-waf.png](images%2F48-blocking-ip-address-alb-cloudfront-waf.png)

- ALB 앞에 CloudFront를 사용하면 VPC 외부에 있는 CloudFront의 Public IP를 허용해야 한다.
- ALB는 클라이언트의 IP를 확인하는 것이 아니라 CloudFront의 Public IP만 확인할 수 있다.
  - 이런 경우 NACL은 특정 IP를 차단하는 데 도움이 되지 않는다.
- 대신 CloudFront에 WAF를 설치할 수 있다.
  - 특정 지역에서 발생하는 요청을 차단하거나 허용할 수 있다.
  - 문제를 발생시키는 IP가 확인되는 경우 WAF를 사용하여 IP 주소 필터링을 할 수 있다.

---

### Amazon Inspector

- 자동화된 보안 평가를 제공한다.
- EC2 인스턴스
  - AWS System Manager(SSM) Agent를 활용하여 인스턴스의 보안을 평가한다.
  - 의도하지 않은 네트워크 접근성으로부터 분석한다.
  - 알려진 취약성에 대해 실행 중인 OS를 분석한다.
- Amazon ECR로 푸시되는 컨테이너 이미지
  - 컨테이너 이미지가 푸시될 때, 이미 알려진 취약점을 분석한다.
- Lambda Function
  - 기능 코드 및 패키지 종속성의 소프트웨어 취약성을 식별한다.
  - Function이 배포될 때, 이미 알려진 취약점을 분석한다.
- 분석이 완료되면 AWS Security Hub에 분석에 대한 리포트를 생성한다.
- 분석 결과를 EventBridge로 전송할 수 있다.

![49-amazon-inspector.png](images%2F49-amazon-inspector.png)

#### 평가 항목

- **"EC2 인스턴스", "컨테이너 이미지", "Lambda Function"에 대해서만 분석을 제공**한다.
- 필요한 경우에만 인프라에 대한 지속적인 검색을 제공한다.
- CVE 데이터베이스를 통해 패키지 취약점을 분석한다.
  - CVE 데이터베이스가 업데이트 되는 경우 Inspector는 자동으로 다시 모든 인프라를 검사한다. 
- EC2 인스턴스의 네트워크 도달 가능성을 조사한다.
- 위험 점수는 우선 순위 지정을 위한 모든 취약성과 연관되어 있다.

---

### AWS Config

- AWS 리소스의 컴플라이언스 감사 및 기록을 지원한다.
- 시간 경과에 따른 구성 및 변경 사항 기록을 지원한다.
- **AWS Config Rule이 작업 수행을 방지할 수는 없다.**
- AWS Config를 사용하면 아래의 질문들에 답변할 수 있다.
  - 보안 그룹에 대한 제한 없는 SSH 액세스가 있는가.
  - 버킷에 Public 접근 권한이 있는가.
  - 시간이 지남에 따라 ALB 구성이 어떻게 변경되었는가.
- 변경 사항에 대한 알림(SNS 알림)을 받을 수 있다.
- AWS Config는 리전 수준의 서비스이므로 여러 리전을 사용하는 경우 모든 리전에서 설정해야 한다.
- 계정 및 리전에 있는 모든 AWS Config 데이터를 하나의 중앙 계정으로 집계할 수 있다.
  - 로그인, 구성 등이 있는 보안 계정의 좋은 사용 사례다.

#### AWS Config Resource

- 시간 경과에 따른 리소스의 규정 준수 보기가 가능하다.
  - 일부 규칙이 해당 리소스에 적용되는 경우 규정을 준수하지 않을 때 빨간색으로 표시된다.
  - 규정을 준수하는 경우 녹색으로 변경된다.

![50-aws-config-resource.png](images%2F50-aws-config-resource.png)

- 시간 경과에 따른 리소스 구성을 확인할 수 있으므로 언제 변경되고 어떤 변경이 발생하는지 확인할 수 있다.

![51-aws-config-resource.png](images%2F51-aws-config-resource.png)

- 활성화되어 있는 경우 CloudTrail API 호출을 AWS Config에서 확인할 수 있다.

#### AWS Config Rule

- AWS 중앙 관리 구성 규칙을 사용할 수 있다. (75개 이상)
- 사용자 지정 구성 규칙을 생성할 수 있다. (AWS Lambda에 정의 필요)
  - EBS 디스크가 gp2 유형인지 평가
  - EC2 인스턴스가 t2.micro인지 평가
- 규칙을 평가/트리거할 수 있다.
  - 각 구성 변경에 대해
  - 일정한 시간 간격으로
- 규칙이 비준수인 경우 Amazon EventBridge를 트리거할 수 있다. (Lambda 체인)
- SSM Automation을 통해 규칙에 자동 업데이트를 적용할 수 있다.
  - 리소스가 규정을 준수하지 않으면 자동 업데이트 적용을 트리거할 수 있다.
  - 예를 들어, 보안 그룹 규칙 업데이트 적용, 승인되지 않은 태그가 있는 인스턴스 중지

---

### AWS Managed Logs

- 로드 밸런서 액세스 로그(ALB, NLB, CLB)는 S3로 전송된다.
  - 로드 밸런서용 액세스 로그를 제공한다.
- CloudTrail 로그는 S3와 CloudWatch Logs로 전송된다.
  - 계정 내에서 API 호출에 대한 로그를 제공한다.
- VPC Flow 로그는 S3, CloudWatch Logs, Kinesis Firehose로 전송된다.
  - VPC의 네트워크 인터페이스에서 송수신되는 IP 트래픽에 대한 정보를 제공한다.
- Route 53 Access Logs는 CloudWatch Logs로 전송된다.
  - Route 53이 수신하는 쿼리에 대한 로그 정보를 제공한다.
- S3 Access Logs는 S3로 전송된다.
  - 서버 액세스 로깅은 버킷에 대한 요청에 대해 상세 레코드를 제공한다.
- CloudFront Access Logs는 S3로 전송된다.
  - CloudFront가 수신하는 모든 사용자 요청에 대한 상세 정보를 제공한다.
- AWS Config는 S3로 전송된다.

---

### Amazon GuardDuty

- AWS 계정 보호를 위한 지능형 위협을 검색한다.
- 머신 러닝 알고리즘, 이상 탐지, 써드파티 데이터를 사용한다.
- 한 번의 클릭으로 활성화할 수 있으며 30일간의 평가판을 제공하고, 소프트웨어를 설치할 필요가 없다.
- 입력 데이터는 다음을 포함한다.
  - CloudTrail Events Logs: 비정상적인 API 호출, 무단 배포
    - CloudTrail Management Events: VPC 서브넷 생성, 추적 생성 등..
    - CloudTrail S3 Data Events: 객체 가져오기, 객체 나열, 객체 삭제 등..
  - VPC Flow Logs: 비정상적인 내부 트래픽, 비정상적인 IP 주소
  - DNS Logs: 손상된 EC2 인스턴스가 DNS 쿼리 내에서 인코딩된 데이터를 전송한다.
  - Optional Feature: EKS 감사 로그, RDS & Aurora, EBS, Lambda, S3 데이터 이벤트 등..
- 발견 시 EventBridge 규칙을 알림으로 설정할 수 있다.
- EventBridge 규칙은 AWS Lambda 또는 SNS를 대상으로 할 수 있다.
- `CryptoCurrency` 공격으로부터 보호할 수 있다.(전용 "검색"기능이 있다.)

![52-amazon-guardduty.png](images%2F52-amazon-guardduty.png)

#### Delegated Administrator

- AWS Organization 회원 계정을 GuardDuty 위임 관리자로 지정할 수 있다.
- Organization의 모든 계정에 대해 GuardDuty를 활성화하고 관리할 수 있는 완전환 권한이 있다.
- 조직 관리 계정을 사용해야만 수행할 수 있다.

![53-delegated-administrator.png](images%2F53-delegated-administrator.png)

---

### IAM Conditions

- IAM Condition은 정책 내에 적용된다.
- 사용자를 위한 정책일 수도 있고 리소스를 위한 정책일 수도 있다.

- **SourceIP**
  - API 호출이 발생하는 클라이언트 IP를 제한하는 데 사용된다.
  - 특정 IP 주소 범위가 아닌 경우 모든 IP를 차단한다.
  - 예를 들어, 회사 네트워크를 등록하고 회사가 아닌 환경에서는 작업하지 못하도록 제한할 수 있다.

```yaml
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
         "NotIpAddress": {
           "aws:SourceIp": ["192.0.2.0/24", "203.0.113.0/24"]
         }
      }
    }
  ]
}
```

- **RequestedRegion**
  - API 호출이 생성되는 지역을 제한한다.
  - 아래의 예시는 사용자가 `eu-contral-1` 또는 `eu-west-1` 리전에 있다면 EC2, RDS, DynamoDB에 대한 작업을 제한한다.

```yaml
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": ["ec2:*", "rds:*", "dynamodb:*"],
      "Resource": "*",
      "Condition": {
         "StringEquals": {
           "aws:RequestedRegion": ["eu-central-1", "eu-west-1"]
         }
      }
    }
  ]
}
```

- **ResourceTag**
  - 리소스 태그를 기반으로 정책을 적용할 수 있다.
  - 아래의 예시에서 `aws:PrincipalTag/Department`는 EC2 인스턴스 태그가 아니라 사용자 태그다.
    - 즉, 작업을 수행하려면 사용자도 부서 데이터의 일부여야 한다.

```yaml
{
  "Version": "2012-10-17",
  "Statement": [
    "Effect": "Allow",
    "Action": ["ec2:startInstances", "ec2:StopInstances"],
    "Resource": "arn:aws:ec2:us-east-1:123456789012:instance/*",
    "Condition": {
       "StringEquals": {
         "ec2:ResourceTag/Project": "DataAnalytics",
         "aws:PrincipalTag/Department": "Data"
       }
    }
  ]
}
```

- **MultiFactorAuthPresent**
  - MFA 활성화를 강제할 수 있다.

```yaml
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": ["ec2:StopInstances", "ec2:TerminateInstances"],
      "Resource": "*",
      "Condition": {
         "BoolIfExists": {
           "aws:MultiFactorAuthPresent": false
         }
      }
    }
  ]
}
```

#### IAM for S3

```yaml
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::test"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::test/*"
    }
  ]
}
```

- S3 버킷에 대한 IAM 정책을 생성할 수 있다.
- 예시에서 `s3:ListBucket`은 버킷 수준의 권한이다.
- `s3:GetObject`, `s3:PutObject`, `s3:DeleteObject`는 객체 수준의 권한이다.

#### Resource Policy & aws:PrincipalOrgID

- `aws:PrincipalOrgID`는 모든 리소스 정책에서 AWS Organization의 구성원인 계정에 대한 액세스를 제한하는 데 사용할 수 있다.

```yaml
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject"],
      "Resource": "arn:aws:s3:::2022-financial-data/*",
      "Condition": {
         "StringEquals": {
           "aws:PrincipalOrgID": ["o-yyyyyyyyyy"]
         }
      }
    }
  ]
}
```

![54-resource-policy-principalorgid.png](images%2F54-resource-policy-principalorgid.png)

---

### EC2 Instance Connect (SendSSHPublicKey API)

![55-ec2-instance-connect.png](images%2F55-ec2-instance-connect.png)

- EC2 인스턴스와 고유한 보안 그룹이 있다.
- 인바운드 규칙에선 포트 22의 SSH를 특정 소스로부터 허용한다.
- 소스는 IP 접두사(`ip_prefix`)를 포함한다.
- EC2 Instance Connect 서비스에 작동한다.
- 22번 포트를 열어 인스턴스에 SSH를 넣는다. 
- 사용자가 EC2 Instance Connect API에 액세스하게 되면 우리가 EC2 인스턴스를 연결할 때마다 뒤에서 한 번 푸시를 업로드하고 SSH 퍼블릭 키를 EC2 인스턴스에 업로드한다.
- 그 키는 60초간 유효하고 `SendSSHPublicKey` API다.
- 이 API를 통해서 EC2 인스턴스와 서비스 연결을 가능하게 하고 사용자들도 사용할 수 있다.
  - EC2 인스턴스에 지정된 SSH 키를 업로드할 수 있다.
  - 즉, 60초 동안 EC2 인스턴스에 대응되는 사설 키를 이용해 연결할 수 있다.
- EC2 Instance Connect API를 호출하면 `SendSSHPublicKey` API를 사용하게 되므로 CloudTrail에 로그가 남게 된다.

---

### AWS Security Hub

- 여러 AWS 계정 간의 보안을 관리하고 보안 감사를 자동화하는 중앙 보안 툴이다.
- 현재 보안 및 규정 준수 상태를 보여주는 통합 대시보드를 통해 신속한 조치를 수행한다.
- 다양한 AWS 서비스 및 AWS 파트너 툴의 사전 정의된 또는 개인 소견 형식으로 경고를 자동으로 집계한다.
  - AWS Config
  - GuardDuty
  - Inspector
  - Macie
  - IAM Access Analyzer
  - AWS Systems Manager
  - AWS Firewall Manager
  - AWS Health
  - AWS Partner Network Solution
- Security Hub가 작동하려면 반드시 AWS Config 서비스를 활성화해야 한다.

![56-aws-security-hub.png](images%2F56-aws-security-hub.png)

- Security Hub는 한 번에 여러 계정을 커버할 수 있고 마치 GuardDuty, Inspector, Firewall Manager, Systems Manager가 제공하는 데이터에서 잠재적 문제와 결과를 찾는다.
- 자동 확인 덕분에 대시보드에서 Security Hub의 결과를 확인할 수 있다.
- 보안 문제가 발생할 때마다 EventBridge에서 이벤트가 발생한다.
- 문제의 근원을 조사하기 위해 Amazon Detective를 사용할 수 있으며 보안 문제가 어디서 발생하는지 빠르게 확인할 수 있다.

---

### Amazon Detective

- GuardDuty, Macie, Security Hub를 사용하여 잠재적인 보안 문제 또는 발견을 식별한다.
- 보안 결과를 통해 근본 원인을 파악하고 조치를 취하기 위해서는 심층적인 분석이 필요한 경우가 있다. - 이는 복잡한 프로세스다.
- Amazon Detector는 보안 문제나 의심스러운 활동의 근본 원인을 분석, 조사하고 신속하게 파악한다. (ML 및 Graphs 사용)
- VPC Flow Logs, CloudTrail, GuardDuty에서 이벤트를 자동으로 수집 및 처리하고 통합 뷰를 생성한다.
- 세부 정보와 상황에 따라 시각화를 수행하여 근본적인 원인을 파악한다.

---

### 참고한 강의

- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)