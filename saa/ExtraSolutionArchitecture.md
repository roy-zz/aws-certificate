# Extra Solution Architecture Discussion

이번 장에서는 SAA를 준비하며 **지금까지 다루지 않은 Solution Architecting** 에 대해서 알아보도록 한다.

---

### Lambda, SNS & SQS

![lambda-sns-sqs.png](images%2FExtraSolutionArchitecture%2Flambda-sns-sqs.png)

#### 다중 SQS로 전달을 위한 Fan-Out 패턴

![fan-out-pattern-multiple-sqs.png](images%2FExtraSolutionArchitecture%2Ffan-out-pattern-multiple-sqs.png)

---

### S3 Event Notifications

- S3:ObjectCreated, S3:ObjectRemoved, S3:ObjectRestore, S3:Replication 등
- 객체 이름을 필터링할 수 있다. (예: `*.jpb`)
- 이미지 썸네일 생성하여 S3에 업로드하는 작업 등에 사용된다.
- 원하는 만큼의 "S3 Event"를 생성할 수 있다.
- "S3 Event Notifications"는 일반적으로 몇 초 안에 이벤트를 전달하지만 때로는 1분 이상 소요될 수 있다.

![s3-event-notifications.png](images%2FExtraSolutionArchitecture%2Fs3-event-notifications.png)

#### S3 Event Notifications with Amazon EventBridge

![s3-event-notifications-eventbridge.png](images%2FExtraSolutionArchitecture%2Fs3-event-notifications-eventbridge.png)

- JSON 규칙을 사용한 고급 필터링 옵션을 제공한다.(메타데이터, 객체의 크기, 이름 등)
- 여러 대상을 제공한다. (Step Functions, Kinesis Stream / Firehose 등)
- Event Bridge 기능: 아카이브, 이벤트 재생, 신뢰성 있는 제공

![eventbridge-intercept-api-call.png](images%2FExtraSolutionArchitecture%2Feventbridge-intercept-api-call.png)

#### API Gateway - AWS 서비스 통합 Kinesis Data Streams

![api-gateway-integration-kinesis-data-streams.png](images%2FExtraSolutionArchitecture%2Fapi-gateway-integration-kinesis-data-streams.png)

#### 캐싱 전략

![caching-strategies.png](images%2FExtraSolutionArchitecture%2Fcaching-strategies.png)

#### IP 주소 차단

- 아래는 NACL 또는 소프트웨어를 통한 IP 주소 차단 방식이다.

![blocking-ip-address.png](images%2FExtraSolutionArchitecture%2Fblocking-ip-address.png)

- 아래는 "Application Load Balancer"를 통해 IP 주소를 차단하는 방식이다.

![blocking-ip-address-alb.png](images%2FExtraSolutionArchitecture%2Fblocking-ip-address-alb.png)

- 아래는 "Network Load Balancer"를 통해 IP 주소를 차단하는 방식이다.

![blocking-ip-address-nlb.png](images%2FExtraSolutionArchitecture%2Fblocking-ip-address-nlb.png)

- 아래는 "CloudFront"를 활용하여 지역을 제한하고, WAF를 사용하여 IP 주소를 차단하는 방식이다.

![blocking-ip-address-cloudfront-waf.png](images%2FExtraSolutionArchitecture%2Fblocking-ip-address-cloudfront-waf.png)

---

### High Performance Computing(HPC)

- 클라우드는 HPC를 수행하기에 완벽한 장소다.
- 매우 많은 리소스를 즉시 생성할 수 있다.
- 리소스를 추가하여 결과 도출 시간을 단축할 수 있다.
- 사용한 리소스에 대해서만 결제가 가능하다.
- 유전체학(Genomic), 전산화학(Computational chemistry), 재무위험 모델링(Financialcial risk modeling), 날씨 예측(Weather prediction), 머신러닝, 딥러닝, 자율 주행 등에 사용된다.

---

### 데이터 관리 및 전송

- **AWS Direct Connect**: GB/s 속도로 사설 보안 네트워크를 통해 클라우드로 데이터를 이동한다.
- **Snowball & Snowmobile**: PB 단위의 데이터를 클라우드로 이동한다.
- **AWS DataSync**: On-premise와 S3, EFS< FSx for Windows 등의 서비스 간에 대용량 데이터를 이동한다.

---

### Compute and Networking

- EC2 인스턴스
  - CPU 최적화, GPU 최적화
  - Spot 인스턴스, Spot Fleets과 Auto Scaling을 활용하여 비용을 절감할 수 있다.
- EC2 배치 그룹: 양호한 네트워크 성능을 위한 클러스터다.

![compute-and-networking.png](images%2FExtraSolutionArchitecture%2Fcompute-and-networking.png)

- **EC2 Enhanced Networking(SR-IOV)**
  - 대역폭 증가, PPS(초당 패킷수) 증가, 지연시간 감소
  - 옵션 1: Elastic Network Adapter(ENA)를 통해 100Gbps까지 속도를 향상시킨다.
  - 옵션 2: Intel 82599VF를 사용하여 10Gbps까지 속도를 향상시킨다.
- **Elastic Fabric Adapter(EFA)**
  - 향상된 HPC용 ENA, Linux에서만 작동한다.
  - 노드간 통신, 긴밀하게 연결된 워크로드에 적합하다.
  - MPI(Message Passing Interface) 표준을 활용한다.
  - 기본 Linux OS를 우회하여 지연 시간이 적고 안정적인 전송을 제공한다.

---

### 스토리지

- 인스턴스 연결 스토리지
  - EBS: io2 Block Express로 최대 256,000 IOPS까지 확장 가능하다.
  - Instance Store: 수백만 IOPS로 확장 가능, EC2 인스턴스에 연결, 짧은 대기 시간을 제공한다.
- 네트워크 스토리지
  - Amazon S3: 파일 시스템이 아닌 큰 Blob
  - Amazon EFS: 총 크기에 따라 IOPS를 확장하거나 프로비저닝된 IOPS를 사용한다.
  - Amazon FSx for Lustre
    - HPC에 최적화된 분산 파일 시스템으로 수백만 IOPS를 지원한다.
    - S3를 지원한다.

---

### 자동화와 오케스트레이션

- AWS 배치
  - AWS 배치는 다중 노드 병렬 작업을 지원하므로 여러 EC2 인스턴스에 걸쳐 단일 작업을 실행할 수 있다.
  - 작업을 쉽게 예약하고 그에 따라 EC2 인스턴스를 실행한다.
- AWS 병렬 클러스터
  - AWS에 HPC를 구축하는 오픈소스 클러스터 관리 툴이다.
  - 텍스트 파일로 구성되어 있다.
  - VPC, 서브넷, 클러스터 유형 및 인스턴스 유형 생성을 자동화한다.
  - 클러스터에서 EFA를 활성화하는 기능을 가지고 있다.(네트워크 성능 향상)

#### 가용성이 높은 EC2 인스턴스 생성

![highly-available-ec2-instance.png](images%2FExtraSolutionArchitecture%2Fhighly-available-ec2-instance.png)

![highly-available-ec2-instance-asg.png](images%2FExtraSolutionArchitecture%2Fhighly-available-ec2-instance-asg.png)

![highly-available-ec2-instance-ebs.png](images%2FExtraSolutionArchitecture%2Fhighly-available-ec2-instance-ebs.png)

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03