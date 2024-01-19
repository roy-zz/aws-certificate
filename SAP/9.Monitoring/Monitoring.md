# Monitoring

- 이번 장에서는 **Solutions Architect Professional (SAP)** 을 준비하며 "모니터링"에 대해서 알아보도록 한다.

---

### CloudWatch

- **CloudWatch Metrics**
  - 많은 AWS 서비스에서 제공한다.
  - EC2 Standard: 5분 간격으로 수집, Detailed Monitoring: 1분 간격으로 수집
  - EC2 RAM은 내장 메트릭이 아니기 때문에 수집되지 않는다.
  - 표준 수집 주기는 1분이며 사용자 지정하여 수집 주기를 1초와 같이 원하는 간격으로 줄일 수 있다.
- **CloudWatch Alarms**
  - 작업 트리거가 가능하다. (EC2 작업: 재부팅, 중지, 종료, 복구 등)
  - Auto Scaling을 지원하고 SNS와 연동할 수 있다.
- **CloudWatch Dashboards**
  - 메트릭 및 경보 표시 기능을 제공한다.
  - 여러 리전의 메트릭을 표시할 수 있다.

#### CloudWatch 알람 통합

![1-cloudwatch-alarms-integrations.png](images%2F1-cloudwatch-alarms-integrations.png)

- CloudWatch 알람을 EC2로 설정할 수 있다.
  - 재부팅, 중지, 종료, 복구 등의 작업에 사용된다.
- EC2 인스턴스를 모니터링하고 상태 체크가 실패하면 EC2 인스턴스를 복구하기 위해 복구 액션을 트리거한다.
  - 복구된 인스턴스는 Private IP와 Public IP가 복구되기 전 인스턴스와 동일하게 유지된다.
- 필요한 시점에 자동으로 확장되도록 Auto Scaling과 통합될 수 있다.
- 알람이 발생할 때, 이메일을 전송하기 위해 SNS와 통합될 수 있다.
- CloudWatch 알람은 EventBridge에 이벤트를 전송할 수 있다.
  - EventBridge를 통해서 Kinesis로 데이터를 전송하거나, Step Function의 워크플로를 트리거하거나, 람다 함수를 트리거할 수 있다.

#### Synthetics Canary

- API, URL, 웹 사이트 등을 모니터링하는 구성 가능한 스크립트다.
- 고객의 영향을 받기 전에 문제를 찾기 위해 고객이 수행하는 작업을 프로그래밍 방식으로 재현한다.
- 엔드포인트의 가용성 및 지연 시간을 확인하고 로드 시간 데이터 및 UI의 스크린샷을 저장할 수 있다.
- CloudWatch 알람과 통합될 수 있다.
- Node.js 또는 Python으로 작성된 스크립트가 사용된다.
- API를 테스트하거나 크롬 브라우저를 통해 바로 액세스할 수 있다.
- 한 번 또는 정기적인 일정으로 실행이 가능하다.

![2-cloudwatch-synthetics-canary.png](images%2F2-cloudwatch-synthetics-canary.png)

- `us-east-1` 리전에서 EC2 인스턴스를 통해 API를 제공하고 있다.
- 사용자는 Route53을 이용하여 EC2 인스턴스에 액세스한다.
- Synthetics Canary는 EC2 인스턴스를 모니터링한다.
  - 예를 들어, 1분 간격으로 API를 호출하거나, 일련의 작업들을 실행하여 상태를 확인한다.
- 만약 상태 확인에 실패하는 경우 사용자도 동일한 실패를 경험할 수 있다.
- 오류가 발생하는 경우 CloudWatch 알람을 트리거하여 Route53의 레코드를 변경하고 다른 EC2 인스턴스로 연결되도록 설정할 수 있다.

#### Synthetics Canary Blueprints

- Heartbeat Monitor: URL 로드, 스크린샷 저장 및 HTTP 아카이브 파일을 확인한다.
- API Canary: REST API의 기본적인 읽기 및 쓰기 기능을 테스트한다.
- Broken Link Checker: 테스트 중인 URL 내부의 모든 링크를 확인한다.
- Visual Monitoring: Canary에서 실행 중에 촬영된 스크린샷과 기준 스크린샷을 비교한다.
- Canary Recorder: CloudWatch Synthetics Recorder와 함께 사용된다. 
  - 웹 사이트에 작업을 기록하고 자동으로 스크립트를 생성한다.
- GUI Workflow Builder: 웹 페이지에서 작업을 수행할 수 있는지 확인한다.
  - 예를 들어, 로그인 양식으로 웹 페이지를 테스트할 수 있다.

---

### CloudWatch Logs

- CloudWatch는 여러 소스들로부터 로그를 수집한다.
  - SDK, CloudWatch Logs Agent, CloudWatch Unified Agent
  - Elastic Beanstalk를 통해 실행된 애플리케이션의 로그
  - ECS로 관리되는 컨테이너들의 로그
  - 람다 함수의 로그
  - VPC Flow Logs의 특정 로그
  - API 게이트웨이
  - 필터 기반의 CloudTrail
  - Route53의 DNS 쿼리 로그

- **Logs Groups**: 임의 이름으로 일반적으로는 애플리케이션을 나타낸다.
- **Log Stream**: 애플리케이션 내 인스턴스, 로그 파일, 컨테이너
- 로그 만료 기간을 설정할 수 있다. (만료되지 않음, 30일 등..)
- 선택적으로 KMS 암호화를 지원한다.
- CloudWatch 로그는 S3, Kinesis Data Stream, Kinesis Data Firehose, Lambda, ElasticSearch로 데이터를 전송할 수 있다.

#### Metric Filter & Insights

- CloudWatch Logs는 필터를 사용할 수 있다.
  - 예를 들어, 로그 내부에서 특정 IP를 찾을 수 있다.
  - 로그에서 "ERROR"가 발생하는 횟수를 계산할 수 있다.
  - 메트릭 필터를 사용하여 알람을 트리거할 수 있다.
- CloudWatch Log Insights를 사용하여 로그를 조회하고 CloudWatch 대시보드에 쿼리를 추가할 수 있다.

![3-cloudwatch-logs-metric-filter-insight.png](images%2F3-cloudwatch-logs-metric-filter-insight.png)

#### S3 Export

![4-cloudwatch-logs-s3-export.png](images%2F4-cloudwatch-logs-s3-export.png)

- S3 버킷은 AES-256(SSE-S3) 또는 SSE-KMS로 암호화해야 한다.
- 로그 데이터를 내보내는데 최대 12시간이 소요될 수 있다.
- `CreateExportTask` API를 호출해야 한다.
- 실시간 또는 거의 실시간이 아니며, CloudWatch 로그 구독을 대신 사용할 수 있다.

#### Logs Subscription

![5-cloudwatch-logs-subscription.png](images%2F5-cloudwatch-logs-subscription.png)

- CloudWatch Logs는 구독 필터(Subscription Filter)를 사용할 수 있고 로그를 필터링할 수 있다.
- 구독 필터는 데이터를 다양한 AWS 서비스로 전달할 수 있다.
  - AWS에 의해 관리되는 람다 함수로 데이터를 전송하여 실시간으로 ElasticSearch로 전송할 수 있다.
  - 구독 필터 위에서 작동하는 람다 함수를 생성하여 원하는 작업을 할 수 있다.
  - 구독 필터에서 Kinesis Data Firehose로 데이터를 전송하여 거의 실시간으로 ElasticSearch나 S3로 데이터를 저장할 수 있다.
  - 구독 필터에서 Kinesis Data Streams로 데이터를 전송하여 KDF, KDA, EC2, 람다 함수로 데이터를 전달할 수 있다.

#### Aggregation (Multi-Account & Multi-Region)

![6-cloudwatch-logs-aggregation.png](images%2F6-cloudwatch-logs-aggregation.png)

- 리전 1에 계정 A, 리전 2에 계정 B, 리전 3에 계정 B가 있다.
- 각 리전과 계정에는 CloudWatch 로그와 구독 필터가 구성되어 있다.
- 각각의 구독 필터는 중앙 로깅 계정의 Kinesis Data Stream으로 로그를 전송한다.
- 전송된 료그는 Kinesis Data Firehose를 통해 거의 실시간으로 S3에 저장된다.

#### Integration with SSM

- SSM 실행 명령을 사용하여 CW 에이전트를 설치한다.

![7-ssm-run-command.png](images%2F7-ssm-run-command.png)

- SSM 상태 관리자를 사용하여 CW 에이전트를 설치한다.

![8-ssm-state-manager.png](images%2F8-ssm-state-manager.png)

- SSM Parameter Store에 `config.`을 저장하여 CW 에이전트를 구성한다.

![9-ssm-parameter-store.png](images%2F9-ssm-parameter-store.png)

---

### Amazon EventBridge

- 예약된 스크립트를 실행하여 Cron 작업을 실행할 수 있다.

![10-eventbridge-cronjob.png](images%2F10-eventbridge-cronjob.png)

- 서비스가 수행하는 것에 대응하기 위한 이벤트 규칙을 생성할 수 있다.

![11-eventbridge-event-pattern.png](images%2F11-eventbridge-event-pattern.png)

- 람다 함수를 트리거하거나 SQS/SNS로 메시지를 전송할 수 있다.

#### EventBridge Rules

![12-eventbridge-rules.png](images%2F12-eventbridge-rules.png)

- 이벤트를 수집할 수 있는 여러 소스가 있다.
  - EC2 인스턴스 (인스턴스 실행)
  - CodeBuild (빌드 실패)
  - S3 Event (객체 업로드)
  - Trusted Advisor (새로운 보안 이슈 발견)
  - CloudTrail (모든 종류의 API 호출)
  - Schedule 또는 Cron (일정 시간 간격)
- 이러한 이벤트들은 EventBridge로 전송되며 필요한 경우 필터를 사용할 수 있다.
- EventBridge는 이벤트에 대한 세부사항을 보여주는 JSON 파일을 생성한다.
- 이러한 JSON 파일은 수많은 목적지로 전송될 수 있다.

#### Event Bus

![13-eventbridge-eventbus.png](images%2F13-eventbridge-eventbus.png)

- AWS에서 제공하는 서비스는 Default 이벤트 버스로 전송된다.
- AWS SaaS 파트너 서비스는 Partner 이벤트 버스로 전송된다.
- 커스텀 애플리케이션은 Custom 이벤트 버스로 전송된다.
- 다른 AWS 계정에서 리소스 기반 정책을 사용하여 이벤트 버스에 액세스할 수 있다.
- 이벤트 버스로 전송된 이벤트를 아카이브할 수 있다.
  - 이벤트는 필터링될 수 있으며 아카이브되는 이벤트는 저장 기간을 설정할 수 있다.
- 보관된 이벤트를 재생하는 기능을 제공한다.

#### Schema Registry

![14-schema-registry.png](images%2F14-schema-registry.png)

- EventBridge는 버스의 이벤트를 분석하여 스키마를 추론할 수 있다.
- 스키마 레지스트리를 사용하면 애플리케이션에 대한 코드를 생성할 수 있으므로 이벤트 버스에서 데이터가 어떻게 구성되는지 미리 알 수 있다.
- 스키마의 버전을 관리할 수 있다.

#### Resource-based Policy

- 특정 이벤트 버스에 대한 사용 권한을 관리할 수 있다.
- 예를 들어, 다른 AWS 계정 또는 AWS 리전의 이벤트를 허용하거나 거부할 수 있다.
- AWS 조직의 모든 이벤트를 단일 AWS 계정 또는 AWS 리전에서 집계하는 작업 등에 사용된다.

![15-eventbridge-resource-based-policy.png](images%2F15-eventbridge-resource-based-policy.png)

---

### AWS X-Ray

- X-Ray는 애플리케이션의 시각 분석과 추적 기능을 제공한다.
- 대기 시간과 오류 비율에 관한 정보를 얻고 네트워크 호출이 인프라에 걸쳐 어떻게 이동하는지 추적할 수 있다.

![16-aws-xray.png](images%2F16-aws-xray.png)

- 마이크로서비스와 같이 분산 시스템 전반에 걸친 요청을 추적할 수 있다.
- 아래의 항목들과 통합될 수 있다.
  - X-Ray 에이전트가 설치된 EC2 인스턴스
  - X-Ray 에이전트 또는 도커 컨테이너가 설치된 ECS
  - 람다
  - Beanstalk (에이전트가 자동으로 설치)
  - API 게이트웨이 (504와 같은 오류를 디버깅하는데 유용)
- X-Ray 에이전트 또는 서비스에는 X-Ray에 대한 IAM 권한이 필요하다.

---

### AWS Personal Health Dashboard

![17-personal-health-dashboard.png](images%2F17-personal-health-dashboard.png)

- 글로벌 서비스다.
- 이벤트가 서비스와 리소스에 어떤 영향을 미치는지 확인하고 싶을 때 사용된다.
- AWS 운영 중단이 사용자에게 직접적으로 어떤 영향을 미치는지 보여준다.
- 문제를 해결하기 위해 수행할 수 있는 작업 목록을 제공한다.
- AWS의 유지 보수 이벤트를 표시한다.
- AWS Health API를 통해 프로그래밍 방식으로 액세스할 수 있다.
- AWS 조직의 여러 계정에 걸친 집계를 제공한다.
- 자세한 내용은 [공식 페이지](https://phd.aws.amazon.com/)에서 확인할 수 있다.

#### Health Event Notification

- EventBridge를 사용하여 AWS 계정의 AWS Health 이벤트 변경 사항에 대응할 수 있다.
- 예를 들어, AWS 계정의 EC2 인스턴스가 업데이트 예정일 떄 이메일 알림을 수신할 수 있다.
- Service Health Dashboard에서 공개 이벤트를 반환하는 데 사용할 수 없다.
- 알림 전송, 이벤트 정보 캡처, 수정 조치 등의 작업에 사용된다.

![18-health-event-notification.png](images%2F18-health-event-notification.png)

---

### 참고한 강의

- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)