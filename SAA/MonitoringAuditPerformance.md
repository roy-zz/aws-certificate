# AWS Monitoring, Audit and Performance

이번 장에서는 SAA를 준비하며 **AWS의 모니터링, 감사 서비스**에 대해서 알아보도록 한다.

---

### Amazon CloudWatch Metrics

- "CloudWatch"는 AWS의 모든 서비스에 대한 메트릭을 제공한다.
- 메트릭은 모니터링할 변수로 CPU 사용률, 네트워크 사용률 등이 있다.
- 메트릭은 네임스페이스에 속한다.
- "Demension"은 메트릭의 속성(인스턴스 ID, 환경 등)이다.
- 메트릭당 최대 30개의 "Demension"을 지원한다.
- 메트릭에는 타임스탬프가 있다.
- 메트릭의 CloudWatch 대시보드를 생성할 수 있다.
- CloudWatch 사용자 지정 메트릭을 생성할 수 있다.

#### CloudWatch Metric Streams

- 거의 실시간으로 제공되고 대기 시간이 짧은 상태에서 원하는 목적지로 CloudWatch 메트릭을 지속적으로 스트리밍할 수 있다.
  - "Amazon Kinesis Data Firehose"
  - 3rd Party 서비스 제공자: Datadog, Dynatrace, New Reric, Splunk, Sumo Logic
- 일부만 스트리밍 하도록 메트릭을 필터링하는 옵션을 제공한다.

#### CloudWatch Logs

- 로그 그룹: 임의의 이름, 일반적으로 응용 프로그램을 나타낸다.
- 로그 스트림: 응용 프로그램 내의 인스턴스/로그 파일/컨테이너
- 로그 만료 정책을 정의할 수 있다.(만료되지 않음, 30일 등)
- CloudWatch Logs는 아래와 같은 목적지로 로그를 전송할 수 있다.
  - Amazon S3(export)
  - Kinesis Data Streams
  - Kinesis Data Firehose
  - AWS Lambda
  - OpenSearch

#### CloudWatch Logs - Sources

- SDK, CloudWatch Logs Agent, CloudWatch Unified Agent
- Elastic Beanstalk: 애플리케이션에서 로그 수집
- ECS: 컨테이너에서 수집
- AWS Lambda: Function에서 로그 수집
- VPC Flow Logs: VPC 특정 로그
- API Gateway
- 필터 기반의 CloudTrail
- Route53: DNS 쿼리 기록

#### Metric Filter & Insight

- CloudWatch 로그에서 필터를 사용할 수 있다.
  - 예를 들어, 로그 내부에서 특정 IP를 찾을 수 있다.
  - 로그에서 "ERROR"가 발생한 횟수를 카운트할 수 있다.
- 메트릭 필터를 사용하여 CloudWatch 경보를 트리거할 수 있다.
- CloudWatch 로그 인사이트를 사용하여 로그를 쿼리하고 CloudWatch 대시보드에 쿼리를 추가할 수 있다.

#### S3 Export

![cloudwatch-logs-s3-export.png](images%2FMonitoringAuditPerformance%2Fcloudwatch-logs-s3-export.png)

- 로그 데이터를 내보내는 데 최대 12시간이 소요될 수 있다.
- API 호출은 `CreateExportTask`다.
- 실시간이 아니기 때문에 실시간이 필요한 경우 아래의 이미지와 같이 "Logs Subscription"을 사용한다.

![cloudwatch-logs-subscription.png](images%2FMonitoringAuditPerformance%2Fcloudwatch-logs-subscription.png)

- 다중 계정 및 다중 Region을 사용하는 경우 아래의 이미지와 같이 로그를 통합할 수 있다.

![cloudwatch-logs-aggregation.png](images%2FMonitoringAuditPerformance%2Fcloudwatch-logs-aggregation.png)

#### CloudWatch Logs for EC2

- 기본적으로 EC2 시스템의 로그가 CloudWatch로 전송되지 않는다.
- EC2에서 CloudWatch 에이전트를 실행하여 원하는 로그 파일을 푸시할 수 있다.
- IAM 권한이 올바르게 설정되어 있는지 확인해야 한다.
- CloudWatch 로그 에이전트를 On-Premise에서 실행 중인 서비스에도 설정할 수 있다.

![cloudwatch-logs-ec2.png](images%2FMonitoringAuditPerformance%2Fcloudwatch-logs-ec2.png)

#### CloudWatch Logs Agent & Unified Agent

- 가상 서버(EC2 Instance, On-Premises Servers)에 사용된다.
- CloudWatch Logs Agent
  - 에이전트의 오래된 버전이다.
  - "CloudWatch Logs"에만 전송이 가능하다.
- CloudWatch Unified Agent
  - RAM, 프로세스 등과 같은 추가적인 시스템 레벨의 메트릭을 수집할 수 있다.
  - "CloudWatch Logs"로 로그를 전송하여 수집할 수 있다.
  - "SSM Parameter Store"를 이용한 중앙 집중식으로 구성할 수 있다.
  - Linux 서버/EC2 인스턴스에서 직접 수집할 수 있다.
  - CPU(active, guest, idle, system, user, steal)
  - Disk metric(free, used, total), Disk IO(writes, reads, bytes, iops)
  - RAM(free, inactive, used, total, cached)
  - Netstat(TCP 및 UDP 연결 수, net packets, bytes)
  - Process(total, dead, blocked, idle, running, sleep)
  - Swap Space(free, used, used %)

---

### CloudWatch Alarms

- 모든 메트릭에 대한 알림을 트리거하는 데 사용된다.
- 샘플링, %, max, min 등 다양한 옵션이 있다.
- "OK", "INSUFFICIENT_DATA", "ALARM" 등 다양한 상태가 있다.
- 메트릭을 평가할 시간을 설정할 수 있다.
- 10초, 30초 또는 60초의 배수로 수집 주기를 설정할 수 있다.
- "Amazon EC2"를 타겟으로 설정하여 중지, 종료, 재부팅 또는 복구에 대한 알림을 트리거할 수 있다.
- "EC2 Auto Scaling"을 타겟으로 설정하여 자동 스케일링 작업을 트리거할 수 있다.
- "Amazon SMS"을 타겟으로 설정하여 알림을 전송할 수 있다.

#### Composite Alarms

- "CloudWatch Alarms"가 단일 메트릭에 있다.
- 복합 알람(Composite Alarms)은 여러 개의 다른 알람의 상태를 모니터링한다.
- AND 또는 OR 조건을 설정할 수 있다.
- 복잡한 복합 알람을 생성하여 "알람 소음(alarm noise)"을 줄이는데 도움이 될 수 있다.

![cloudwatch-alarm-composite.png](images%2FMonitoringAuditPerformance%2Fcloudwatch-alarm-composite.png)

#### EC2 Instance Recovery

- Status Check
  - 인스턴스 상태: EC2 VM의 상태를 확인한다.
  - 시스템 상태: 기반 하드웨어의 상태를 확인한다.

![ec2-instance-recovery.png](images%2FMonitoringAuditPerformance%2Fec2-instance-recovery.png)

- Recovery: 동일한 Private, Public, Elastic IP, metadata, 배치 그룹으로 복구한다.

#### 참고

- "CloudWatch Logs Metrics Filters"를 기반으로 알람을 생성할 수 있다.

![cloudwatch-alarm-good-to-know.png](images%2FMonitoringAuditPerformance%2Fcloudwatch-alarm-good-to-know.png)

- 경보 및 알람을 테스트 하려면 CLI를 통해서 진행할 수 있다.
  `aws cloudwatch set-alarm-state --alarm-name "myalarm" --state-value ALARM --state-reason "testing purposes"`

---

### Amazon EventBridge (CloudWatch Event의 새로운 버전)

- Schedule: Cron jobs(시케줄링된 스크립트)
- Event Pattern: 서비스가 어떤 일을 할 때 반응하는 이벤트 규칙
- "Lambda Function" 트리거, SQS 또는 SNS에 메시지 전송을 할 수 있다.
- 아래의 이미지와 같이 "EventBridge 규칙"을 설정할 수 있다.

![eventbridge-rules.png](images%2FMonitoringAuditPerformance%2Feventbridge-rules.png)

- 다른 AWS 계정에서 리소스 기반 정책을 사용하여 이벤트 버스에 액세스할 수 있다.
- 이벤트 버스로 전송된 이벤트를 아카이브할 수 있다.
- 아카이브된 이벤트를 반복할 수 있다.

![eventbridge.png](images%2FMonitoringAuditPerformance%2Feventbridge.png)

#### Schema Registry

- "EventBridge"는 이벤트를 분석하고 스키마를 추론할 수 있다.
- 스키마 레지스트리를 사용하면 응용프로그램에 대한 코드를 생성할 수 있으며, 이 코드는 이벤트 버스에서 데이터가 어떻게 구성되는지 미리 알 수 있다.
- 스키마를 버전화할 수 있다.

![eventbridge-schema-registry.png](images%2FMonitoringAuditPerformance%2Feventbridge-schema-registry.png)

#### Resource-based Policy

- 특정 이벤트 버스에 대한 사용 권한을 관리할 수 있다.
- 예를 들어, 다른 AWS 계정 또는 AWS 영역의 이벤트를 허용하거나 거부할 수 있다.
- AWS 조직의 모든 이벤트를 단일 AWS 계정 또는 AWS Region에 집계하는 데 사용될 수 있다.

![eventbridge-resource-based-policy.png](images%2FMonitoringAuditPerformance%2Feventbridge-resource-based-policy.png)

---

### CloudWatch Container Insights

- 컨테이너로 부터 메트릭 및 로그를 수집, 집계, 요약할 수 있다.
- 아래와 같은 컨테이너 서비스에 사용될 수 있다.
  - Amazon Elastic Container Service (Amazon ECS)
  - Amazon Elastic Kubernetes Service (Amazon EKS)
  - Kubernetes platforms on EC2
  - Fargate (ECS or EKS)
- "CloudWatch Insights"는 "Amazon EKS"와 "Kubernetes"에서 컨테이너형 버전의 "CloudWatch Agent"를 사용하여 컨테이너를 검색할 수 있다.

![cloudwatch-container-insights.png](images%2FMonitoringAuditPerformance%2Fcloudwatch-container-insights.png)

---

### CloudWatch Lambda Insights

- "AWS Lambda"에서 실행되는 서버리스 애플리케이션의 모니터링 및 문제 해결을 할 수 있다.
- CPU 시간, 메모리, 디스크 및 네트워크를 포함한 시스템 수준의 메트릭 수집, 집계 및 요약을 제공한다.
- Cold Start, Lambda worker shutdown 등 진단정보 수집, 집계 및 요약이 제공된다.
- "Lambda Insights"는 "Lambda Layer"로 제공된다.

![cloudwatch-lambda-insights.png](images%2FMonitoringAuditPerformance%2Fcloudwatch-lambda-insights.png)

---

### CloudWatch Contributor Insights

- 로그 데이터를 분석하고 기여자(Contributor) 데이터를 표시하는 시계열 데이터를 생성한다.
  - 상위 N개 기여자에 대한 메트릭 조회
  - 고유한 기여자의 총 수와 사용 시간 조회
- 이를 통해 최고의 대화 상대를 찾고 시스템 성능에 영향을 미치는 사용자 또는 대상을 파악할 수 있다.
- AWS에서 생성한 모든 로그(VPC, DNS 등)에 대해 작동한다.
- 예를 들어, 불량 호스트를 찾거나, 가장 무거운 네트워크 사용자를 식별하거나, 가장 많은 오류를 생성하는 URL을 찾을 수 있다.
- 처음부터 규칙을 만들 수도 있고, "CloudWatch Logs"를 활용하여 AWS에서 만든 샘플 규칙을 사용할 수도 있다.
- "CloudWatch"는 다른 AWS 서비스의 메트릭을 분석하는 데 사용할 수 있는 기본 제공 규칙도 제공한다.

![cloudwatch-contributor-insights.png](images%2FMonitoringAuditPerformance%2Fcloudwatch-contributor-insights.png)

---

### CloudWatch Application Insights

- 모니터링되는 애플리케이션의 잠재적인 문제를 보여주는 자동 대시보드를 제공하여 지속적인 문제를 파악할 수 있도록 지원한다.
- 애플리케이션은 "Amazon EC2" 인스턴스에서 실행되며, 이는 선택한 기술(Java, .NET, Microsoft IIS 웹 서버, 데이터베이스 등)만 포함된다.
- "Amazon EBS", "RDS", "ELB", "ASG", "Lambda", "SQS", "DynamoDB", "S3 버킷" 등 다른 AWS 리소스를 사용할 수 있다.
- "SageMaker"에 의해 작동된다.
- 애플리케이션 상태에 대한 가시성 향상으로 애플리케이션 문제 해결 및 복구에 소요되는 시간을 단축할 수 있다.
- 결과 및 경고는 "Amazon EventBridge" 및 "SSM OpsCenter"로 전송된다.

---

### CloudWatch Insights and Operational Visibility

- **CloudWatch Container Insights**
  - "ECS", "EKS", "Kubernetes on EC2", "Fargate", "Kubernetes 에이전트 필요"
  - 메트릭 및 로그를 확인할 수 있다.
- **CloudWatch Lambda Insights**
  - 서버 없는 애플리케이션 문제 해결을 위한 세부 메트릭을 확인할 수 있다.
- **CloudWatch Contributors Insights**
  - CloudWatch 로그를 통해 "Top-N"개의 기여자를 찾는다.
- **CloudWatch Application Insights**
  - 애플리케이션 및 관련 AWS 서비스 문제 해결을 위한 자동 대시보드를 제공한다.

---

### AWS CloudTrail

- AWS 계정에 대한 거버넌스, 컴플라이언스 및 감사를 제공한다.
- "CloudTrail"은 기본적으로 활성화되어 있다.
- AWS 계정 내에서 발생한 Event/API 호출 기록을 저장하고 있다. (Console, SDK, CLI, AWS Services)
- "CloudTrail"의 로그를 "CloudWatch" 로그 또는 "S3"에 저장할 수 있다.
- 추적은 모든 Region(기본값) 또는 단일 지역에 적용할 수 있다.
- AWS에서 리소스가 삭제된 경우 "CloudTrail"을 먼저 확인하여 검색할 수 있다.

![cloudtrail-diagram.png](images%2FMonitoringAuditPerformance%2Fcloudtrail-diagram.png)

#### Events

- **Management Events**:
  - AWS 계정의 리소스에 대해 수행되는 작업이다.
  - 대표적으로 아래와 같은 작업이 있을 수 있다.
    - 보안 구성(`IAM AttachRolePolicy`)
    - 라우팅 데이터에 대한 규칙 구성(`Amazon EC2 CreateSubnet`)
    - 로깅 설정(`AWS CloudTrail CreateTrail`)
  - 기본적으로 "CloudTrail"은 관리 이벤트를 기록하도록 구성된다.
  - 읽기 이벤트(리소스를 수정하지 않는)와 쓰기 이벤트(리소스를 수정할 수 있는)를 분리할 수 있다.
- **Data Events**:
  - 기본적으로 데이터 이벤트는 기록되지 않는다.(대략의 작업 때문에)
  - "Amazon S3" 개체 수준의 활동(예: `GetObject`, `DeleteObject`, `PutObject`): 읽기 및 쓰기 이벤트를 구분할 수 있다.

#### CloudTrail Insights

- "CloudTrail Insights"를 통해 계정에서 비정상적인 활동을 감지할 수 있다.
  - 부정확한 리소스 프로비저닝
  - 서비스 한도 초과
  - "AWS IAM" 작업 버스트
  - 주기적인 유지관리 활동의 공백
- "CloudTrail Insights"는 정상적인 관리 이벤트를 분석하여 기준선을 생성한다.
- 쓰기 이벤트를 지속적으로 분석하여 특이한 패턴을 탐지한다.
  - "CloudTrail" 콘솔에 이상 징후가 나타난다.
  - 이벤트는 "Amazon S3"로 전송된다.
  - "EventBridge" 이벤트가 생성된다.(자동화 요구사항에 맞도록)

![cloudtrail-insights.png](images%2FMonitoringAuditPerformance%2Fcloudtrail-insights.png)

#### CloudTrail Events Retention

- 이벤트는 "CloudTrail"에 90일 동안 저장된다.
- 이벤트를 이 기간 이후로 유지하려면 "S3"에 기록하고 "Athena"를 사용해야 한다.

![cloudtrail-events-retention.png](images%2FMonitoringAuditPerformance%2Fcloudtrail-events-retention.png)

- "Amazon EventBridge"와 통합되어 "DynamoDB"의 테이블 변경 API 호출을 탐지하고 "SNS"를 통해 알림을 발생시킬 수 있다.

![cloudtrail-intercept-api-calls.png](images%2FMonitoringAuditPerformance%2Fcloudtrail-intercept-api-calls.png)

---

### AWS Config

- AWS 리소스 컴플라이언스 감사 및 기록을 지원한다.
- 시간 경과에 따른 구성 및 변경 사항을 기록한다.
- "AWS Config"에서는 아래와 같은 질문에 답변할 수 있도록 도움을 줄 수 있다.
  - 보안 그룹에 제한 없이 SSH에 액세스할 수 있는가?
  - 나의 S3 버킷에 누구든 Public 접근할 수 있는가?
  - 시간이 지남에 따라 ALB 구성이 어떻게 변경되었는가?
- 변경 사항에 대한 알림(SNS Notifications)을 받을 수 있다.
- "AWS Config"는 지역별(per-region) 서비스다.
- 여러 지역 및 계정에 걸쳐 집계가 가능하다.
- "S3"에 구성 데이터를 저장하고 "Athena"로 분석이 가능하다.

#### Rules

- AWS 관리 구성 규칙 및 사용이 가능하다.(75개 이상)
- 사용자 지정 규칙을 만들 수 있으며 "AWS Lambda"에 정의해야 한다.
  - 예: 각 "EBS" 디스크가 "gp2" 유형인지 평가
  - 예: 각 "EC2" 인스턴스가 "t2.micro"인지 평가
- 규칙을 평가/트리거할 수 있다.
  - 각 규칙 변경 시마다
  - 일정한 시간 간격으로
- "AWS Config Rules"에서 작업이 발생하는 것을 방지하지 않는다.
- "Free tier"에서는 사용이 불가능하며, Region에 기록된 구성당 $0.003, Reion에 기록된 규칙 평가당 $0.001이다.

#### Resource

- 시간 경과에 따른 리소스 컴플라이언스 보기

![config-resource-compliance.png](images%2FMonitoringAuditPerformance%2Fconfig-resource-compliance.png)

- 시간 경과에 따른 리소스 구성 보기

![config-resource-overtime.png](images%2FMonitoringAuditPerformance%2Fconfig-resource-overtime.png)

- 시간 경과에 따른 리소스의 "CloudTrail" API 호출 보기

![config-resource-api-call.png](images%2FMonitoringAuditPerformance%2Fconfig-resource-api-call.png)

#### Remediations

- "SSM Automation Documents"를 사용하여 비준수 리소스에 대한 교정 자동화를 할 수 있다.
- AWS에서 관리하는 자동화 문서를 사용하거나 사용자 지정 자동화 문서를 작성할 수 있다.
  - "Lambda Function"를 호출하는 사용자 지정 자동화 문서를 생성할 수 있다.
- 자동 업데이트 적용 후 리소스가 여전히 비준수인 경우 업데이트 적용 재시도를 설정할 수 있다.

![config-remediations.png](images%2FMonitoringAuditPerformance%2Fconfig-remediations.png)

#### Notifications

- AWS 리소스가 규정을 준수하지 않을 경우, "EventBridge"를 사용하여 알림을 트리거할 수 있다.

![config-eventbridge.png](images%2FMonitoringAuditPerformance%2Fconfig-eventbridge.png)

- 구성 변경 및 규정 준수 상태 알림을 "SNS"로 전송하는 기능을 사용할 수 있다. (모든 이벤트 - SNS 필터링 또는 클라이언트 사이드 필터링 사용)

![config-sns.png](images%2FMonitoringAuditPerformance%2Fconfig-sns.png)

---

### CloudWatch vs CloudTrail vs Config

- **CloudWatch**
  - 성능 모니터링(메트릭, CPU, 네트워크 등) 및 대시보드
  - 이벤트 및 알림
  - 로그 집계 및 분석
- **CloudTrail**
  - 모든 사용자가 계정 내에서 수행한 API 호출 기록
  - 특정 리소스에 대한 추적을 정의
  - 글로벌 서비스
- **Config**
  - 구성 변경사항 기록
  - 규정 준수 규칙에 따라 리소스 평가
  - 변경 사항 및 규정 준수 일정 파악

### Elastic Load Balancer 와의 통합

- **CloudWatch**
  - 들어오는 연결 메트릭 모니터링
  - 시간 경과에 따라 오류 코드를 %로 시각화
  - 대시보드를 만들어 로드 밸런서 성능을 파악
- **Config**
  - 로드 밸런서에 대한 Security Group 규칙 추적
  - 로드 밸런서에 대한 구성 변경 사항 추적
  - SSL 인증서가 항상 로드 밸런서에 할당되었는지 확인(Compliance)
- **CloudTrail**
  - API 호출을 사용하여 로드 밸런서를 변경한 사용자 추적

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03