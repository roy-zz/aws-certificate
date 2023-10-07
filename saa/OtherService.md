# Other Service

이번 장에서는 SAA를 준비하며 **지금까지 다루지 않았지만 시험에 출제 있는 서비스** 에 대해서 알아보도록 한다.

---

### CloudFormation

- "CloudFormation"은 대부분의 리소스에 대해 AWS 인프라스트럭처의 개요를 선언적으로 설명하는 방식이다.
- 예를 들어, CloudFormation 템플릿에 아래와 같이 설정할 수 있다.
  - 보안 그룹 설정
  - 두개의 EC2 인스턴스와 보안 그룹 설정
  - S3 버킷 설정
  - 시스템 앞에 로드 밸런서 설정
- 설정 이후 CloudFormation이 지정한 정확한 구성을 사용하여 올바른 순서로 작업을 생성할 수 있다.

#### 이점

- 인프라를 코드로 사용할 수 있다.(IaS, Infrastructure as Code)
  - 리소스를 수동으로 생성하지 않아 리소스 제어에 유용하다.
  - 인프라스트럭처의 변경 사항을 코드를 통해 검토할 수 있다.
- 비용
  - 스택 내의 각 리소스에는 식별자 태그가 지정되어 있으므로 스택 비용을 쉽게 파악할 수 있다.
  - CloudFormation 템플릿을 사용하여 리소스 비용을 추정할 수 있다.
  - 저장 전략: 개발 서버에서는 템플릿 삭제를 오후 5시에 자동화하고 오전 8시에 안전하게 재생성할 수 있다. (서버가 중지되어있는 동안 비용 절감)
- 생산성
  - 클라우드의 인프라스트럭처를 즉시 파괴하고 재구축할 수 있는 능력
  - 템플릿에 대한 다이어그램 자동 생성
  - 선언적 프로그래밍(순서와 오케스트레이션을 파악할 필요 없음)
- 새로운 템플릿을 재개발 불필요
  - 웹에서 기존 템플릿을 활용
  - 문서 활용
- 거의 모든 AWS 리소스 지원
  - 지원되지 않는 리소스에 대해 "사용자 지정 리소스"를 사용 가능

#### Stack Designer

- WordPress Cloud Formation 스택 예시다.

![cloudformation-stack-designer.png](images%2FOtherServices%2Fcloudformation-stack-designer.png)

- 우리는 모든 리소스를 한 눈에 파악할 수 있다.
- 구성 요소들 사이의 관계를 파악할 수 있다.

---

### Amazon Simple Email Service(Amazon SES)

- 완벽하게 관리되는 서비스를 통해 안전하게, 전세계적으로 그리고 규모에 맞게 이메일을 전송할 수 있다.
- 인바운드/아웃바운드 전자 메일을 허용한다.
- 평판 대시보드, 성능 통찰력, 안티스팸 피드백을 활용할 수 있다.
- 이메일 전송, 바운스, 피드백 루프 결과, 이메일 열기 등의 통계를 제공한다.
- DKIM(Domain Keys Identified Mail) 및 SPF(Sender Policy Framework)를 지원한다.
- 유연한 IP 구축: Public IP, Private IP 및 고객이 소유한 IP를 활용할 수 있다.
- 애플리케이션을 사용하여 AWS 콘솔, API 또는 SMTP를 사용하여 이메일을 전송할 수 있다.
- 트랜잭션, 마케팅 및 대량 이메일 커뮤니케이션 등에 활용된다.

![amazon-ses.png](images%2FOtherServices%2Famazon-ses.png)

---

### Amazon Pinpoint

- 확장 가능한 양방향(아웃바운드/인바운드) 마케팅 커뮤니케이션 서비스
- 이메일, SMS, 푸시, 음성 및 인앱 메시징을 제공한다.
- 고객에게 적합한 콘텐츠를 제공하는 메시지를 세분화하고 개인화할 수 있는 능력을 제공한다.
- 원하는 경우 회신도 가능하다.
- 매일 수십억 개의 메시지로 확장이 가능하다.
- 마케팅, 대량, 트랜잭션 SMS 메시지를 전송하여 캠페인을 실행하는 작업 등에 사용된다.
- Amazon SNS, Amazon SES와의 차이점
  - "SNS" & "SES"에서는 각 메시지의 시청자, 컨텐츠, 전송 일정 등을 관리할 수 있다.
  - "Amazon Pinpoint"에서 메시지 템플릿, 전송 일정, 대상이 높은 세그먼트 및 전체 캠페인을 만들 수 있다.

![amazon-pinpoint.png](images%2FOtherServices%2Famazon-pinpoint.png)

---

### System Manager - SSM Session Manager

- EC2 및 On-premises 서버에서 보안 Shell을 실행할 수 있다.
- SSH 액세스, Bastion 호스트, SSH 키가 필요 없다.
- 22번 포트를 개방할 필요가 없으므로 보안이 향상된다.
- Linux, macOS 및 Windows를 지원한다.
- 세션 로그 데이터를 S3 또는 CloudWatch Logs로 전송한다.

![ssm-session-manager.png](images%2FOtherServices%2Fssm-session-manager.png)

#### Run Command

- 문서(=스크립트)를 실행하거나 커맨드만 실행한다.
- 여러 인스턴스(리소스 그룹 사용)에 걸쳐 커맨드를 실행한다.
- SSH 접속이 필요하지 않다.
- 커맨드 출력은 AWS 콘솔에서 S3 버킷 또는 CloudWatch Logs로 전송할 수 있다.
- 커맨드 상태에 대한 알림을 SNS에 전송할 수 있다.(In progress, Success, Failed 등)
- IAM & CloudTrail과 통합된다.
- EventBridge를 사용하여 호출이 가능하다.

![run-command.png](images%2FOtherServices%2Frun-command.png)

#### Patch Manager

- 관리 인스턴스 패치 적용 프로세스를 자동화한다.
- OS 업데이트, 애플리케이션 업데이터, 보안 업데이트를 지원한다.
- EC2 인스턴스 및 On-premise 서버를 지원한다.
- Linux, macOS, Windows를 지원한다.
- **Maintenance Windows**를 사용하여 온디맨드 또는 스케줄에 따라 패치를 적용한다.
- 인스턴스 검색 및 패치 준수 보고서를 생성한다.

![patch-manager.png](images%2FOtherServices%2Fpatch-manager.png)

#### Maintenance Windows

- 인스턴스에 대한 작업을 수행할 시기에 대한 일정을 정의한다.
- OS 패치 적용, 드라이버 업데이트, 소프트웨어 설치 등에 사용된다.
- Maintenance Windows에는 아래의 항목이 포함된다.
  - 스케줄
  - 지속시간
  - 등록된 인스턴스 집합
  - 등록된 태스크 집합

![maintenance-windows.png](images%2FOtherServices%2Fmaintenance-windows.png)

#### Automation

- EC2 인스턴스 및 기타 AWS 리소스의 일반적인 유지보수 및 배포 작업을 간소화한다.
- 인스턴스 재시작, AMI, EBS 스냅샷 생성 등의 작업에 사용된다.
- **Automation Runbook**: SSM 문서를 통해 EC2 인스턴스 또는 AWS 리소스(사전 정의 또는 사용자 지정)에 대해 사전에 수행된 작업을 정의한다.
- 아래의 항목을 사용하여 트리거할 수 있다.
  - 수동으로 AWS Console, AWS CLI 또는 SDK에서 작업
  - Amazon EventBridge
  - Maintenance Windows를 사용하는 스케줄
  - 규칙 업데이트 적용을 위한 AWS 구성 기준

![automation.png](images%2FOtherServices%2Fautomation.png)

---

### Cost Explorer

- AWS 비용 및 사용량을 시간에 따라 시각화, 이해 및 관리한다.
- 비용 및 사용량 데이터를 분석하는 사용자 정의 보고서를 만든다.
- 전체 계정에 걸친 총 비용 및 사용량에 대해 높은 수준의 데이터를 분석한다.
- 월별, 시간별, 리소스 수준으로 세분화할 수 있다.
- 최적의 저축 계획을 선택하여 청구서 가격을 절감할 수 있다.
- 이전 사용량을 기준으로 최대 12개월까지 사용량을 예측할 수 있다.

![cost-explorer-monthly-cost.png](images%2FOtherServices%2Fcost-explorer-monthly-cost.png)

![cost-explorer-hourly-resource-level.png](images%2FOtherServices%2Fcost-explorer-hourly-resource-level.png)

- **Savings Plan**을 사용하여 예약 인스턴스(RI)를 대체할 수 있다.

![cost-explorer-saving-plan.png](images%2FOtherServices%2Fcost-explorer-saving-plan.png)

![cost-explorer-forecast-usage.png](images%2FOtherServices%2Fcost-explorer-forecast-usage.png)

---

### Amazon Elastic Transcoder

- **Elastic Transcoder**는 S3에 저장된 미디어 파일을 고객의 기기에서 필요로 하는 포맷의 미디어로 변환하는데 사용된다.
- 아래와 같은 이점을 가지고 있다.
  - 사용 편의성 향상
  - 높은 확장성: 대용량 미디어 파일과 대용량 파일 크기를 처리할 수 있다.
  - 비용 효율적: 지속 기간 기반(duration-based) 가격 모델을 제공한다.
  - 완벽하게 관리되고 안전하며 사용하는 제품에 대한 비용을 지불한다.

![amazon-elastic-transcoder.png](images%2FOtherServices%2Famazon-elastic-transcoder.png)

---

### AWS Batch

- 모든 규모에서 완벽하게 관리되는 완전 관리형 배치 프로세스다.
- AWS에서 10만 개의 컴퓨팅 배치 작업을 효율적으로 실행한다.
- 배치 작업은 시작과 끝(연속적인 것과 반대)이 있는 작업이다.
- 배치는 EC2 인스턴스 또는 Spot 인스턴스를 동적으로 실행한다.
- "AWS Batch"에서 적절한 양의 CPU/메모리를 제공한다.
- 배치 작업을 제출하거나 예약하면 "AWS Batch"가 나머지 작업을 수행한다.
- 배치 작업은 도커 이미지로 정의되고 ECS에서 실행된다.
- 배용 최적화에 도움이 되며 인프라스트럭처에 대한 관심을 줄일 수 있다.
  
- 아래는 "AWS Batch"를 통해서 이미지를 프로세싱하는 예시이다.

![batch-simplified-example.png](images%2FOtherServices%2Fbatch-simplified-example.png)

#### Batch vs Lambda

- **Lambda**
  - 시간 제한이 있다.
  - 제한된 런타임을 가진다.
  - 제한된 임시 디스크 공간이 있다.
  - Serverless로 작동한다.
- **Batch**
  - 시간 제한이 없다.
  - 도커 이미지로 포장된 모든 런타임을 제공한다.
  - 디스크 공간을 EBS/인스턴스 스토어에 의존한다.
  - EC2에 의존하며 AWS에 의해 관리된다.

---

### Amazon AppFlow

- SaaS(Software-as-a-Service) 애플리케이션과 AWS 간에 데이터를 안전하게 전송할 수 있도록 완전 관리되는 통합 서비스다.
- Source: Salesforce, SAP, Zendesk, Slack, ServiceNow와 같은 서비스에서 데이터를 가져올 수 있다.
- Destination: Amazon S3, Amazon Redshift와 같은 AWS 서비스 또는 SnowFlake와 Salesforce와 같은 AWS에서 제공하지 않는 서비스에 데이터를 전달할 수 있다.
- 주기(Frequency): 일정, 이벤트에 대한 응답 또는 요청 시
- 필터링 및 검증과 같은 데이터 변환 기능을 가진다.
- Public 인터넷을 통해 암호화되거나 AWS PrivateLink를 통해 비공개로 암호화된다.
- 통합(Integration)을 작성하는데 시간을 소요하지 않고, API를 즉시 활용할 수 있다.

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03