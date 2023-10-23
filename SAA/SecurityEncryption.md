# AWS Security & Encryption

이번 장에서는 SAA를 준비하며 **AWS의 보안과 암호화**에 대해서 알아보도록 한다.

---

### Encryption

#### 전송 중 암호화(Encryption in flight (SSL))

- 데이터는 전송 전에 암호화되고 수신 후에 복호화된다.
- SSL 인증서를 통해서 HTTPS 암호화를 할 수 있다.
- 전송 중 암호화를 통해 중간 공격자인 "MITM"(man in the middle attack)이 발생하지 않도록 보장한다.

![encryption-in-flight.png](images%2FSecurityEncryption%2Fencryption-in-flight.png)

#### 저장 시 암호화(Encryption at rest)

- 데이터는 서버가 수신한 후 암호화된다.
- 데이터를 전송하기 전에 암호화가 복호화된다.
- 키를 통해서 암호화된 형태로 저장된다.
- 암호화/복호화 키는 어딘가에서 관리되어야 하며, 서버는 암호화/복호화 키에 액세스할 수 있어야 한다.

![serverside-encryption-at-rest.png](images%2FSecurityEncryption%2Fserverside-encryption-at-rest.png)

#### 클라이언트 측 암호화

- 데이터는 클라이언트에 의해 암호화되고 서버에 의해 복호화되지 않는다.
- 수신받은 클라이언트에 의해 데이터가 복호화된다.
- 서버에서 데이터 암호화를 복호화할 수 없다.
- "Envelope"암화화를 활용할 수 있다.

![clientside-encryption.png](images%2FSecurityEncryption%2Fclientside-encryption.png)

---

### AWS KMS(Key Management Service)

- AWS 서비스에 대해 "암호화"라는 말을 들으면 대부분 "KMS"관련 문제일 가능성이 높다.
- AWS에서 암호화 키를 관리해 준다.
- 승인(authorization)을 위해 IAM과 완전히 통합된다.
- 데이터에 대한 액세스를 손쉽게 제어할 수 있는 방법이다.
- "CloudTrail"을 사용하여 KMS Key 사용량을 감사할 수 있다.
- 대부분의 AWS 서비스(EBS, S3, RDS, SSM 등)와 원활하게 통합된다.
- 절대로 **Secrets**을 평문이나 코드에 저장해서는 안된다.
  - KMS Key Encryption은 API 호출(SDK, CLI)를 통해서도 사용할 수 있다.
  - 암호화된 **Secrets**을 코드/환경변수에 저장할 수 있다.

#### 키 유형

- "KSM Keys"는 "KMS Customer Master Key"의 새로운 이름이다.
- **Symmetric(대칭, AES-256 keys)**
  - 암호화 및 복호화에 사용되는 단일 암호화 키다.
  - KMS와 통합된 AWS 서비스는 "Symmetric CMK"를 사용한다.
  - 암호화되지 않은 KMS키에 액세스할 수 없으며, 사용하려면 KMS API를 호출해야 한다.
- **Asymmetric(비대칭, RSA & ECC 키 쌍)**
  - Public(암호화) 및 Private(복호화) 쌍으로 이루어져 있다.
  - 암호화/복호화 또는 서명/확인 작업에 사용된다.
  - Public 키는 다운로드할 수 있지만 암호화되지 않은 Private 키에는 액세스할 수 없다.
  - KMS API 호출이 불가능한 사용자가 AWS 외부에서 암호화할 때 사용된다.

- "KMS 키 유형"
  - "AWS Owned Keys(free)": SSE-S3, SSE-SQS, SSE-DDB (default 키)
  - "AWS Managed Keys(free)": 서비스 이름, RDS, EBS
  - "KMS에서 생성된 고객 관리 키": $1/month
  - "외부에서 가져온 고객 관리 키": 대칭 키만 허용되며 $1/month, 추가로 KMS에 API 호출 비용을 지불해야 한다.
- 자동 키 갱신
  - AWS에서 관리하는 KMS Key: 1년마다 자동으로 갱신된다.
  - 고객 관리형 KMS Key: 활성화되어 있는 경우 1년마다 갱신된다.
  - 외부에서 가져온 고객 관리 키: 별칭을 사용하여 수동으로 갱신해야 한다.

- 아래의 이미지와 같이 "EBS"의 데이터를 KMS 키를 통해서 암호화하고 스냅샷 또한 KMS 키를 통해서 암호화할 수 있다.
- 암호화 이후 다른 Region의 S3 버킷에 복제를 생성할 수 있다.

![copying-snapshots-across-region.png](images%2FSecurityEncryption%2Fcopying-snapshots-across-region.png)

#### Key 정책

- S3 버킷 정책과 유사한 KMS 키에 대한 액세스를 제어할 수 있다.
- 차이점은 KMS 키가 없다면 액세스를 제어할 수 없다.
- 기본 KMS Key 정책:
  - 특정 KMS 키 정책을 제공하지 않을 경우 생성된다.
  - Root 사용자는 전체 AWS 계정에 대한 키에 대해 접근할 수 있다. 
- 사용자 지정 KMS Key 정책:
  - KMS 키에 액세스할 수 있는 사용자, 역할을 정의할 수 있다.
  - 키를 관리할 수 있는 사용자를 정의할 수 있다.
  - KMS 키 교차 계정(cross-account) 액세스에 유용하다.

- 교차 계정에서 스냅샷을 복제하는 방법에 대해서 알아본다.
  1. 자신의 KMS 키(Customer Managed Key)로 암호화된 스냅샷을 생성한다.
  2. 계정 간 액세스 권한을 부여하기 위해 KMS 키 정책을 추가한다.
  3. 암호화된 스냅샷을 공유한다.
  4. 대상 스냅샷 복사본을 만들고 계정의 CMK로 암호화한다.
  5. 스냅샷에서 볼륨을 생성한다.

![copying-snapsionts-across-accounts.png](images%2FSecurityEncryption%2Fcopying-snapsionts-across-accounts.png)

#### Multi-Region Keys

![kms-multi-region-key.png](images%2FSecurityEncryption%2Fkms-multi-region-key.png)

- 서로 다른 AWS Region에서 동일한 KMS 키로 상호 교환하여 사용할 수 있다.
- Multi-Region 키들은 "Key ID", "Key 소재", "Automatic Rotation" 동일한 값을 가진다. 
- 한 영역에서 암호화하고 다른 영역에서 복호화할 수 있다.
- 영역 간 API를 호출하거나 다시 암호화를 할 필요가 없다.
- "KMS Multi-Region"은 글로벌 서비스가 아니며 "Primary"와 "Replicas"로 구성되어 있다.
- 각 "Multi-Region" 키는 독립적으로 관리된다.
- "Global 클라이언트 사이드 암호화", "Global DynamoDB 암호화", "Global Aurora" 등에 사용된다.

#### DynamoDB 글로벌 테이블 & Multi-Region 키 클라이언트 사이드 암호화

- "DynamoDB Encryption Client"를 사용하여 DynamoDB 테이블에서 특정 속성을 암호화할 수 있다.
- 글로벌 테이블과 결합하여 클라이언트 사이드 암호화된 데이터를 다른 영역으로 복제할 수 있다.
- "DynamoDB 글로벌 테이블"과 동일한 영역에 복제된 다중 영역 키를 사용하는 경우, 해당 영역의 클라이언트는 해당 영역의 KMS에 대한 지연 시간이 짧은 API를 호출하여 데이터 클라이언트 측의 암호를 복호화할 수 있다.
- 클라이언트 사이드 암호화를 사용하면 특정 필드를 보호하고 클라이언트가 API 키에 액세스할 수 있는 경우에만 복호화를 보장할 수 있다.

![dynamodb-global-table-kms-multiregion-encryption.png](images%2FSecurityEncryption%2Fdynamodb-global-table-kms-multiregion-encryption.png)

#### Global Aurora & Multi-Region 키 클라이언트 사이드 암호화

- "AWS Encryption SDK"를 사용하여 Aurora 테이블의 특정 속성을 암호화할 수 있다.
- "Aurora Global Tables"와 결합하여 클라이언트 사이드에서 암호화된 데이터를 다른 영역으로 복제할 수 있다.
- "Global Aurora DB"와 동일한 영역에 복제된 Multi-Region 키를 사용하는 경우, 이러한 영역의 클라이언트는 해당 영역의 KMS에 대한 지연 시간이 짧은 API 호출을 사용하여 클라이언트 사이드의 암호를 복호화할 수 있다.
- 클라이언트 사이드 암호화를 사용하여 특정 필드를 보호하고 클라이언트가 API 키에 액세스할 경우에만 복호화를 보장할 수 있도록 데이터베이스 관리자로부터도 특정 필드를 보호할 수 있다.

![globalaurora-multiregion-clientside-encryption.png](images%2FSecurityEncryption%2Fglobalaurora-multiregion-clientside-encryption.png)

#### S3 복제의 암호화 고려사항

- 암호화되지 않은 객체 및 SSE-S3로 암호화된 객체는 기본적으로 복제된다.
- SSE-C(고객 제공 키)로 암호화된 객체는 복제되지 않는다.
- SSE-KMS로 암호화된 객체의 경우 옵션을 활성화해야 한다.
  - 대상 버킷 내의 객체를 암호화할 KMS 키를 지정한다.
  - 대상 키에 대한 KMS 키 정책을 조정한다.
  - IAM 역할(KMS 포함): 소스 KMS 키 및 KMS에 대한 암호화: 대상 KMS 키에 대한 암호화
  - KMS 키를 조회할 때 병목 현상이 발생할 수 있으며, 이 경우 서비스 할당량 증가를 요청할 수 있다.
- Multi-Region AWS KMS 키를 사용할 수 있지만 현재 "Amazon S3"에서 독립적인 키로 처리된다.(객체는 여전히 복호화된 후 암호화)

#### KMS를 통해 암호화된 AMI 공유 프로세스

1. 소스 계정의 AMI가 소스 계정의 KMS 키로 암호화한다.
2. 지정된 대상 AWS 계정에 해당하는 "Launch Permission"을 추가하려면 이미지 속성을 수정해야 한다.
3. AMI 참조 스냅샷을 암호화하는 데 사용되는 KMS 키를 대상 계정 / IAM 역할과 공유해야 한다.
4. 대상 계정의 IAM 역할/사용자에게 DescriptionKey, ReEncrypted, CreateGrant, Decrypt에 대한 권한이 있어야 한다.
5. AMI에서 EC2 인스턴스를 시작할 때, 선택적으로 대상 계정이 자신의 계정에 새 KMS 키를 지정하여 볼륨을 다시 암호화할 수 있다.

![ami-sharing-process-encrypted-kms.png](images%2FSecurityEncryption%2Fami-sharing-process-encrypted-kms.png)

#### SSM Parameter Store

- 구성(Configuration) 및 시크릿(Secrets)을 위한 안전한 스토리지다.
- KMS를 사용한 원활한 암호화가 가능하다.
- 서버리스, 확장성, 내구성, 간편한 SDK를 제공한다.
- 구성/시크릿의 버전을 추적할 수 있다.
- IAM을 통한 보안을 제공한다.
- "Amazon EventBridge"를 통한 알림을 제공한다.
- "Cloud Formation"과 통합되어 사용될 수 있다.

![ssm-parameter-store.png](images%2FSecurityEncryption%2Fssm-parameter-store.png)

#### SSM Parameter Store 계층

![ssm-parameter-store-hierarchy.png](images%2FSecurityEncryption%2Fssm-parameter-store-hierarchy.png)

- "Standard"와 "Advanced"의 차이는 아래의 이미지와 같다.

![ssm-parameter-store-standard-advanced.png](images%2FSecurityEncryption%2Fssm-parameter-store-standard-advanced.png)

- "Advanced"의 경우 파라미터 정책을 설정할 수 있다.
- 매개 변수(만료일)에 TTL을 할당하여 암호와 같은 중요한 데이터를 강제로 업데이트하거나 삭제하도록 허용할 수 있다.
- 여러 정책을 한 번에 할당할 수 있다.

![ssm-parameter-store-parameter-policies.png](images%2FSecurityEncryption%2Fssm-parameter-store-parameter-policies.png)

---

### AWS Secrets Manager

- "Secrets"을 저장하기 위한 새로운 서비스다.
- "N"일마다 "Secrets"을 강제로 순환시키는 기능을 가지고 있다.
- 교체 시 "Secrets" "생성을 "Lambda"를 사용하여 자동화한다.
- "Amazon RDS(MySQL, PostgreSQL, Aurora)"와 통합된다.
- "KMS"를 사용하여 "Secrets"을 암호화한다.
- 대부분 "RDS" 통합을 위하여 사용된다.

#### Multi-Region Secrets

- 여러 AWS Region에 걸쳐 "Secrets"을 복제한다.
- "Secrets Manager"는 읽은 복제본을 기본 "Secrets"과 동기화하여 보관한다.
- 읽은 복제본 "Secrets"을 독립 실행형 "Secrets"로 승격시키는 기능을 가지고 있다.
- Multi-Region APP, 재해 복구 전략, Multi-Region DB 등에 활용된다.

![secrets-manager-multiregion-secrets.png](images%2FSecurityEncryption%2Fsecrets-manager-multiregion-secrets.png)

---

### AWS Certificate Manager (ACM)

- TLS 인증서를 쉽게 프로비저닝, 관리 및 배포할 수 있다.
- 웹 사이트(HTTP)에 대한 전송 중 암호화를 지원한다.
- 공인 TLS 인증서와 사설 TLS 인증서를 모두 지원한다.
- 공인 TLS 인증서를 무료로 제공한다.
- TLS 인증서 자동 갱신 기능을 제공한다.
- "Load TLS 인증서"를 활성화하여 아래의 항목들과 통합될 수 있다.
  - Elastic Load Balancer(CLB, ALB, NLB)
  - CloudFront Distributions
  - API Gateway 상의 API
- ACM을 직접적으로 EC2와 함께 사용할 수 없다.

![certificate-manager.png](images%2FSecurityEncryption%2Fcertificate-manager.png)

#### 공인 인증서(Public Certificates) 요청

1. 인증서에 포함할 도메인 이름을 나열한다.
   - FQDN(Fully Qualified Domain Name): corm.example.com
   - Wildcard Domain: *.example.com
2. 유효성 검사 방법을 선택한다. DNS 검증 또는 이메일 검증
   - 자동화를 위해 DNS 유효성 검사가 선호된다.
   - 전자 메일 유효성 검사는 WHOIS 데이터베이스의 연락처 주소로 전자메일을 전송한다.
   - DNS 유효성 검사는 CNAME 레코드를 활용하여 DNS를 구성한다.(예: Route 53)
3. 이러한 유효성 검사는 확인되는 데 몇 시간이 소요된다.
4. 공인 인증서가 자동 갱신을 위해 등록된다.
   - ACM은 만료 60일 전에 ACM에서 생성한 인증서를 자동으로 갱신한다.

#### 공인 인증서 가져오기(Importing)

- ACM 외부에서 인증서를 생성한 다음 가져오는 옵션이다.
- 자동 갱신은 불가능하며 만료 전에 새로운 인증서를 가져와야 한다.
- ACM은 만료 45일 전부터 메일 만료 이벤트를 전송한다.
  - 몇일 전부토 발송할 것인지 구성할 수 있다.
  - "EventBridge"에 이벤트가 생성된다.
- "AWS Config"에는 만료되는 인증서를 확인하기 위한 `acm-certificate-expiration-check(구성 가능한 일 수)`라는 관리 규칙이 있다.

![certificate-manager-importing-public-certificates.png](images%2FSecurityEncryption%2Fcertificate-manager-importing-public-certificates.png)

- "ACM"은 아래의 이미지처럼 ALB와 통합되어 사용될 수 있다.

![certificate-manager-integration-with-alb.png](images%2FSecurityEncryption%2Fcertificate-manager-integration-with-alb.png)

#### API Gateway - Endpoint 유형

- **Edge-Optimized(기본값)**: 글로벌 글라이언트용
  - "CloudFront Edge Location"을 통해 요청을 라우팅한다.
  - "API Gateway"가 여전히 한 Region에서만 작동한다.
- **Regional**:
  - 동일한 지역 내 클라이언트의 경우
  - "CloudFront"와 수동으로 결합 가능하다.(캐싱 전략 및 배포에 대한 통제력을 강화한다.)
- **Private**:
  - "Interface VPC Endpoint"를 사용하여 VPC에서만 액세스할 수 있다.
  - 리소스 정책을 사용하여 액세스를 정의한다.

#### ACM - API Gateway와의 통합

- "API Gateway"에서 사용자 지정 도메인 이름을 생성한다.
- **Edge-Optimized(기본값)**: 
  - 글로벌 클라이언트의 경우에 사용된다.
  - "CloudFront Edge Location"을 통해 요청이 라우팅된다.
  - "API Gateway"는 하나의 Region에만 존재한다.
  - TLS 인증서는 "CloudFront"와 동일한 지역(`us-east-1`)에 있어야 한다.
  - "Route 53"에서 CNAME 또는 A-Alias 레코드를 설정한다.
- **Regional**:
  - 동일한 Region 내 클라이언트의 경우에 사용된다.
  - TLS 인증서는 "API Gateway"에서 API Stage와 동일한 Region으로 가져와야 한다.
  - "Route 53"에서 CNAME 또는 A-Alias 레코드를 설정한다.

![certificate-manager-integration-api-gateway.png](images%2FSecurityEncryption%2Fcertificate-manager-integration-api-gateway.png)

---

### AWS WAF - Web Application Firewall

- 일반적인 웹 공격으로부터 웹 응용프로그램을 보호한다.(Layer 7)
- Layer 7은 HTTP (Layer 4는 TCP/UDP)
- 배포되는 위치는 아래와 같다.
  - Application Load Balancer
  - API Gateway
  - CloudFront
  - AppSync GraphQL API
  - Cognito User Pool

- Web ACL(Web Access Control List) 규칙 정의
  - IP Set: 최대 10,000개의 IP 주소 - 더 많은 IP에 대해 여러 규칙을 사용할 수 있다.
  - HTTP 헤더, HTTP 바디, URI 문자열 공통 공격, SQL 주입 및 사이트간 스크립팅(XSS)으로부터 보호한다.
  - 크기 제한, geo-match (국가 제한)
  - 속도 기반 규칙(이벤트 발생 횟수 계산) - DDoS를 보호한다.
- "CloudFront"를 제외한 Region별 "Web ACL"
- 규칙 그룹은 "Web ACL"에 추가할 수 있는 재사용 가능한 규칙의 집합이다.

#### WAF와 Load Balancer를 사용하는 동안 고정 IP

- "WAF"가 네트워크 로드 밸런서(Layer 4)를 지원하지 않는다.
- "ALB"의 고정 IP 및 WAF에 대해 "Global Accelerator"를 사용할 수 있다.

![fixed-ip-waf-load-balancer.png](images%2FSecurityEncryption%2Ffixed-ip-waf-load-balancer.png)

#### AWS Shield: protected from DDoS attack

- DDoS: 분산형 서비스 거부 서비스다. 
- **AWS Shield Standard**
  - 모든 AWS 고객을 대상으로 활성화되는 무료 서비스다.
  - SYN/UDP Flood, Reflection 공격 및 기타 Layer 3/Layer 4 등의 공격으로부터 보호한다.
- **AWS Shield Advanced**
  - DDoS 완화 서비스(Organization 당 월 $3,000가 발생한다.)
  - "Amazon EC2", "ELB(Elastic Load Balancing)", "Amazon CloudFront", "AWS Global Accelerator" 및 "Route 53"에 대한 보다 정교한 공격으로부터 보호한다.
  - "AWS DDoS 대응 팀(DRP)"의 연중무휴 액세스를 제공한다.
  - DDoS로 인해 사용량이 급증할 때 더 높은 수수료로부터 보호한다.
  - "Shield Advanced" 자동 애플리케이션 DDoS 완화를 통해 "AWS WAF" 규칙을 자동으로 생성, 평가 및 배포하여 Layer 7 공격을 완화한다.

#### AWS Firewall Manager

- AWS 조직의 모든 계정에서 규칙 관리를 한다.
- 보안 정책: 공통 보안 규칙 집합
  - WAF 규칙(Application Load Balancer, API Gateway, CloudFront)
  - AWS Shield Advanced (ALB, CLB, NLB, Elastic IP, CloudFront)
  - VPC의 EC2, Application Load Balancer 및 ENI 리소스에 대한 Security Group
  - AWS Network Firewall(VPC 레벨)
  - Amazon Route 53 Resolver DNS 방화벽
  - Region 수준에서 정책이 생성된다.
- 규정은 조직의 모든 계정과 향후 계정에 걸쳐 생성(규정에 적합)될 때, 새 리소스에 적용된다.

#### WAF vs Firewall Manager vs Shield

- 포괄적인 보호를 위해 WAF, Shield 및 Firewall Manager가 함께 사용된다.
- WAF에서 웹 ACL 규칙 정의
- 리소스를 세분화하여 보호하려면 WAF만 사용하는 것이 올바른 선택이다.
- AWS WAF를 여러 계정에 걸쳐 사용하려면 WAF 구성을 가속화하고 새로운 리소스 보호를 자동화하고 Firewall Manager를 AWS WAF와 함께 사용해야 한다.
- "Shield Advanced"는 SRT(Shield Response Team)의 전용 지원 및 고급 보고 기능과 같은 AWS WAF에 추가 기능을 제공한다.
- DDoS 공격이 자주 발생하는 경우에는 "Shield Advanced"를 구입하는 것을 고려할 수 있다.

#### DDoS 복원력 Edge Location 완화(BP1, BP3) 모범 사례

- **BP1 - CloudFront**
  - Edge에서의 웹 애플리케이션 제공
  - DDoS 공통 공격(SYN Flood, UDP Reflection 등..)으로부터 보호
- **BP1 - Global Accelerator**
  - Edge에서 응용프로그램에 액세스
  - DDoS 보호를 위한 Shield와의 통합
  - 백엔드가 CloudFront와 호환되지 않는 경우에도 도움이 된다.
- **BP3 - Route 53**
  - Edge의 도메인 이름 확인
  - DDoS 보호 메커니즘

![ddos-resiliency-1.png](images%2FSecurityEncryption%2Fddos-resiliency-1.png)

#### DDoS 완화를 통한 DDoS 복원력 모범 사례

- **인프라 계층 방어(BP1, BP3, BP6)**
  - 높은 트래픽으로부터 Amazon EC2 보호
  - "Global Accelerator", "Route 53", "CloudFront", "Elastic Load Balancing" 사용이 포함된다.
- **Amazon EC2 with Auto Scaling(BP7)**
  - 플래시 군중(Flash Crowd) 또는 DDoS 공격과 같은 갑작스러운 트래픽 급증 시 확장 가능하다.
- **Elastic Load Balancing(BP6)**
  - Elastic Load Balancing은 트래픽 증가에 따라 확장되며 트래픽을 많은 EC2 인스턴스로 분산한다.

![ddos-resiliency-2.png](images%2FSecurityEncryption%2Fddos-resiliency-2.png)

#### Application 계층 방어를 통한 DDoS 복원력 모범 사례

- **악성 웹 요청 탐지 및 필터링(BP1, BP2)**
  - "CloudFront"는 정적 컨텐츠를 캐싱하고 Edge Location에서 서비스하여 백엔드를 보호한다.
  - "AWS WAF"는 "CloudFront" 및 "Application Load Balancer" 위에서 요청 서명을 기반으로 요청을 필터링하고 차단하는 데 사용된다.
  - "WAF" 속도 기반 규칙을 통해 불량 행위자의 IP를 자동으로 차단할 수 있다.
  - "WAF"에서 관리되는 규칙을 사용하여 IP 평판에 기반한 공격을 차단하거나 익명 IP들을 차단할 수 있다.
  - "CloudFront"는 특정 지역을 차단할 수 있다.
- **Shield Advanced(BP1, BP2, BP6)**
  - "Shield Advanced"는 자동 애플리케이션 계층 DDoS 완화를 통해 AWS WAF 규칙을 자동으로 생성, 평가 및 배포하여 Layer 7 공격을 완화한다.

![ddos-resiliency-3.png](images%2FSecurityEncryption%2Fddos-resiliency-3.png)

#### DDoS 탄력성 공격 표면 감소(Attack surface reduction)에 대한 모범 사례

- **AWS 리소스 난독화(BP1, BP4, BP6)**
  - "CloudFront", "API Gateway", "Elastic Load Balancing"을 사용하여 백엔드 리소스(Lambda Function, EC2 인스턴스)를 숨긴다.
- **Security groups and Networks ACLs(BP5)**
  - Security Group 및 NACL을 사용하여 서브넷 또는 ENI 수준의 특정 IP를 기반으로 트래픽을 필터링한다.
  - 탄력적인 IP는 "AWS Shield Advanced"에 의해 보호된다.
- **Protecting API endpoints(BP4)**
  - EC2, Lambda를 다른 곳으로 숨긴다.
  - Edge-optimized 모드, CloudFront + Regional 모드(DDoS에 대한 제어 강화)
  - WAF + API Gateway: 버스트(burst) 제한, 헤더 필터링, API 키 사용

![ddos-resiliency-4.png](images%2FSecurityEncryption%2Fddos-resiliency-4.png)

---

### Amazon GuardDuty

- AWS 계정을 보호하는 지능형 위협을 검색한다.
- 머신러닝 알고리즘, 이상 탐지, 3rd party 데이터를 사용한다.
- 클릭 한 번으로 활성화(30일 평가판)할 수 있으며, 소프트웨어 설치가 필요없다.
- 입력 데이터는 아래와 같은 데이터를 포함하고 있다.
  - **CloudTrail Events Logs** - 비정상적인 API 호출, 무단 배포
    - **CloudTrail Management Events** - VPC 서브넷 생성, trail 생성 등..
    - **CloudTrail S3 Data Events** - 객체 가져오기, 객체 나열, 객체 삭제 등..
  - **VPC FLow Logs**: 비정상적인 내부 트래픽, 비정상적인 IP 주소
  - **DNS Logs**: 손상된 EC2 인스턴스가 DNS 쿼리 내에서 인코딩된 데이터 전송.
  - **Kubernetes Audit Logs**: 의심스러운 활동 및 EKS 클러스터 손상 가능성.
- 발견 시 알림을 받을 "EventBridge" 규칙을 설정할 수 있다.
- "EventBridge" 규칙은 "AWS Lambda" 또는 SNS를 대상으로 할 수 있다.
- "CryptoCurrency" 공격으로부터 보호가 가능하다.

![guardduty.png](images%2FSecurityEncryption%2Fguardduty.png)

---

### Amazon Inspector

- 자동화된 보안 평가를 제공한다.
- **EC2 인스턴스**
  - AWS SSM(System Manager) 에이전트를 활용한다.
  - 의도하지 않은 네트워크 액세스 가능성에 대비하여 분석한다.
  - 알려진 취약성에 대해 실행 중인 OS를 분석한다.
- **컨테이너 이미지의 경우 Amazon ECR로 Push**
  - 컨테이너 이미지를 푸시할 때 평가한다.
- **Lambda Function**
  - 함수 코드 및 패키지 종속성에 대한 소프트웨어 취약성을 식별한다.
  - Functions 들이 배포될 때 평가한다.
- 보고서 작성 및 "AWS Security Hub"와 통합된다.
- "Amazon Event Bridge"로 결과를 전송할 수 있다.

![inspector.png](images%2FSecurityEncryption%2Finspector.png)

- "Amazon Inspector"는 아래의 항목들을 평가한다.
  - **Remember**: EC2 인스턴스, Container Image & Lambda Function
  - 필요한 경우 인프라를 지속적으로 스캔한다.
  - 패키지 취약성(EC2, EC2 & Lambda) - CVE 데이터베이스
  - 네트워크 도달 가능성(EC2)
  - 위험 점수는 우선 순위 지정을 위한 모든 취약성과 관련이 있다.

---

### AWS Macie

- "Amazon Macie"는 기계 학습 및 패턴 매칭을 사용하여 AWS에서 중요한 데이터를 검색하고 보호하는 완전 관리형 데이터 보안 및 데이터 개인 정보 보호 서비스다.
- "Macie"는 PII(Personally Identifiable Information)와 같은 민감한 데이터를 식별하고 이를 알려주는 데 도움을 준다.

![macie.png](images%2FSecurityEncryption%2Fmacie.png)

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03