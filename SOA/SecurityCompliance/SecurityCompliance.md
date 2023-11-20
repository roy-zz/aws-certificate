# Security & Compliance

이번 장에서는 **SysOps Administrator**를 준비하며 **보안과 규정 준수**에 대해서 알아보도록 한다.

---

### 공유 보안 책임

- AWS 책임 - 클라우드 보안
  - 모든 AWS 서비스를 실행하는 인프라(하드웨어, 소프트웨어, 시설, 네트워킹) 보호
  - S3, DynamoDB, RDS와 같은 관리형 서비스
- 고객 책임 - 클라우드 보안
  - EC2 인스턴스의 경우 고객은 게스트 OS (보안 패치 및 업데이트 포함), 방화벽 및 네트워크 구성, IAM
  - 애플리케이션 데이터의 암호화
- 공유 제어
  - 패치 관리, 구성 관리, 인식 및 교육

#### RDS 예시

- AWS 책임
  - 기본 EC2 인스턴스를 관리하고 SSH 액세스를 비활성화한다.
  - 자동화된 DB 패치
  - 자동화된 OS 패치
  - 기본 인스턴스와 디스크를 감사하고 작동을 보장한다.
- 고객 책임
  - DB의 보안 그룹에서 포트/IP/인바운드 규칙을 확인한다.
  - 데이터베이스 내 사용자 생성 및 권한을 관리한다.
  - 공개 액세스 여부에 관계없이 데이터베이스를 생성한다.
  - 매개변수 그룹이나 DB가 SSL 연결만 허용하도록 구성되어 있는지 확인한다.
  - 데이터베이스 암호화를 설정한다.

#### S3 예시

- AWS 책임
  - 무제한 저장용량 보장한다.
  - 암호화를 보장한다.
  - 서로 다른 고객 간의 데이터 분리를 보장한다.
  - AWS 직원이 고객의 데이터에 액세스할 수 없도록 보장한다.
- 고객 책임
  - 버킷 구성
  - 버킷 정책/공개 설정
  - IAM 사용자 및 역할
  - 암호화 활성화

![1-shared-responsibility-model.png](images%2F1-shared-responsibility-model.png)

#### DDoS란?

- Distributed Denial of Service의 약자다.

![2-what-is-ddos.png](images%2F2-what-is-ddos.png)

- 해커는 여러 개의 마스터 서버를 실행하고 이 서버들은 많은 봇을 실행한다.
- 봇들은 애플리케이션 서버에 수 많은 요청을 보내고, 애플리케이션 서버는 많은 부하 때문에 요청을 처리할 수 없게 된다.
- 일반 사용자들은 애플리케이션을 정상적으로 이용할 수 없게 된다.

#### DDoS 보호

- **AWS Shield Standard**: 추가 비용 없이 모든 고객을 대상으로 웹 사이트 및 애플리케이션을 DDoS 공격으로부터 보호한다.
- **AWS Shield Advanced**: 24/7 프리미엄 DDoS 방어 서비스다.
- **AWS WAF**: 규칙을 기반으로 특정 요청을 필터링한다.
- **CloudFront & Route 53**
  - 글로벌 엣지 네트워크를 이용하여 가용성을 보호한다.
  - AWS Shield와 결합하여 엣지에서 공격 완화를 제공한다.

![3-ddos-sample-reference-architecture.png](images%2F3-ddos-sample-reference-architecture.png)

- 사용자는 Route 53에 있는 DNS를 통해 라우트되는데 DNS는 "AWS Shield"로 보호되어 DNS가 DDoS 공격으로부터 안전하다.
- CloudFront Distribution을 이용해 엣지 로케이션에 콘텐츠가 캐시되도록 해야 한다.
- CloudFront에서 공격에서 필터링과 보호가 필요한 경우 WAF를 사용한다.

#### AWS Shield

- AWS Shield Standard
  - 모든 AWS 고객에게 활성화되는 무료 서비스다.
  - SYN/UDP Flood, Reflection 공격, Layer 3/Layer 4 공격과 같은 공격으로 부터 보호한다.
- AWS Shield Advanced
  - DDoS 완화 서비스 옵션(Organization 당 $3,000)
  - EC2, ELB, CloudFront, Global Accelerator 및 Route 53에 대한 더욱 정교한 공격으로부터 보호한다.
  - AWS DDoS 대응팀(DRP)에 연중무휴 24시간 액세스를 지원한다.
  - DDoS로 인해 사용량이 급증하는 동안 발생하는 높은 비용(스케일 아웃으로 인한)으로부터 보호한다.

#### AWS WAF - Web Application Firewall

- 일반적인 웹 공격으로부터 웹 애플리케이션을 보호한다. (Layer 7)
- Layer 7은 HTTP다.
- Application Load Balancer, API Gateway, CloudFront에 배포한다.
- Web ACL(Web Access Control List)을 정의한다.
  - 규칙에는 IP 주소, HTTP 헤더, HTTP Body 또는 URI 문자열이 포함될 수 있다.
  - 일반적인 공격으로부터 보호 - SQL Injection 및 XSS(Cross-Site Scripting)
  - 크기 제약, 지리적 일치(특정 국가 차단)를 지원한다.
  - 비율 기반 규칙(이벤트 발생 횟수 계산) - DDoS 보호용

#### 관통(Penetration) 테스트

- AWS 고객은 8가지 서비스에 대해 사전 승인 없이 AWS 인프라에 대한 보안 평가 또는 침투 테스트를 수행할 수 있다.
  - Amazon EC2 인스턴스, NAT Gateway, Elastic Load Balancer
  - Amazon RDS
  - Amazon CloudFront
  - Amazon Aurora
  - Amazon API Gateway
  - AWS Lambda and Lambda Edge Function
  - Amazon Lightsail Resource
  - Amazon Elastic Beanstalk 환경
- 테스트를 수행할 수 있는 항목은 시간이 지남에 따라 증가할 수 있다.
- 금지된 활동
  - Route 53 호스팅 영역을 통한 DNS 영역 탐색
  - 서비스 거부(DoS), 분산 서비스 거부(DDoS), 시뮬레이션된 DoS, 시뮬레이션된 DDoS
  - Port Flooding
  - Protocol Flooding
  - Request Flooding(Login Request Flooding, API Request Flooding)
- 기타 시뮬레이션된 이벤트는 AWS(aws-security-simulated-event@amazon.com)에 문의해야 한다.

---

### Amazon Inspector

- **자동화된 보안 평가를 제공**한다.
- EC2 인스턴스의 경우
  - AWS System Manager(SSM) 에이전트 활용
  - 의도하지 않은 네트워크 접근성 분석
  - 알려진 취약점에 대해 실행 중인 OS를 분석
- 컨테이너 이미지(ECR로 푸시된)의 경우
  - 푸시된 컨테이너 이미지 평가
- Lambda 함수의 경우
  - 함수 코드 및 패키지 종속성의 소프트웨어 취약점을 식별한다.
  - 배포 시 기능을 평가한다.
- AWS Security Hub와의 보고 및 통합할 수 있다.
- Amazon EventBridge로 결과를 전송할 수 있다.

![4-amazon-inspector.png](images%2F4-amazon-inspector.png)

#### 평가 대상

- EC2 인스턴스, 컨테이너 이미지, Lambda Function만 평가한다.
- 필요한 경우에만 인프라를 지속적으로 검사한다.
- 패키지 취약점(EC2, ECR 및 Lambda) - CVE 데이터베이스
- 네트워크 접근성(EC2)
- 위험 점수는 우선순위 지정을 위해 모든 취약점과 연결된다.

#### 보안 및 규정 준수를 위한 로깅

- 규정 준수 요구 사항을 지원하기 위해 AWS는 다양한 서비스별 보안 및 감사 로그를 제공한다.
- 서비스 로그에는 아래의 항목이 포함된다.
  - CloudTrail - 모든 API 호출 추적
  - Config Rule - 시간 경과에 따른 구성 및 규정 준수
  - CloudWatch Logs - 전체 데이터 보존용
  - VPC Flow Logs - VPC 내의 IP 트래픽
  - ELB Access Logs - 로드 밸런서에 대한 요청의 메타데이터
  - CloudFront Logs - 웹 배포 액세스 로그
  - WAF Logs - 서비스에서 분석한 모든 요청의 전체 로깅
- **로그가 S3에 저장되는 경우 AWS Athena를 사용하여 분석할 수 있다.**
- **S3의 로그를 암호화하고 IAM 및 버킷 정책, MFA를 사용하여 액세스를 제어해야 한다.**
- **비용 절감을 위해 로그를 Glacier로 이동해야 한다.**

---

### Amazon GuardDuty

- AWS 계정을 보호하기 위한 지능형 위협을 검색한다.
- 기계 학습 알고리즘, 이상 탐지, 타사 데이터를 사용한다.
- 클릭 한 번으로 활성화할 수 있고 소프트웨어를 설치할 필요가 없다. 30일간의 평가판을 제공한다.
- 입력 데이터에는 아래의 항목이 포함된다.
  - CloudTrail Event Logs - 비정상적인 API 호출, 무단 배포
    - CloudTrail Management Events: VPC 서브넷 생성, 추적 생성 등..
    - CloudTrail S3 Data Events: 객체 가져오기, 객체 냐열, 객체 삭제 등..
  - VPC Flow Logs: 비정상적인 내부 트래픽, 비정상적인 IP 주소
  - DNS Logs: DNS 쿼리 내에서 인코딩된 데이터를 전송하는 손상된 EC2 인스턴스
  - 선택적 기능: EKS 감사 로그, RDS 및 Aurora, EBS, Lambda, S3 데이터 이벤트 등..
- 발견 시 알림을 받도록 EventBridge 규칙을 설정할 수 있다.
- EventBridge 규칙은 AWS Lambda 또는 SNS를 대상으로 할 수 있다.
- CryptoCurrency 공격으로부터 보호할 수 있다.(전용 찾기 기능이 있음)

![5-guardduty.png](images%2F5-guardduty.png)

- VPC Flow Logs, CloudTrail Logs, DNS Logs로부터 새로운 것을 발견할 수 있다.
- 이런 것들이 발견되면 EventBridge에서 Event가 발생하여 SNS, Lambda로 전달되고 사용자에게 전달된다.

---

### AWS Macie

- Amazon Macie는 기계 학습 및 패턴 일치를 사용하여 AWS에서 중요한 데이터를 검색하고 보호하는 완전 관리형 데이터 보안 및 데이터 개인 정보 보호 서비스다.
- Macie는 개인 정보 식별(Personally identifiable information, PII)와 같은 민감 데이터를 식별하고 경고하는 데 도움을 준다.

![6-macie.png](images%2F6-macie.png)

- S3 버킷에 PII 데이터가 저장되고 Macie에 의해 검출되면 EventBridge를 통해 사용자에게 알림을 전달할 수 있다.

---

### Trusted Advisor

- 아무 것도 설치할 필요가 없이, **높은 수준의 AWS 계정 평가를 제공**한다.
- 사용자의 AWS 계정을 분석하고 5가지 카테고리에 대한 추천을 제공한다.
  - 비용 최적화(Cost Optimization)
  - 성능(Performance)
  - 보안(Security)
  - 내결함성(Fault Tolerance)
  - 서비스 한도(Service Limit)

![7-trusted-advisor.png](images%2F7-trusted-advisor.png)

#### Support Plan

- 7 중요 검사(Basic & Developer 지원 계획)
  - S3 버킷 권한
  - 보안 그룹 - 특정 포트 제한 없음
  - IAM 사용 (IAM 사용자 최소 1명)
  - 루트 계정의 MFA
  - EBS 공개 스냅샷
  - RDS 공개 스냅샷
  - 서비스 한도
- 모든 검사(Business & Enterprise 지원 계획)
  - 5개의 카테고리에 대한 전체 확인이 가능하다.
  - 한도에 도달하면 CloudWatch 경보를 설정하는 기능이 있다.
  - AWS Support API를 사용한 프로그램이 방식의 액세스를 제공한다.

---

### Encryption

#### Encryption in flight (SSL)

- 데이터는 전송 전에 암호화되고 수신 후에 복호화된다.
- SSL 인증서는 암호화에 도움이 된다.(HTTPS)
- 전송 중 암호화를 통해 MITM(man in the middle attack)이 발생하지 않도록 보장한다.

![8-encryption-in-flight.png](images%2F8-encryption-in-flight.png)

#### Server-Side Encryption 

- 데이터는 서버에서 수신된 후 암호화된다.
- 데이터는 전송되기 전에 복호화된다.
- 키 (일반적으로 데이터 키) 덕분에 암호화된 형태로 저장된다.
- 암호화/복호화 키는 어딘가에서 관리되어야 하며 서버는 이에 액세스할 수 있어야 한다.

![9-server-side-encryption-at-rest.png](images%2F9-server-side-encryption-at-rest.png)

#### Client-Side Encryption

- 데이터는 클라이언트에 의해 암호화되며 서버에 의해 복호화되지 않는다.
- 데이터는 수신 클라이언트에 의해 복호화된다.
- 서버는 데이터를 복호화할 수 없어야 한다.
- Envelope 암호화를 활용할 수 있다.

![10-client-side-encryption.png](images%2F10-client-side-encryption.png)

---

### AWS KMS (Key Management Service)

- AWS 서비스에 대한 "암호화"는 대부분 KMS와 연관되어 있다.
- AWS가 암호화 키를 관리한다.
- 승인을 위해 IAM과 완전히 통합된다.
- 데이터에 대한 액세스를 쉽게 제어할 수 있다.
- CloudTrail을 사용하여 KMS 키 사용을 감사할 수 있다.
- 대부분의 AWS 서비스(EBS, S3, RDS, SSM 등)에 원활하게 통합된다.
- **코드에 Secret을 일반 텍스트로 저장해서는 안된다.**
  - API 호출(SDK, CLI)을 통해서도 KSM 키 암호화가 가능하다.
  - 암호화된 Secret은 코드/환경 변수에 저장될 수 있다.

#### Key Types

- **"KSM Key"는 "KSM Customer Master Key"의 새로운 이름**이다. 
- **대칭(Symmetric, AES-256 Key)**
  - 암호화 및 복호화에 사용되는 단일 암호화 키다.
  - KMS와 통합된 AWS 서비스는 대칭 CMK를 사용한다.
  - 암호화되지 않은 KMS 키에는 절대 액세스할 수 없다. (사용하려면 KMS API를 호출)
- **비대칭(Asymmetric, RSA & ECC Key Pair)**
  - Public(Encrypt) 키와 Private(Decrypt) 키 쌍이다.
  - 암호화/복호화 또는 서명/확인 작업에 사용된다.
  - 공개 키를 다운로드할 수 있지만 암호화되지 않은 개인 키에는 액세스할 수 없다.
  - 사용 사례: KSM API를 호출할 수 없는 사용자가 AWS 외부에서 암호화

- KMS 키 유형
  - AWS Owned Keys (무료): SSE-S3, SSE-SQS, SSE-DDB (기본 키)
  - AWS Managed Key (무료): `aws/service-name`, `aws/rds`, `aws/ebs`)
  - Customer managed Keys created in KMS: $1 / month
  - Customer managed Keys imported: $1 / month (반드시 대칭이어야 함.)
  - KMS API를 호출 비용을 지불해야 한다. ($0.03 / 10,000 호출)
- Automatic Key rotation
  - AWS Managed KMS Key: 1년마다 자동으로 갱신된다.
  - Customer Managed KMS Key: 활성화되는 경우 1년마다 자동으로 갱신된다.
  - Imported KMS Key: 별칭(Alias)을 사용하여 수동 교체만 가능하다.

#### 교차 지역간 스냅샷 복제

![11-copying-snapshot-across-region.png](images%2F11-copying-snapshot-across-region.png)

- EBS 스냅샷은 AZ에 종속되어 있다. 
- KMS키를 통해 암호화되어 있는 스냅샷을 다른 리전으로 복제하기 위해서는 스냅샷을 복호화하고 다른 리전에서 다시 암호화(ReEncrypt)해야 한다.

#### Key Policy

- S3 버킷 정책과 유사한 KMS 키에 대한 액세스 제어가 있다.
- 차이점으로는 KMS의 경우 정책없이는 액세스 제어가 불가능하다.
- **Default KMS Key Policy**
  - 특정 KMS 키 정책을 제공하지 않는 경우 생성된다.
  - 루트 사용자의 키에 대한 완전한 액세스를 제공한다. (모든 AWS 계정)
- **Custom KMS Key Policy**
  - KMS 키에 액세스할 수 있는 사용자, 역할을 정의한다.
  - 키를 관리할 수 있는 사람을 정의한다.
  - KMS 키의 교차 계정 액세스에 유용하게 사용된다.

#### 교차 계정간 스냅샷 복제

![12-copying-snapshot-across-accounts.png](images%2F12-copying-snapshot-across-accounts.png)

- 자신의 KMS 키(고객 관리 키)로 암호화된 스냅샷을 생성한다.
- KMS 키 정책을 연결하여 교차 계정 액세스를 승인한다.
- 암호화된 스냅샷을 공유한다.
- 대상에서 스냅샷 사본을 생성하고 계정의 CMK로 암호화한다.
- 스냅샷에서 볼륨을 생성한다.

#### Automatic Key Rotation

- Customer Managed CMK를 위한 기능이다. (AWS Managed CMK는 아님)
- 할성화된 경우, 1년마다 자동 키 순환이 발생한다.
- 이전 키는 활성 상태로 유지되므로 이전 데이터를 복호화할 수 있다.
- 새 키는 동일한 CMK ID를 갖는다. (지원 키만 변경됨)

![13-kms-automatic-key-rotation.png](images%2F13-kms-automatic-key-rotation.png)

#### Manual Key Rotation

- 90일, 180일 등으로 키를 순환하려는 경우에 사용한다.
- 새 키는 다른 CMK ID를 갖게된다.
- 이전 데이터를 복호화할 수 있도록 이전 키를 활성 상태로 유지해야 한다.
- 애플리케이션에서 키의 변경을 알지 못하도록 별칭(Alias)을 사용하는 것이 좋다.

![14-kms-alias-updating.png](images%2F14-kms-alias-updating.png)

- 자동 순환에 적합하지 않은 CMK(비대칭 CMK 등)를 순환하는 데 적합한 솔루션이다.

![15-manual-key-rotation.png](images%2F15-manual-key-rotation.png)

#### 암호화된 EBS 볼륨의 KMS 키 변경

- EBS 볼륨에 사용되는 암호화 키는 변경할 수 없다.
- EBS 스냅샷을 생성하고 새 EBS 볼륨을 생성하고 새로운 KMS 키를 지정해야 한다.

![16-changing-kms-encrypted-ebs-volume.png](images%2F16-changing-kms-encrypted-ebs-volume.png)

#### KMS 암호화된 RDS DB 스냅샷 공유

- KMS CMK로 암호화된 RDS DB 스냅샷을 다른 계정과 공유할 수 있지만 먼저 키 정책을 사용하여 KMS CMK를 대상 계정과 공유해야 한다.

![17-sharing-kms-encrypted-rds-db-snapshot.png](images%2F17-sharing-kms-encrypted-rds-db-snapshot.png)

#### Key 삭제

- 7 ~ 30일의 대기 기간을 두고 CMK 삭제를 예약한다.
  - 대기 기간 동안 CMK 상태는 "삭제 보류 중"이다.
- CMK 삭제 대기 기간 동안
  - **CMK는 암호화 작업에 사용할 수 없다.** (예를 들어, SSE-KMS에서 KMS로 암호화된 객체를 복호화할 수 없음)
  - 삭제가 예약된 경우 키가 순환되지 않는다.
- 대기 기간 동안에는 키 삭제를 취소할 수 있다.
- 확실하지 않은 경우 키를 삭제하는 대신 비활성화하는 것이 좋다.
- CloudTrail, CloudWatch Logs, CloudWatch Alarm, Amazon SNS를 사용하여 누군가 암호화 작업(암호화, 복호화 등)에서 "삭제 보류 중"인 CMK를 사용하려고 할 때 알림을 받을 수 있다.

![18-key-deletion-cloudwatch-alarm.png](images%2F18-key-deletion-cloudwatch-alarm.png)

---

### CloudHSM

- AWS가 암호화용 소프트웨어를 관리한다.
- CloudHSM => AWS가 암호화 하드웨어를 프로비저닝한다.
- 전용 하드웨어를 사용한다.(HSM = Hardware Security Module)
- 자체 암호화 키를 전적으로 관리한다. (AWS가 아님)
- HSM 장치는 변조 방지 기능을 갖추고 FIPS 140-2 레벨 3을 준수한다.
- 대칭 및 비대칭 암호화 모두 지원한다. (SSL/TLS 키)
- Free Tier에서는 사용할 수 없다.
- CloudHSM 클라이언트 소프트웨어를 사용해야 한다.
- Redshift는 데이터베이스 암호화 및 키 관리를 위해 CloudHSM을 지원한다.
- SSE-C 암호화와 함께 사용하기 좋은 옵션이다.

![19-hsm-diagram.png](images%2F19-hsm-diagram.png)

- IAM Permission
  - HSM 클러스터 CRUD
- CloudHSM Software
  - Key 관리
  - User 관리

#### High Availability

- CloudHSM 클러스터는 다중 AZ(HA)에 분산되어 있다.
- 가용성과 내구성이 뛰어나다.

![20-hsm-high-availability.png](images%2F20-hsm-high-availability.png)

#### AWS 서비스와의 통합

- AWS KMS와의 통합을 통해 AWS 서비스들과 통합될 수 있다.
- CloudHSM으로 KMS 사용자 지정 키 스토어를 구성할 수 있다.
- EBS, S3, RDS와 같은 예시가 있다.

![21-integration-with-aws-service.png](images%2F21-integration-with-aws-service.png)

#### CloudHSM vs KMS

- CloudHSM과 KMS는 아래 표와 같은 차이점이 있다.

![22-cloudhsm-vs-kms-1.png](images%2F22-cloudhsm-vs-kms-1.png)

![23-cloudhsm-vs-kms-2.png](images%2F23-cloudhsm-vs-kms-2.png)

---

### AWS Artifact

- **고객에게 AWS 규정 준수 문서 및 AWS 계약에 대한 온디맨드 액세스를 제공하는 포털**이다.
- Artifact Report: AWS ISO 인증, PCI(Payment Card Industry), SOC(System and Organization Control) 보고서와 같은 타사 감사자로부터 AWS 보안 및 규정 준수 문서를 다운로드할 수 있다.
- Artifact Agreement: 개인 계정 또는 조직에 대한 BAA(Business Associate Addendum) 또는 HIPAA(Health Insurance Portability and Accountability Act)와 같은 AWS 계약 상태를 검토, 수락 및 추적할 수 있다.
- 내부 감사 또는 규정 준수를 지원하는 데 사용할 수 있다.

---

### AWS Certificate Manager (ACM)

- SSL/TLS 인증서를 쉽게 프로비저닝, 관리, 배포할 수 있다.
- 웹사이트에 진행 중인 암호화를 제공하는 데 사용된다. (HTTPS)
- Public 및 Private TLS 인증서를 모두 지원한다.
- Public TLS 인증서는 무료다.
- 자동 TLS 인증서 갱신을 지원한다.
- Elastic Load Balancer, CloudFront Distributions, API, API Gateway와 통합된다.

![24-aws-certificate-manager.png](images%2F24-aws-certificate-manager.png)

---

### AWS Secrets Manager

- Secert 저장을 위한 서비스다.
- X일마다 보안 Secret을 강제로 교체하는 기능이 있따.
- 교체 시 암호 생성을 자동화한다. (Lambda 사용)
- Amazon RDS(MySQL, PostgreSQL, Aurora)와 통합된다.
- Secret은 KMS를 사용하여 암호화된다.
- 주로 RDS 통합을 의미한다.

#### Multi-Region Secrets

- 여러 AWS 리전에 걸쳐 Secret을 복제한다.
- Secrets Manager는 읽기 전용 복제본을 기본 보안 Secret과 동기화된 상태로 유지한다.
- 읽기 전용 복제본 Secret을 독립형 Secret으로 승격하는 기능을 가지고 있다.
- 다중 리전 앱, 재해 복구 전략, 다중 지역 DB 등의 경우에 사용된다.

![25-secret-manager-multi-region-secrets.png](images%2F25-secret-manager-multi-region-secrets.png)

#### Monitoring

- CloudTrail은 Secrets Manager API에 대한 API 호출을 캡처한다.
- CloudTrail은 AWS 계정에 보안이나 규정 준수에 영항을 미칠 수 있거나 운영 문제를 해결하는 데 도움이 될 수 있는 기타 관련 이벤트를 캡처한다.
- CloudTrail은 이러한 이벤트를 비 API(non-API) 서비스 이벤트로 기록한다.
  - RotationStarted 이벤트
  - RotationSucceeded 이벤트
  - RotationFailed 이벤트
  - RotationAbandoned 이벤트 - 자동 교체 대신 암호를 수동으로 변경한다.
  - StartSecretVersionDelete 이벤트
  - CancelSecretVersionDelete 이벤트
  - EndSecretVersionDelete 이벤트
- 자동화를 위해 CloudWatch Logs 및 CloudWatch Alarm과 통합될 수 있다.

![26-secret-manager-monitoring.png](images%2F26-secret-manager-monitoring.png)

#### Troubleshooting Rotation

![27-secret-manager-trobuleshooting-rotation.png](images%2F27-secret-manager-trobuleshooting-rotation.png)

---

### SSM Parameter Store

- 구성 및 Secret을 위한 안전한 저장소다.
- KMS를 사용한 원활한 암호화 옵션을 제공한다.
- 서버리스, 확장성, 내구성, 간편한 SDK를 지원한다.
- 구성/Secret의 버전 추적이 가능하다.
- 경로 및 IAM을 사용한 구성 관리가 용이하다.
- CloudWatch 이벤트를 통해 변경사항이 있는 경우 알림 전송이 가능하다.
- CloudFormation과 통합될 수 있다.

![28-ssm-parameter-store.png](images%2F28-ssm-parameter-store.png)

#### SSM Parameter Store vs Secrets Manager

- **Secrets Manager**
  - AWS Lambda를 통한 Secrets 자동 교체를 지원한다.
  - RDS, Redshift, DocumentDB에 대한 Lambda 기능을 제공한다.
  - KMS 암호화가 필수다.
  - CloudFormation과 통합될 수 있다.
- **SSM Parameter Store**
  - 간단한 API로 사용이 가능하다.
  - Secret 자동 교체 기능이 없으며, EventBridge에 의해 트리거된 Lambda를 사용하여 교체를 활성화할 수 있다.
  - KMS 암호화는 선택 사항이다.
  - CloudFormation과 통합될 수 있다.
  - SSM Parameter Store API를 사용하여 Secrets Manager Secerts을 가져올 수 있다.

![29-parameter-store-secret-manager-rotation.png](images%2F29-parameter-store-secret-manager-rotation.png)

- Secret Manager의 경우 Secret이 자동으로 순환되기 때문에 자동으로 순환된 이후 Lambda Function을 호출하여 RDS의 비밀번호를 변경하면 된다.
- Parameter Store의 경우 Secret이 자동으로 순환되지 않기 때문에 Lambda Function을 통해 수동으로 Secret을 순환하고 RDS의 비밀번호를 변경해야 한다.

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/)
- [DDoS Attack Mitigation](https://aws.amazon.com/answers/networking/aws-ddos-attack-mitigation/)
- [Penetration Testing](https://aws.amazon.com/security/penetration-testing/)
- [AWS Security Logging](https://d0.awsstatic.com/whitepapers/compliance/AWS_Security_at_Scale_Logging_in_AWS_Whitepaper.pdf)