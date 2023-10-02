# Integration & Messaging

이번 장에서는 SAA를 준비하며 **Integration(통합)과 Messaging(메시징)**에 대해서 알아보도록 한다.

---

## Overview

- 여러 애플리케이션을 구축하기 시작하면 애플리케이션 간에 통신이 필요할 수 밖에 없다.
- 애플리케이션 통신에는 아래의 이미지와 같이 크게 두 가지 패턴이 있다.

![integration-and-messaging.png](images%2FIntegrationAndMessaging%2Fintegration-and-messaging.png)

- 갑자기 트래픽이 급증하는 경우 애플리케이션 간 동기화에 문제가 발생할 수 있다.
- 일반적으로 10개의 비디오만 인코딩하지만, 갑자기 1000개의 비디오를 인코딩해야 하는 경우가 발생할 수 있다.
- 이러한 경우 애플리케이션을 분리(Decouple)하는 것이 좋다.
  - SQS: 대기열 모델을 사용한다.
  - SNS: Publish/Subscribe 모델을 사용한다.
  - Kinesis: Real-time 스트리밍 모델을 사용한다.
- 이러한 서비스는 애플리케이션과 독립적으로 확장할 수 있다.

### SQS

![whats-a-queue.png](images%2FIntegrationAndMessaging%2Fwhats-a-queue.png)

#### Standard Queue

- 10년 이상된 오래된 서비스다.
- 애플리케이션을 분리하는 데 사용되는 완벽한 관리가 가능하다.
- 아래와 같은 속성이 있다.
  - 무제한 처리량, 대기열에 있는 메시지 수의 제한이 없다.
  - 메시지의 기본 보존 기한은 4일이며, 최대 14일까지 보관이 가능하다.
  - 낮은 지연 시간을 제공한다.(publish 부터 receive까지 10ms 이내에 처리가 가능)
  - 메시지는 256KB로 제한된다.
- 메시지는 최소 한번 전달되면 간헐적으로 중복된 메시지가 전달될 수 있다.
- 전달이 불가능한 메시지를 가질 수 있다.

#### Producing Messages

- SDK(Send Message API)를 이용하여 SQS에 데이터를 제공할 수 있다.
- 메시지는 소비자(Consumer)가 데이터를 소비할 때까지 SQS에 지속(Persist)된다.
- 메시지는 기본적으로 4일, 최대 14일까지 보존된다.
- SQS Standard는 처리량이 제한되지 않는다.

![sqs-producing-messages.png](images%2FIntegrationAndMessaging%2Fsqs-producing-messages.png)

#### Consuming Messages

- 소비자(Consumer)는 EC2 Instance, Servers, AWS Lambda 등이 될 수 있다.
- SQS에서 메시지를 Polling하며 한번에 최대 10개의 메시지까지 가져올 수 있다.
- 메시지를 RDS Database에 삽입하는 것과 같은 작업을 할 수 있다.
- DeleteMessage API를 사용하여 메시지를 삭제할 수 있다.

![sqs-consuming-messages.png](images%2FIntegrationAndMessaging%2Fsqs-consuming-messages.png)

#### Multiple EC2 Instances Consumers

![multiple-ec2-instances-consumers.png](images%2FIntegrationAndMessaging%2Fmultiple-ec2-instances-consumers.png)

- 소비자(Consumer)는 메시지를 병렬로 수신하고 처리한다.
- 메시지는 1회 이상 배송된다.
- Best-Effort 메시지 순서를 지정한다.
- 소비자(Consumer)는 메시지 처리 후에 삭제한다.
- 소비자(Consumer)를 수평적으로 확장하여 처리량을 개선할 수 있다.

#### SQS with Auto Scaling Group (ASG)

- SQS Queue의 길이를 CloudWatch가 관측하고 일정 수준 이상이 되는 경우 Auto Scaling Group을 통해 EC2 Instance를 Scale-Up 할 수 있다.

![auto-scaling-group-with-sqs.png](images%2FIntegrationAndMessaging%2Fauto-scaling-group-with-sqs.png)

#### SQS to Decouple Between Application Tiers

- Front-end 프로젝트와 Back-end 프로젝트 사이에 SQS를 두어 프로젝트를 분리(Decouple)할 수 있다.

![decouple-between-application-tiers.png](images%2FIntegrationAndMessaging%2Fdecouple-between-application-tiers.png)

#### Security

- **Encryption**:
  - HTTPS API를 통해서 전송 중 암호화가 가능하다.
  - KMS Keys를 통해 암호화가 가능하다.
  - 클라이언트가 암호화를 진행하는 경우, 클라이언트 사이드 암호화/복호화가 가능하다.
- **Access Controls**:
  - IAM 정책을 통해서 SQS API에 대한 접근을 제한할 수 있다.
- **SQS Access Policies**:
  - SQS 대기열에 대한 계정 간 액세스에 유용하다.
  - 다른 서비스(SNS, S3)가 SQS Queue에 데이터를 제공할 수 있도록 허용하는 데 유용하다.

#### Message Visibility Timeout

- 소비자(Consumer)가 메시지를 Polling한 후 다른 소비자(Consumer)에게는 보이지 않게 된다.
- 기본적으로 "메시지 가시성 시간 초과(Message Visibility Timeout)"는 30초다.
- 메시지 처리 시간이 30초임을 의미한다.
- 메시지 가시성 시간 초과가 끝나면 메시지가 SQS에서 "보이는(Visible)" 상태가 된다.

![message-visibility-timeout.png](images%2FIntegrationAndMessaging%2Fmessage-visibility-timeout.png)

- 만약 "Visibility Timeout" 시간 내에 메시지가 처리되지 않으면 두 번 처리된다.
- 소비자(Consumer)가 ChangeMessageVisibility API를 호출하여 더 많은 시간을 확보할 수 있다.
- "Visibility Timeout"이 시간이 높고 소비자가 충돌하면 재처리에 시간이 소요된다.
- "Visibility Timeout"이 너무 낮으면 중복이 발생할 수 있다.

#### Long Polling

- 소비자(Consumer)가 대기역에서 메시지를 요청할 때, 대기열에 메시지가 없을 경우 선택적으로 메시지가 도착할 때까지 "기다릴(Wait)" 수 있으며, 이것을 "Long Polling"이라고 한다.
- Long Polling은 SQS에 호출되는 API의 수를 줄이면서 애플리케이션의 효율성을 높이고 지연 시간을 줄인다.
- 대기 시간은 1초에서 20초 사이일 수 있으며, 20초가 권장된다.
- 짧은 폴링 시간보다는 긴 폴링 시간이 권장된다.
- 대기열 수준에서 긴 폴링을 활성화할 수 있다.
- API Level에서 "WaitTimeSeconds"를 통해서 Long Polling을 설정할 수 있다.

![long-polling.png](images%2FIntegrationAndMessaging%2Flong-polling.png)

#### FIFO Queue

- FIFO: First In First Out(Queue의 Message Ordering)

![fifo-queue.png](images%2FIntegrationAndMessaging%2Ffifo-queue.png)

- 제한된 처리량: 배치 없이 300 msg/s을 제공한다.
- 중복 제거를 통해 정확히 한 번만 메시지를 전송한다.
- 소비자(Consumer)가 메시지를 순서대로 처리한다.

#### SQS with Database

- 부하가 큰 경우에 몇몇 트랜잭션이 유실될 수 있다.

![lost-transaction.png](images%2FIntegrationAndMessaging%2Flost-transaction.png)

- SQS가 무한대로 확장할 수 있기 때문에 애플리케이션 중간에서 버퍼(Buffer) 역할을 할 수 있다.

![sqs-buffer.png](images%2FIntegrationAndMessaging%2Fsqs-buffer.png)

---

### Amazon SNS

- 하나의 메시지를 많은 수신자(Subscriber)에게 보내고 싶은 경우에는 Amazon SNS를 사용할 수 있다.

![amazon-sns-overview.png](images%2FIntegrationAndMessaging%2Famazon-sns-overview.png)

- "이벤트 제공자(Producer)"는 하나의 SNS 주제에만 메시지를 보낸다.
- SNS 토픽 알림을 수신하고 싶은 만큼의 "이벤트 수신자(Subscription)"에게 토픽을 전달할 수 있다.
- 각 토픽의 구독자(Subscriber)는 모든 메시지를 받는다.
- 토픽 하나당 최대 12,500,000개 까지 구독이 가능하다.
- 최대 토픽은 10만개로 제한된다.

![amazon-sns-overview-2.png](images%2FIntegrationAndMessaging%2Famazon-sns-overview-2.png)

- Amazon SNS는 AWS의 수많은 서비스와 직접적으로 통합될 수 있다.

![amazon-sns-overview-3.png](images%2FIntegrationAndMessaging%2Famazon-sns-overview-3.png)

#### 게시(Publish) 방식

- Topic Publish(SDK 사용)
  - 토픽 생성
  - 구독(Subscription) 생성, 여러개의 구독 생성 가능
  - 토픽에 Publish
- Direct Publish(Mobile App SDK 사용)
  - 플랫폼 애플리케이션 생성
  - 플랫폼 엔드포인트(Endpoint) 생성
  - 플랫폼 엔드포인트에 Publish
  - Google GCM, Apple APNS, Amazon ADM과 통합되어 작동

#### 보안(Security)

- 암호화(Encryption)
  - HTTPS API를 사용한 전송 중 암호화
  - KMS 키를 사용한 REST 암호화
  - 클라이언트에서 암호화/복호화를 원하는 경우 클라이언트 사이드에서 암호화/복호화
- 접근 제어(Access Controls)
  - SMS API에 대한 접근을 규제(Regulate)하는 IAM 정책
- SNS 접근 정책(Access Policies, S3 버킷 정책과 유사)
  - SNS 주제에 대한 계정 간 접근에 유용하게 사용
  - S3와 같은 다른 서비스가 SNS 주제에 글을 쓸 수 있도록 허용하는데 유용하게 사용

#### SNS + SQS: Fan Out

![sns-sqs-fanout.png](images%2FIntegrationAndMessaging%2Fsns-sqs-fanout.png)

- SNS에서 한 번 Push하고 구독자인 모든 SQS 대기열에서 수신
- 완전하게 분리(Decoupled)되어 데이터 손실 없음
- SQS를 통해 제공되는 기능: 데이터 지속성, 지연된 처리 및 작업 재시도
- 시간이 지남에 따라 더 많은 SQS Subscribers 추가 가능
- SQS 대기열 액세스 정책에서 SNS 쓰기를 허용하는지 확인
- Cross-Region Delivery: 다른 Region의 SQS Queues와 함께 작동

#### 여러 대기열에 대한 S3 이벤트

![event-to-multiple-queues.png](images%2FIntegrationAndMessaging%2Fevent-to-multiple-queues.png)

- 이벤트 유형(예: Object 생성) 및 접두사의 동일한 조합인 경우(예: /images) S3 Event 규칙을 하나만 가질 수 있다.
- 동일한 S3 이벤트를 많은 SQS 큐에 전송하려면 팬아웃(fan-out)을 사용할 수 있다.

#### SNS를 통해 Amazon S3로 Kinesis Data Firehose

- SNS는 Kinesis로 데이터를 보낼 수 있으므로 아래와 같은 아키텍처를 설계할 수 있다.

![sns-s3-through-kinesis.png](images%2FIntegrationAndMessaging%2Fsns-s3-through-kinesis.png)

#### FIFO Topic

- FIFO는 First In First Out의 약자로 Topic의 메시지를 정렬한다.

![sns-fifo-topic.png](images%2FIntegrationAndMessaging%2Fsns-fifo-topic.png)

- SQS FIFO와 유사한 기능을 제공한다
  - 메시지 그룹 ID별 순서를 지정(동일한 그룹의 모든 메시지 순서 지정)
  - ID 또는 콘텐츠 기반의 중복 제거
- SQS FIFO 대기열만 Subscriber로 사용할 수 있다.
- SQS FIFO와 동일한 처리량으로 제한된 처리량을 가진다.

#### SNS FIFO + SQS FIFO: Fan Out

- Fan-out, 정렬, 중복 제거가 필요하다면 아래와 같은 아키텍처를 설계할 수 있다.

![sns-sqs-fanout.png](images%2FIntegrationAndMessaging%2Fsns-sqs-fanout.png)

#### SNS Message Filtering

- SNS Topic의 Subscriber로 보낸 메시지를 필터링하는 데 사용되는 JSON 정책을 사용할 수 있다.
- 만약 Filter 정책이 없는 경우 모든 메시지를 수신한다.

![sns-message-filtering.png](images%2FIntegrationAndMessaging%2Fsns-message-filtering.png)

---

### Kinesis Overview

- 실시간으로 손쉽게 스트리밍 데이터를 수집, 처리 및 분석이 가능하다.
- 실시간 데이터를 수집한다. 애플리케이션 로그, 메트릭, 웹 사이트 클릭 스트림, IoT 원격 측정 데이터
- **Kinesis Data Streams**: 데이터 스트림 캡처, 처리 및 저장
- **Kinesis Data Firehose**: 데이터 스트림을 AWS 데이터 저장소로 적재한다.
- **Kinesis Data Analytics**: SQL 또는 Apache Flink로 데이터 스트림을 분석한다.
- **Kinesis Video Streams**: 비디오 스트림 캡처, 처리 및 저장한다.

### Kinesis Data Streams

![kinesis-data-streams.png](images%2FIntegrationAndMessaging%2Fkinesis-data-streams.png)

- 1 ~ 365일 사이의 데이터 보존 기간
- 데이터 재처리(재생) 기능
- Kinesis에서 한 번 데이터를 삽입하면 삭제가 불가능(데이터 불변성)
- 동일한 파티션을 공유하는 데이터가 동일한 Shard로 이동
- Producers: AWS SDK, Kinesis Producer Library(KPL), Kinesis Agent
- Consumers:
  - 데이터 생성: Kinesis Client Library(KCL), AWS SDK
  - 관리: AWS Lambda, Kinesis Data Firehose, Kinesis Data Analytics

#### Kinesis Data Streams - Capacity Modes

- **프로비저닝 모드(Provisioned Mode)**:
  - 프로비저닝된 샤드의 수를 선택하거나 수동으로 확장하거나 API를 사용할 수 있다.
  - 각 샤드는 1MB/s의 입력(또는 초당 1,000개의 레코드).
  - 각 샤드당 2MB/s의 출력(클래식 또는 향상된 fan-out 소비자(Consumer)).
  - 시간당 프로비저닝 된 샤드당 지불
- **온디멘드 모드(On-demand Mode)**:
  - 용량을 프로비저닝하거나 관리할 필요가 없다.
  - 프로비저닝된 기본 용량(초당 4MB 또는 초당 4,000개 레코드)
  - 지난 30일 동안 관측된 처리량 피크(Peak)를 기반으로 자동 확장된다.
  - 시간당 & 스트림당 지불하며 GB당 데이터 In/Out을 기반으로 지불한다.

#### Kinesis Data Streams Security

- IAM 정책을 사용하여 Access/Authorization 제어
- HTTPS 엔드포인트를 사용하여 전송 중 암호화
- KSM를 통한 REST 암호화
- 클라이언트 측에서 데이터의 암호화/복호화를 구현
- Kinesis에서 VPC 내에서 액세스할 수 있는 VPC Endpoint를 구축
- CloudTrail을 사용하여 API 호출 모니터링

![kinesis-data-streams-security.png](images%2FIntegrationAndMessaging%2Fkinesis-data-streams-security.png)

---

### Kinesis Data Firehose

![kinesis-data-firehose.png](images%2FIntegrationAndMessaging%2Fkinesis-data-firehose.png)

- 완전 관리형 서비스로 관리가 필요없고 자동으로 확장하며 Serverless
  - AWS: Redshift, S3, OpenSearch
  - 타사 파트너: Splunk, MongoDB, DataDog, NewRelic
  - 사용자 지정: 원하는 HTTP 엔드포인트로 전송
- Firehose를 통과하는 데이터에 대한 비용을 지불
- Near Real Time
  - 전체 배치가 아닌 최소 60초 지연 시간 또는 한 번에 최소 1MB의 데이터를 전송
- 다양한 데이터 형식, 변환, 압축을 지원
- AWS Lambda를 사용하여 맞춤형 데이터 변환을 지원
- 백업 S3 버킷에 실패한 데이터 또는 모든 데이터를 전송 가능

#### Kinesis Data Stream vs Firehose

- **Kinesis Data Streams**
  - 대규모 데이터 수집을 위한 스트리밍 서비스
  - 사용자 지정 코드 작성(Producer/Consumer)
  - 실시간(~200ms)
  - 확장 관리(Shard Splitting/Merging)
  - 1 ~ 365일 기간 동안의 데이터 저장
  - 재생(Replay) 기능 지원
- **Kinesis Data Firehose**
  - S3/Redshift/OpenSearch/3rd Party/사용자 지정 HTTP
  - 완전 관리형 서비스
  - 거의 실시간(Near Real-Time, 버퍼 시간 최소 60초)
  - 자동 스케일링
  - 데이터 저장을 미지원
  - 재생(Replay) 기능 미지원

#### Ordering data into Kinesis

![ordering-data-into-kinesis.png](images%2FIntegrationAndMessaging%2Fordering-data-into-kinesis.png)

- 도로에서 100대의 트럭(truck_1, truck_2 ...)이 GPS 위치를 정기적으로 AWS 전송하고 있다고 가정
- 각 트럭에 대한 데이터를 순서대로 사용하여 트럭의 움직임을 정확하게 추적이 필요
- **`truck_id`의 Partition Key 값을 사용하여 데이터를 전송**
- **동일한 키는 항상 동일한 Shard로 이동**

#### Ordering data into SQS

- SQS Standard의 경우 정렬(Ordering) 기능이 없음
- SQS FIFO의 경우 그룹 ID를 사용하지 않으면 메시지가 발송(Send)된 순서대로 소비(Consume)되며 소비자(Consumer)는 한 명뿐이다.

![ordering-data-into-sqs-1.png](images%2FIntegrationAndMessaging%2Fordering-data-into-sqs-1.png)

- 소비자 수를 확장하려고 하지만 메시지가 서로 관련되어 있을 때 메시지를 "그룹화"하기를 원하는 경우 Group ID를 사용할 수 있다.(Kinesis의 Partition Key와 유사)

![ordering-data-into-sqs-2.png](images%2FIntegrationAndMessaging%2Fordering-data-into-sqs-2.png)

#### Kinesis vs SQS Ordering

- 트럭 100대와 5개의 Kinesis Shard, 1개의 FIFO SQS가 있다고 가정한다.
- Kinesis Data Streams:
  - 평균적으로 한 샤드당 20대의 트럭을 보유하게 된다.
  - 트럭은 각 샤드 내에서 데이터가 정렬된다.
  - 병렬 소비자(Consumer)의 최대 용량은 5다.
  - 최대 5MB/s의 데이터를 수신할 수 있다.
- SQS FIFO
  - SQS FIFO Queue가 하나만 있다.
  - 100개의 Group ID를 가지게 된다.
  - 최대 100명의 소비자를 보유할 수 있다.(100개의 Group ID 때문에)
  - 초당 최대 300개의 메시지를 처리할 수 있다.(배치를 사용하는 경우 3,000개)

#### SQS vs SNS vs Kinesis

- **SQS**:
  - 소비자(Consumer)가 데이터를 가져간다.(Pull)
  - 데이터는 소비된 이후에 삭제된다.
  - 원하는 만큼 소비자(Consumer)를 보유할 수 있다.
  - 처리량 프로비저닝이 필요 없다.
  - FIFO 대기역에 대해서만 정렬 보증이 가능하다.
  - 개별 메시지 지연 기능이 있다.
- **SNS**:
  - 여러 구독자(Subscriber)에게 데이터를 전송한다.(Push)
  - 최대 12,500,000개의 구독자를 가질 수 있다.
  - 데이터가 전송되지 않은 경우 데이터가 지속되지 않는다.
  - Pub/Sub 구조를 가진다.
  - 최대 100,000개의 Topic을 생성할 수 있다.
  - 처리량 프로비저닝이 필요 없다.
  - fan-out 아키텍처 패턴을 위해 SQS와 통합된다.
  - SQS FIFO에 대한 FIFO를 지원한다.
- **Kinesis**:
  - Standard: 데이터를 가져간다.(Pull, 샤드당 2MB)
  - 향상된 fan-out: 데이터를 전송한다.(Push, 소비자당 & 샤드당 2MB)
  - 데이터 재생(Replay)이 가능하다.
  - 실시간 빅데이터, 분석, ETL을 위해 사용된다.
  - Shard 레벨에서 정렬이 가능하다.
  - 1 ~ 365 기간 내에서 데이터 저장 주기를 설정할 수 있다.
  - "프로비저닝 모드" 또는 "온디맨드 용량 모드"로 설정할 수 있다.

---

### Amazon MQ

- SQS, SNS는 "클라우드 네이티브" 서비스로 AWS의 독점 프로토콜이 사용된다.
- 온프레미스에서 실행되는 기존 애플리케이션은 MQTT, AMQP, STOMP, Openwire, WSS와 같은 개방형 프로토콜을 사용할 수 있다.
- 클라우드로 마이그레이션할 때, 애플리케이션을 재설계하여 SQS와 SNS를 사용하는 대신 Amazon MQ를 사용할 수 있다.
- Amazon MQ는 RabbitMQ와 ActiveMQ의 **관리형 메시지 브로커 서비스**다.
- Amazon MQ는 SQS/SNS 만큼 "확장"되지 않는다.
- Amazon MQ는 서버에서 실행되며, Multi-AZ에서 Failover와 함께 실행할 수 있다.
- Amazon MQ에는 Queue 기능(~SQS)와 토픽 기능(~SNS)가 모두 있다.

#### 고가용성(High Availability)

![mq-high-availability.png](images%2FIntegrationAndMessaging%2Fmq-high-availability.png)

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03