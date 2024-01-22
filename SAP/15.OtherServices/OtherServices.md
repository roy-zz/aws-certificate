# Other Services

- 이번 장에서는 **Solutions Architect Professional (SAP)** 을 준비하며 "기타 서비스들"에 대해서 알아보도록 한다.

---

### CI/CD

#### Continuous Integration

- 개발자들은 코드 저장소로 코드를 자주 푸시한다. (GitHub / CodeCommit / Bitbucket 등..)
- 테스트 / 빌드 서버는 코드를 푸시하자마자 코드를 확인한다. (CodeBuild / Jenkins CI 등..)
- 개발자는 통과/실패한 테스트 및 점검에 대한 피드백을 받는다.
- 버그를 조기에 찾아내고 버그를 수정할 수 있다.
- 코드 테스트를 신속하게 완료할 수 있다.
- 자주 배포할 수 있는 환경을 만들 수 있다.

![1-continuous-integration.png](images%2F1-continuous-integration.png)

#### Continuous Delivery

- 필요할 때마다 소프트웨어를 안정적으로 출시할 수 있도록 한다.
- 배포가 자주 진행되고 신속하게 이루어진다.
- 예를 들어, "3개월에 한 번 배포"되던 애플리케이션을 "하루에 5번 배포"할 수 있도록 전환한다.
- 아래의 항목들은 일반적으로 자동화된 배포를 제공한다.
  - CodeDeploy, Jenkins CD, Spinnaker

![2-continuous-delivery.png](images%2F2-continuous-delivery.png)

#### Technology Stack

![3-technology-stack-cicd.png](images%2F3-technology-stack-cicd.png)

- AWS에는 CICD를 위한 기술이 있다.
- 빌드, 테스트, 배포, 프로비저닝 단계를 제공한다.
- CodeCommit은 AWS에서 제공하는 코드 리포지토리다. 
  - GitHub과 같은 3rd Party 코드 리포지토리도 있다.
- CodeBuild는 AWS에서 제공하는 빌드 및 테스트 서비스다.
  - Jenkins CI와 같은 3rd Party 서비스도 있다.
- AWS Elastic Beanstalk를 사용하여 효과적으로 배포하고 프로비저닝할 수 있다.
  - AWS CodeDeploy나 CloudFormation을 사용하여 애플리케이션을 배포하거나 인프라를 배포할 수 있다.
- 이러한 모든 과정을 AWS CodePipeline을 통해서 관리할 수 있다.

#### CI/CD Architecture

![4-cicde-architecture.png](images%2F4-cicde-architecture.png)

- 코드는 CodeCommit에 있고 코드를 위해 다른 브랜치를 사용하게 된다.
- 개발 환경을 위한 `dev` 브랜치와 운영 환경을 위한 `prod` 브랜치가 있다.
- 개발 브랜치에 대한 "Pull Request" 또는 "Merge"가 발생하면 "DEV" CodePipeline에 의해 처리된다.
  - CodeBuild를 통해 코드가 테스트 및 빌드된다.
  - CodeDeploy를 통해서 코드가 배포되거나 Elastic Beanstalk를 통해서 개발 환경에 배포된다.
- 운영 브랜치 또한 동일하게 작동된다.
- 여기서 중요한 점은 환경 하나당 하나의 CodePipeline이 필요하다는 점이다.

#### CodeCommit Trigger AWS Lambda

- CodeCommit으로 푸시할 때마다 람다 함수롤 트리거할 수 있다.
- 람다 함수는 모든 코드 푸시에서 유출된 AWS 자격 증명을 검색하고 자동으로 비활성화하여 문제를 해결할 수 있다.

![5-codecommit-trigger-lambda.png](images%2F5-codecommit-trigger-lambda.png)

#### Good to Know

- COdePipeline에서 수동 승인 단계를 사용할 수 있다.
- 단위 테스트를 실행한다. (CodeCommit + CodeBuild + CodePipeline)

![6-running-unit-test.png](images%2F6-running-unit-test.png)

- 도커 이미지 생성 및 저장 (CodeBuild + ECR)

![7-build-store-docker-images.png](images%2F7-build-store-docker-images.png)

- 자동화된 CloudFormation 배포: CodePipeline

#### GitHub Integration

![8-github-integration.png](images%2F8-github-integration.png)

- CodePipeline과 GitHub이 통합되어 사용될 수 있다.
- 첫 번째 방법은 CodePipelin이 정기적으로 GitHub의 상태를 확인하는 방법이다.
  - 효율적인 방법이 아니다.
- 두 번째 방법은 GitHub에 변화가 발생하였을 때 HTTP Webhook을 통해 CodePipeline에게 알리는 방법이다.
- 세 번째 방법을 사용하면 GitHub과 CodePipeline은 CodeStar Source Connection(GitHub APP)을 통해 통합되어 있다.
  - 자동으로 GitHub으로 푸시된 변화가 CodePipeline으로 전달된다.

---

### Amazon CodeGuru

- 자동화된 코드 컴토 및 애플리케이션 성능 권장을 위한 ML 기반의 서비스다.
- 두 가지 기능을 제공한다.
  - CodeGuru Reviewer: 정적 코들 분석을 위한 자동 코드 리뷰 (개발)
  - CodeGuru Profiler: 런타임 중 애플리케이션 성능에 대한 가시성/권장사항 (운영)

![9-codeguru.png](images%2F9-codeguru.png)

#### CodeGuru Reviewer

- 중요한 문제, 보안 취약성 및 찾기 어려운 버그를 식별한다.
- 예를 들어, 일반적인 코딩 모범 사례, 리소스 유출, 보안 탐지, 입력 유효성 검사 등이 있다.
- 머신러닝 및 자동화된 추론을 사용한다.
- 1,000개의 오픈 소스 및 Amazon 저장소 수백만 건의 코드 리뷰를 통해 학습한 권장사항을 제공한다.
- Java 및 Python을 지원한다.
- GitHub, BitBucket 및 CodeCommit과 통합된다.
- 자세한 내용은 [공식 홈페이지](https://aws.amazon.com/codeguru/features/)를 확인한다.

![10-codeguru-reviewer.png](images%2F10-codeguru-reviewer.png)

#### CodeGuru Profiler

- 애플리케이션의 런타임 동작을 이해하는데 도움이 된다.
- 예를 들어, 로깅 루틴에서 애플리케이션이 과도한 CPU 용량을 사용하고 있는지 확인한다.
- 아래와 같은 특징이 있다.
  - 코드 비효율성 식별 및 제거
  - 애플리케이션 성능 향상(예: CPU 사용률 감소)
  - 컴퓨팅 비용 절감
  - 힙 메모리 요약(메모리를 사용하는 객체 식별)을 제공
  - 이상 탐지
- AWS 또는 온프레미스에서 실행되는 애플리케이션을 지원한다.
- 애플리케이션에 대한 오버헤드를 최소화한다.
- 자세한 내용은 [공식 홈페이지](https://aws.amazon.com/codeguru/features/)를 확인한다.

![11-codeguru-profiler.png](images%2F11-codeguru-profiler.png)

---

### Alexa for Business, Lex & Connect

- **Alexa for Business**
  - Alexa를 사용하여 직원들이 회의실 및 데스크에서 보다 효율적으로 업무를 수행할 수 있도록 지원한다.
  - 작업장의 회의실 활용도를 측정하고 개선할 수 있다.
- **Amazon Lex**
  - Alexa를 작동시키는 기술과 동일한 기술을 사용한다.
  - 음성을 텍스트로 변환하는 ASR(Automatic Speech Recognition)을 지원한다.
  - 텍스트와 발신자의 의도를 인식하기 위해 자연어를 이해한다.
- **Amazon Connect**
  - 전화 수신, 연락처 흐름 생성, 클라우드 기반 가상 연락처 센터다.
  - 다른 CRM 시스템 또는 AWS와 통합할 수 있다.

![12-alex-lex-connect.png](images%2F12-alex-lex-connect.png)

---

### Kinesis Video Streams

- 스트리밍 디바이스(Producer)당 한 개의 비디오 스트림을 제공한다.
  - 보안 카메라, 신체 착용 카메라, 스마트폰 등의 장치가 될 수 있다.
  - Kinesis Video Streams Producer 라이브러리를 사용할 수 있다.
- 기본 데이터는 S3에 저장되지만 액세스 권한은 없다.
- 스트림 데이터를 바로 S3에 출력할 수 없으며, 이를 위해서는 사용자 정의 솔루션을 구축해야 한다.
- 소비자(Consumer)는 아래와 같은 특징이 있다.
  - 실시간 분석을 위해 EC2 인스턴스에서 사용하거나 일괄적으로 사용할 수 있다.
  - Kinesis Video Stream Parser Library 활용 가능하다.
  - 안면 인식을 위해 AWS Recognition과 통합될 수 있다.

#### Video Streaming & Rekognition

![13-video-streaming-rekognition.png](images%2F13-video-streaming-rekognition.png)

- 프로듀서들이 Kinesis Video Stream으로 비디오 스트림 데이터를 전송한다.
- 비디오 스트림을 처리할 수 있는 EC2 인스턴스 플릿을 연동할 수 있고 스트림을 수정할 수 있다.
- Amazon Rekognition을 통해서 머신러닝에 대한 메타데이터 스트림을 생성할 수 있다.

---

### Amazon WorkSpaces

- 관리되는 안전한 클라우드 데스크톱이다.
- 사내 VDI(Virtual Desktop Infrastructure) 관리 제거에 탁월하다.
- 가격은 주문형(시간당 지불) 또는 월간 구독 형태로 지불한다.
- 안전하고 암호화되며, 네트워크 격리를 지원한다.
- Microsoft Active Directory와 통합될 수 있다.

![14-workspace.png](images%2F14-workspace.png)

- **WorkSpaces Application Manager (WAM)**
  - 애플리케이션을 가상화된 애플리케이션 컨테이너로 배포 및 관리한다.
  - 규모에 맞게 프로비저닝하고 WAM을 사용하여 애플리케이션의 업데이트를 유지한다.
- **Windows Update**
  - 기본적으로 Amazon Workspace는 소프트웨어 업데이트를 설치하도록 구성되어 있다.
  - Windows가 설치된 WorkSpace에서 Windows Update가 실행된다.
  - Windows Update 빈도를 완전히 제어할 수 있다.
- **Maintenance Windows**
  - 업데이트는 유지 관리 기간 동안 설치된다. (사용자 정의)
  - Always On WorkSpaces: 기본값은 일요일 아침 00:00 ~ 04:00에 실행된다.
  - AutoStop WorkSpaces: 한 달에 한 번 자동으로 시작하여 업데이트를 설치한다.
  - 수동 유지보수: Windows를 정의하고 유지보수를 수행한다.

#### Cross Region Redirection

![15-cross-region-redirection.png](images%2F15-cross-region-redirection.png)

- WorkSpaces Directory를 서로 다른 여러 리전에서 실행할 수 있다.
- Primary 리전이 있고 페일오버를 위해 사용되는 리전이 있다.
- 사용자가 접속하면 Primary 리전으로 연결된다.
  - Microsoft AD에 연결되어 사용자를 저장하고 WorkSpaces에서 세션을 열 수 있다.
- 페일오버를 위한 리전에서는 AD 커넥터를 생성해야 한다.
  - Microsoft AD는 Multi-AZ를 지원하지 않기 때문에 AD Connector를 설치해야 한다.
- 두 개의 리전이 생성되고 연결 별칭을 설정해야 한다.

#### IP Access Control Groups

- WorkSpaces 보안 그룹과 유사하다.
- 사용자가 연결할 권한이 있는 IP 주소 / CIDR 주소 범위 목록이다.
- 사용자가 VPN 또는 NAT를 통해 WorkSpaces에 액세스하는 경우 IP Access Control Group은 Public IP를 허용해야 한다.

![16-ip-access-control-groups.png](images%2F16-ip-access-control-groups.png)

---

### Amazon AppStream 2.0

- 데스크톱 애플리케이션 스트리밍 서비스다.
- 인프라를 확보하지 않고 프로비저닝하지 않고도 모든 컴퓨터에 제공된다.
- 웹 브라우저 내에서 애플리케이션을 제공한다.

![17-appstream.png](images%2F17-appstream.png)

#### AppStream 2.0 vs WorkSpaces

- **WorkSpaces**
  - 완벽하게 관리되는 VDI 및 데스크톱 사용이 가능하다.
  - 사용자는 VDI에 연결하여 네이티브 또는 WAM 애플리케이션을 실행한다.
  - 작업 공간은 온디맨드 또는 상시 가동된다.
- **AppStream 2.0**
  - 데스크톱 애플리케이션을 웹 브라우저로 스트리밍한다. (VDI에 연결할 필요 없음)
  - 웹 브라우저가 있는 모든 장치에서 작동한다.
  - 애플리케이션 유형별 인스턴스 유형(CPU, RAM, GPU) 구성을 허용한다.

---

### AWS Device Farm

- 모바일 및 웹 애플리케이션을 위한 애플리케이션 테스트 서비스다.
- 실제 브라우저 및 실제 모바일 장치 전반에 걸쳐 테스트가 가능하다.
- 프레임워크를 사용하여 완전 자동화된다.
- 웹 및 모바일 앱의 품질 향상에 도움을 준다.
- 발생한 문제를 문서화하기 위한 비디오 및 로그를 생성한다.
- 디버깅을 위해 원격으로 장치에 로그인할 수 있다.

![18-device-farm.png](images%2F18-device-farm.png)

---

### AWS Macie

- Macie는 완벽하게 관리되는 데이터 보안 및 데이터 개인 정보 보호 서비스로 머신러닝 및 패턴 매칭을 사용하여 AWS에서 중요한 데이터를 검색하고 보호한다.
- Macie는 개인 식별 정보(PII)와 같은 중요한 데이터를 식별하고 경고할 수 있도록 지원한다.

![19-macie.png](images%2F19-macie.png)

---

### Amazon Simple Email Service (SES)

- 완벽하게 관리되는 서비스를 통해 안전하고 전 세계적으로 대규모로 이메일을 전송할 수 있다.
- 인바운드/아웃바운드 전자 메일을 허용한다.
- 평판 댓보드, 성능 통찰력, 안티스팸 피드벡을 제공한다.
- 전자 메일 배달, 바운스, 피드백 루프 결과, 전자 메일 열기와 같은 통계를 제공한다.
- 도메인 키 식별 메일(DKIM) 및 발신자 정책 프레임 워크(SPF)를 지원한다.
- 유연하게 IP를 구축하여 공유하고 전용 및 고객 소유의 IP를 사용할 수 있다.
- AWS 콘솔, API, SMTP를 사용하여 애플리케이션을 사용하여 이메일을 전송할 수 있다.
- 트랜잭션, 마케팅 및 대량 이메일 통신 등에 사용된다.

![20-simple-email-service.png](images%2F20-simple-email-service.png)

#### Configuration Set

- 구성 집합을 사용하면 전자 메일 전송 이벤트를 사용자 지정하고 분석할 수 있다.
- 이벤트 대상은 아래와 같다.
  - Kinesis Data Firehose: 각 이메일에 대한 메트릭(전송 횟수, 배달, 열기, 클릭, 바운스, 불만건수)를 수신한다.
  - SNS: 바운스 및 불만 정보에 대한 즉각적인 피드백을 제공한다.
- IP 풀 관리: IP 풀을 사용하여 특정 유형의 이메일을 보낸다.

![21-ses-configuration-sets.png](images%2F21-ses-configuration-sets.png)

---

### EC2 Image Builder

- VM 또는 컨테이너 이미지 생성 자동화에 사용된다.
- 예를 들어, EC2 AMI 생성 자동화, 유지보수, 검증 및 테스트 등이 있다.
- 일정에 따라 실행 가능하다. (매주, 패키지 업데이트 시 등..)
- 무료 서비스로 기본 리소스에 대한 비용만 지불한다.
- 여러 리전 및 여러 계정에 AMI를 게시할 수 있다.

![22-ec2-image-builder.png](images%2F22-ec2-image-builder.png)

#### CI/CD Architecture

![23-cicd-architecture.png](images%2F23-cicd-architecture.png)

---

### AWS IoT Core

- IoT는 데이터를 수집하고 전송할 수 있는 인터넷 연결 장치 네트워크인 "Internet of Things"의 약자다.
- AWS IoT Core를 통해 IoT 기기를 AWS 클라우드에 쉽게 연결할 수 있다.
- 수십억 개의 장치와 수조 개의 메시지에 대한 서버리스, 보안 및 확장성을 제공한다.
- 많은 AWS 서비스(람다, S3, SageMaker 등)과 통합된다.
- 데이터를 수집, 처리, 분석 및 조치하는 IoT 애플리케이션을 구축할 수 있다.

![24-iot-core.png](images%2F24-iot-core.png)

#### Integrations

![25-iot-core-integrations.png](images%2F25-iot-core-integrations.png)

- IoT 토픽은 SNS 토픽과 마찬가지로 데이터를 수집한다.
  - 대표적으로 MQTT 프로토콜이 있다.
- 메시지들을 위한 IoT 규칙을 설정한다.
  - 규칙들은 동작들을 가지고 있고, 이 동작들은 AWS 서비스와 관련이 있다.
  - 예를 들어, IoT 토픽에 있는 메시지를 키네시스에 보내거나 DynamoDB로 보낼 수 있다.

#### Kinesis Data Firehose

- IoT 코어 규칙은 "Kinesis Data Firehose"와 통합될 수 있다.

![26-iot-core-kinesis-data-firehose.png](images%2F26-iot-core-kinesis-data-firehose.png)

---

### 참고한 강의

- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)