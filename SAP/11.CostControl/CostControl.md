# Cost Control

- 이번 장에서는 **Solutions Architect Professional (SAP)** 을 준비하며 "비용 관리"에 대해서 알아보도록 한다.

---

### Cost Allocation Tags

- 태그를 사용하여 서로 관련된 리소스를 추적할 수 있다.
- 비용 할당 태그를 사용하면 상세 비용 보고서를 실행할 수 있다.
- 태그와 마찬가지로 보고서에 열로 표시된다.
- AWS 생성 비용 할당 태그
  - 생성한 리소스에 자동으로 적용된다.
  - 접두사 `aws`로 시작한다. (예: `aws:createdBy`)
  - 활성화 전에 생성된 리소스에는 적용되지 않는다.
- 사용자 태그
  - 사용자에 의해 정의된다.
  - 접두사 `user`로 시작한다.
- 비용 할당 태그가 청구 콘솔에 표시된다.
- 보고서에 태그가 표시되는데 최대 12시간이 소요된다.

#### AWS Tag Editor

- 여러 리소스의 태그를 한 번에 관리할 수 있다.
- 태그 추가/업데이트/삭제가 가능하다.
- 모든 AWS 리전에서 태그를 지정하거나 태그가 지정되지 않은 리소스를 검색할 수 있다.

![1-aws-tag-editor.png](images%2F1-aws-tag-editor.png)

---

### Trusted Advisor

- 어떤 것도 설치할 필요가 없으며, 높은 수준에서 AWS 계정을 평가한다.
- AWS 계정에 대한 분석 및 권장 사항을 제공한다.
  - 비용 최적화 및 권장 사항, 성능, 보안, 내결함성, 서비스 제한
- 핵심 점검 및 권장 사항을 모든 고객에게 제공한다.
- 콘솔에서 매주 전자 메일 알림을 실행할 수 있다.
- Full Trusted Advisor: 비즈니스 및 엔터프라이즈 지원 계획에 사용 가능하다.
  - 한계에 도달했을 때, CloudWatch 알람을 설정할 수 있는 기능을 제공한다.
  - AWS 지원 API를 이용한 프로그램 액세스를 제공한다.

#### AWS Support Plans

- Basic, Developer, Business, Enterprise 총 네 개의 플랜이 있다.

![2-aws-support-plans.png](images%2F2-aws-support-plans.png)

- Business, Enterprise 플랜은 프로그래밍 방식의 AWS 지원 API를 제공한다.

#### Good to Know

- S3 버킷의 공개 여부를 확인할 수 있다.
  - 버킷 내부에 공개된 S3 객체는 확인할 수 없다.
  - EventBridge, S3 Event, AWS Config Rule을 사용할 수 있다.
- 서비스 제한을 확인할 수 있다.
  - Trusted Advisor에서만 서비스 제한을 모니터링할 수 있으며 변경할 수 없다.
  - AWS 지원 센터에서 케이스를 수동으로 생성해야 제한이 늘어난다.
  - 대신 "AWS Service Quotas" 서비스를 사용할 수 있다.

![3-tursted-advisor-good-to-know.png](images%2F3-tursted-advisor-good-to-know.png)

- Trusted Advisor를 간헐적으로 새로고침하는 람다 함수가 있고 Trusted Advisor를 확인한다.
- EventBridge 규칙을 실행한다.
- Service Quota에서 vCPU를 확인하여 한도를 초과한 경우 이벤트 규칙을 불러올 수 있다.
- 하나의 이벤트 버스에서 주요 이벤트 버스로 이동할 수 있다.
- EventBridge의 목적지는 여럿일 수 있으며 SNS 토픽 관리자나 SQS 큐로 이메일을 받을 수 있다.
- 람다 함수가 한계를 자동으로 요약하고 DynamoDB 테이블로 보낼 수 있다.
- 메시지가 정상적으로 처리되지 않은 경우 메시지는 DLQ로 이동한다.

---

### AWS Service Quotas

- 서비스 할당량 값 임계값에 근접하면 알림을 발생한다.
- Service Quotas 콘솔에서 CloudWatch 알람을 생성할 수 있다.
- 예를 들어, 람다 함수 동시 실행 제한이 있다.
- 제한에 도달하기 전에 리소스 할당량 증가 또는 종료를 요청해야 하는지 여부를 알 수 있도록 도와준다.

![4-aws-service-quotas.png](images%2F4-aws-service-quotas.png)

---

### EC2 인스턴스 실행 유형

- On Demand Instance: 짧은 워크로드, 예측 가능한 가격, 신뢰성이 높다.
- Spot Instance: 짧은 워크로드, 저렴한 비용으로 인해 인스턴스가 손실될 수 있다.
- Reserved: 최소 1년 약정이 있다.
  - Reserved Instance: 긴 워크로드
  - Convertible Reserved Instance: 유연한 인스턴스 유형을 사용하는 긴 워크로드
- Dedicated Instance: 다른 사용자와 하드웨어를 공유하지 않는다.
- Dedicated Host: 전체 물리적 서버를 예약하고 인스턴스 배치를 제어한다.
  - 코어 또는 소켓 레벨에서 작동하는 소프트웨어 라이센스에 적합하다.
  - 인스턴스가 재부팅될 때 동일한 호스트에 유지되도록 호스트 선호도를 정의할 수 있다.

#### Saving Plan

- 장기적인 사용량에 따라 할인을 받을 수 있는 새로운 가격 모델이다.
- 특정 유형의 사용을 약속한다.
  - 예를 들어, 1 ~ 3년동안 시간당 $10의 비용을 설정
- Saving Plan을 초과하는 사용량은 온디맨드 가격으로 청구된다.
- **EC2 Instance Saving Plan**
  - 최대 72%의 할인을 받을 수 있으며 표준 RI와 동일한 할인이다.
  - 인스턴스 패밀리(예: M5, C5 등)을 선택해야 하며 특정 리전에 종속된다.
  - 유연한 크기(`m5.large` ~ `m5.xlarge`)와 다양한 OS, 테넌시(dedicated 또는 기본)를 제공한다.
- **Compute Saving plan**
  - 최대 66%까지 할인을 받을 수 있으며 Convertible RI와 동일한 할인이다.
  - 인스턴스 패밀리 간 이동(C5 -> M5), 리전간 이동을 지원한다.
- **SageMaker Saving Plan**
  - 최대 64%까지 할인받을 수 있다.

---

### S3 Storage Classes

- S3에는 다양한 종류의 스토리지 클래스가 있다.
  - Amazon S3 Standard - General Purpose
  - Amazon S3 Standard-Infrequent Access (IA)
  - Amazon S3 One Zone-Infrequent Access
  - Amazon S3 Glacier Instant Retrieval
  - Amazon S3 Glacier Flexible Retrieval
  - Amazon S3 Glacier Deep Archive
  - Amazon S3 Intelligent Tiering
- 클래스 간에 수동 또는 S3 라이프사이클을 구성하여 이동할 수 있다.

#### S3 비용 절감

- S3 Select & Glacier Select: 네트워크 및 CPU 비용을 절감할 수 있다.
- S3 Lifecycle Rule: 계층 간 객체를 전환할 수 있다.
- 객체를 압축하여 저장 공간을 절약할 수 있다.
- S3 요청자가 비용을 지불하도록 설정할 수 있다. (Requester Pays)
  - 일반적으로 버킷 소유자는 버킷과 관련된 모든 S3 스토리지 및 데이터 전송 비용을 지불한다.
  - Requester Pays 버킷을 사용하면 버킷 소유자 대신 Requester가 버킷에서 요청 및 데이터 다운로드 비용을 지불한다.
  - 버킷 소유자는 항상 데이터 저장 비용을 지불한다.
  - 대규모 데이터셋을 다른 계정과 공유하고자 할 때 유용하게 사용된다.
  - IAM 역할이 할당된 경우 해당 역할의 소유자 계정이 요청에 대해 비용을 지불한다.

#### Durability & Availability

- Durability(내구성)
  - 여러 AZ에 컬쳐 객체의 높은 내구성을 제공한다. (99.99999999999%)
  - 10,000,000개의 객체를 S3에 저장할 경우 평균적으로 10,000년에 한 번씩 객체 하나의 손실이 발생할 것으로 예상할 수 있다.
  - 모든 스토리지 클래스에서 동일하다.
- Availability(가용성)
  - 서비스를 얼마나 쉽게 이용할 수 있는지 측정한다.
  - 스토리지 클래스에 따라 상이하다.
  - 예를 들어, S3 Standard는 99.99%의 가용성을 제공하며 이는 1년에 53분동안 사용할 수 없음을 의미한다.

#### S3 Standard - General Purpose

- 99.99%의 가용성을 제공한다.
- 자주 액세스하는 데이터에 유용하게 사용된다.
- 짧은 대기 시간와 높은 처리량을 제공한다.
- AWS 측에서 두 개의 동시 시설 장애를 견딜 수 있다.
- 빅데이터, 모바일 및 게임 애플리케이션, 콘텐츠 배포에 주로 사용된다.

#### Infrequent Access

- 액세스 빈도가 낮지만 필요할 때 신속하게 액세스해야 하는 데이터의 경우에 사용된다.
- S3 Standard보다 낮은 비용이 발생한다.
- **Amazon S3 Standard-Infrequent Access (S3 Standard-IA)**
  - 99.99%의 가용성을 제공한다.
  - 재해 복구, 백업 등의 작업에 사용된다.
- **Amazon S3 One Zone-Infrequent Access (S3 One Zone-IA)**
  - 단일 AZ에서 높은 내구성(99.999999999%)을 제공하고 AZ 파괴 시에 데이터가 손실된다.
  - 99.5%의 가용성을 제공한다.
  - 온프레미스 데이터의 보조 백업 복사본, 재생성 가능한 데이터의 저장 등에 사용된다.

#### Glacier Storage Class

- 아카이빙/백업을 위한 저비용 객체 스토리지다.
- 스토리지 비용과 검색 비용을 지불해야 한다.
- **Amazon S3 Glacier Instant Retrieval**
  - 밀리초 단위의 검색 기능으로 분기별로 한 번씩 액세스하는 데이터에 적합하다.
  - 최소 스토리지 저장 기간은 90일이다.
- **Amazon S3 Glacier Flexible Retrieval**
  - 신속(1~5분), 표준(3~5시간), 대량(5~12시간) 검색 유형이 있다.
  - 최소 스토리지 저장 기간은 90일이다.
- **Amazon S3 Glacier Deep Archive**
  - 장기간 데이터를 저장해야 할 때 적합하다.
  - 표준(12시간), 대량(48시간) 검색 유형이 있다.
  - 최소 스토리지 저장 기간은 180일이다.

#### Intelligent-Tiering

- 사용 패턴에 기반해 계층 사이에서 객체를 이동시킬 수 있도록 해준다.
- 객체 모니터링에 대한 비용을 지불해야 한다.
- 스토리지 검색 비용이 부과되지 않는다.
- Frequent Access tier (자동): 기본 계층이다.
- Infrequent Access tier (자동): 30일 동안 객체에 액세스 되지 않은 경우
- Archive Instant Access tier (자동): 90일 동안 객체에 액세스 되지 않은 경우
- Archive Access tier (선택): 90일에서 700일 이상까지 구성 가능하다.
- Deep Archive Access tier (선택): 180일에서 700일 이상까지 구성 가능하다.

#### Storage Class 비교

- S3 스토리지 간 차이는 아래의 표를 참고하면 되고 자세한 내용은 [공식 홈페이지](https://aws.amazon.com/s3/storage-classes/)를 확인하면 된다.

![5-s3-storage-classes-comparison.png](images%2F5-s3-storage-classes-comparison.png)

- S3 스토리지 간 비용 차이는 아래의 표를 참고하면 되고 자세한 내용은 [공식 홈페이지](https://aws.amazon.com/s3/pricing/)를 확인하면 된다.

![6-s3-storage-classes-price-comparison.png](images%2F6-s3-storage-classes-price-comparison.png)

---

### AWS Budgets

- 예산을 작성하고 비용이 예산을 초과하는 경우 알람을 전송한다.
- 4가지 유형의 예산이 있다.
  - 사용, 비용, 예약, Saving Plan
- RI의 경우 사용률을 추적할 수 있다.
  - 트랙 활용도
  - EC2 ElasticCache, RDS, Redshift를 지원한다.
- 예산당 최대 5개의 SNS 알림을 지원한다.
- 서비스, 링크된 계정, 태그, 구매 옵션, 인스턴스 유형, 리전, 가용 영역, API 작업 등으로 필터링할 수 있다.
- AWS Cost Explorer와 동일한 옵션을 제공한다.
- 2개의 예산은 무료로 사용할 수 있으며, 초과하는 경우 하나의 예산당 하루에 $0.02의 비용이 발생한다.

#### Budget Actions

- 예산이 특정 비용 또는 사용량 임계값을 초과할 경우 사용자 대신 작업을 실행한다.
- 3가지 작업 유형을 지원한다.
  - 사용자, 그룹 또는 IAM 역할에 IAM 정책 적용
  - OU에 서비스 제어 정책(SCP) 적용
  - EC2 또는 RDS 인스턴스 중지
- 작업을 자동으로 실행하거나 워크플로 승인 프로세스가 필요하다.
- 계정의 의도하지 않은 과소비를 감소시킬 수 있다.

![7-budget-actions.png](images%2F7-budget-actions.png)

- "Management 계정"과 "Member 계정"이 있는 "Dev OU"가 있다.
- 예산이 특정 임계값에 도달하면 "Dev OU"에 자동으로 SCP를 적용해서 "Member 계정"을 제한할 수 있다.
- 위의 예제에서는 개발자는 "Dev OU"에 맞춰 예산을 책정한다.

#### 예산 관리의 중앙화

![8-centralized-budget-management.png](images%2F8-centralized-budget-management.png)

- 중앙 집중식으로 예산을 관리할 수 있다.
- "Management 계정"과 "Member 계정"이 있는 "Current OU", "Member 계정"이 있는 "Restrictive OU"가 있다.
- "Management 계정"에서 모니터링하고자 하는 모든 "Member 계정"에 대해서 예산을 설정한다.
- 만약 10개의 "Member 계정"이 있다면 "Management 계정"에서 10개의 예산을 생성하지만 각 예산에 필터를 적용할 수 있다.
- 예산이 임계값을 초과하면 중앙에 있는 계정의 SNS에 알림을 보낼 수 있고, 해당 알림은 람다 함수를 트리거한다.
- 임계값을 위반한 계정을 실제 SCP가 이미 적용된 보다 제한적인 OU(Restrictive OU)로 이동시킬 수 있다.
  - 제한된 OU로 이동된 계정은 새로운 리소스 생성이 허용되지 않는다.
- 계정이 이동된 것을 알리기 위해서 SNS 토픽을 통해 이메일을 전송할 수 있다.

#### 예산 관리의 탈중앙화

![9-decentralized-budget-management.png](images%2F9-decentralized-budget-management.png)

- 예산이 중앙에서 관리되지 않도록 할 수 있다.
- 예산이 "Management 계정"에서 직접 관리되지만 예산을 배포하고 각 "Member 계정"에서 생성하도록 할 수 있다.
  - 예산 배포를 단순화하기 위해 CloudFormation StackSets을 사용할 수 있다.
- 예산이 특정 임계값에 도달하기 직전에 SNS 토픽에 메시지를 보내거나 초과하는 경우에 SNS 토픽에 메시지를 전송할 수 있다.
- 예를 들어, 람다 함수를 트리거하여 모든 EC2 인스턴스를 종료시킬 수 있다.

#### Cost Explorer

- 시간이 지남에 따라 AWS 비용 및 사용량을 시각화하여 비용에 대한 이해와 관리를 편리하게 해준다.
- 비용 및 사용량 데이터를 분석하는 사용자 정의 보고서를 생성한다.
- 높은 수준으로 데이터를 분석하여 모든 계정에서 총 비용 및 사용량을 확인할 수 있다.
- 월별, 시간별, 리소스 수준에서 세분화된 정보를 확인할 수 있다.
- 최적의 Saving Plan을 선택하여 청구서의 가격을 낮출 수 있다.
- 이전 사용량을 기준으로 최대 12개월까지의 사용량을 예측할 수 있다.

---

### AWS Compute Optimizer

- 워크로드에 최적의 AWS 리소스를 추천하여 비용 절감 및 성능을 향상시킬 수 있다.
- 최적의 구성을 선택하고 워크로드 크기를 적절히 조정할 수 있도록 지원한다.
  - 오버/언더 프로비저닝을 방지할 수 있다.
- 머신러닝을 사용하여 리소스의 구성 및 활용률 CloudWatch 메트릭을 분석한다.
- EC2 인스턴스, ASG, EBS 볼륨, 람다 함수를 지원한다.
- 비용을 최대 25%까지 절감할 수 있다.
- 권장 사항을 S3로 내보낼 수 있다.

![11-compute-optimizer.png](images%2F11-compute-optimizer.png)

#### CloudWatch Agent

- 메모리를 분석하기 위해서 필요하다.
- CPU, Network In/Out, DiskReadOps, DiskWriteOps 등의 지표를 확인할 때는 필요하지 않다.

![12-cloudwatch-agent.png](images%2F12-cloudwatch-agent.png)

---

### Reserved Instance

- AWS Organization에서 예약된 인스턴스
  - 모든 계정이 예약된 인스턴스 및 Saving Plan을 공유한다.
  - 조직의 지급자 계정(관리 계정)은 지급자 계정을 포함하여 해당 조직의 모든 계정에 대해 RI 할인 및 Saving Plan 공유를 해제할 수 있다.
- 예약 인스턴스 갱신
  - 예약된 인스턴스를 미리 예약할 수 있다.
  - RI를 갱신하려면 이전 RI 구매가 만료될 때마다 인스턴스를 대기시키면 된다.

---

### 참고한 강의

- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)