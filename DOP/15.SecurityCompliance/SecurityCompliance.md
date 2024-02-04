# Security & Compliance

- 이번 장에서는 **DevOps Engineer Professional (DOP)** 을 준비하며 "보안과 규정준수"에 대해서 알아보도록 한다.

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

#### Resource

- 시간 경과에 따른 리소스의 규정 준수 보기가 가능하다.
  - 일부 규칙이 해당 리소스에 적용되는 경우 규정을 준수하지 않을 때 빨간색으로 표시된다.
  - 규정을 준수하는 경우 녹색으로 변경된다.

![1-aws-config-resource.png](images%2F1-aws-config-resource.png)

- 시간 경과에 따른 리소스 구성을 확인할 수 있으므로 언제 변경되고 어떤 변경이 발생하는지 확인할 수 있다.

![2-aws-config-resource.png](images%2F2-aws-config-resource.png)

- 활성화되어 있는 경우 CloudTrail API 호출을 AWS Config에서 확인할 수 있다.

#### Remediations

- SSM 자동화 문서를 사용하여 비준수 리소스 교정을 자동화한다.
- AWS 관리형 자동화 문서를 사용하거나 사용자 지정 자동화 문서를 생성한다.
  - 람다 함수를 호출하는 사용자 지정 자동화 문서를 생성할 수 있다.
- 자동 교정 후에도 리소스가 여전히 비준수 상태인 경우 교정 재시도를 설정할 수 있다.

![3-rules-remediations.png](images%2F3-rules-remediations.png)

- 사용자가 IAM 키가 만료되었는지 모니터링하고 있다.
  - 예를 들어, 90일이 넘으면 준수하지 않는다고 표시해야 한다.
- Config가 이 규칙을 준수하지 않는 경우를 막아줄 수는 없다.
  - 하지만 리소스가 규칙을 준수하지 않을 경우 수정 작업을 트리거한다.
- 예를 들어, SSM 문서가 있고 `RevokeUnusedIAMUserCredentials`라는 권한을 가지고 있다.
  - 인스턴스에는 이 경우 IAM 접근 키를 비활성화한다.
  - 즉, AWS에서 관리하는 문서를 사용하거나 자동화 문서를 직접 생성하더라도 미준수 리소스에 대해 수정을 할 수 있다.

#### Notifications

- AWS 리소스가 규정을 준수하지 않을 때, EventBridge를 사용하여 알림을 트리거한다.

![4-rules-notifications.png](images%2F4-rules-notifications.png)

- 구성 변경 사항 및 규정 준수 상태 알림을 SNS로 보내는 기능을 제공한다.
  - 모든 이벤트 - SNS 필터링 사용 또는 클라이언트 필터링

![5-rules-notifications-2.png](images%2F5-rules-notifications-2.png)

#### Configuration Recorder

- AWS 리소스의 구성을 "구성 항목(Configuration Items)"으로 저장한다.
- 구성 항목: AWS 리소스의 다양한 속성에 대한 특정 시점 보기다.
  - AWS Config가 리소스(예: 속성, 관계, 구성, 이벤트 등..)에 대한 변경 사항을 감지할 때마다 생성된다.
- 사용자가 지정하는 리소스 유형만 기록하는 사용자 정의 구성 레코더다.
- AWS Config가 리소스를 추적하기 전에 생성되어야 한다.
  - AWS CLI 또는 AWS 콘솔을 사용하여 AWS Config를 활성화할 때 자동으로 생성된다.

![6-configuration-recorder.png](images%2F6-configuration-recorder.png)

- "구성 레코더"를 활성화해서 Config를 Organization 내 모든 계정에 사용하는 한 가지 방법은 스택 세트를 사용하는 것이다.
- 스택 세트는 모든 구성원의 계정에 AWS Config 구성 레코더를 배포하게 된다.
  - 그러면 Config가 전체에 활성화된다.
- Config가 활성화된 상태를 유지하려면 Organization의 루트 권한으로 SCP 정책을 등록하여 사용자가 Config를 비활성화하거나 구성 레코더를 삭제하지 않도록 한다.
  - 모든 구성원 계정에 적용된다.

#### Aggregators

![7-aggregators.png](images%2F7-aggregators.png)

- 집계자는 하나의 중앙 집계자 계정에 생성된다.
- 여러 계정 및 지역에 걸쳐 규칙, 리소스 등을 집계한다.
- AWS Organizations를 사용하는 경우 개별 인증이 필요하지 않다.
- 규칙은 각 개별 소스 AWS 계정에서 생성된다.
- CloudFormation StackSets를 사용하여 여러 대상 계정에 규칙을 배포할 수 있다.

#### Conformance Pack

- AWS Config 규칙 및 해결 작업을 수집한다.
- 팩은 YAML 형식의 파일로 생성되며 CloudFormation과 유사하다.
- AWS 계정 및 리전에 배포하거나 AWS Organizations에 배포한다.
- 사전 구축된 샘플 팩 또는 자신만의 맞춤형 적합성 팩을 생성할 수 있다.
- 리소스가 구성 규칙을 준수하는지 여부를 평가하기 위해 람다 함수로 지원되는 사용자 지정 구성 규칙을 포함할 수 있다.
- 파라미터 섹션을 통해 입력을 전달하여 더욱 유연하게 만들 수 있다.
- AWS Organizations에 적합성 팩을 배포하도록 위임된 관리자를 지정할 수 있다. (회원 계정일 수 있음)

![8-conformance-pack.png](images%2F8-conformance-pack.png)

- 템플릿의 첫 번재 섹션은 파라미터로 구성되어 있다.
- 규칙이 어떻게 작동할지 사용자가 정의할 수 있다.
- 실제로 규칙이 정의되며 세 개가 정의되어 있다.
  - 하나는 왼쪽 아래, 두 개는 오른쪽에 있다.
- 각 Config 규칙은 속성이 몇 가지 있다.
- AWS가 만든 규칙이라면 소유자는 AWS가 되고 직접 만든 규칙을 배포한다면 Custom이 된다.
- 이 경우에는 람다 함수도 함께 배포해야 한다.

![9-conformance-pack-cicd.png](images%2F9-conformance-pack-cicd.png)

- CI/CD를 이용해 Organization 전반에 이런 팩을 자동으로 배포하는 방법도 있다.
- 개발자가 CodeCommit에 모든 적합성 팩을 등록하면 CodeBuild가 해당 팩을 빌드하고 배포하는 것을 관리해 준다.
  - 관리형 규칙, 사용자 생성 규칙 모두 포함된다.
- 규칙 미준수 대상으로 자동으로 수정을 적용하는 데 필요한 SSM 자동화도 모두 포함된다.
- CloudFormation을 활용하면 모든 것을 여러 계정에 배포할 수 있다.

#### Organizational Rules

- AWS Organization 내의 모든 계정을 관리할 수 있는 AWS Config 규칙이다.

![10-organizational-rules.png](images%2F10-organizational-rules.png)

- 관리 계정이나 위임된 관리자에서 규칙을 정의하고 그 다음 모든 사용자 계정에 이 규칙이 배포된다.

#### Organizational Rules vs Conformance Packs

![11-organizational-rules-conformance-packs.png](images%2F11-organizational-rules-conformance-packs.png)

- Organizational 규칙의 범위는 Organization에 한정되지만 Conformance 팩은 여러 계정이나 Organization에도 적용할 수 있다.
- Organizational 규칙은 Organization 슈준에서 정의되고, 하나의 규칙이 있으며 한 번에 하나의 규칙을 배포한다.
- Conformance 팩은 한 번에 여러 규칙에 접근할 수 있다.
  - 여러 규칙과 여러 람다 함수와, 여러 수정 작업을 배포할 수 있고 이들의 준수 여부는 계정에서 확인할 수 있다.
- 첫 번째 규칙은 Organization 수준에 있고 두 번째 규칙은 각 계정 수준에 있으며, 각 준수 여부를 확인할 수 있다.

---

### AWS Organizations

![12-aws-organization.png](images%2F12-aws-organization.png)

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

![13-organization-account-access-role.png](images%2F13-organization-account-access-role.png)

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

![14-organizational-units.png](images%2F14-organizational-units.png)

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

![15-moving-accounts.png](images%2F15-moving-accounts.png)

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

![16-scp-hierarchy.png](images%2F16-scp-hierarchy.png)

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

![17-scp-blocklist.png](images%2F17-scp-blocklist.png)

- 명시적으로 모든 것을 허용하지만 명시적으로 DynamoDB를 차단하고 있다.

![18-scp-allowlist.png](images%2F18-scp-allowlist.png)

- EC2와 CloudWatch에 대해서만 접근을 허용한다.
- SCP 전략에 대한 자세한 내용은 [여기](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_examples.html)에서 확인할 수 있다.

#### IAM Policy Evaluation Logic

![19-iam-policy-evaluation-logic.png](images%2F19-iam-policy-evaluation-logic.png)

- IAM 정책을 평가할 때, 여러 단계마다 평가할 수 있다.
- 모든 정책을 평가하고 마지막으로 특정 IAM 작업을 허용하거나 거부한다.
- IAM 정책 평가 로직에 대한 자세한 내용은 [여기](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)에서 확인할 수 있다.

#### SCP 사용 예시

- **예시 1**
- IAM SCP 정책을 사용하여 태그를 제한하는데 사용될 수 있다.

![20-restricting-tags-with-iam-policy.png](images%2F20-restricting-tags-with-iam-policy.png)

- `aws:TagKeys` Condition 키를 사용하면 된다.
  - IAM 정책의 태그 키를 기준으로 리소스에 연결된 태그 키의 유효성을 확인한다.
- 예시를 보면 "Env" 및 "CostCenter" 태그가 있는 경우에만 IAM 사용자가 EBS 볼륨을 만들 수 있도록 허용하고 있다.
- `ForAllValues`는 모든 키를 포함하고 있는 경우에만 허용되며, `ForAnyValue`는 여러 키 중에 하나만 일치하더라도 허용된다.

- **예시 2**
- `aws:RequestRegion` Condition 키를 사용하여 전체 리전을 거부하는 SCP를 생성할 수 있다.

![21-deny-region-aws-requestregion.png](images%2F21-deny-region-aws-requestregion.png)

- 모든 것이 거부되는 리전은 `eu-central-1`, `eu-west-1`이 될 수 있고, `ec2:*`, `rds:*`, `dynamodb:*`와 같이 다양한 AWS 서비스를 지정할 수 있다.
- 특정 지역이나 특정 지역으로부터 멤버 계정을 제한하는 데 사용될 수 있다.

- **예시 3**
- 적절한 태그가 없는 리소스 생성을 제한하기 위해 SCP를 사용할 수 있다.

![22-restrict-creating-resources-without-appropriate-tags.png](images%2F22-restrict-creating-resources-without-appropriate-tags.png)

- 영향을 받는 구성원 계정의 IAM 사용자/역할이 특정 태그를 가지고 있지 않은 경우 리소스를 만들지 못하도록 제한할 수 있다.
- 예시에서는 EC2 인스턴스에 "Project" 및 "CostCenter" 태그가 없는 경우 리소스 시작을 제한하고 있다.

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

![23-account-factory.png](images%2F23-account-factory.png)

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

![24-detect-remediate-policy-violations.png](images%2F24-detect-remediate-policy-violations.png)

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

#### Landing Zone

- AWS 모범 사례를 기반으로 자동으로 프로비저닝되고 한전하며 규정을 준수하는 다중 계정 환경이다.
- 랜딩 존은 아래의 항목들로 구성된다.
  - AWS Organization: 다중 계정 구저 생성 및 관리
  - Account Factory: 규성 및 정책을 준수하도록 새 계정을 쉽게 구성한다.
  - Organizational Units: 목적에 따라 계정을 그룹화하고 분류한다.
  - Service Control Policy (SCP): 세분화된 권한 및 제한을 시행한다.
  - IAM ID Center: 계정 및 서비스에 대한 사용자 액세스를 중앙에서 관리한다.
  - Guardrails: 보안, 규정 준수, 모범 사례를 시행하기 위한 규칙 및 정책이다.
  - AWS Config: 리소스의 Guardrail 준수 여부를 모니터링하고 평가한다.

![25-landing-zone.png](images%2F25-landing-zone.png)

- "Management 계정"으로 구성된 Organization이 있고, "다른 OU"가 있고 "Security OU" 내에 "Log Archive 계정"과 "Audit 계정"이 있다.
- Account Factory 시설은 Control Tower 내에서 새로운 계정을 생성하고 제대로 구성되었는지 확인한다.
- IAM ID 센터는 한 번의 로그인으로 모든 계정에 안전하게 로그인할 수 있게 한다.
- 예방 가드레일은 SCP를 Organization 수준에 적용할 수 있고 탐지형 가드레일은 Config를 사용해 Organization과 계정 내 모든 것의 준수 여부를 모니터링한다.

#### Account Factory Customization (AFC)

- Account Factory를 통해 생성된 신규 및 기존 계정의 리소스를 자동으로 사용자 정의한다.
- Custom Blueprint
  - 계저어에서 사용자 정의하려는 리소스와 구성을 정의하는 CloudFormation 템플릿이다.
  - 서비스 카탈로그 제품의 형태로 정의된다.
  - 모든 사용자 정의 블루프린트을 저장하는 허브 계정에 저장된다. (관리 계정을 사용하지 않는 것이 권장됨)
  - AWS 파트너가 생성한 사전 정의된 블루프린트도 사용 가능하다.
- **하나의 블루프린트만 계정에 배포할 수 있다.**

![26-account-factory-customization.png](images%2F26-account-factory-customization.png)

- 관리자 계정이 허브 계정을 사용하고 Serviec Catalog에서 Control Tower에서 만드는 새로운 계정에 배포할 블루프린트를 생성한다.
- 관리 계정에서 부여하는 IAM 역할을 생성한다.
  - Control Tower에서 새로운 계정을 만들 때마다 Service Catalog에서 CloudFormation 템플릿으로 블루프린트가 배포된다.

#### Account Factory

- EventBridge를 사용하여 생성된 새 계정에 대응할 수 있다.

![27-account-factory.png](images%2F27-account-factory.png)

- 예를 들어, SNS로 알림을 보내거나 람다 함수를 사용하여 코드를 보내는 등의 자동화 작업이 가능하다.

#### AWS Account to Control Tower

![28-aws-account-control-tower.png](images%2F28-aws-account-control-tower.png)

- AWS Organization와 관리자 계정이 있고 Organization에 새로운 계정을 추가하는 상황이다.
- OU를 정의하고 새로운 계정이 들어갈 대상 OU는 "Dev OU"다.
  - 하지만 일단 등록되지 않은 OU에 이동해야 하며, 저장 장소라고 볼 수 있다.
  - 그 다음 Organization으로 계정을 이동한다.
  - 계정은 Organization의 일부이며 OU에 소속된다.
- IAM 역할을 생성하는데 Control Tower가 이 계정을 관리할 수 있게 한다.
  - `AWSControlTowerExecution` 역할이라 한다.
- 그런 다음 이 계정에 Config Conformance 팩을 배포할 수 있다.
- 계정은 Organization에서 어떻게 동작해야 하는지 규정을 준수해야 한다.
- Conformance 팩의 결과를 평가하면 모든 규칙과 규정과 수정 작업이 이 새로운 계정에 적용된다.
- 모두 완료되면 이 계정을 "Dev OU"로 옮길 수 있다.

#### Customizations for AWS Control Tower (CfCT)

- AWS에서 생성된 GitOps 스타일의 사용자 정의 프레임워크다.
- 사용자 지정 CloudFormation 템플릿과 SCP를 사용하여 랜딩 존에 사용자 지정을 추가하는 데 도움이 된다.
- Account Factory를 사용하여 생성된 새 AWS 계정에 리소스를 자동으로 배포한다.
- 참고로 CfCT는 AFC(Account Factory Customization, 블루프린트)와 다르다.

![29-customizations-control-tower.png](images%2F29-customizations-control-tower.png)

- 모든 계정에서 일종의 배포를 관리하게 된다.
- 모든 Control Tower 계정이 같은 SCP를 가지게 하여, 기본적으로 템플릿이 동일하도록 한다.
- 계정 블루프린트와 기능 측면에서 매우 유사하지만 다른 점이 있다.
  - 계정 블루프린트는 하나의 CloudFormation 템플릿만 전송할 수 있다.
  - 여기서는 여러 개의 CloudFormation을 전송할 수 있으며, SCP도 동일하고, 모두 파이프라인에 통합된다.

![30-customizations-aws-control-tower.png](images%2F30-customizations-aws-control-tower.png)

- 모든 CloudFormation 템플릿과 SCP는 S3나 CodeCommit에 배포될 수 있다.
  - 그런 다음 CodeBuild로 파이프라인을 시작한다.
  - 이 부분을 마무리하면서 Step Functions를 호출한다.
  - Step Functions를 통해 CloudFormation 템플릿이 CloudFormation 내 스택 세트의 일부로 모든 관리 계정에 배포된다.
- SCP도 마찬가지로 코드를 통해 관리되기 때문에 Organization에 자동으로 배포되어 OU 기반으로 모든 계정에 적용된다.
- 새로운 계정이 생기면 Control Tower에 이벤트가 발생한다.
  - 이 이벤트는 EventBridge에서 가로채서 일종의 자동화가 가능해진다.
  - 자동화로 SQS FIFO 큐에 메시지를 보내 이런 이벤트를 하나씩 처리할 람다 함수를 받아오고 람다 함수는 이 이벤트를 가지고 CodePipeline을 호출한다.
  - CodePipeline은 계정에 모든 템플릿과 모든 SCP를 다시 구축한다.
- AWS CfCT로 제공하는 것의 일부로 배포되는 아키텍처다.
- 모든 클라우드 템플릿과 SCP를 코드로 관리하는 전체 파이프라인을 제공한다.
- Control Tower에서 모든 계정에 배포되도록하며 새로운 계정이나 이미 존재하는 계정도 포함된다.

#### AWS Config Integration

- Control Tower는 AWS Config를 사용하여 "Detective Guardrails"를 구현한다.

![31-config-integration.png](images%2F31-config-integration.png)

- 내부적으로는 모든 혜왼 계정에 배포되는 Config 규칙이다.
- 준수하든 준수하지 않든 모든 구성 기록과 준수 기록은 로그 보관 계정으로 전송된다.
  - 여기에서 계정의 모든 클라이언트에 대한 정보를 볼 수 있다.
- Control Tower는 활성화된 리전에서 AWS Config를 자동으로 활성화한다.
- AWS 구성 기록 및 스냅샷이 중앙 집중식으로 중앙 계정으로 전달된다.
- Config를 모든 계정에 배포하기 위해 CloudFormation 스택 세트를 사용한다.
  - 그리고 Config Aggregator, CloudTrail과 중앙 로깅도 사용할 수 있다.

#### AWS Config Conformance Packs

- 적합성 팩을 Control Tower 전체에 배포할 수 있다.
- 적합성 팩은 많은 규칙과 수정을 YAML 템플릿으로 쉽게 배포하여 Organization에 배포하거나 개인 계정에 배포할 수 있도록 한다.

![32-config-conformance-packs.png](images%2F32-config-conformance-packs.png)

- 새로운 계정이 Control Tower에 생성되면 EventBridge가 SQS FIFO 큐에 메시지를 전송하고, 람다 함수가 호출된다.
- 람다 함수는 배포 리전이나 계정을 CloudFormation에 추가한다.
- 그러면 스택 세트에서 모든 적합성 팩을 배포(`us-east-1`, `us-west-2`)한다.

#### Account Factory for Terraform (AFT)

- 배포 파이프라인을 사용하여 테라폼을 통해 Control Tower에서 AWS 계정을 프로비저닝하고 사용자 정의하는데 도움이 된다.
- 계정 프로비저닝을 위한 AFT 워크플로를 트리거하는 계정 요청 테라폼 파일을 생성한다.
- 내장된 기능 옵션을 제공하며, 기본적으로는 비활성화되어 있다.
  - "AWS CloudTrail Data Events": Trail 생성 및 CloudTrail 데이터 이벤트를 활성화한다.
  - "AWS Enterprise Support Plan": Enterprise Support Plan을 활성화한다.
  - "Delete The AWS Default VPC": 모든 AWS 리전에서 기본 VPC를 삭제한다.
- AWS에서 유지 관리하는 테라폼 모듈이다.
- 테라폼 오픈 소스, 테라폼 엔터프라이즈 및 테라폼 클라우드와 함께 작동한다.

![33-account-factory-for-terraform.png](images%2F33-account-factory-for-terraform.png)

- 계정 요청 파일을 CodeCommit으로 푸시하려면 CodePipeline이 CodeBuild로 전송하고 CodeBuild가 이를 분석한 다음 테라폼을 사용하여 실제로 Control Tower의 계정과 리소스를 프로비저닝한다.
- 계정 요청 테라폼 파일에는 테라폼 파일을 배포할 모듈의 소스가 무엇인지, 계정에 필요한 파라미터가 무엇인지 정의된다.
  - 추가로 "로그 보관 계정", "검사 계정", "관리자 계정", "홈과 백엔드의 리전", "선택적 기능 플래그"에 대한 정보가 포함된다.

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

![34-iam-identity-center-login-flow.png](images%2F34-iam-identity-center-login-flow.png)

- 로그인 페이지로 가서 사용자 이름과 비밀번호를 입력하면, AWS IAM ID 센터로 이동된다.
  - AWS 계정이 활성화되어 있으며, 원하는 계정을 선택하면 관리 콘솔에 연결된다.
- 이런 방식으로 로그인하는 경우 특정 콘솔에 로그인하는 방법을 알 필요가 없다.
  - 단순히 IAM ID 센터 포털에 로그인하면 되고 SSO가 있기 때문에 비밀번호를 입력할 필요가 없다.
  - SSO로 계정과 비즈니스 애플리케이션에 접근할 수 있다.
- 실제로 여러 AWS 계정이 있는 경우 사용하는 것이 권장된다.

![35-iam-identity-center-1.png](images%2F35-iam-identity-center-1.png)

- 브라우저 인터페이스가 AWS IAM ID 센터의 로그인 페이지로 연결된다.
  - 사용자와 그룹을 관리하기 위해서 다른 사용자 스토어와 통합해야 하며, "Active Directory"가 될 수 있다.
  - IAM ID 센터를 사용할 수 있고, 내장 ID 스토어다.
  - 예를 들어, 사용자와 그룹을 정의할 수 있고 IAM이 사용된다.
- 그런 다음 Organizaiton에 ID 센터를 SSO와 통합한다.
  - 아니면, Windows EC2 인스턴스나 비즈니스 클라우드 애플리케이션, 사용자 정의 SAML2.0 활성화 애플리케이션에 통합할 수 있다.
- 한 번의 로그인으로 이 모든 것이 가능하다.
- 로그인만으로 모든 서비스에 접근할 수는 없으며 "권한 세트(Permission Sets)"를 정의해야 한다.
  - 어떤 사용자가 어디에 접근 가능한지 ID 센터에 정의한다.

![36-iam-identity-center-2.png](images%2F36-iam-identity-center-2.png)

- 권한, 사용자, 그룹, Orgamization이 있고 "관리 계정"에서 IAM ID 센터를 설정했다.
- "개발 OU"와 "운영 OU"가 있고 회사에는 두 명의 개발자인 "Bob"과 "Alice"가 있다.
- "Bob"과 "Alice"로 개발자 그룹을 생성한다.
  - "Bob"과 "Alice" 둘에게 개발을 위한 OU 전체 접근 계정이 필요하다.
  - 권한을 부여하기 위해서 권한 세트를 생성하고, 관리자 권한을 허용한다.
- 권한 세트를 특정 OU에 연관시킨다.
  - FullAccess 권한이 있는 권한 세트를 "개발 OU"에 연결하고 개발자 그룹에게 권한 세트를 할당한다.
  - 운영 OU에 대해서 ReadOnlyAccess가 있는 권한 세트를 생성하고 개발자 그룹에 권한 세트를 할당한다.
- IAM ID 센터에서 사용자를 그룹과 권한 세트 특정 계정 할당에 연결하는 방법이다.

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

![37-iam-identity-center-fine-grained-permissions.png](images%2F37-iam-identity-center-fine-grained-permissions.png)

- IAM ID 센터가 있고 사용자의 Organization과 통합되어 있다.
- 데이터베이스 관리자에 대한 권한 세트를 정의하며 IAM 정책 집합이다.
  - 예를 들어, 사용자가 개발 계정에서 RDS 및 Aurora에 접근할 수 있고 운영 계정의 RDS 및 Aurora에도 접근 가능하다.
  - 사용자의 해당 IAM 역할을 자동으로 생성한다.
  - 따라서 데이터베이스 관리자가 있으면 그룹이 될 수 있으며 사용자는 그룹에 포함되게 된다.
- 데이터베이스 관리자의 권한 세트를 지정한다.
  - 그리고 사용자가 IAM ID 센터를 통해 로그인할 때마다 개발 계정이나 운영 계정의 콘솔에 접근하면 해당 계정에 자동으로 IAM 역할을 부여한다.

#### External IdPs

- SAML 2.0: 외부 ID 제공자를 사용하는 방식이다.
  - 사용자는 기업 ID를 사용하여 AWS 액세스 포털에 로그인한다.
  - Okta, Azure AD, OneLogin을 지원한다.
  - SAML 2.0은 사용자 및 그룹에 대해 알아보기 위해 IdP에 쿼리하는 방법을 제공하지 않는다.
  - IAM ID 센터에서는 외부 IdP의 사용자 및 그룹과 동일한 사용자 및 그룹을 생성해야 한다.
- SCIM: 외부 IdP의 사용자 ID를 IAM ID 센터로 자동 프로비저닝한다. (동기화)
  - SCIM은 "System for Cross-domain Identity Management"의 약자다.
  - 외부 IdP의 지원을 받아야 한다.
  - SAML 2.0 사용을 완벽하게 보완한다.

![38-external-idps.png](images%2F38-external-idps.png)

- IAM ID 센터는 SAML 2.0을 사용해 "Active Directory"와 통합된다.
- 어떤 로그인 요청이 전달되면 일단 사용자와 그룹을 외부 IdP에서 정의한 것과 동일하게 생성해야 한다.
  - 수작업이 많이 들어가는 방식이지만 SCIM 프로토콜을 사용하면 간편하게 해결할 수 있다.
- 활성화하면 자동으로 사용자와 그룹이 IAM ID 센터에 생성되고 IAM ID 센터에서 수동으로 사용자를 생성할 필요가 없어진다.

#### Attribute-Based Access Control (ABAC)

- IAM ID 센터 ID 저장소에 저장된 사용자 속성을 기반으로 세분화된 권한을 제공한다.
- 사용자 석송은 IAM ID 센터 권한 집합 및 리소스 기반 정책에서 사용할 수 있다.
- 예를 들어, Cost Center, Title, Locale 등이 있다.
- 권한을 한 번 정의한 후 속성을 변경하여 AWS 액세스를 수정하는 등의 작업에 사용된다.
- 사용자 속성은 IdP에서 Key-Value 쌍으로 매핑된다.

![39-attribute-based-access-control.png](images%2F39-attribute-based-access-control.png)

- 세 개의 그룹이 있고 각각 다른 태그를 가지고 있다.
- AWS에는 다른 리소스 세트가 있으며 빨강, 파랑, 녹색으로 태그가 추가된다.
- ID 센터에서 올바른 권한 세트를 정의하여 빨간색 사용자는 빨간색 리소스, 녹색 사용자는 녹색 리소스, 파란색 사용자는 파란색 리소스에만 접근하도록 한다.
- 이러한 속성은 ID 제공자가 Key-Value 쌍으로 정의할 수 있으며 AWS에 매핑된다.

![40-attribute-based-access-control.png](images%2F40-attribute-based-access-control.png)

- EC2 인스턴스가 있고 "AccessProject-StarProject" 태그를 추가한다.
- IAM ID 센터에서 이 태그를 그룹 A에 할당하고, 그룹 B에는 적용하지 않는다.
- "인스턴스 시작"과 "인스턴스 중지"가 정의된 권한 세트의 조건을 보면 `ec2:ResourceTag/AccessProject` 태그가 지정되어 있으며, 사용자의 태그를 의미하낟.

#### Multi-Factor Authentication

- 인증 모드를 통해 MFA(Multi-Factor Authentication)를 지원한다.
  - Every Time They Sign-in(Always-on): 사용자가 로그인할 때마다 MFA를 요구한다.
  - Only When Their Sign-in Context Changes(Context-aware): 인증 컨텍스트가 변경될 때만 MFA를 요구한다. 예를 들어, 사용자 행동이 바뀌는 상황이다.

![41-multi-factor-authentication.png](images%2F41-multi-factor-authentication.png)

- ID 센터에서 MFA를 "Always-on"으로 설정하면 사용자는 MFA 디바이스가 필요하다.
  - 예를 들어, Google Authenticator가 있다.
  - 사용자 이름과 비밀번호로 로그인 후 MFA를 요구한다.
- "Authenticator"에서 MFA 코드를 입력하면 로그인을 완료할 수 있다.

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

#### Security Policy

- Policy Type:AWS WAF
  - AWS Organization의 모든 계정에 있는 모든 ALB에 WebACL 적용을 시행한다.
  - 규정을 준수하지 않으며 자동으로 교정되지 않는 리소스를 식별한다. 리소스에 WebACL를 적용하지 않고 각 계정에서 WebACL을 생성한다.
  - 비준수 리소스 자동 교정한다. WebACL을 기존 리소스에 자동으로 적용한다.
- Policy Type:Shield Advanced
  - AWS Organization의 모든 계정에 Shield Advanced 보호를 적용한다.
  - "규정 준수만 확인" 또는 "자동 해결 옵션"을 제공한다.
- Security Group Policy Type:Common Security Group
  - AWS Organization의 모든 계정에 있는 모든 EC2 인스턴스에 SG 적용을 시행한다.
- Security Group Policy Type: Auditing of Security Group Policy
  - AWS Organization 내 모든 계정의 SG 규칙을 확인하고 관리한다.
- Security Group Policy Type: Usage Audit Security Group Policy
  - 사용되지 않거나 중복된 SG를 모니터링하고 선택적으로 정리를 수행한다.
- Policy Type:Network Firewall
  - AWS Organization의 모든 계정에서 네트워크 방화벽을 중앙에서 관리한다.
  - Distributed: 각 VPC에서 방화벽 엔드포인트를 생성하고 유지한다.
  - Centralized: 중앙 집중식 VPC에서 방화벽 엔드포인트를 생성하고 유지 관리한다.
  - Import Existing Firewall: 리소스 세트를 사용하여 기존 방화벽을 가져온다.
- Policy Type:Route 53 Resolver DNS Firewall
  - Resolver DNS 방화벽 규칙 그룹과 AWS Organization의 모든 계정에 있는 VPC 간의 연결을 관리한다.

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

![44-amazon-guardduty.png](images%2F44-amazon-guardduty.png)

#### Multi-Account Strategy

- GuardDuty에서 여러 계정을 관리할 수 있다.
- 회원 계정을 관리자 계정과 연결한다.
  - AWS Organization을 통해 연결할 수 있다.
  - GuardDuty를 통해 초대를 보낼 수 있다.
- 관리자 계정은 아래의 항목들을 수행할 수 있다.
  - 회원 계정 추가 및 제거를 수행할 수 있다.
  - 연결된 회원 계정 내에서 GuardDuty를 관리한다.
  - 결과, 억제 규칙, 신뢰할 수 있는 IP 목록, 위협 목록을 관리하낟.
- AWS Organization에서는 회원 계정을 GuardDuty에 대한 조직의 위임된 관리자로 지정할 수 있다.

![45-multi-account-strategy.png](images%2F45-multi-account-strategy.png)

- Organization에 두 개의 OU와 회원 계정이 있고, GuardDuty의 관리자 계정을 만들고 지정하려한다.
- 다른 모든 회원 계정에서 GuardDuty를 관리한다.

#### Findings Automated Response

- 조사 결과는 AWS 계정에서 발생하는 악의적인 이벤트에 대한 잠재적인 보안문제다.
- EventBridge를 사용하여 GuardDuty 결과에서 밝혀진 보안 문제에 대한 대응을 자동화한다.
- SNS로 알림을 보내거나 이메일을 전송하거나 람다 함수를 트리거할 수 있다.
- 이벤트는 관리자 계정과 해당 이벤트가 시작된 회원 계정 모드에 게시된다.

![46-finding-automated-response.png](images%2F46-finding-automated-response.png)

#### Findings

- GuardDuty는 CloudTrail Logs(관리 이벤트, 데이터 이벤트), VPC Flow Logs 또는 EKS 로그에서 직접 독립적인 데이터 스트림을 가져온다.
- 각 결과의 심각도 값은 0.1부터 8+까지의 값을 가지고 있고 "높음", "중간", "낮음" 세 단계로 분류된다.
- 명명 방법이 있다.
  - `ThreatPurpose`: 위협의 주요 목적이다. (예: `Backdoor`, `CryptoCurrency`)
  - `ResourceTypeAffected`: 대상인 AWS 리소스를 의미한다. (예: S3, EC2)
  - `ThreatFamilyName`: 잠재적인 악성 활동을 설명한다. (예: `NetworkPortUnusual`)
  - `DetectionMechanism`: 결과를 감지하는 GuardDuty 메서드다. (예: TCP, UDP)
  - `Artifact`: 악설 활동에 사용되는 리소스를 설명한다. (예: DNS)
- GuardDuty에서 샘플 결과를 생성하여 자동화를 테스트할 수 있다.

#### Findings Types

- EC2 Finding Types: 
  - `UnauthorizedAccess:EC2/SSHBruteForce`: 누군가 22번 포트로 침입하여 강제로 접근하는 것을 탐지한다.
  - `CryptoCurrency:EC2/BitcoinToo.B!DNS`: EC2에 대한 암호화폐 공격을 탐지한다.
- IAM Finding Types: 
  - `Stealth:IAMUser/CloudTrailLoggingDisabled`: CloudTrail 로깅이 비활성화되는 것을 탐지한다. 
  - `Policy:IAMUser/RootCredentialUsage`: 사용해서는 안되는 루트 자격을 사용하는 사용자를 탐지한다.
- Kubernetes Audit Logs Finding Types: 
  - `CredentialAccess:Kubernetes/MaliciousIPCaller`: 악성 IP에서 K8S의 자격 증명으로 접근하는 것을 탐지한다.
- Malware Protection Finding Types: 
  - `Execution:EC2/SuspiciousFile`: EC2의 맬웨어 보호를 위한 의심스러운 파일을 탐지한다.
  - `Execution:EC2/SuspiciousFile`: ECS의 맬웨어 보호를 위한 의심스러운 파일을 탐지한다.
- RDS Protection Finding Types: 
  - `CredentialAccess:RDS/AnomalousBehavior.SuccessfulLogin`: 사용자가 변칙적인 방식으로 계정의 RDS 데이터베이스에 성공적으로 로그인하는 것을 탐지한다.
- S3 Finding Types: 
  - `Policy:S3/AccountBlockPublicAccessDisabled`: S3 공개 액세스가 차단되었는지 확인한다.
  - `PenTest:S3/KaliLinux`: Kali Linux 머신에서 S3 API가 간접적으로 호출되는 것을 탐지한다.

#### Architecture

![47-architecture.png](images%2F47-architecture.png)

- VPC에서 EC2 인스턴스가 있고 GuardDuty는 VPC Flow Logs를 분석하고 있다.
  - VPC Flow Logs는 GuardDuty에 의해 관리되며 GuardDuty 결과를 보면 VPC Flow Logs와 유사하다.
- 누군가 `SSHBruteForce` 유형의 공격을 EC2 인스턴스에 수행한다.
  - 공격이 탐지되면 EventBridge가 이벤트를 생성한다.
  - 이메일로 알림을 받기 위해 SNS에 연결할 수 있다.
- 람다 함수를 사용하여 두 가지 일을 수행할 수 있다.
  - WAF가 로드 밸런서 등과 연결된 경우에 의심스러운 요청을 차단하기 위한 WebACL을 생성할 수 있다.
  - EC2 인스턴스의 경우 NACL을 자동으로 업데이트하여 악성 IP를 차단한다.

![48-architecture.png](images%2F48-architecture.png)

- AWS Network Firewall 기능을 사용하기 때문에 "방화벽 서브넷"이 있다.
- EC2 인스턴스를 사용하면 VPC Flow Logs가 GuardDuty에서 의심스러운 행동을 감지한다.
  - 그 결과는 우리의 EC2에 백도가 있다는 것을 의미한다.
- EventBridge는 Step Function 워크플로를 트리거할 수 있다.
  - 여기에는 여러 람다 함수가 포함될 수 있다.
  - 공격자의 IP가 악성 IP 데이터베이스에 있는지 확인하고 아니라면 트래픽을 차단한다.
  - 트래픽을 차단함으로써 람다 함수가 NACL API를 호출하여 네트워크 방화벽 엔드포엔트에 적용되는 방화벽 정책에 IP를 추가할 수 있다.
  - 그 결과를 EC2 인스턴스 트래픽이 방화벽 서브넷을 통과하는 동안 EC2 인스턴스는 연결할 수 없다.
- 결과적으로 의심스러운 호스트에 연결하지 못하고 완전히 차단된다.
- "블록 트래픽" 람다 함수는 차단 성공, 실패에 대한 주제를 SNS로 전송한다.

#### Trusted & Threat IP List

- Public IP에 대해서만 작동한다.
- Trusted IP List
  - 신뢰하는 IP 주소 및 CIDR 범위 목록을 제공한다.
  - GuardDuty는 신뢰할 수 있는 목록에 대한 결과를 생성하지 않는다.
- Threat IP List
  - 알려진 악성 IP 주소 및 CIDR 범위 목록을 제공한다.
  - GuardDuty는 이러한 위협 목록을 기반으로 결과를 생성한다.
  - 타사 위협 인텔리전스를 통해 제공되거나 사용자 정의로 생성될 수 있다.
- 다중 계정 GuardDuty 설정에서는 GuardDuty 관리자 계정만 해당 목록을 관리할 수 있다.

![49-trusted-threat-ip-list.png](images%2F49-trusted-threat-ip-list.png)

#### CloudFormation Integration

- CloudFormation 템플릿을 사용하여 GuardDuty를 활성화할 수 있다.
- GuardDuty가 이미 활성화된 경우 CloudFormation 스택 배포가 실패한다.
- GuardDuty가 아직 활성화되지 않은 경우 CloudFormation Custom Resource(람다)를 사용하여 조건부로 활성화한다.
- 스택 세트를 사용하여 조직 전체에 배포할 수 있다.

![50-cloudformation-integration.png](images%2F50-cloudformation-integration.png)

- CloudFormation 스택은 사용자 리소스를 생성하고 람다 함수는 코드를 실행하여 GuardDuty가 활성화되지 않았을 때만 활성화한다.

---

### Amazon Detective

- GuardDuty, Macie, Security Hub를 사용하여 잠재적인 보안 문제 또는 발견을 식별한다.
- 보안 결과를 통해 근본 원인을 파악하고 조치를 취하기 위해서는 심층적인 분석이 필요한 경우가 있다. - 이는 복잡한 프로세스다.
- Amazon Detector는 보안 문제나 의심스러운 활동의 근본 원인을 분석, 조사하고 신속하게 파악한다. (ML 및 Graphs 사용)
- VPC Flow Logs, CloudTrail, GuardDuty에서 이벤트를 자동으로 수집 및 처리하고 통합 뷰를 생성한다.
- 세부 정보와 상황에 따라 시각화를 수행하여 근본적인 원인을 파악한다.

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

![51-amazon-inspector.png](images%2F51-amazon-inspector.png)

#### 평가 항목

- **"EC2 인스턴스", "컨테이너 이미지", "Lambda Function"에 대해서만 분석을 제공**한다.
- 필요한 경우에만 인프라에 대한 지속적인 검색을 제공한다.
- CVE 데이터베이스를 통해 패키지 취약점을 분석한다.
  - CVE 데이터베이스가 업데이트 되는 경우 Inspector는 자동으로 다시 모든 인프라를 검사한다.
- EC2 인스턴스의 네트워크 도달 가능성을 조사한다.
- 위험 점수는 우선 순위 지정을 위한 모든 취약성과 연관되어 있다.

#### Systems Manager

- Inspector에서 EC2 인스턴스를 설정하려면 다음의 항목들이 필요하다.
  - SSM 에이전트가 실행 중이어야 한다.
  - SSM 관리형 인스턴스이어야 한다. (IAM 역할 또는 기본 호스트 관리 구성)
  - Systems Manager 엔드포인트로 아웃바운드 443을 허용해야 한다.

![52-inspector-systems-manager.png](images%2F52-inspector-systems-manager.png)

---

### 가용지역간 EC2 Instance Migration

![53-instance-migration-between-az.png](images%2F53-instance-migration-between-az.png)

- 인스턴스를 하나의 AZ에서 다른 AZ로 마이그레이션하고 싶다면 AMI를 이용하는 방법이 있다.
- 이미지를 예시로 "us-east-1a"에서 "us-east-1b"로 옮길 수 있다.
- 먼저 AMI를 EC2 인스턴스에서 가져오고 AMI를 다른 AZ로 새 EC 인스턴스로 복원한다.
- AMI가 원본에서 옮겨졌기 때문에 기본값으로 같은 데이터와 파일 시스템 애플리케이션을 가지게 된다.

#### Cross-Account AMI 공유

![54-cross-account-ami-sharing.png](images%2F54-cross-account-ami-sharing.png)

- AMI를 다른 AWS 계정과 공유할 수 있다.
- AMI 공유는 AMI 소유권에 영향을 주지 않는다.
- 암호화되지 않은 볼륨과 고객 관리형 키로 암호화된 볼륨이 있는 AMI만 공유할 수 있다.
- 암호화된 볼륨과 AMI를 공유하는 경우 이를 암호화하는 데 사용되는 고객 관리형 키도 공유해야 한다.

#### KMS로 암호화된 AMI 공유

![55-ami-sharing-with-kms-encryption.png](images%2F55-ami-sharing-with-kms-encryption.png)

- AMI가 CMK로 암호화되어 있다.
- AMI를 "Account B"와 공유하기 위해서는 KMS 키도 공유해야 한다.
- 대상 계정에 복호화와 재암호화 등을 위하여 키에 대한 권한을 제공해야 한다.

#### Cross-Account AMI 복제

![56-ami-sharing-with-copy.png](images%2F56-ami-sharing-with-copy.png)

- 사용자의 계정과 공유된 AMI를 복사하면 사용자의 계정에 있는 AMI의 소유자가 된다.
- 소스 AMI의 소유자는 AMI(EBS 스냅샷)를 지원하는 스토리지에 대한 읽기 권한을 부여해야 한다.
- 공유 AMI에 암호화된 스냅샷이 있는 경우 사용자는 키를 공유해야 한다.
- 복사하는 동안 자체 CMK로 AMI를 암호화할 수 있다.

![57-ami-copy-kms-encryption.png](images%2F57-ami-copy-kms-encryption.png)

- 공유된 AMI를 암호화하면서 복사를 할 수 있다.
- 기본 EBS 스냅샷을 공유하고 KMS 키에 대상 계정에 권한을 부여한다.
- 대상 계정은 복사 명령을 실행할 수 있다.
  - CMK-A를 이용해 복호화하여 EBS 스냅샷을 다시 암호화하고 CMK-B와 그 고유 계정으로 다시 암호화한다.
- 공유된 AMI가 있다면 AMI를 복사할 수 있다. 그럼 나중에 다른 것을 공유하지 않고도 자신만의 AMI를 갖게 된다.

---

### Trusted Advisor

- 아무 것도 설치할 필요가 없이, **높은 수준의 AWS 계정 평가를 제공**한다.
- 사용자의 AWS 계정을 분석하고 5가지 카테고리에 대한 추천을 제공한다.
  - 비용 최적화(Cost Optimization)
  - 성능(Performance)
  - 보안(Security)
  - 내결함성(Fault Tolerance)
  - 서비스 한도(Service Limit)

![58-trusted-advisor.png](images%2F58-trusted-advisor.png)

#### Support Plan

- 7 중요 검사(Basic & Developer 지원 계획)
  - S3 버킷 권한
  - 보안 그룹 - 특정 포트 제한 없음
  - IAM 사용 (IAM 사용자 최소 1명)
  - 루트 계정의 MFA
  - EBS 공개 스냅샷
  - RDS 공개 스냅샷
  - 서비스 한도
- 모든 검사(Business & Enterprise 지원 계획)
  - 5개의 카테고리에 대한 전체 확인이 가능하다.
  - 한도에 도달하면 CloudWatch 경보를 설정하는 기능이 있다.
  - AWS Support API를 사용한 프로그램이 방식의 액세스를 제공한다.

#### Monitoring

![59-trusted-advisor-monitoring.png](images%2F59-trusted-advisor-monitoring.png)

- EC2 인스턴스의 사용률이 낮을 때마다 발생하는 이벤트가 있다.
  - 14일 평균 CPU 사용률이 매우 낮다는 의미다.
  - 즉, EC2 인스턴스가 아무것도 사용하지 않고 아무 목적 없이 비용을 청구하고 있을 가능성이 있다.
- 이러한 경우에는 물론 EventBridge에서 이벤트가 발생하고 SNS 토픽으로 메시지를 보낼 수 있다.
  - 또는 자동화를 위해 람다 함수를 트리거할 수 있다.

![60-service-quotas.png](images%2F60-service-quotas.png)

- Trusted Advisor를 사용하여 서비스 할당량을 모니터링할 수 있다.
- 약 50개 이상의 서비스 할당량이 Trusted Advisor에서 지원되지만 모든 할당량이 지원되지는 않는다.
- EC2 온디맨드 인스턴스 제한이 있고 다양한 알림을 받을 수 있다.
  - 예를 들어, 제한의 80%, 100%에 도달했다는 알림을 EventBridge에 메시지 전송을 트리거할 수 있다. 

---

### 참고한 강의

- [DevOps Engineer Professional](https://www.udemy.com/course/aws-certified-devops-engineer-professional-korean)
- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)