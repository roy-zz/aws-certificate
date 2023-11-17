# Account Management

이번 장에서는 **SysOps Administrator**를 준비하며 **계정 관리**에 대해서 알아보도록 한다.

---

### AWS Health Dashboard

- 모든 지역, 모든 서비스 상태를 표시한다.
- 매일의 기록 정보를 표시한다.
- 구독할 수 있는 RSS 피드가 있다.
- 이전에는 "AWS 서비스 Heath Dashboard"라고 불렀다.

![1-health-dashboard-service-history.png](images%2F1-health-dashboard-service-history.png)

- 이전에는 "AWS Personal Health Dashboard(PHD)"로 불렸다.
- "AWS Account Health Dashboard"는 AWS의 사용자에게 영향을 줄 수 있는 이벤트가 발생할 때 경고 및 해결 지침을 제공한다.
- 서비스 상태 대시보드에는 AWS 서비스의 일반 상태가 표시되지만, 계정 상태 대시보드는 AWS 리소스의 기반이 되는 AWS 서비스의 성능과 가용성에 대한 맞춤형 보기를 제공한다.
- 대시보드는 진행 중인 이벤트를 관리하는 데 도움이 되는 관련 정보를 시기적절하게 표시하고 예정된 활동을 계획하는 데 도움이 되는 사전 알림을 제공한다.

![2-health-dashboard-your-account-1.png](images%2F2-health-dashboard-your-account-1.png)

- 글로벌 서비스로 전체 AWS 조직에서 데이터를 집계할 수 있다.
- AWS 중단이 사용자와 사용자의 AWS 리소스에 어떤 직접적인 영향을 미치는지 보여준다.
- 경고, 해결, 사전 예방적, 예정된 활동

![3-health-dashboard-your-account-2.png](images%2F3-health-dashboard-your-account-2.png)

#### Health Event Notification

![4-health-event-notification.png](images%2F4-health-event-notification.png)

- EventBridge를 사용하여 AWS 계정의 AWS 상태 이벤트 변경 사항에 대응할 수 있다.
- 예를 들어, AWS 계정의 EC2 인스턴스 업데이트가 예약되면 이벤트 알림을 받는다.
- 이는 계정 이벤트(계정에 영향을 받는 리소스) 및 공개 이벤트(서비스의 지역적 가용성)에 대해 가능하다.
- 사용 사례: 알림 보내기, 이벤트 정보 캡처, 시정 조치 수행 등..

![5-health-dashboard-example.png](images%2F5-health-dashboard-example.png)

---

### AWS Organization

- 글로벌 서비스다.
- 여러 AWS 계정을 관리할 수 있다.
- 기본 계정은 관리(Management) 계정이다.
- 이외의 계정은 회원(Member) 계정이다.
- 회원 계정은 하나의 조직에만 속할 수 있다.
- 모든 계정에 대한 통합 결제를 지원하며 단일 결제 수단으로 결제한다.
- 집계된 사용량으로 인한 가격 혜택을 받을 수 있다. (EC2, S3 등)
- **계정 간 공유 예약 인스턴스 및 Savings Plans 할인**을 받을 수 있다.
- API를 사용하여 AWS 계정 생성을 자동화할 수 있다.

![6-organizations.png](images%2F6-organizations.png)

- 여러 기준으로 Organizational Unit을 나눌 수 있다.
- 비즈니스 성향에 맞게 Sales OU, Retail OU, Finance OU로 나누고 해당 OU에 회원 계정을 포함시킬 수 있다.
- 개발 환경 라이프사이클에 맞게 운영 OU, 개발 OU, 테스트 OU로 나누고 해당 OU에 회원 계정을 포함시킬 수 있다.
- 프로젝트의 성향에 맞게 Project1 OU, Project2 OU, Project3 OU로 나누고 해당 OU에 회원 계정을 포함시킬 수 있다. 

![7-organizational-units.png](images%2F7-organizational-units.png)

#### 장점

- 다중 계정 덕분에 다중 VPC를 가진 하나의 계정보다 더 나은 보안을 가질 수 있다.
- 청구 목적으로 태그 표준을 지정할 수 있다.
- 모든 계정에서 CloudTrail을 활성화하고 중앙 S3 계정으로 로그를 보낸다.
- CloudWatch Logs를 중앙 로깅 계정으로 보낸다.
- 관리 목적을 위한 교차 계정 역할을 설정할 수 있다.

#### 보안 - Service Control Policy (SCP)

- 사용자 및 역할을 제한하기 위해 OU 또는 계정에 적용되는 IAM 정책을 사용할 수 있다.
- 마스터 계정에는 적용되지 않는다. (전체 관리자 권한)
- IAM과 같이 기본적으로 아무것도 허용하지 않기 때문에 명시적인 허용이 있어야 한다.

#### SCP Hierarchy

![8-scp-hierarchy.png](images%2F8-scp-hierarchy.png)

- **Management Account**
  - 모든 작업을 할 수 있다.
  - SCP가 적용되지 않는다.
- **Account A**
  - 모든 작업을 할 수 있다.
  - Redshift 액세스는 제외된다. (OU에서 명시적으로 거부)
- **Account B**
  - 모든 작업을 할 수 있다.
  - Redshift 액세스는 제외된다. (Prod OU의 명시적 거부)
  - Lambda 액세스는 제외된다. (HR OU의 명시적 거부)
- **Account C**
  - 모든 작업을 할 수 있다.
  - Redshift 액세스는 제외된다. (Prod OU의 명시적 거부)

#### Blocklist and Allowlist Strategy

![9-blocklist-allowlist-strategy.png](images%2F9-blocklist-allowlist-strategy.png)

- 좌측 이미지의 경우 모든 액션을 허용하지만 DynamoDB에 대한 접근은 제한된다.
- 우측 이미지의 경우 EC2 인스턴스와 CloudWatch에 대한 모든 권한을 가진다.

#### Reserved Instance

- 청구 목적을 위해 AWS Organizations의 **통합 청구 기능은 조직의 모든 계정을 하나의 계정**으로 처리한다.
- 이는 조직의 **모든 계정이 다른 계정에서 구매한 예약 인스턴스의 시간당 비용 혜택을 받을 수 있음**을 의미한다.
- 조직의 지급 계정(마스터 계정)은 지급 계정을 포함하여 해당 조직의 모든 계정에 대해 예약 인스턴스(RI) 할인 및 Savings Plans 할인 공유를 해제할 수 있다.
- 공유가 꺼진 계정 간에는 RI 및 Savings Plans 할인이 공유되지 않음을 의미한다.
- RI 또는 Savings Plans 할인을 계정과 공유하려면 두 계정 모두 공유가 켜져 있어야 한다.

#### IAM Policy

- 리소스 기반 정책에서 `aws:PrincipalOrgID` 조건 키를 사용하여 AWS 조직의 계정에서 IAM 보안 주체에 대한 액세스를 제한한다.

![10-organization-iam-policy.png](images%2F10-organization-iam-policy.png)

#### Tag Policy

- AWS 조직의 리소스 전반에 걸쳐 태그를 표준화하는 데 도움이 된다.
- 일관성 있는 태그를 보장하고, 태그가 지정된 리소스를 감사하고, 적절한 리소스 분류를 유지하는 등의 작업을 수행한다.
- 태그 키와 허용되는 값을 정의한다.
- AWS 비용 할당 태그 및 속성 기반 액세스 제어에 도움이 된다.
- 지정된 서비스 및 리소스에 대한 비준수 태깅 작업을 방지한다. 태그가 없는 리소스에는 영향을 미치지 않는다.
- 태그가 지정되거나 규정을 준수하지 않는 모든 리소스를 나열하는 보고서를 생성할 수 있다.
- CloudWatch 이벤트를 사용하여 비준수 태그를 모니터링할 수 있다.

![11-organization-tag-policy.png](images%2F11-organization-tag-policy.png)

---

### AWS Control Tower

- 모범 사례를 기반으로 안전하고 규정을 준수하는 다중 계정 AWS 환경을 쉽게 설정하고 관리하는 방법이다.
- 아래와 같은 이익을 얻을 수 있다.
  - 몇 번의 클릭만으로 환경 설정을 자동화한다.
  - 가드레일을 사용하여 지속적인 정책 관리를 자동화한다.
  - 정책 위반을 감지하고 해결한다.
  - 대화형 대시보드를 통해 규정 준수 모니터링을 실시한다.
- AWS Control Tower는 AWS Organizations 위에서 실행된다.
  - 자동으로 AWS Organizations를 설정하여 계정을 구성하고 SCP(서비스 제어 정책)를 구현한다.

---

### AWS Service Catalog

- AWS를 처음 사용하는 사용자에게는 옵션이 너무 많으며 조직의 나머지 부분과 호환되지 않거나 일치하지 않는 스택을 생성할 수 있다.
- 관리자가 사전에 정의한 승인된 제품 세트를 실행할 수 있는 빠른 셀프 서비스 포털을 제공한다.
- 가상 머신, 데이터베이스, 스토리지 옵션 등을 제공한다.

![12-service-catalog-diagram.png](images%2F12-service-catalog-diagram.png)

- 관리자와 사용자가 있고 관리자는 제품을 생성한다. 여기서 제품은 CloudFormation 템플릿이다.
- 그런 다음 CloudFormation 템플릿의 집합인 포트폴리오를 생성한다.
- 누가 포트폴리오에 접근할 수 있는지에 대한 IAM 사용 권한 같은 컨트롤을 정의할 수 있다.
- 관리자가 설정한 IAM 사용 권한으로 인해 사용자들은 사용할 수 있는 제품 리스트를 가지고 있다.
- 사용자가 CloudFormation 목록에서 선택하고 포트폴리오를 구성해 안전하게 인프라를 구축할 수 있다.
- 사용자는 AWS가 아닌 Service Catalog에서만 접근할 수 있고 CloudFormation에 직접 접근하지 않고도 풀스택을 실행할 수 있다.

#### Sharing Catalog

- 개별 AWS 계정 또는 AWS Organizations와 포트폴리오를 공유할 수 있다.
- 아래와 같은 공유 옵션이 있다.
  - 포트폴리오 참조를 공유한 후 공유 포트폴리오를 수신자 계정으로 가져온다. (원본과 동기화를 유지)
  - 포트폴리오 사본을 수신자 계정에 배포한다. (모든 업데이트를 다시 배포해야 함)
- 가져온 포트폴리오의 제품을 로컬 포트폴리오에 추가하는 기능을 제공한다.

![13-service-catalog-sharing-catalog.png](images%2F13-service-catalog-sharing-catalog.png)

#### TagOptions Library

- 프로비저닝된 제품의 태그를 쉽게 관리할 수 있다.
- 태그 옵션
  - AWS Service Catalog에서 관리되는 Key-Value 쌍
  - AWS 태그 생성에 사용
- 포트폴리오 및 제품과 연결할 수 있다.
- 적절한 리소스 태깅, 허용된 태그 정의 등에 사용된다.
- 다른 AWS 계정 및 AWS Organizations와 공유할 수 있다.

![14-service-catalog-tagoptions-library.png](images%2F14-service-catalog-tagoptions-library.png)

---

### AWS Billing Alarms

- **결제 데이터 지표는 CloudWatch us-east-1에 저장**된다.
- 결제 데이터는 전세계 전체 AWS 비용에 대한 것이다.
- 프로젝트 비용이 아니라 실제 발생한 비용이다.

---

### Cost Explorer

- 시간 경과에 따른 AWS 비용 및 사용량을 시각화, 이해 및 관리한다.

![15-monthly-cost-aws-service.png](images%2F15-monthly-cost-aws-service.png)

- 비용 및 사용량 데이터를 분석하는 맞춤형 보고서를 생성한다.
- 높은 수준에서 데이터를 분석하고, 모든 계정의 총 비용 및 사용량을 확인한다.
- 월별, 시간별, 리소스 수준에서 세분화할 수 있다.

![16-hourly-resource-level.png](images%2F16-hourly-resource-level.png)

- 비용 절감을 위해 최적의 Savings Plan을 선택할 수 있다.

![17-savings-plan-alternative-ri.png](images%2F17-savings-plan-alternative-ri.png)

- 이전 사용량을 기준으로 최대 12개월까지 사용량을 예측할 수 있다.

![18-forecast-usage.png](images%2F18-forecast-usage.png)

---

### AWS Budget

- 예산을 생성하고 비용이 예산을 초과하면 알람을 보낸다.
- 4가지 유형의 예산이 있다: 사용량, 비용, 예약, Savings Plan
- RI의 경우
  - 활용도를 추적한다.
  - EC2, ElastiCache, RDS, Redshift를 지원한다.
- 예산당 최대 5개의 SNS 알림을 제공한다.
- 필터링 기준: 서비스, 연결 계정, 태그, 구매 옵션, 인스턴스 유형, 지역, 가용 영역, API 작업 등..
- AWS Cost Explorer와 동일한 옵션을 제공한다.
- 2가지 Budget은 무료로 제공하며, 초과하는 경우 하나당 하루에 $0.02를 지불해야 한다.

---

### Cost Allocation Tags

- 비용 할당 태그를 사용하여 AWS 비용을 세부적으로 추적한다.
- **AWS Generated Tags**
  - 생성한 리소스에 자동으로 적용된다.
  - aws 접두사로 시작한다. (예. `aws:createdBy`)
- **User-defined Tags**
  - 사용자가 정의한다.
  - 접두사가 `user:`로 시작한다.

![19-cost-allocation-tags.png](images%2F19-cost-allocation-tags.png)

#### Cost & Usage Report

![20-cost-usage-report.png](images%2F20-cost-usage-report.png)

- AWS 비용 및 사용량에 대해 자세하게 확인할 수 있다.
- AWS 비용 및 사용 보고서에는 사용 가능한 가장 포괄적인 AWS 비용 및 사용 데이터 세트가 포함되어 있다.
- AWS 서비스, 가격 및 예약(예. EC2 예약 인스턴스)에 대한 추가 메타데이터를 포함한다.
- AWS 비용 및 사용 보고서에는 각각에 대한 AWS 사용량이 나열되어 있다.
  - 계정에서 사용하는 서비스 범주
  - 시간별 또는 일일 광고 항목
  - 비용 할당 목적으로 활성화한 태그
- S3로 매일 내보내도록 구성할 수 있다.
- Athena, Redshift 또는 QuickSight와 통합할 수 있다.

![21-cost-usage-report.png](images%2F21-cost-usage-report.png)

---

### AWS Compute Optimizer

- 워크로드에 최적의 AWS 리소스를 추천하여 비용을 절감하고 성능을 향상시킨다.
- 최적의 구성을 선택하고 워크로드 크기를 적절하게 조정하는 데 도움이 된다. (over/under 프로비저닝 방지)
- **기계 학습을 사용하여 리소스 구성 및 활용도 CloudWatch 지표를 분석**한다.
- 지원되는 리소스 목록은 아래와 같다.
  - EC2 인스턴스
  - EC2 Auto Scaling Group
  - EBS Volume
  - Lambda Function
- 비용을 최대 25% 절감할 수 있다.
- 권장 사항을 S3로 내보낼 수 있다.

![22-compute-optimizer.png](images%2F22-compute-optimizer.png)

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [SCP 예시](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_example-scps.html)