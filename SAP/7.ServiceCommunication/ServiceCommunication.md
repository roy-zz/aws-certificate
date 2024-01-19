# Service Communication

- 이번 장에서는 **Solutions Architect Professional (SAP)** 을 준비하며 "서비스간 통신"에 대해서 알아보도록 한다.

---

### AWS Step Functions

- 서버리스로 시각적 워크플로를 만들어 람다 함수를 구성하는 것을 돕는다.
- 흐름을 "JSON state machine"으로 구성한다.
- 시퀀스, 병렬작업, 조건, 타임아웃, 에러 핸들링 등의 기능을 제공한다.
- 최대 실행 시간은 1년이다.
- 계속 작업을 진행하고 싶다면 사람이 승인하도록 기능을 구현할 수 있다.
- "Step Function"을 사용하여 람다 함수를 연결하는 경우 모든 호출 사이에 대기 시간이 있다는 것을 염두에 두어야 한다.

![1-workflow-step-function.png](images%2F1-workflow-step-function.png)

- "JSON state machine"에서 AWS 클라우드가 생성되는 그림이다.
- 성공, 실패, 취소, 완료가 되었을 경우 다음 단계로 진행되는 단계별 함수의 전체 흐름을 볼 수 있다.

#### Integration

- **Optimized Integrations**
    - 람다 함수를 호출할 수 있다.
    - AWS 배치 작업을 실행할 수 있다.
    - ECS 작업을 실행하고 완료될 때까지 대기한다.
    - DynamoDB에서 항목을 삽입한다.
    - SNS, SQS에 메시지를 게시한다.
    - EMR, Glue 또는 SageMaker 작업을 시작한다.
    - 다른 Step Function의 워크플로를 시작한다.
- **AWS SDK Integrations**
    - "State Machine"에서 200개 이상의 AWS 서비스에 액세스할 수 있다.
    - 표준 AWS SDK API 호출처럼 작동한다.

![2-step-function-integration.png](images%2F2-step-function-integration.png)

#### Workflow Triggers

- 다음의 항목들을 활용하여 Step Function Workflow(State Machine)을 호출할 수 있다.
    - AWS Management Console
    - AWS SDK (`StartExecution` API 호출)
    - AWS CLI (`start-execution`)
    - AWS Lambda (`StartExecution` API 호출)
    - API Gateway
    - EventBridge
    - CodePipeline
    - Step Function

![3-workflow-trigger.png](images%2F3-workflow-trigger.png)

#### Sample Project

- [AWS 콘솔](https://console.aws.amazon.com/states/home?region=us-east-1#/sampleProjects)에서 많은 샘플 프로젝트를 확인할 수 있다.

![4-sample-project.png](images%2F4-sample-project.png)

- 수많은 샘플 프로젝트를 참고해서 원하는 Step Function을 구축할 수 있다.
    - SQS에서 발생하는 대량의 메시지를 처리하는 람다 함수를 구축하는 프로젝트.
    - SageMaker, 람다, S3를 동기화하며 러닝머신 모델 훈련하는 프로젝트.
    - Fargate 작업을 실행하여 컨테이너 작업을 실행하는 프로젝트.

#### Tasks

- **Lambda Task**
    - 람다 함수츷 호출한다.
- **Activity Task**
    - 활동 작업자(HTTP, Activity Worker)를 만들어야 한다.
        - EC2 인스턴스, 모바일 장치, On-Premise 데이터 센터로 사용될 수 있다.
    - Step Function 서비스를 끌어오고 Step Function에서 액티비티를 끌어온다.
- **Service Task**
    - 지원되는 AWS 서비스에 연결한다.
    - 람다 함수, ECS 태스크, Fargate, DynamoDB, Batch job, SNS 토픽, SQS 큐
- **Wait Task**
    - 기간 또는 타임스탬프가 표시될 때까지 대기한다.
- **Step Function은 AWS Mechanical Turk와 기본적으로 통합되지 않는다.**

#### Standard vs Express

- 워크플로의 종류에는 "Standard"와 "Express"가 있으며 차이점은 아래의 표와 같다.

![5-standard-express.png](images%2F5-standard-express.png)

#### Express Workflow - Synchronous vs Asynchronous

- **Synchronous Express Workflows**
    - 워크플로가 완료될 떄까지 기다렸다가 결과를 반환한다.
    - 예: 마이크로서비스 오케스트레이션, 에러 핸들링, 재시도, 병렬 작업 등..

![6-express-workflow-synchronous.png](images%2F6-express-workflow-synchronous.png)

- 동기 방식의 워크플로는 아래와 같이 작동한다.
    - 사용자가 API 게이트웨이에 요청한다.
    - API 게이트웨이는 워크플로를 실행하고 결과를 받을 때까지 대기한다.
    - 결과를 받은 API 게이트웨이는 사용자에게 결과를 전달한다.

- **Asynchronous Express Workflows**
    - 워크플로가 완료되기를 기다리지 않는다.
    - 예: 즉각적인 응답, 메시징 등이 필요 없는 워크플로 등..

![7-express-workflow-asynchronous.png](images%2F7-express-workflow-asynchronous.png)

- 비동기 방식의 워크플로는 아래와 같이 작동한다.
    - 사용자가 API 게이트웨이로 요청을 보낸다.
    - API 게이트웨이는 워크플로를 실행시키고 실행되었다는 응답을 반환받는다.
    - API 게이트웨이는 정상적으로 워크플로가 실행되었다는 응답을 사용자에게 반환한다.

#### Error Handling

- 오류 처리, 재시도 및 "Step Function State Machine"에 경고를 추가할 수 있다.
- 예: "State Machine" 실행이 실패하는 경우 이메일을 통해 알림을 받도록 EventBridge를 설정할 수 있다.

![8-error-handling.png](images%2F8-error-handling.png)

- 사용자는 State Machine을 호출해 람다 함수를 호출한다.
- 람다 함수가 때때로 실패할 수 있다는 것을 알지만 다시 시도할 수 있다.
    - 실패하는 경우를 대비하여 재시도 블록을 미리 추가할 수 있다.
    - 예시에서는 실패하는 경우 최대 2번까지 재시도한다.
- 재시도 횟수를 초과하는 경우 SNS 토픽을 호출하도록 설정할 수 있다.
    - SNS는 이메일을 통해서 사용자에게 작업이 실패하였음을 전달한다.

#### Solutions Architecture

![9-step-functions-solution-architecture.png](images%2F9-step-functions-solution-architecture.png)

- AWS SDK/CLI를 통해서 Step Function을 트리거할 수 있다.
- Step Function은 "API 게이트웨이"와 서비스 프록시로 통합될 수 있다.
    - HTTP REST API를 호출하는 모든 사용자는 기능을 비동기 또는 동기 방식으로 실행할 수 있다.
- EventBridge 통합을 사용하여 어떠한 서비스라도 Step Function 워크플로를 트리거하도록 구성할 수 있다.
- "Step Function State Machine"은 SQS 큐, 람다 함수, DynamoDB와 같이 많은 AWS 서비스와 통합될 수 있다.

---

### Amazon SQS

- 서버리스로 관리형 큐 서비스이며 IAM과 통합되어 있다.
- 프로비저닝이 필요 없는 극한의 확장성을 통하여 수많은 데이터를 처리할 수 있다.
- 서비스를 분리(디커플링)하는 용도로 사용된다.
- 최대 256KB의 메시지 크기를 지원한다.
    - 만약 최대 크기를 초과하는 파일을 전송하고 싶다면 S3에 파일을 업로드하고 객체 키를 SQS로 전달하는 방법을 사용할 수 있다.
- EC2 인스턴스(선택적으로 ASG), 람다 함수에서 데이터를 읽을 수 있다.
- DynamoDB의 쓰기 버퍼로 사용될 수 있다.
- SQS FIFO
    - 발송된 순서대로 메시지를 처리한다.
    - 초당 300개의 메시지를 처리할 수 있으며, 배치를 사용하는 경우 초당 3,000개의 메시지를 처리할 수 있다.

#### Dead Letter Queue (DLQ)

- 소비자가 가시성 시간 초과 내에 메시지를 처리하지 못하는 경우 메시지는 다시 대기열로 들어간다.
- 메시지가 대기열로 돌아갈 수 있는 횟수의 임계값을 설정할 수 있다.
- 최대 임계값인 `MaximumReceives`를 초과하면 메시지가 DLQ(Dead Letter Queue)에 들어간다.
- 디버깅에 유용하게 사용된다.
- **FIFO 대기열의 경우 DLQ도 FIFO 대기열이어야 한다.**
- **Standard 대기열의 경우 DLQ도 Standard 대기열이어야 한다.**
- DLQ 메시지가 만료되기 전에 메시지를 처리해야 한다.
    - DLQ 메시지의 리텐션 기간을 설정하여 메시지가 처리되지 못한 이유를 찾는 것이 좋다. (예: 14일)

![10-dead-letter-queue.png](images%2F10-dead-letter-queue.png)

#### DLQ - Redrive to Source

- DLQ에서 메시지를 소비하여 문제가 된 부분을 이해하는 데 도움이 되는 기능이다.
- 코드가 수정되면 사용자 지정 코드를 작성하지 않고도 DLQ의 메시지를 소스 큐 또는 다른 큐로 다시 전달할 수 있다.

![11-dlq-redrive-to-source.png](images%2F11-dlq-redrive-to-source.png)

#### 중복 처리

- 메시지는 소비자가 두 번 처리할 수 있다. (장애, 시간 초과 등의 경우)
- 해당 문제를 방지하기 위해 소비자 차원에서 "idempotency"를 구현한다.
- 같은 메시지를 두 번 처리한다고 하더라도 결과는 동일하도록 구현한다는 것을 의미한다.

![12-sqs-idempotency.png](images%2F12-sqs-idempotency.png)

- EC2 인스턴스 소비자가 비용과 지연 시간을 최소화하기 위하여 "SQS에서 롱폴링"하여 메시지를 가져온다.
- 데이터를 DynamoDB에 입력하면 EC2 인스턴스 소비자는 다른 소비자가 처리하지 못하도록 SQS의 메시지를 지운다.
- 하지만 여러가지 이유로 EC2 인스턴스 소비자는 DynamoDB에 데이터를 여러번 쓸 수 있다.
    - 동일한 이벤트에 대해서 DynamoDB에 동일한 데이터를 삽입하는 것은 바람직한 프로그래밍이 아니다.
    - DynamoDB의 기본 키를 활용하여 이미 동일한 데이터가 있는 경우 새로운 데이터를 삽입하는 것이 아니라 덮어쓰거나 업데이트하도록 구현할 수 있다. (Upsert)

#### Event Source Mapping - SQS & SQS FIFO

- "Event Source Mapping"은 SQS에서 배치를 폴링하여 람다 함수를 호출한다.

![13-event-source-mapping.png](images%2F13-event-source-mapping.png)

- "Event Source Mapping"은 SQS를 폴링하거나 롱폴링한다.
- 1 ~ 10개의 메시지로 배치의 크기를 조절할 수 있다.
- "대기열 가시 시간 초과"를 "람다 함수 시간 초과"의 6배로 설정하는 것이 권장된다.
- DLQ를 사용하려면
    - 람다가 아닌 SQS 큐 레벨에서 설정한다. 람다의 DLQ는 비동기 호출에만 해당된다.
    - 실패한 경우를 대비한 다른 람다 함수를 구성한다.

#### Request / Response (async)

- 요청과 응답을 비동기로 분리(디커플링)하는 방법을 알아본다.

![14-request-response-async.png](images%2F14-request-response-async.png)

- 클라이언트가 요청을 "SQS Request Queue"에 집어넣고 대기하지 않는다.
- 워크 프로세서는 "SQS Request Queue"에서 데이터를 가져와 처리하고 결과를 "SQS Response Queue"로 전달한다.
- 클라이언트는 "SQS Response Queue"에서 요청에 대한 결과를 가져온다.

---

### Amazon MQ

- SQS와 SNS는 AWS의 독점 프로토콜인 "클라우드 네이티브" 서비스다.
- On-Premise에서 실행되는 기존 애플리케이션은 다음과 같은 개방형 프로토콜을 사용할 수 있다.
    - MQTT, AMQP, STOMP, Openwire, WSS
- 클라우드로 마이그레이션할 때, SQS와 SNS를 사용하도록 애플리케이션을 재설계하는 대신 Amazon MQ를 사용할 수 있다.
- Amazon MQ는 RabbitMQ 및 ActiveMQ의 관리형 메시지 브로커 서비스다.
- Amazon MQ는 SQS/SNS만큼 "스케일링"하지 않는다.
- Amazon MQ는 서버에서 실행되며 Multi-AZ에서 페일오버를 실행할 수 있다.
- Amazon MQ에는 대기열 기능(SQS)과 토픽 기능(SNS)가 모두 있다.

#### Re-platform

- IBM MQ, TIBCO EMS, Rabbit MQ 및 Apache Active MQ를 Amazon MQ로 마이그레이션할 수 있다.
- 마이그레이션에 대한 자세한 내용은 [공식문서](https://aws.amazon.com/blogs/compute/migrating-from-ibm-mq-to-amazon-mq-using-a-phased-approach/)를 참고한다.

![15-amazonmq-replatform.png](images%2F15-amazonmq-replatform.png)

---

### Amazon SNS

![16-amazonsns.png](images%2F16-amazonsns.png)

- 메시지 소비자들과 직접적으로 통합되야 한다면 소비자가 늘어날 때마다 메시지를 전달하는 목적지를 추가해야 한다.
- 만약 Pub/Sub 방식으로 SNS 토픽을 사용하고 메시지를 소비해야 하는 새로운 소비자가 추가된다면 기존의 코드는 변경할 필요없이 새로운 소비자가 SNS 토픽을 바라보도록 구현하면 된다.

![17-amazonsns.png](images%2F17-amazonsns.png)

- "이벤트 생산자(Producer)"는 하나의 SNS 토픽에만 메시지를 보낸다.
- SNS 토픽 알림을 받고 싶은 "이벤트 구독자(Subscriber)"를 원하는만큼 추가할 수 있다.
- 토픽에 대한 각 구독자는 모든 메시지를 받게 된다.
    - 필요에 따라 메시지를 필터링할 수 있다.
- 토픽은 최대 12,500,000개의 구독자를 가질 수 있다.
- 계정당 토픽은 100,000개로 제한된다.

#### AWS 서비스와의 통합

- 많은 AWS 서비스는 알림을 위해 데이터를 SNS로 직접 전송할 수 있다.

![18-sns-integrate-aws-service.png](images%2F18-sns-integrate-aws-service.png)

#### 메시지 게시(Publish) 방식

- Topic Publish (SDK 사용)
    - 토픽을 생성한다.
    - 여러개의 구독을 생성한다.
    - 토픽에 데이터를 게시한다.
- Direct Publish (모바일 앱 SDK 사용)
    - 플랫폼 애플리케이션을 생성한다.
    - 플랫폼 엔드포인트를 생성한다.
    - 플랫폼 엔드포인트를 게시한다.
    - Google GCM, Apple APNS, Amazon ADM 등이 모바일 앱에 알림을 전송할 수 있다.

#### Security

- **Encryption**
    - HTTPS API를 사용하여 전송 중 암호화를 사용한다.
    - KSM 키를 사용하여 저장할 때 암호화할 수 있다.
    - 클라이언트가 직접 암호화/복호화를 수행하려는 경우 클라이언트 사이드 암호화를 할 수 있다.
- **Access Control**
    - SNS API 접근을 제한하기 위한 IAM 정책을 생성할 수 있다.
- **SNS Access Policy** (S3 버킷 정책과 유사)
    - SNS 토픽에 대한 계정 간 액세스에 유용하다.
    - 다른 서비스가 SNS 토픽에 글을 쓸 수 있도록 허용하는 데 유용하다.

#### SNS + SQS - Fan Out

![19-sns-sqs-fan-out.png](images%2F19-sns-sqs-fan-out.png)

- SNS에 한 번 푸시하면 구독자인 모든 SQS 대기열에서 수신하도록 구축할 수 있다.
- 데이터 손실 없이 완벽하게 분리된다.
- SQS를 통해서 "데이터 지속성", "지연된 처리", "작업 재시도"가 가능하다.
- 서비스가 확장함에 따라 더 많은 SQS 구독자를 추가할 수 있다.
- SQS 대기열 액세스 정책이 SNS에서 쓰기를 허용하는지 확인해야 한다.
- 서로 다른 리전에 있는 SQS에 메시지를 전달할 수 있다.

#### S3 이벤트 다중 대기열로 전달

- 이벤트 유형(예: 객체 생성)과 접두사(예: `images/`)의 동일한 조압의 경우 S3 이벤트 규칙을 하나만 가질 수 있다.
- 동일한 S3 이벤트를 여러 SQS 대기열에 전송하려면 Fan-out을 사용하면 된다.

![20-sqs-events-multiple-queue.png](images%2F20-sqs-events-multiple-queue.png)

- S3 버킷에 객체가 생성되면 이벤트를 SNS 토픽으로 전달한다.
- 전달된 이벤트는 SNS 토픽 모든 구독자인 SQS 큐, 람다 함수로 전달된다.

#### KDF를 통해 S3 전달

- SNS는 Kinesis로 데이터를 전달할 수 있기 때문에 아래와 같은 아키텍처가 가능하다.

![21-sns-s3-kinesis-data-firehose.png](images%2F21-sns-s3-kinesis-data-firehose.png)

- Buying 서비스가 SNS 토픽으로 데이터를 전송한다.
- SNS를 통해 데이터가 Kinesis Data Firehose로 전달된다.
- Kinesis Data Firehose에서 S3 버킷이나 지원되는 목적지로 데이터를 전송할 수 있다.

#### FIFO 토픽

- FIFO는 First In First Out의 약자로 토픽내의 메시지가 정렬된다.

![22-sns-fifo-topic.png](images%2F22-sns-fifo-topic.png)

- SQS FIFO와 유사한 기능이다.
    - "메시지 그룹 ID"를 기반으로 동일한 그룹의 모든 메시지가 정렬된다.
    - "중복 제거 ID" 또는 "컨텐츠 기반 중복 제거"를 통해 중복을 제거한다.
- SQS Standard 및 FIFO 큐를 구독자로 가질 수 있다.
- SQS FIFO와 동일한 제한된 처리량을 가진다.

#### SNS FIFO & SQS FIFO - Fan-out

- Fan-out, 정렬, 중복제거가 모두 필요한 경우 아래와 같은 아키텍처를 설계할 수 있다.

![23-sns-fifo-sqs-fifo.png](images%2F23-sns-fifo-sqs-fifo.png)

- Buying 서비스가 데이터를 SNS FIFO 토픽으로 전달한다.
- SNS FIFO 토픽의 데이터가 SQS FIFO 큐로 전달된다.
- Fraud 서비스와 Shipping 서비스는 각각의 큐에서 메시지를 전달받는다.

#### Message Filtering

- SNS 토픽의 구독자들에게 보내는 메시지를 필터링하는 데 사용되는 JSON 정책이다.
- 구독에 필터링 정책이 없는 경우 모든 메시지를 수신한다.

![24-message-filtering.png](images%2F24-message-filtering.png)

#### Message Delivery Retry

- SNS 구독자에게 메시지를 전달할 때 서버 측 오류가 발생하는 경우 전달 정책이 적용된다.

![25-message-delivery-retry.png](images%2F25-message-delivery-retry.png)

![26-message-delivery-retry.png](images%2F26-message-delivery-retry.png)

- AWS에서 관리하는 엔드포인트를 보면 KDF, 람다, SQS가 있고 실패하는 경우 지체 없이 바로 세 번 재시도한다.
- Pre-backoff 단계에서는 1초 간격으로 두 번 재시도한다.
- 이후 Backoff 단계로 들어가 10번 재시도한다.
- Post-backoff 단계가 되면 20초 간격으로 10만번의 재시도를 한다.
- 계속 실패하는 경우 23일동안 총 100,015번의 재시도를 한다.

- 고객 관리형 엔드포인트의 경우 SMTP 프로토콜을 사용하고, 실패하는 경우 재시도하지 않는다.
- Pre-backoff 단계에서는 10초 간격으로 두 번 재시도한다.
- 이후 Backoff 단계로 들어가면 10초부터 600초 간격으로 10번의 재시도를 한다.
- Post-backoff 단계가 되면 600초 간격으로 38번의 재시도를 한다.
- 계소 실패하는 경우 6시간동안 총 50번의 재시도를 한다.

#### Custom Delivery Policy

- HTTP/S만 지원하는 사용자 지정 정책을 지원한다.

![27-custom-delivery-policy.png](images%2F27-custom-delivery-policy.png)

#### Dead Letter Queue

- 전달 정책을 모두 사용한 후 DLQ를 설정하지 않으면 전달되지 않은 메시지는 폐기된다.
- DLQ는 SQS 큐가 대기하는 것이고 FIFO는 SNS에서 FIFO를 사용하는 경우다.
- DLQ는 SNS 토픽 수준에서 작동하는 것이 아니라 구독자 수준에서 작동한다.

![28-dead-letter-queue.png](images%2F28-dead-letter-queue.png)

- SNS는 메시지를 받고 있고 이메일 구독자와 HTTP 구독자를 가지고 있다.
    - 이메일 구독자에 문제가 생기는 경우 사용될 SQS DLQ를 생성할 수 있다.
    - HTTP 구독자에 문제가 생기는 경우 사용될 SQS DLQ를 생성할 수 있다.

---

### 참고한 강의

- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)