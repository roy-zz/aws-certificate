# AWS Lambda

이번 장에서는 **SysOps Administrator**를 준비하며 **Lambda**에 대해서 알아보도록 한다.

---

### Lambda 사용 이유

- **EC2**
  - 클라우드의 가상 서버
  - RAM 및 CPU에 의해 제한된다.
  - 지속적으로 실행된다.
  - EC2에서 확장이란 서버를 추가/제거하기 위한 개입을 의미한다.
- **Lambda**
  - 가상 함수 - 관리할 서버가 없다.
  - 시간 제한 - 짧은 실행 시간을 가진다.(최대 15분)
  - 주문형(On-Demand) 실행
  - 스케일링이 자동으로 이루어진다.

#### 이점

- 간편한 가격 정책
  - 요청 및 컴퓨팅 시간당 비용을 지불한다.
  - 1,000,000개의 AWS Lambda 요청과 400,000GB의 컴퓨팅 시간을 제공하는 프리 티어를 사용할 수 있다.
- 전체 AWS 서비스와 통합된다.
- 다양한 프로그래밍 언어와 통합된다.
- AWS CloudWatch를 통해 손쉽게 모니터링할 수 있다.
- 기능(Function)별로 더 많은 리소스를 쉽게 얻을 수 있다.(최대 RAM 10GB)
- RAM을 늘리면 CPU 성능과 네트워크 성능이 향상된다.

- 많은 개발 언어를 지원하고 있다.
  - Node.js (javascript)
  - Python
  - Java (Java 8 compatible)
  - C# (.NET Core)
  - Golang
  - C# / Powershell
  - Ruby
  - Customer Runtime API (커뮤니티 지원)
  - Lambda Container Image
    - 컨테이너 이미지는 "Lambda Runtime API"를 구현해야 한다.
    - 임의의 Docker 이미지를 실행하려면 ECS(Fargate)가 추천된다.

#### 사용 예시

![1-serverless-thumbnail-creation.png](images%2F1-serverless-thumbnail-creation.png)

- S3 버킷에 이미지가 업로드되는 이벤트가 "Lambda Function"을 트리거한다.
- "Lambda Function"은 썸네일 이미지를 S3 썸네일 버킷으로 업로드하고 이미지에 대한 메타데이터를 DynamoDB에 입력한다.

![2-serverless-cron-job.png](images%2F2-serverless-cron-job.png)

- 한 시간마다 실행되는 "CloudWatch Event"가 있다고 가정해본다.
- 만약 EC2 인스턴스에서 서비스가 실행된다면 대부분의 시간은 작업을 하지않지만 컴퓨팅 비용을 지불해야 한다.
- "Lambda Function"을 사용하면 실제로 실행된 시간에 대해서만 비용을 지불하기 때문에 비용최적화를 할 수 있다. 

#### 비용 정보

- 전반적인 가격 정보는 [여기](https://aws.amazon.com/lambda/pricing/)에서 확인할 수 있다.
- 호출 당 비용
  - 처음 1,000,000개의 요청은 무료다.
  - 이후 요청 100만 개당 $0.20를 지불한다.
- 기간 당 지불(1ms 단위)
  - 무료로 매월 400,000GB 초의 컴퓨팅 시간을 제공한다.
  - 1GB RAM을 사용한다면 400,000초를 사용할 수 있다.
  - 128MB RAM을 사용한다면 3,200,000초를 사용할 수 있다.
  - 이후 600,000GB(초) 당 $1.00를 지불해야 한다.
- "AWS Lambda"를 실행하는 것은 일반적으로 매우 저렴하므로 인기가 많다.

---

### CloudWatch Events / EventBridge

![3-cloudwatch-events-eventbridge.png](images%2F3-cloudwatch-events-eventbridge.png)

- Cron 또는 Rate를 사용하여 EventBridge 규칙을 생성하여 1시간마다 "Lambda Function"을 호출할 수 있다.
- "CodePipeline" 규칙을 생성하여 변경이 발생할 때마다 "Lambda Function"을 호출할 수 있다.

---

### S3 Events Notifications

![4-s3-events-notifications.png](images%2F4-s3-events-notifications.png)

- S3:ObjectCreated, S3:ObjectRemoved, S3:ObjectRestore, S3:Replication 등의 알람을 받을 수 있다.
- 객체명 필터링이 가능하다.(예: `*.jpg`)
- S3에 업로드된 이미지의 썸네일을 만드는 작업을 만드는 작업 등에 사용된다.
- S3 이벤트 알림은 일반적으로 몇 초 안에 이벤트를 전달하지만 경우에 따라 1분 이상 걸릴 수도 있다.
- 버전이 지정되지 않은 단일 객체에 동시에 두 개의 쓰기가 수행되는 경우 단일 이벤트 알림만 전송될 수 있다.
- 쓰기가 성공할 때마다 이벤트 알림이 전송되도록 하려면 버킷에서 버전 관리를 활성화해야 한다.

#### Event Pattern - Metadata Sync

![5-s3-events-pattern-metadata-sync.png](images%2F5-s3-events-pattern-metadata-sync.png)

- S3 버킷에 새로운 파일이 업로드되면 Lambda Function으로 알림을 전송한다.
- Lambda Function은 해당 파일을 처리하고 RDS 또는 DynamoDB 테이블에 메타데이터를 저장한다.

---

### Lambda Execution Role (IAM Role)

- IAM Role은 반드시 Lambda Function에 부착되어 있어야 한다.
- AWS 서비스/리소스에 Lambda Function 권한을 부여한다.
  - `AWSLambdaBasicExecutionRole`: CloudWatch에서 로그를 업로드한다.
  - `AWSLambdaKinesisExecutionRole`: Kinesis에서 데이터를 읽는다.
  - `AWSLambdaDynamoDBExecutionRole`: DynamoDB 스트림에서 데이터를 읽는다.
  - `AWSLambdaSQSQueueExecutionRole`: SQS에서 데이터를 읽는다.
  - `AWSLambdaVPCAccessExecutionRole`: VPC에 Lambda 함수를 배포한다.
  - `AWSXRayDaemonWriteAccess`: 추적하는 데이터를 X-Ray에 업로드한다.
- **이벤트 소스 매핑을 사용하여 함수를 호출하면 Lambda는 실행 역할을 사용하여 이벤트 데이터를 읽는다.**
- **모범 사례는 함수 하나당 하나의 Lambda 실행 역할을 생성하는 것이다.**

#### Resource Based Policy

- **리소스 기반 정책을 사용하여 다른 계정과 AWS 서비스에 Lambda 리소스를 사용할 수 있는 권한을 부여한다.**
- S3 버킷에 대한 S3 버킷 정책과 유사하다.
- IAM 주체는 Lambda에 액세스할 수 있다.
  - 보안 주체가 연결된 IAM 정책이 이를 승인하는 경우(예: 사용자 액세스)
  - 리소스 기반 정책이 승인하는 경우(예: 서비스 액세스)
- S3와 같은 AWS 서비스가 Lambda Function을 호출하면 리소스 기반 정책에 따라 액세스 권한이 부여된다.

---

### Lambda Logging & Monitoring

- CloudWatch Logs
  - AWS Lambda 실행 로그는 AWS CloudWatch Logs에 저장된다.
  - **AWS Lmabda Function에 CloudWatch Logs에 대한 쓰기 권한을 부여하는 IAM 정책이 포함된 실행 역할이 있는지 확인해야 한다.**
- CloudWatch Metric
  - AWS Lambda 지표는 AWS CloudWatch 지표에 표시된다.
  - Invocation, Duration, Concurrent Execution
  - Error Count, Success Rate, Throttles
  - Async Delivery Failures
  - IteratorAge(Kinesis & DynamoDB Streams)

#### X-Ray를 통한 추적

- Lambda 구성에서 활성화해야 한다.(활성 추적)
- X-Ray 데몬을 실행한다.
- 코드에서 AWS X-Ray SDK를 사용한다.
- Lambda Function에 올바른 IAM 실행 역할이 있는지 확인한다.
  - 관리형 정책은 `AWSXRayDaemonWriteAccess`다.
- X-Ray와 통신하기 위한 환경변수는 아래와 같다.
  - `_X_AMZN_TRACE_ID`: 추적 헤더를 포함한다.
  - `AWS_XRAY_CONTEXT_MISSING`: 기본적으로 LOG_ERROR
  - `AWS_XRAY_DAEMON_ADDRESS`: X-Ray 데몬 IP_ADDRESS:PORT

---

### Lambda Function Configuration

- RAM
  - 1MB 단위로 128MB ~ 10GB 범위에서 사용할 수 있다.
  - RAM을 증설할수록 vCPU 크레딧을 더 많이 얻게되며, 직접적으로 vCPU의 숫자를 설정할 수 없다.
    암묵적으로 vCPU를 더 얻으려면 RAM을 증설해야 한다.
  - 1,792MB에서 하나의 Function은 전체 vCPU 1개에 해당한다.
  - 1,792MB 이후에는 두 개 이상의 CPU를 얻게 되며, 이를 활용하려면 코드에서 멀티스레딩을 사용해야 한다.(최대 6개의 vCPU 사용 가능)
- **애플리케이션이 CPU-bound가 많은 작업을 한다면 실행 시간을 줄이기 위해서 RAM을 증설해야 한다.**
- Timeout: 기본적으로 3초로 설정되어 있으며 3초가 지나면 오류가 발생한다.
  최대 900초(15분)까지 설정이 가능하다.
- 만약 15분을 초과하는 작업이 있다면 ECS(Fargate) 또는 EC2 인스턴스를 사용하는 것이 적합하다.

#### Execution Context

- 실행 컨텍스트는 Lambda 코드의 외부 종속성을 초기화하는 임시 런타임 환경이다.
- 데이터베이스 연결, HTTP 클라이언트, SDK 클라이언트에 적합하다.
- 실행 컨텍스트는 다른 Lambda 함수 호출을 예상하여 한동안 유지된다.
- 다음 함수 호출에서는 컨텍스트를 실행 시간에 "재사용"하고 연결 객체 초기화 시간을 절약할 수 있다.
- 실행 컨텍스트에는 `/tmp` 디렉터리가 포함된다.

- 아래의 코드는 "좋지 못한" 코드다.

![6-bad-execution-context.png](images%2F6-bad-execution-context.png)

- DB 연결이 함수 호출 시마다 설정된다.
- 사용자를 확보하려면 먼저 데이터베이스에 연결해야 한다.
- Lambda Function이 실행될 때마다 데이터베이스 연결도 실행되야 한다.

- 아래의 코드는 "좋은" 코드다.

![7-good-execution-context.png](images%2F7-good-execution-context.png)

- DB 연결은 한 번 설정되고 호출 전반에 걸쳐 재사용된다.
- 한 번만 초기화되고 함수 호출에서 재사용이 가능하기 때문에 함수 성능이 크게 향상될 수 있다.
- 시험 문제에서는 데이터베이스 연결, HTTP 클라이언트, SDK 클라이언트가 생성된 코드의 위치에 따라 좋은 코드를 구분하는 문제가 출제된다.

#### /tmp space

- Lambda Function이 작동하기 위해 큰 파일을 다운로드해야 하는 경우 사용된다.
- Lambda Function이 작업을 수행하기 위해 디스크 공간이 필요한 경우 사용된다.
- `/tmp` 디렉토리를 사용할 수 있다.
- 최대 10GB 크기까지 사용할 수 있다.
- 실행 컨텍스트가 고정되어도 디렉터리 내용은 그대로 유지되어 여러 호출에서 사용할 수 있는 임시 캐시를 제공한다. (작업을 검사하는데 도움이 됨)
- 객체의 영구적인 지속성이 필요하다면 S3를 사용해야 한다.
- `/tmp`에 저장되는 파일을 암호화하려면 KMS 데이터 키를 생성해야 한다.

---

### Lambda Concurrency & Throttling

- 최대 1,000개까지 동시에 실행이 가능하다.

![8-lambda-concurrency-throttling.png](images%2F8-lambda-concurrency-throttling.png)

- 동시 실행 횟수를 제한할 수 있으며, 권장되는 방식이다.
- 함수 수준에서 "reserved concurrency"을 설정할 수 있다.
- 동시성 제한을 초과하는 각 호출은 "Throttle"를 트리거한다.
- "Throttle" 동작
  - 동기 호출: ThrottleError(429) 반환
  - 비동기 호출: 자동으로 재시도한 후 DLQ로 이동
- 1,000이 넘는 더 많은 한도가 필요하다면 지원 티켓을 사용할 수 있다.

#### Concurrency Issue

- 만약 동시성을 예약(=제한)하지 않으면 아래와 같은 문제가 발생할 수 있다.

![9-lambda-concurrency-issue.png](images%2F9-lambda-concurrency-issue.png)

- 많은 사용자가 ALB를 통해서 요청을 하고 있다.
- 적은 사용자가 API Gateway와 SDK/CLI를 통해서 요청을 하고 있다.
- 요청의 수가 적을 때는 문제가 없지만 요청의 수가 많아져서 ALB에서 사용하는 함수의 수가 1,000개가 된다면 API Gateway와 SDK/CLI를 통해서 처리되는 함수는 확장될 수 없다.
- **계정에 모든 함수에 동시성 한계가 적용된다는 점을 기억해야 한다.**

#### Asynchronous Invocations

![10-concurrency-asynchronous-invocations.png](images%2F10-concurrency-asynchronous-invocations.png)

- S3 버킷에 파일이 많이 업로드되면 Lambda 함수가 동시에 많이 실행된다.
- 만약 한계에 도달해서 확장할 수 없다면 추가 요청은 제한된다.
- 함수에 모든 이벤트를 처리하는데 사용할 수 있는 동시성이 충분하지 않은 경우 추가 요청이 제한된다.
- Throttling Error(429) 및 시스템 오류(5xx)의 경우 Lambda는 이벤트를 대기열에 반환하고 최대 6시간 동안 함수를 다시 실행하려고 시도한다.
- 재시도 간격은 첫 번째 시도 후 1초에서 최대 5분까지 기하급수적으로 늘어난다.

---

### Cold Start & Provisioned Concurrency

- Cold Start
  - 새로운 인스턴스: 코드가 로드되고 핸들러 외부의 코드가 실행된다.(init)
  - 초기화가 큰 경우(code, dependencies, SDK...) 이 프로세스에 시간이 걸릴 수 있다.
  - 새로운 인스턴스에서 제공되는 첫 번째 요청은 나머지 인스턴스보다 지연 시간이 더 길다.
- Provisioned Concurrency
  - 동시성은 함수가 호출되기 전에 할당된다.(미리)
  - 따라서 Cold Start가 발생하지 않으며 모든 호출의 지연 시간이 짧다.
  - Application Auto Scaling은 동시성(일정 또는 목표 활용도)을 관리할 수 있다.
- 2019년 10월/11월에 VPC의 Cold Start가 크게 감소하였다.

![11-reserved-provisioned-concurrency.png](images%2F11-reserved-provisioned-concurrency.png)

---

### Lambda Monitoring - CloudWatch Metrics

- **Invocations**: 함수가 호출된 횟수(성공/실패)
- **Duration**: 함수가 이벤트를 처리하는 데 소요되는 시간
- **Errors**: 오류를 발생시킨 호출의 수
- **Throttles**: 조절(Throttling)된 호출 요청 수(동시성 사용 불가)
- **DeadLetterErrors**: Lambda가 DLQ에 이벤트를 보내지 못한 횟수(비동기 호출)
- **IteratorAge**: Lambda가 레코드를 수신하는 시점과 이벤트 소스 매핑이 이벤트를 함수로 보내는 시점 사이의 시간
  스트림에서 읽는 이벤트 소스 매핑의 경우
- **ConcurrentExecutions**: 이벤트를 처리하는 함수 인스턴스의 수

![12-lambda-metrics-dashboard.png](images%2F12-lambda-metrics-dashboard.png)

#### CloudWatch Alarms

- Lambda 지표를 통해서 여러가지 알람을 구성할 수 있다. 아래는 예시이다.
  - **Invocations** CloudWatch 지표를 사용하여 지난 1시간 동안 Lambda 호출이 없었다는 알람을 생성한다.
  - **Error** CloudWatch 지표를 사용하여 에러가 발생한 경우 알람을 생성한다.
  - **Throttles** CloudWatch 지표를 사용하여 Throttle이 0보다 큰 경우 알람을 생성한다.

#### CloudWatch Logs

![13-cloudwatch-logs.png](images%2F13-cloudwatch-logs.png)

- CloudWatch는 Lambda 관점에서 로깅을 위해서도 사용된다.
- Lambda Function은 고유 로그 그룹이 있는 CloudWatch 로그에 정보를 보낸다.
- 각 Function 인스턴스에는 특정 로그 스트림이 있다.
- 이러한 기능을 활성화시키려면 Lambda Function에 실행 역할이 있어서 로그 그룹을 생성하고, 로그 스트림을 만들고 로그 이벤트를 넣어야 한다.

#### CloudWatch Logs Insights

- CloudWatch Logs Insights를 사용하면 모든 Lambda Function 로그를 검색할 수 있다.
- 아래는 지난 7일 동안 Lambda Function이 얼마나 많은 오류를 발생시켰는지 조회하는 쿼리다.

![14-cloudwatch-logs-insights.png](images%2F14-cloudwatch-logs-insights.png)

- 이외에도 아래의 표를 보면 Insight를 통해서 데이터를 추출하는 여러가지 예시가 있다.

![15-cloudwatch-logs-insights-examples.png](images%2F15-cloudwatch-logs-insights-examples.png)

#### Lambda Insights

- 시스템 메트릭을 수집, 집계, 요약하는 방법이다.
  - **System-level Metrics**: CPU 시간, 메모리, 디스크, 네트워크 같은 시스템 메트릭을 수집하고 요악할 수 있다.
  - **Diagnostic Information**: Cold Start, Lambda Worker Shutdown을 확인할 수 있다.
- Lambda Function 관련 문제를 격리하고 신속하게 해결하는 데 도움이 된다.
- CloudWatch Lambda 확장을 사용한다. (Lambda 계층으로 제공된다.)

![16-lambda-insights.png](images%2F16-lambda-insights.png)

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [Lambda 가격 정보](https://aws.amazon.com/lambda/pricing/)
- [Lambda를 위한 VPC 네트워킹](https://aws.amazon.com/blogs/compute/announcing-improved-vpc-networking-for-aws-lambda-functions/)