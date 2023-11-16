# AWS Monitoring, Audit & Performance

이번 장에서는 **SysOps Administrator**를 준비하며 **AWS의 모니터링, 감사, 성능**에 대해서 알아보도록 한다.

---

### AWS CloudWatch

#### Metric

- CloudWatch는 AWS의 모든 서비스에 대한 지표를 제공한다.
- "Metric"은 모니터링할 변수다.(`CPUUtilization`, `NetworkIn` 등..)
- "Metric"은 네임스페이스에 속한다.
- "Dimension"은 "Metric"의 속성이다. (인스턴스 ID, 환경 등..)
- "Metric"당 최대 30개의 "Dimension"을 가질 수 있다.
- "Metric"에는 타임스탬프가 있다.
- "Metric"의 CloudWatch 대시보드를 생성할 수 있다.

#### EC2 상세 모니터링

- EC2 인스턴스 지표에는 "5분 마다" 지표가 있다.
- 상세한 모니터링(유료)을 통해 "1분 마다" 데이터를 얻을 수 있다.
- ASG를 더 빠르게 확장하려면 상세한 모니터링을 사용해야 한다.
- AWS 프리 티어를 사용하면 10가지 세부 모니터링 지표를 얻을 수 있다.
- EC2 메모리 사용량은 기본적으로 수집되지 않으며, 사용자 지정 지표로 인스턴스 내부에서 전송해야 한다.

#### Custom Metric

- 자체 사용자 지정 지표를 정의하고 CloudWatch에 전송할 수 있다.
- 예를 들어, 메모리 사용량, 디스크 공간, 로그인한 사용자의 수가 있다.
- `PutMetricData` API를 호출하면 된다.
- "Dimension"을 활용하여 "Metric"을 분류하는 기능이 있다.
  - Instance.id
  - Environment.name
- Metric 확인 (`StorageResolution` API 파라미터에 두 가지 값이 있음)
  - 표준: 1분(60초)
  - 짧은 주기: 1/5/10/30초 간격으로 수집하지만 비용이 높다.
- **지난 2주 및 향후 2시간의 지표 데이터 포인트를 허용**한다. (EC2 인스턴스의 시간을 올바르게 구성해야 한다.)

#### Dashboard

- 주요 지표 및 경보에 빠르게 액세스할 수 있도록 맞춤형 대시보드를 설정하는 가장 좋은 방법이다.
- **대시보드는 글로벌 서비스**다.
- **대시보드에는 여러 AWS 계정 및 Region의 그래프가 포함될 수 있다.**
- 대시보드의 시간대 및 시간 범위를 변경할 수 있다.
- 자동 새로고침(10초, 1분, 2분, 5분, 15분)을 설정할 수 있다.
- 대시보드는 AWS 계정이 없는 사람과 공유할 수 있다.
  - Public으로 공개, 이메일, 3rd SSO 제공자를 통한 Cognito
- 대시보드 3개(최대 지표 50개)는 무료로 사용할 수 있고, 이후 "$3.00/대시보드/월" 이용료를 지불해야 한다.

---

### CloudWatch Logs

- **Log Group**: 임의의 이름, 일반적으로 애플리케이션을 나타낸다.
- **Log Stream**: 애플리케이션 내 인스턴스/로그 파일/컨테이너를 나타낸다.
- 로그 만료 정책 정의가 가능하다.(만료되지 않음, 1일 ~ 10년 등..)
- **CloudWatch Logs는 아래의 대상으로 로그를 보낼 수 있다.**
  - Amazon S3
  - Kinesis Data Streams
  - Kinesis Data Firehose
  - AWS Lambda
  - OpenSearch
- 로그는 기본적으로 암호화된다.
- 자신의 키로 KMS 기반 암호화를 설정할 수 있다.

#### CloudWatch Logs - Source

- CloudWatch Logs는 아래와 같은 로그 소스를 가질 수 있다.
  - SDK, CloudWatch Logs Agent, CloudWatch Unified Agent
  - Elastic Beanstalk: 애플리케이션에서 로그 수집
  - ECS: 컨테이너에서 수집
  - AWS Lambda: Function 로그에서 수집
  - VPC Flow Logs: VPC 특정 로그 수집
  - API Gateway
  - 필터 기반의 CloudTrail
  - Route53: DNS 쿼리를 기록한다.

#### CloudWatch Logs - Insights

- CloudWatch Logs Insights를 통해 통찰력을 얻을 수 있다.

![1-cloudwatch-logs-insights.png](images%2F1-cloudwatch-logs-insights.png)

- CloudWatch Logs에 저장된 로그 데이터를 검색 및 분석할 수 있다.
- 예를 들어, 로그 내에서 특정 IP를 찾거나, 로그에서 "ERROR" 발생 횟수를 검색할 수 있다.
- "purpose-built" 쿼리 언어가 제공된다.
  - AWS 서비스 및 JSON 로그에서 필드를 자동으로 검색한다.
  - 원하는 이벤트 필드 가져오기, 조건에 따른 필터링, 집계 통계 계산, 이벤트 정렬, 이벤트 수 제한 등..
  - 쿼리를 저장하고 CloudWatch 대시보드에 추가할 수 있다.
- 서로 다른 AWS 계정의 여러 로그 그룹을 쿼리할 수 있다.
- 실시간 엔진이 아닌 쿼리 엔진이다.

![2-cloudwatch-logs-insights.png](images%2F2-cloudwatch-logs-insights.png)

#### S3 Export

![3-s3-export.png](images%2F3-s3-export.png)

- 로그 데이터를 내보낼 수 있게 되기까지 최대 12시간이 걸릴 수 있다.
- API 호출은 `CreateExportTask`다.
- 배치로 내보내기 때문에 거의 실시간이 아니거나 실시간이 아니다.

#### CloudWatch Logs Subscription

- 처리 및 분석을 위해 CloudWatch Logs에서 실시간으로 로그 이벤트를 가져온다.
- Kinesis Data Streams, Kinesis Data Firehose, Lambda로 전송한다.
- **Subscription Filter**: 어떤 로그가 대상으로 전달되는 이벤트인지 필터링한다.

![4-cloudwatch-logs-subscription.png](images%2F4-cloudwatch-logs-subscription.png)

#### CloudWatch Logs Aggregations - Multi-Account & Multi Region

![5-cloudwatch-logs-aggregation.png](images%2F5-cloudwatch-logs-aggregation.png)

- Subscription Filter를 통해 CloudWatch Logs에서 특정 계정이나 Region의 Kinesis Data Stream으로 데이터를 집계할 수 있다.

![6-cloudwatch-logs-subscription.png](images%2F6-cloudwatch-logs-subscription.png)

- 교차 계정 구독을 위해서 다른 AWS 계정(KDS, KDF)의 리소스로 이벤트를 보낼 수 있다.
- KDS나 KDF로 데이터를 Put하기 위해서 적절한 IAM 역할을 제공해야 한다.

![7-cloudwatch-logs-subscription-2.png](images%2F7-cloudwatch-logs-subscription-2.png)

---

### CloudWatch Alarms

- "Alarm"은 모든 지표에 대한 알림을 트리거하는 데 사용된다.
- 다양한 옵션(샘플링, `%`, 최대값, 최소값 등)을 제공한다.
- Alarm 상태:
  - OK
  - INSUFFICIENT_DATA
  - ALARM
- 기간:
  - "Metric"을 평가하는 데 걸리는 시간(초)
  - 짧은 수집 주기의 맞춤형 "Metric": 10초, 30초 또는 60초의 배수

#### Alarm Target

- EC2 인스턴스 중단, 종료, 재부팅 또는 복구 시점에 알람을 발생시킬 수 있다.
- Auto Scaling 액션을 트리거할 수 있다.
- SNS에 알림을 보낼 수 있다.

#### Composite Alarm

- CloudWatch Alarm은 단일 지표에 대해서 발생한다.
- **복합 경보는 여러 다른 경보의 상태를 모니터링**한다.
- AND 및 OR 조건을 사용할 수 있다.
- 복잡한 복합 알림을 생성하여 "알림 소음"을 줄이는 데 도움이 된다.

![8-cloudwatch-alarm-composite-alarm.png](images%2F8-cloudwatch-alarm-composite-alarm.png)

#### EC2 인스턴스 복구

- 상태 확인
  - 인스턴스 상태: EC2 가상 머신의 상태를 확인한다.
  - 시스템 상태: 기본 하드웨어의 상태를 확인한다.

![9-ec2-instance-recovery.png](images%2F9-ec2-instance-recovery.png)

- 복구하면 동일한 Private & Public & Elastic IP, 메타데이터, 배치 그룹을 얻을 수 있다.

#### 참고 사항

- CloudWatch Logs 지표 필터를 기반으로 경보를 생성할 수 있다.

![10-cloudwatch-alarm-good-to-know.png](images%2F10-cloudwatch-alarm-good-to-know.png)

- 경보 및 알림을 테스트하려면 CLI를 통해 `aws cloudwatch set-alarm-state --alarm-name "myalarm" --state-value ALARM --state-reason "testing purposes"`를 사용하여 경보 상태를 경보로 설정할 수 있다.

#### CloudWatch Synthetics Canary

- API, URL, 웹사이트 등을 모니터링하는 구성 가능한 스크립트다.
- 고객이 영향을 받기 전에 문제를 찾기 위해 고객이 수행하는 작업을 프로그래밍 방식으로 재현한다.
- 엔드포인트의 가용성과 대기 시간을 확인하고 UI의 로드 시간 데이터와 스크린샷을 저장할 수 있다.
- CloudWatch Alarm과 통합할 수 있다.
- 스크립트는 Node.js 또는 Python으로 작성할 수 있다.
- 헤드리스 Google Chrome 브라우저에 프로그래밍 방식으로 액세스할 수 있다.
- 한 번 또는 정기적으로 실행할 수 있다.

![11-cloudwatch-synthetics-canary.png](images%2F11-cloudwatch-synthetics-canary.png)

- **Heartbeat Monitor**: URL 부하, 스크린샷 및 HTTP 아카이브 파일 저장
- **API Canary**: REST API의 기본 읽기 및 쓰기 기능을 테스트한다.
- **Broken Link Checker**: 테스트 중인 URL 내부의 모든 링크를 확인한다.
- **Visual Monitoring**: Canary 실행 중에 찍은 스크린샷을 기준 스크린샷과 비교한다.
- **Canary Recorder**: CloudWatch Synthetics Recorder와 함께 사용한다. (웹 사이트에서 작업을 기록하고 이에 대한 스크립트를 자동으로 생성)
- **GUI Workflow Builder**: 웹페이지에서 작업을 수행할 수 있는지 확인한다. (예. 로그인 양식으로 웹페이지 테스트)

---

### Amazon EventBridge

- CloudWatch Event의 새로운 버전이다.
- Schedule: Cron 작업(예약된 스크립트)

![12-eventbridge-cron-job.png](images%2F12-eventbridge-cron-job.png)

- Event Pattern: 서비스가 수행하는 작업에 반응하는 이벤트 규칙

![13-eventbridge-event-pattern.png](images%2F13-eventbridge-event-pattern.png)

- Lambda Function 트리거, SQS/SNS 메시지 전송

#### Rules

![14-eventbridge-rule.png](images%2F14-eventbridge-rule.png)

- EventBridge는 EC2 인스턴스, S3 등의 서비스로부터 이벤트를 수신하여 여러 AWS 서비스로 전달할 수 있다.

![15-eventbridge.png](images%2F15-eventbridge.png)

- 리소스 기반 정책을 사용하여 다른 AWS 계정에서 이벤트 버스에 액세스할 수 있다.
- 이벤트 버스로 전송된 이벤트를 원하는 기간만큼 보관할 수 있다.
- 보관된 이벤트를 재실행하는 기능을 제공한다.

#### Schema Registry

- EventBridge는 버스의 이벤트를 분석하고 스키마를 추론할 수 있다.
- 스키마 레지스트리를 사용하면 이벤트 버스에서 데이터가 어떻게 구성되어 있는지 미리 알 수 있는 애플리케이션용 코드를 생성할 수 있다.
- 스키마 버전 관리가 가능하다.

![16-eventbridge-schema-registry.png](images%2F16-eventbridge-schema-registry.png)

#### Resource-based Policy

- 특정 이벤트 버스에 대한 권한을 관리한다.
- 예를 들어, 다른 AWS 계정 또는 AWS 리전의 이벤트를 허용하거나 거부할 수 있다.
- 단일 AWS 계정 또는 AWS 리전에서 AWS 조직의 모든 이벤트를 집계하는 등의 작업에 사용된다.

![17-resource-based-policy.png](images%2F17-resource-based-policy.png)


#### Service Quotas

- 서비스 할당량 및 임계값에 가까워지면 알림을 생성한다.
- Service Quotas 콘솔에서 CloudWatch 경보를 생성한다.
- 예를 들어, Lambda 동시 실행 수를 설정하여 동시 실행 수가 설정한 수를 넘으면 알람을 발생시킬 수 있다.
- 한도에 도달하기 전에 할당량 증가 또는 리소스 종료를 요청해야 하는지 알 수 있다.

![18-service-quotas.png](images%2F18-service-quotas.png)

#### 대안: Trusted Advisor

- Trusted Advisor에서 제한된 수의 서비스 제한을 최대 50개까지 확인할 수 있다.
- Trusted Advisor는 점검 결과를 CloudWatch에 게시한다.
- 서비스 할당량 사용량에 대한 CloudWatch 경보를 생성할 수 있다.

![19-trusted-advisor-cw-alarms.png](images%2F19-trusted-advisor-cw-alarms.png)

---

### AWS CloudTrail

- AWS 계정에 대한 거버넌스, 규정 준수 및 감사 기능을 제공한다.
- CloudTrail은 기본적으로 활성화되어 있다.
- 아래의 항목을 통해 AWS 계정 내에서 발생한 이벤트/API 호출 기록을 가져온다.
  - 콘솔
  - SDK
  - CLI
  - AWS 서비스
- CloudTrail의 로그를 CloudWatch Logs 또는 S3에 넣을 수 있다.
- 추적은 모든 Region(기본값) 또는 단일 Region에 적용될 수 있다.
- 만약 AWS 리소스가 삭제되었다면 CloudTrail을 확인하면 된다.

![20-cloudtrail-diagram.png](images%2F20-cloudtrail-diagram.png)

#### Events

- CloudTrail에는 세가지 종류의 이벤가 있다.
- **Management Events**
  - AWS 계정의 리소스에 대해 수행되는 작업
  - 예를 들어, 아래와 같은 작업들이 있다.
    - 보안 구성(IAM `AttachRolePolicy`)
    - 데이터 라우팅 규칙 구성(Amazon EC2 `CreateSubnet`)
    - 로깅 설정(AWS CloudTrail `CreateTrail`)
  - **기본적으로 추적은 관리 이벤트를 기록하도록 구성**된다.
  - 읽기 이벤트와 쓰기 이벤트를 분리할 수 있다.
- **Data Events**
  - **기본적으로 데이터 이벤트는 기록되지 않는다.** (대량 작업으로 인해)
  - S3 객체 수준 활동(예: `GetObject`, `DeleteObject`, `PutObject`), 읽기 및 쓰기 이벤트를 분리할 수 있다.
  - AWS Lambda Function 실행 활동(Invoke API)
- **CloudTrail Insights Events**

#### CloudTrail Insights

- CloudTrail Insights를 활성화하여 **계정의 비정상적인 활동을 감지**한다.
  - 부정확한 리소스 프로비저닝
  - 서비스 한도에 도달
  - IAM 작업의 급증
  - 정기 유지 관리 활동의 공백
- CloudTrail Insights는 일반적인 관리 이벤트를 분석하여 기준선을 생성한다.
- 쓰기 이벤트를 지속적으로 분석하여 비정상적인 패턴을 감지한다.
  - CloudTrail 콘솔에 이상이 나타난다.
  - 이벤트가 S3로 전송된다.
  - EventBridge 이벤트가 생성된다. (자동화가 필요한 경우)

![21-cloudtrail-insights.png](images%2F21-cloudtrail-insights.png)

#### Events Retention

- 이벤트는 CloudTrail에 90일 동안 저장된다.
- 이 기간 이후에도 이벤트를 보관하려면 S3에 기록하고 Athena를 사용하면 된다.

![22-cloudtrail-event-retention.png](images%2F22-cloudtrail-event-retention.png)

#### EventBridge와의 통합

![23-eventbridge-intercept-api-call.png](images%2F23-eventbridge-intercept-api-call.png)

- API 호출을 중간에서 가로채기 위해서 EventBridge와 통합될 수 있다.
- DynamoDB의 테이블이 삭제되는 경우 CloudTrail에서 이벤트를 수집하고 EventBridge로 전달하여 SNS를 통해 알람을 받을 수 있다.

![24-eventbridge-cloudtrail.png](images%2F24-eventbridge-cloudtrail.png)

- IAM, EC2 인스턴스와 같은 서비스의 API를 호출하는 경우에도 CloudTrail은 EventBridge와 통합되어 사용자에게 알람을 전송할 수 있다.

![26-integration-with-eventbridge.png](images%2F26-integration-with-eventbridge.png)

- 사용자의 계정에서 발생하는 모든 API 호출에 반응하는 데 사용된다.
- **CloudTrail은 실시간이 아니다.**
  - API 호출 후 15분 이내에 이벤트를 전달한다.
  - 5분마다 S3 버킷에서 로그 파일을 전달한다.

#### Log File 무결성 검증

- **Digest Files**
  - 지난 한 시간 동안의 로그 파일을 참조하고 각 로그 파일의 해시를 포함한다.
  - 로그 파일과 동일한 다른 경로의 S3 버킷에 저장된다.
- CloudTrail이 로그 파일을 전송한 후 로그 파일이 수정/삭제되었는지 확인하는 데 도움이 된다.
- SHA-256을 사용한 해싱, RSA와 함께 SHA-256을 사용한 디지털 서명을 제공한다.
- 버킷 정책, 버전 관리, MFA 삭제 보호, 암호화, 객체 잠금을 사용하여 S3 버킷을 보호한다.
- IAM을 사용하여 CloudTrail을 보호한다.

![25-log-file-integrity-validation.png](images%2F25-log-file-integrity-validation.png)

#### Organizations Trails

- AWS 조직의 모든 AWS 계정에 대한 모든 이벤트를 기록하는 추적이다.
- 관리 및 회원 계정에 대한 로그 이벤트다.
- 모든 AWS 계정에 동일한 이름의 추적이 생성된다. (IAM 허용)
- 멤버 계정은 조직 추적을 제거하거나 수정할 수 없으며 볼 수만 있다.

![27-organizations-trails.png](images%2F27-organizations-trails.png)

---

### AWS Config

- AWS 리소스의 규정 준수를 감사하고 기록하는 데 도움이 된다.
- 시간 경과에 따른 구성 및 변경 사항을 기록하는 데 도움이 된다.
- AWS Config로 아래와 같은 정보들을 확인할 수 있다.
  - 보안 그룹에 대한 무제한 SSH 액세스가 허용되어 있는가
  - 내 버킷에 Public 액세스 권한이 있는가
  - 시간이 지남에 따라 ALB의 구성이 어떻게 변경이 되었는가
- 변경사항에 대한 SNS Notification을 받을 수 있다.
- AWS Config는 Region에 속하는 서비스다.
- Region 및 계정 전반에 걸쳐 집계할 수 있다.
- 구성(Configuration) 데이터를 S3에 저장할 수 있고 Athena를 통해서 분석할 수 있다.

#### Rules

- AWS 관리 구성 규칙(75개 이상)을 사용할 수 있다.
- 사용자 지정 구성 규칙을 만들 수 있다.(AWS Lambda에서 정의해야 함)
  - 예를 들어, 각 EBS 디스크가 gp2 유형인지 평가하거나, EC2 인스턴스가 t2.micro인지 평가할 수 있다.
- 규칙은 각 구성 변경에 대해 평가되거나 트리거 될 수 있다.
- 규칙은 일정한 시간 간격으로 평가되거나 트리거 될 수 있다.
- **AWS Config 규칙은 작업 발생을 사전에 방지하지 않는다.**

#### Config Resource

- 시간 경과에 따른 리소스 규정 준수 확인

![28-resource-compliance-over-time.png](images%2F28-resource-compliance-over-time.png)

- 시간 경과에 따른 리소스 구성 확인

![29-configuration-over-time.png](images%2F29-configuration-over-time.png)

- 시간 경과에 따른 리소스의 CloudTrail API 호출 확인

![30-cloudtrail-over-time.png](images%2F30-cloudtrail-over-time.png)

#### Remediations

- SSM 자동화 문서를 사용하여 비준수 리소스 교정을 자동화한다.
- AWS 관리형 자동화 문서를 사용하거나 사용자 지정 자동화 문서를 생성한다.
  - Lambda Function을 호출하는 사용자 지정 자동화 문서를 생성할 수 있다.
- 자동 교정 후에도 리소스가 여전히 비준수 상태인 경우 교정 재시도를 설정할 수 있다.

![31-rules-remediations.png](images%2F31-rules-remediations.png)

#### Notifications

- AWS 리소스가 규정을 준수하지 않을 때, EventBridge를 사용하여 알림을 트리거한다.

![32-rules-notifications.png](images%2F32-rules-notifications.png)

- 구성 변경 사항 및 규정 준수 상태 알림을 SNS로 보내는 기능을 제공한다.
  모든 이벤트 - SNS 필터링 사용 또는 클라이언트 필터링

![33-rules-notifications-2.png](images%2F33-rules-notifications-2.png)

#### Aggregators

![34-config-aggregators.png](images%2F34-config-aggregators.png)

- 집계자는 하나의 중앙 집계자 계정에 생성된다.
- 여러 계정 및 Region에 걸쳐 규칙, 리소스 등을 집계한다.
- AWS Organizations를 사용하는 경우 개별 인증이 필요하지 않다.
- 규칙은 각 개별 AWS 계정에서 생성된다.
- CloudFormation StackSets을 사용하여 여러 대상 계정에 규칙을 배포할 수 있다.

---

### CloudWatch vs CloudTrail vs Config

- **CloudWatch**
  - 성능 모니터링(지표, CPU, 네트워크 등) 및 대시보드
  - 이벤트 및 경고
  - 로그 집계 및 분석
- **CloudTrail**
  - 모든 사람이 사용자의 계정 내에서 수행한 API 호출을 기록한다.
  - 특정 리소스에 대한 추적을 정의할 수 있다.
  - 글로벌 서비스다.
- **Config**
  - 규성 변경 사항을 기록한다.
  - 규정 준수 규칙에 따라 리소스를 평가한다.
  - 변경 및 규정 준수 일정을 확인한다.

- 로드 밸런서를 예로 들면 아래와 같다.
- **CloudWatch**
  - 들어오는 연결 Metric을 모니터링한다.
  - 시간 경과에 따른 오류 코드를 %로 시각화한다.
  - 로드 밸런서 성능에 대한 아이디어를 얻기 위한 대시보드를 생성한다.
- **Config**
  - 로드 밸런서에 대한 보안 그룹 규칙을 추적한다.
  - 로드 밸런서에 구성 변경 사항을 추적한다.
  - 로드 밸런서에 SSL 인증서가 항상 할당되어 있는지 확인한다. (규정준수)
- **CloudTrail**
  - API 호출을 통해 누가 로드 밸런서를 변경했는지 추적한다.

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [CloudWatch Logs Insights](https://mng.workshop.aws/operations-2022/detect/cwlogs.html)