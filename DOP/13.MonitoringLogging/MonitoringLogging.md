# Monitoring & Logging

- 이번 장에서는 **DevOps Engineer Professional (DOP)** 을 준비하며 "AWS의 모니터링과 로깅을 위한 서비스들"에 대해서 알아보도록 한다.

---

### Amazon CloudWatch Metrics

- "CloudWatch"는 AWS의 모든 서비스에 대한 메트릭을 제공한다.
- 메트릭은 모니터링할 변수로 CPU 사용률, 네트워크 사용률 등이 있다.
- 메트릭은 네임스페이스에 속한다.
- "Dimension"은 메트릭의 속성(인스턴스 ID, 환경 등)이다.
- 메트릭당 최대 30개의 "Dimension"을 지원한다.
- 메트릭에는 타임스탬프가 있다.
- 메트릭의 CloudWatch 대시보드를 생성할 수 있다.
- CloudWatch 사용자 지정 메트릭을 생성할 수 있다.

#### CloudWatch Metric Streams

- 거의 실시간으로 제공되고 대기 시간이 짧은 상태에서 원하는 목적지로 CloudWatch 메트릭을 지속적으로 스트리밍할 수 있다.
  - "Amazon Kinesis Data Firehose"
  - 3rd Party 서비스 제공자: Datadog, Dynatrace, New Reric, Splunk, Sumo Logic
- 모든 네임스페이스에 모든 지표를 스트리밍하도록 선택하거나 네임스페이스 몇 곳으로만 필터링해 지표의 하위 집합만을 필터링할 수 있다.

![1-cloudwatch-metric-streams.png](images%2F1-cloudwatch-metric-streams.png)

- CloudWatch 지표가 거의 실시간으로 KDF로 스트리밍된다.
- KDF에서 S3 버킷으로 데이터를 전송하고 Athena를 사용해 S3 버킷의 데이터를 분석할 수 있다.
- 아니면 Redshift를 사용해 지표에 데이터 웨어하우징을 적용하거나 OpenSearch를 사용해 OpenSearch 대시보드를 만들거나 분석할 수도 있다.

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

#### Anomaly Detection

- ML 알고리즘을 사용하여 지표를 지속적으로 분석하여 정상 기준선과 이상 징후를 파악한다.
- 과거 데이터를 기반으로 메트릭의 기대값 모델을 생성한다.

![2-anomaly-detection.png](images%2F2-anomaly-detection.png)

- 그래프의 어떤 값이 정상 범위를 벗어났는지를 보여준다.
- 지표의 예상 값(정적 임계값 대신)을 기반으로 경보를 생성할 수 있다.
- 지정된 기간이나 이벤트를 기계 학습에서 제외하는 기능을 제공한다.

#### Amazon Lookout for Metric

- 머신러닝을 사용하여 지표 내 이상 징후를 자동으로 감지하고 근본 원인을 식별한다.
- 수동 개입 없이 데이터 내 오류를 감지하고 진단한다.
- AppFlow를 통해 다양한 AWS 서비스 및 타사 SaaS 앱과 통합된다.
- SNS, Lambda, Slack, Webhooks 등을 지원한다.

![3-lookout-for-metric.png](images%2F3-lookout-for-metric.png)

- 예를 들어 지표데이터는 S3, Redshift에서도 사용할 수 있다.
  - CloudWatch의 과거 데이터나 CloudWatch 지표의 연속 데이터를 둘 수도 있으며 RDS나 Salesforce, Marketo, Zendesk도 마찬가지로 AppFlow를 통해 가능하다.
- Amazon Lookout은 모든 것과 연결되어 머신러닝을 사용해 이상 변경 사항을 알아내고 근본 원인을 파악할 수 있다.

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

![4-cloudwatch-logs-insights.png](images%2F4-cloudwatch-logs-insights.png)

- CloudWatch Logs에 저장된 로그 데이터를 검색 및 분석할 수 있다.
- 예를 들어, 로그 내에서 특정 IP를 찾거나, 로그에서 "ERROR" 발생 횟수를 검색할 수 있다.
- "purpose-built" 쿼리 언어가 제공된다.
  - AWS 서비스 및 JSON 로그에서 필드를 자동으로 검색한다.
  - 원하는 이벤트 필드 가져오기, 조건에 따른 필터링, 집계 통계 계산, 이벤트 정렬, 이벤트 수 제한 등..
  - 쿼리를 저장하고 CloudWatch 대시보드에 추가할 수 있다.
- 서로 다른 AWS 계정의 여러 로그 그룹을 쿼리할 수 있다.
- 실시간 엔진이 아닌 쿼리 엔진이다.

![5-cloudwatch-logs-insights.png](images%2F5-cloudwatch-logs-insights.png)

#### S3 Export

![6-s3-export.png](images%2F6-s3-export.png)

- 로그 데이터를 내보낼 수 있게 되기까지 최대 12시간이 걸릴 수 있다.
- API 호출은 `CreateExportTask`다.
- 배치로 내보내기 때문에 거의 실시간이 아니거나 실시간이 아니다.

#### CloudWatch Logs Subscription

- 처리 및 분석을 위해 CloudWatch Logs에서 실시간으로 로그 이벤트를 가져온다.
- Kinesis Data Streams, Kinesis Data Firehose, Lambda로 전송한다.
- **Subscription Filter**: 어떤 로그가 대상으로 전달되는 이벤트인지 필터링한다.

![7-cloudwatch-logs-subscription.png](images%2F7-cloudwatch-logs-subscription.png)

- 구독 필터는 데이터를 KDS로 전송한다.
- 이 방법이 적합한 경우는 KDF, KDA 또는 EC2 및 람다와 통합하려는 경우다.
- KDF에 직접 전송해 S3로 실시간에 가깝게 전송하거나 OpenSearch 서비스를 사용할 수 있다.
- 람다의 경우 사용자 지정 람다 함수를 직접 작성하거나, 관리형 람다 함수를 사용할 수 있다.
- OpenSearch 서비스에 실시간으로 데이터를 

![8-cloudwatch-logs-subscription.png](images%2F8-cloudwatch-logs-subscription.png)

- 발신자 계정과 수신자 계정이 있고 CloudWatch Logs 구독 필터를 생성하면 구독 대상으로 보내진다.
- 이는 수신자 계정에 있는 KDS의 가상 대표다.
- 그런 다음 대상 액세스 정책을 연결해 첫 번째 계정에서 실제로 이 대상에 데이터를 전송할 수 있게 한다.
- 그런 다음, 수신자 계정에 IAM 역할을 만들고 KDS에 레코드를 전송할 권한을 부여한다.
  - 이 역할은 첫 번째 계정에 주어져야 한다.
- 모든 사항을 준비하면 한 계정에 있는 CloudWatch Logs의 데이터를 다른 계정의 대상으로 전송할 수 있다.

![9-cloudwatch-logs-subscription-2.png](images%2F9-cloudwatch-logs-subscription-2.png)

#### CloudWatch Logs Aggregations - Multi-Account & Multi Region

![10-cloudwatch-logs-aggregation.png](images%2F10-cloudwatch-logs-aggregation.png)

- 서로 다른 계정과 리전에 대해 CloudWatch Logs에서 데이터를 집계해 공통 대상으로 보내는 것도 가능하다.
- Subscription Filter를 통해 CloudWatch Logs에서 특정 계정이나 Region의 Kinesis Data Stream으로 데이터를 집계할 수 있다.
- 거의 실시간에 가깝게 S3로 데이터를 집계할 수 있다.

#### Metric Filter

- CloudWatch Logs는 필터를 사용할 수 있다.
  - 예를 들어, 로그 내부에서 특정 IP를 찾을 수 있다.
  - 로그에서 "ERROR"가 발생하는 횟수를 계산할 수 있다.
  - 메트릭 필터를 사용하여 알람을 트리거할 수 있다.
- 필터는 데이터를 소급하여 필터링하지 않는다.
  - 필터는 필터가 생성된 후에 발생한 데이터에 대해 측정항목 데이터 포인트만 게시한다.
- 지표 필터에 대해 최대 3개의 Dimension을 지정하는 기능이 있다.(선택 사항)

![11-cloudwatch-logs-metric-filter.png](images%2F11-cloudwatch-logs-metric-filter.png)

- CloudWatch Logs 에이전트가 EC2 인스턴스에 설치되어 있다.
- CloudWatch Logs에서 로그를 스트리밍한다.
- 선택한 필터 식에 따라 실제 CloudWatch 지표가 생성된다.
- 예를 들어, 이것을 이용하여 CloudWatch 알람과 통합하여 로그에서 1분 안으로 오류가 다섯 번 발생하면 이 상황을 알려주도록 SNS 토픽에서 경고를 받게 구축할 수 있다.

---

### 모든 종류의 로그

- Application Logs
  - 애플리케이션 코드에 의해 생성된 로그를 수집한다.
  - 사용자 정의 로그 메시지, 스택 추적 등이 포함되어 있다.
  - 파일 시스템의 로컬 파일에 기록된다.
  - 일반적으로 EC2의 CloudWatch 에이전트를 사용하여 CloudWatch Logs로 스트리밍된다.
  - 람다를 사용하는 경우 CloudWatch Logs와 직접 통합될 수 있다.
  - ECS 또는 Fargate를 사용하는 경우 CloudWatch Logs와 직접 통합될 수 있다.
  - Elastic Beanstalk를 사용하는 경우 CloudWatch Logs와 직접 통합될 수 있다.
- Operating System Logs (이벤트 로그, 시스템 로그)
  - 운영 체제(EC2 or On-Premise 인스턴스)에서 생성된 로그를 수집한다.
  - 시스템 동작 알림을 수집한다. (예 `/var/log/messages` or `/var/log/auth.log`)
  - 일반적으로 CloudWatch 에이전트를 사용하여 CloudWatch Logs로 스트리밍된다.
- Access Logs
  - 사람들이 웹사이트에서 요청한 개별 파일에 대한 모든 요청 목록을 수집한다.
  - 예를 들어, 아파치 웹서버의 액세스 로그는 `/var/log/apache/access.log`에 저장된다.
  - 일반적으로 로드 밸런서, 프록시, 웹 서버 등에 사용된다.
  - AWS는 일부 액세스 로그를 제공한다.

#### AWS Managed Logs

- Load Balancer Access Logs (ALB, NLB, CLB): S3에 저장
  - 로드 밸런서에 대한 액세스 로그를 저장한다.
- CloudTrail Logs: S3 및 CloudWatch Logs에 저장
  - 계정 내에서 이루어진 API 호출에 대한 로그를 저장한다.
- VPC Flow Logs: S3 및 CloudWatch Logs에 저장
  - VPC의 네트워크 인터페이스를 오가는 IP 트래픽에 대한 정보를 저장한다.
- Route 53 액세스 로그: CloudWatch Logs에 저장
  - Route 53이 수신하는 쿼리에 대한 로그 정보를 저장한다.
- S3 Access Logs: S3에 저장
  - 서버 액세스 로깅은 버킷에 대한 요청에 대한 자세한 기록을 제공한다.
- CloudFront Access Logs: S3에 저장
  - CloudFront가 수신하는 모든 사용자 요청에 대한 세부 정보를 저장한다.

#### CloudWatch Logs for EC2

- 기본적으로 EC2 시스템의 로그가 CloudWatch로 전송되지 않는다.
- EC2에서 CloudWatch 에이전트를 실행하여 원하는 로그 파일을 푸시할 수 있다.
- IAM 권한이 올바르게 설정되어 있는지 확인해야 한다.
- CloudWatch 로그 에이전트를 온프레미스에서 실행 중인 서비스에도 설정할 수 있다.

![12-cloudwatch-logs-ec2.png](images%2F12-cloudwatch-logs-ec2.png)

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

#### Unified Agent - Metrics

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

#### CloudWatch Alarm Targets

- 알람의 주요 목표는 세 가지다.
- 첫 번째는 EC2 인스턴스 상의 작업으로 중단, 종료, 재부팅하거나 인스턴스 복구 등이 있다.
- 두 번째는 자동 크기 조정 작업을 트리거하여 확장이나 축소할 수 있다.
- 마지막은 SNS 서비스에 알림을 보낼 수 있다.
  - 예를 들어, SNS 서비스에 람다 함수로 연결하여 임계값을 넘은 알림에 따라 람다 함수로 원하는 작업을 수행할 수 있다.

![13-cloudwatch-alarm-targets.png](images%2F13-cloudwatch-alarm-targets.png)

#### Composite Alarms

- "CloudWatch Alarms"가 단일 메트릭에 있다.
- 복합 알람(Composite Alarms)은 여러 개의 다른 알람의 상태를 모니터링한다.
- AND 또는 OR 조건을 설정할 수 있다.
- 복잡한 복합 알람을 생성하여 "알람 소음(alarm noise)"을 줄이는데 도움이 될 수 있다.

![14-cloudwatch-alarm-composite.png](images%2F14-cloudwatch-alarm-composite.png)

- EC2 인스턴스가 있고 복합 알람을 만드는 상황이다.
- 첫 번째 기본 알람을 만들어 Alarm-A라 부르고, 이 알람은 EC2 인스턴스의 CPU를 모니터링한다.
- 그 다음 Alarm-B를 만들고 EC2 인스턴스의 IOPS를 모니터링한다.
- 그런 다음 복합 알람은 Alarm-A와 Alarm-B의 연결 지점으로 정의된다.
  - Alarm-A가 ALARM 상태이고, Alarm-B가 ALARM이라고 가정한다.
  - 이런 경우 복합 알람 자체도 ALARM 상태가 되어 SNS 알림을 트리거할 수 있다.

#### EC2 Instance Recovery

- Status Check
  - 인스턴스 상태: EC2 VM의 상태를 확인한다.
  - 시스템 상태: 기반 하드웨어의 상태를 확인한다.
- Recovery: 동일한 Private, Public, Elastic IP, metadata, 배치 그룹으로 복구한다.

![15-ec2-instance-recovery.png](images%2F15-ec2-instance-recovery.png)

- EC2 VM을 확인하기 위한 상태 검사가 있고 시스템 상태 검사는 기본 하드웨어를 검사하며 두 확인 작업 모두 CloudWatch 알람을 정의할 수 있다.
- 알람 임계값이 넘는 경우 EC2 인스턴스 복구를 시작하여 EC2 인스턴스를 하나의 호스트에서 다른 호스트로 옮길 수 있다.
- 복구된 인스턴스는 Private IP, Public IP, Elsatic IP가 동일하고 메타데이터도 같으며 인스턴스 그룹 위치도 동일하다.
- SNS 토픽에 경보를 보내 EC2 인스턴스가 복구되었다고 알릴 수도 있다.

#### Good to Know

- "CloudWatch Logs Metrics Filters"를 기반으로 알람을 생성할 수 있다.
- 경보 및 알람을 테스트 하려면 CLI를 통해서 진행할 수 있다.
  - `aws cloudwatch set-alarm-state --alarm-name "myalarm" --state-value ALARM --state-reason "testing purposes"`

![16-cloudwatch-alarm-good-to-know.png](images%2F16-cloudwatch-alarm-good-to-know.png)

- CloudWatch Logs에는 지표 필터가 있다.
  - 지표 필터는 CloudWatch 알람에 연결되어 있다.
  - 특정 인스턴스에서 Error라는 단어를 많이 받는 경우 SNS로 메시지를 보낼 수 있다.

#### CloudWatch Synthetics Canary

- API, URL, 웹사이트 등을 모니터링하는 구성 가능한 스크립트다.
- 고객이 영향을 받기 전에 문제를 찾기 위해 고객이 수행하는 작업을 프로그래밍 방식으로 재현한다.
- 엔드포인트의 가용성과 대기 시간을 확인하고 UI의 로드 시간 데이터와 스크린샷을 저장할 수 있다.
- CloudWatch Alarm과 통합할 수 있다.
- 스크립트는 Node.js 또는 Python으로 작성할 수 있다.
- 헤드리스 Google Chrome 브라우저에 프로그래밍 방식으로 액세스할 수 있다.
- 한 번 또는 정기적으로 실행할 수 있다.

![17-cloudwatch-synthetics-canary.png](images%2F17-cloudwatch-synthetics-canary.png)

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

### Amazon Athena

- S3에 저장된 데이터를 분석하는 서버리스 쿼리 서비스다.
- 표준 SQL 언어(Presto 기반)를 사용하여 파일을 조회할 수 있다.
- CSV, JSON, ORC, Avro 및 Parquet를 지원한다.
- 스캔한 데이터 1TB당 $5.00를 지불해야 한다.
- 일반적으로 보고/대시보드용으로 Amazon Quicksight와 함께 사용된다.
- 비지니스, 인텔리전스/분석/보고, VPC Flow Logs, ELB Logs, CloudTrail 등을 분석 및 쿼리할 때 사용한다.

![18-athena.png](images%2F18-athena.png)

#### 성능 향상

- 비용 절감을 위해 컬럼 데이터를 사용하여 스캔되는 양을 줄인다.
  - Apache Parquet 또는 ORC가 권장된다.
  - 성능이 크게 향상된다.
  - Glue를 사용하여 데이터를 Parquet 또는 ORC로 변환할 수 있다.
- 소규모 검색을 위해 데이터 압축(bzip2, gzip, lz4, snapy, zlip, zstd 등)을 지원한다.
- 가상 컬럼에서 쉽게 조회할 수 있도록 S3의 파티션 데이터 셋을 지원한다.
  - `s3://yourBucket/pathToTable`
  - `s3://athena-examples/flight/parquet/year=1991/month=1/day=1/`
- 128MB보다 더 큰 파일을 사용하여 오버헤드를 최소화하고 성능을 향상시킬 수 있다.
  - S3에 아주 작고 많은 파일이 있는 경우보다 큰 파일을 사용하는 경우가 더 높은 성능이 나온다.

#### Federated Query

- 관계형, 비관계형, 객체형 및 사용자 정의 데이터 소스(AWS 또는 온프레미스)에 저장된 데이터 전반에 걸쳐 SQL 쿼리를 실행할 수 있다.
- AWS 람다에서 실행되는 데이터 소스 커넥터를 사용하여 연합 쿼리를 실행한다. (예: CloudWatch Logs, DynamoDB, RDS 등)
- 결과를 S3에 다시 저장한다.

![19-athena-federated-query.png](images%2F19-athena-federated-query.png)

---

### 참고한 강의

- [DevOps Engineer Professional](https://www.udemy.com/course/aws-certified-devops-engineer-professional-korean)
- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)