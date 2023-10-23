# AWS Lambda

이번 장에서는 SAA를 준비하며 **AWS Lambda**에 대해서 알아보도록 한다.

---

### Serverless Overview

- 서버리스(Serverless)는 개발자들이 서버를 더 이상 관리할 필요가 없는 새로운 패러다임이다.
- 단지 코드를 배포하거나, Function을 배치하는 것만으로 작동한다.
- 처음에는 AWS Lambda에서만 Serverless 개념이 사용되었지만 현재는 "데이터베이스, 메시징, 스토리지 등" 관리되는 모든 것을 포함한다.
- Serverless라고 해서 서버가 없는 것은 아니며, 개발자가 직접 서버를 Manage/Provision 할 필요가 없다는 것을 의미한다.

- 아래는 AWS에서 제공되는 여러 Serverless 서비스다.
  - AWS Lambda
  - DynamoDB
  - AWS Cognito
  - AWS API Gateway
  - Amazon S3
  - AWS SNS & SQS
  - AWS Kinesis Data Firehose
  - Aurora Serverless
  - Step Functions
  - Fargate

![serverless-in-aws.png](images%2FServerless%2Fserverless-in-aws.png)

---

### AWS Lambda

- **Amazon EC2**:
  - 클라우드의 가상 서버다.
  - RAM 및 CPU에 의해 제한된다.
  - 지속적으로 실행된다.
  - 스케일링은 서버의 추가나 제거를 위한 개입을 위미한다.
- **Amazon Lambda**:
  - Virtual Functions: 관리할 서버가 없다.
  - 시간 제한: 짧은 실행시간을 가진다.
  - 온-디멘드 실행
  - 스케일링이 자동화된다.

#### Benefits

- 간편한 가격 정책
  - 요청당, 컴퓨팅 시간당 지불하면 된다.
  - 1,000,000개의 AWS Lambda 요청 및 400,00GB의 컴퓨팅 시간이 Free Tier에 포함된다.
- AWS 전체 서비스 제품군과 통합된다.
- 많은 프로그래밍 언어와 통합된다.
- AWS CloudWatch를 통한 간편한 모니터링이 가능하다.
- 기능당 더 많은 리소스 확보가 용이하다.(최대 10GB RAM)
- RAM을 증설하는 경우, CPU와 Network 성능도 향상된다.

#### 언어 지원

- AWS Lambda는 아래의 언어 및 프레임워크를 지원한다.
  - Node.js (Javascript)
  - Python
  - Java (Java 8 호환)
  - C# (.NET Core)
  - Golang
  - C# / Powershell
  - Ruby
  - Custom Runtime API (커뮤니티 지원, 대표적으로 Rust)
  - Lambda Container Image
    - 컨테이너 이미지는 Lambda Runtime API를 구현해야 한다.
    - ECS/Fargate는 임의의 Docker 이미지를 실행하는 데 선호된다.

- AWS Lambda는 아래와 같이 많은 AWS 서비스와 통합된다.

![lambda-integrations-main-ones.png](images%2FServerless%2Flambda-integrations-main-ones.png)

- AWS Lambda를 사용하여 아래와 같이 썸네일을 생성할 수 있다.

![servless-thumbnail-creation.png](images%2FServerless%2Fservless-thumbnail-creation.png)

- CloudWatch EventBridge를 사용하여 일정 시간마다 CRON Job을 실행시킬 수 있다.

![serverless-cron-job.png](images%2FServerless%2Fserverless-cron-job.png)

#### Pricing

- 전반적인 가격 정보는 [여기](https://aws.amazon.com/lambda/pricing/)에서 확인할 수 있다.
- 호출당 가격:
  - 최초 1,000,000건의 요청은 무료로 제공된다.
  - 이후 요청 100만 건당 $0.2(요청당 $0.0000002)
- 지속되는 시간당 가격(1ms 단위로 증가):
  - 매달 400,000GB 초의 무료 컴퓨팅 시간이 제공된다.
  - == 기능이 1GB RAM인 경우 400,000초
  - == 기능이 128MB RAM인 경우 3,200,000초
  - 이후 60만 GB-second에 1달러를 지불하면 된다.
- AWS Lambda를 운영하는 것은 일반적으로 매우 저렴하기 때문에 인기가 많다.

#### Lambda 제한(Limit) - 지역별

- 실행(Execution)
  - 메모리 할당: 128MB ~ 10GB(1MB씩 증분)
  - 최대 실행시간: 900초(15분)
  - 환경변수(4KB)
  - "Function Container"의 디스크 용량(`/tmp`): 512MB ~ 10GB
  - 동시 실행: 1,000(증가 가능)
- 배포(Deployment)
  - Lambda 함수의 배포 사이즈(compressed.zip): 50MB
  - 압축되지 않은 배포 사이즈(코드 + 의존성): 250MB
  - 시작할 때 `/tmp` 디렉토리를 사용하여 다른 파일을 로드할 수 있다.
  - 환경 변수의 크기: 4KB

---

### Customization At The Edge

- 현대의 많은 응용 프로그램들은 Edge에서 어떤 형태로든 로직을 실행한다.
- **Edge Function**:
  - CloudFront 배포판에 사용자가 작성하고 첨부하는 코드다.
  - 사용자 가까이에서 실행하여 지연 시간을 최소화한다.
- CloudFront는 CloudFront Function과 Lambda@Edge 두 가지 유형을 제공한다.
- 전 세계에 구축된 서버를 관리할 필요가 없다.
- 사용 사례: CDN 컨텐츠 사용자 정의
- 사용하는 것에 대해서만 지불하면 된다.
- 완전 Serverless

#### CloudFront Functions & Lambda@Edge 사용 사례

- 웹사이트 보안 및 개인정보 보호
- Edge의 동적 웹 애플리케이션
- 검색엔진 최적화(SEO)
- 오리진 및 데이터 센터 전반에 걸친 지능적인 경로
- Edge의 Bot 완화
- 실시간 이미지 변환
- A/B 테스트
- 사용자 인증 및 권한 부여
- 사용자 우선순위 지정
- 사용자 추적 및 분석

#### CloudFront Functions

- Javascript로 작성된 경량 함수다.
- 대기 시간에 민감한 대규모 CDN Customization이다.
- Sub-ms의 시작 시간을 가지고 있으며, 수백만 건의 요청을 초단위로 처리한다.
- 뷰어 요청(Viewer Request) 및 응답(Viewer Response)을 변경하는 데 사용한다.
  - Viewer Request: CloudFront가 viewer로 부터 요청을 수신한 후
  - Viewer Response: CloudFront가 View에게 응답을 전달하기 전
- CloudFront의 기본 기능(Code를 전적으로 CloudFront 내에서 관리)

![cloudfront-functions.png](images%2FServerless%2Fcloudfront-functions.png)

#### Lambda@Edge

- NodeJS 또는 Python으로 작성된 Lambda 함수다.
- 초당 1,000개의 요청으로 확장한다.
- CloudFront 요청 및 응답 변경에 사용된다.
  - Viewer Request - CloudFront에서 Viewer로 부터 요청을 받은 후
  - Origin Request - CloudFront가 Origin에 요청을 전달하기 전
  - Origin Response - CloudFront가 Origin으로부터 응답을 받은 후
  - Viewer Response - CloudFront가 Viewer에게 응답을 전달하기 전
- 하나의 AWS 영역(Region)에서 Function을 작성한 후 CloudFront가 해당 위치로 복제한다.

![lambda-edge.png](images%2FServerless%2Flambda-edge.png)

#### CloudFront Functions vs Lambda@Edge

![cloudfront-function-vs-lambda-edge.png](images%2FServerless%2Fcloudfront-function-vs-lambda-edge.png)

- **CloudFront Functions 사용 사례**
  - **캐시키 정규화**: 요청 속성(헤더, 쿠키, 쿼리 문자열, URL)을 변환하여 최적의 캐시 키를 생성한다.
  - **헤더 조작**: HTTP 요청 및 응답의 헤더 삽입/수정/삭제
  - URL 다시 쓰기 또는 Redirects
  - **요청 인증(Authentication) 및 인가(Authorization)**: 요청을 허용/거부하기 위한 사용자 생성 토큰(예 JWT) 생성 및 검증
- **Lambda@Edge 사용 사례**
  - 실행 시간이 길어짐(ms)
  - 조정 가능한 CPU 또는 메모리
  - 코드는 제3의 라이브러리(예: 다른 AWS 서비스에 액세스하기 위한 AWS SDK)에 의존한다.
  - 처리를 위해 외부 서비스를 사용해야 하므로 네트워크에 액세스한다.
  - 파일 시스템 액세스 또는 HTTP 요청 Body에 대해 액세스한다.

#### Lambda by default

- 기본적으로 Lambda Function은 VPC 외부(AWS 소유의 VPC)에서 실행된다.
- VPC의 리소스(RDS, ElastiCache, 내부 ELB)에 액세스할 수 없다.

![default-lambda-deployment.png](images%2FServerless%2Fdefault-lambda-deployment.png)

- VPC ID, 서브넷 및 Security Group을 정의해야 한다.
- Lambda는 서브넷에 ENI(Elastic Network Interface)를 생성한다.

![lambda-in-vpc.png](images%2FServerless%2Flambda-in-vpc.png)

#### Lambda with RDS Proxy

- Lambda Function가 데이터베이스에 직접 액세스할 경우 높은 부하가 발생하는 경우 너무 많은 DB Connection을 생성할 수 있다.
- RDS Proxy
  - DB 풀링 및 공유를 통해 확장성을 향상한다.
  - Failover 시간을 66% 단축하고 연결을 유지하여 가용성을 향상 시킨다.
  - IAM 인증을 시행하고 Secrets Manager에 자격 증명을 저장하여 보안을 향상시킨다.
- RDS Proxy는 공개적(Public)으로 액세스할 수 없기 때문에 Lambda Function을 VPC에 배포해야 한다.

![lambda-with-rds-proxy.png](images%2FServerless%2Flambda-with-rds-proxy.png)

- DB 인스턴스 내에서 Lambda 함수를 호출할 수 있다.
- 데이터베이스 내에서 데이터 이벤트를 처리할 수 있다.
- PostgreSQL 및 Aurora MySQL에 대한 RDS를 지원한다.
- DB 인스턴스(Public, NAT GW, VPC Endpoint) 내에서 Lambda 함수에 대한 아웃바운드 트래픽을 허용해야 한다.
- DB 인스턴스가 Lambda 함수(Lambda Resource-based Policy & IAM Policy)를 호출하는 데 필요한 권한을 가지고 있어야 한다.

![invoking-lambda-rds-aurora.png](images%2FServerless%2Finvoking-lambda-rds-aurora.png)

#### RDS Event Notifications

- DB 인스턴스 자체(생성, 중지, 시작)에 대한 정보를 알려주는 알림
- 데이터 자체에 대한 정보가 없다.
- DB 인스턴스, DB 스냅샷, DB 매개변수 그룹, DB Security Group, RDS Proxy, Custom Engine Version 등 이벤트 범주에 구독(Subscribe)한다.
- 실시간에 가까운 이벤트(최대 5분)
- SNS로 알림 전송 또는 Event Bridge를 사용한 이벤트 구독

![rds-event-notifications.png](images%2FServerless%2Frds-event-notifications.png)

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03