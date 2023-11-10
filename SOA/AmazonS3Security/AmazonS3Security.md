# Amazon S3 Security

이번 장에서는 **SysOps Administrator**를 준비하며 **S3 보안**에 대해서 알아보도록 한다.

---

### Encryption

- 4가지 방법 중 하나를 사용하여 S3 버킷의 객체를 암호화할 수 있다.
- Server-Side 암호화(SSE)
  - Amazon S3-Managed Keys(SSE-S3)를 사용한 서버 측 암호화 - 기본적으로 활성화된다.
    - AWS에서 처리, 관리 및 소유한 키를 사용하여 S3 객체를 암호화한다.
  - AMS KMS(SSE-KMS)에 저장된 KMS 키를 사용한 서버 측 암호화
    - AWS Key Management Service(AWS KMS)를 활용하여 암호화 키를 관리한다.
  - Customer-Provider Keys(SSE-C) 고객 제공 키를 사용하여 서버측 암호화를 한다.
    - 자체 암호화 키를 관리하려는 경우에 사용한다.
- Client-Side 암호화

#### SSE-S3

- AWS에서 처리, 관리 및 소유한 키를 사용하여 암호화한다.
- 객체는 서버 측에서 암호화된다.
- 암호화 유형은 "AES-256"이다.
- 헤더에 `x-amz-server-side-encryption:AES256` 설정을 해야 한다.
- 새로운 버킷 및 새로운 객체에 대해 기본적으로 활성화된다.

![1-encryption-sse-s3.png](images%2F1-encryption-sse-s3.png)

#### SSE-KMS

- AWS KMS(Key Management Service)에서 처리하고 관리하는 키를 사용하여 암호화한다.
- KMS를 사용하면 CloudTrail을 사용한 사용자 제어와 감사(audit)키로 사용할 수 있다는 장점이 있다.
- 객체는 서버 측에서 암호화된다.
- 헤더에 `x-amz-server-side-encryption:aws:kms` 설정을 해야 한다.

![2-encryption-sse-kms.png](images%2F2-encryption-sse-kms.png)

#### SSE-KSM 제한

- SSE-KMS를 사용하는 경우 KMS 제한의 영향을 받을 수 있다.
- 파일을 업로드하면 GenerateDataKey KMS API가 호출된다.
- 다운로드 시 Decrypt KMS API를 호출한다.
- 초당 KMS 할당량에 포함된다.
  - 5500, 10000, 30000 요청/초로 지역에 따라 다르다.
- Service Quotas 콘솔을 사용하여 할당량 증가를 요청할 수 있다.

![3-sse-kms-limitation.png](images%2F3-sse-kms-limitation.png)

#### SSE-C

- AWS 외부에서 고객이 완전히 관리하는 키를 사용하여 서버 측에서 암호화한다.
- Amazon S3는 사용자가 제공한 암호화 키를 저장하지 않는다.
- 반드시 HTTPS를 사용해야 한다.
- 모든 HTTP 요청에 대해 암호화 키가 HTTP 헤더에 제공되어야 한다.

![4-encryption-sse-c.png](images%2F4-encryption-sse-c.png)

#### Client-Side Encryption

- S3 클라이언트 측 암호화 라이브러리와 같은 클라이언트 라이브러리를 사용해야 한다.
- 클라이언트는 S3로 전송하기 전에 데이터를 자체 암호화해야 한다.
- S3에서 데이터를 검색할 때, 클라이언트가 직접 데이터를 복호화해야 한다.
- 고객이 키와 암호화 주기를 완전히 관리한다.

![5-encryption-client-side-encryption.png](images%2F5-encryption-client-side-encryption.png)

#### 전송 중 암호화 (SSL/TLS)

- 전송 중 암호화를 SSL/TLS라고 한다.
- Amazon S3는 두 개의 엔드포인트를 노출한다.
  - HTTP 엔드포인트 - 암호화되지 않는다.
  - HTTPS 엔드포인트 - 전송 중 암호화된다.
- HTTPS 사용이 권장된다.
- SSE-C를 사용하는 경우 HTTPS가 필수다.
- 대부분의 클라이언트는 기본적으로 HTTPS 엔드포인트를 사용한다.

![6-force-encryption-in-transit.png](images%2F6-force-encryption-in-transit.png)

- 버킷 정책(`aws:SecureTransport`)을 통해서 HTTPS를 사용한 객체만 업로드되도록 설정할 수 있다.

#### Default Encryption vs Bucket Policy

- SSE-S3 암호화는 S3 버킷에 저장된 새로운 객체에 자동으로 적용된다.
- 선택적으로 버킷 정책을 사용하여 "암호화를 강제"하고 암호화 헤더(SSE-KMS 또는 SSE-C)없이 S3 객체를 PUT하는 API 호출을 거부할 수 있다.

![7-default-encryption-bucket-policy.png](images%2F7-default-encryption-bucket-policy.png)

- **버킷 정책은 기본 암호화 이전에 평가된다.**

---

### CORS

- CORS는 Cross-Origin Resource Sharing의 약자다.
- Origin = Scheme(protocol) + Host(domain) + port
  - 예를 들어, `https://wwww.example.com` (암시된 포트는 HTTPS의 경우 443, HTTP의 경우 80)
- 기본 Origin을 방문하는 동안 다른 Origin에 대한 요청을 허용하는 웹 브라우저 기반의 메커니즘이다.
- `http://example.com/app1` 와 `http://example.com/app2` 는 동일한 Origin이다.
- `http://www.example.com` 와 `http://other.example.com`은 다른 Origin이다.
- CORS 헤더(예: `Access-Control-Allow-Origin`)을 사용하여 다른 Origin에서 요청을 허용하지 않으면 요청이 이행되지 않는다.

![8-what-is-cors.png](images%2F8-what-is-cors.png)

- 클라이언트가 S3 버킷에 교차 출처 요청(Cross-Origin Request)하는 경우 올바른 CORS 헤더를 활성화해야 한다.
- 특정 Origin 또는 `*`(모든 Origin)을 허용할 수 있다.

![9-s3-cors.png](images%2F9-s3-cors.png)

---

### MFA Delete

- MFA(Multi-Factor Authentication): S3에서 중요한 작업을 수행하기 전에 사용자가 장치(일반적으로 휴대폰 또는 하드웨어)에서 코드를 생성하도록 강제한다.
- 아래의 항목을 수행하기 위해서 MFA가 필요하다.
  - 객체 버전을 영구적으로 삭제한다.
  - 버킷의 버전 관리를 일시 중지한다.
- 아래의 항목을 수행하기 위해서 MFA가 필요하지 않다.
  - 버전 관리를 활성화한다.
  - 삭제된 버전을 나열한다.
- MFA 삭제를 사용하려면 버킷에서 버전 관리를 활성화해야 한다.
- 버킷 소유자(루트 계정)만 MFA 삭제를 활성화하거나 비활성화할 수 있다.

---

### Access Log

- 감사(audit)를 목적으로 S3 버킷에 대한 모든 액세스를 기록할 수 있다.
- 승인되거나 거부된 모든 계정에서 S3에 대한 모든 요청은 다른 S3 버킷에 기록된다.
- 해당 데이터는 데이터 분석 도구를 사용하여 분석할 수 있다.
- 대상 로깅 버킷은 동일한 리전에 있어야 한다.

![10-access-log.png](images%2F10-access-log.png)

- 로그를 생성하는 버킷을 로그가 저장되는 버킷으로 설정하면 안된다.
- 로깅 루프가 생성되고 버킷에 저장되는 데이터가 기하급수적으로 증가한다.

![11-access-log-warning.png](images%2F11-access-log-warning.png)

---

### Pre-Signed URL

- S3 콘솔, AWS CLI 또는 SDK를 사용하여 사전 서명된 URL을 생성한다.
- URL 만료
  - S3 콘솔: 1 ~ 720분(12시간)
  - AWS CLI: `--expires-in` 파라미터를 사용하여 초 단위로 만료를 구성할 수 있다.(기본값 3600초, 최대 604800초(168시간))
- Pre-Signed URL이 부여된 사용자는 GET/PUT용 URL을 생성한 사용자의 권한을 상속받는다.
- 예를 들어, 아래와 같은 작업에 사용된다.
  - 로그인한 사용자만 S3 버킷에서 프리미엄 비디오를 다운로드하도록 허용한다.
  - URL을 동적으로 생성하여 끊임없이 변화하는 사용자 목록에서 파일을 다운로드할 수 있도록 허용한다.
  - 일시적으로 사용자가 S3 버킷의 정확한 위치에 파일을 업로드하도록 허용한다.

![12-pre-signed-url.png](images%2F12-pre-signed-url.png)

---

### Glacier Vault Lock

- WORM(Write Once Read Many) 모델을 채택한다.
- Vault Lock Policy를 생성한다.
- 향후 수정을 위해 정착을 잠근다. 더 이상 변경하거나 삭제할 수 없다.
- 규정 준수 및 데이터 보존에 도움이 된다.

![13-glacier-vault-lock.png](images%2F13-glacier-vault-lock.png)

#### Object Lock

- 버전 관리가 반드시 활성화되어 있어야 한다.
- WORM(Write Once Read Many) 모델을 채택한다.
- 지정된 시간 동안 객체 버전 삭제를 차단한다.
- **Retention mode - Compliance**:
  - 루트 사용자를 포함한 어떤 사용자도 객체 버전을 덮어쓰거나 삭제할 수 없다.
  - 객체 보관 모드는 변경할 수 없으며, 보관 기간은 단축할 수 없다.
- **Retention mode - Governance**:
  - 대부분의 사용자는 객체 버전을 덮어쓰거나 삭제할 수 없으며 객체의 잠금 설정을 변경할 수도 없다.
  - 일부 사용자에게는 보존을 변경하거나 객체를 삭제할 수 있는 특별한 권한이 있다.
- **Retention Period**: 일정기간 동안 객체를 보호하며, 연장할 수 있다.
- **Legal Hold**:
  - 보존 기간에 관계없이 객체를 무기한 보호한다.
  - `s3:PutObjectLegalHold` IAM 권한을 사용하여 자유롭게 배치하고 제거할 수 있다.

---

### Access Point

![14-access-point.png](images%2F14-access-point.png)

- 액세스 포인트는 S3 버킷의 보안 관리를 단순화한다.
- 각 액세스 포인트에는 아래의 항목이 포함된다.
  - 자체 DNS 이름(Internet Origin 또는 VPC Origin)
  - 액세스 포인트 정책(버킷 정책과 유사): 대규모 보안 관리

#### VPC Origin

- VPC 내에서만 액세스할 수 있도록 액세스 포인트를 정의할 수 있다.
- 액세스 포인트(Gateway 또는 Interface 엔드포인트)에 액세스하려면 VPC 엔드포인트를 생성해야 한다.
- VPC 엔드포인트 정책은 대상 버킷 및 액세스 포인트에 대한 액세스를 허용해야 한다.

![15-access-point-vpc-origin.png](images%2F15-access-point-vpc-origin.png)

#### Multi-Region Access Point

![16-multi-region-access-point.png](images%2F16-multi-region-access-point.png)

- 여러 AWS Region의 S3 버킷에 걸쳐 있는 글로벌 엔드포인트를 제공한다.
- 가장 가까운 S3 버킷으로 요청을 동적으로 라우팅한다.(최저 지연 시간)
- 지역 간 데이터 동기화를 유지하기 위해 양방향 S3 버킷 복제 규칙이 생성된다.
- **장애 조치 제어** - 몇 분 안에 다양한 AWS Region의 S3 버킷 간에 요청을 이동할 수 있다.(Active-Active 또는 Active-Passive)

![17-multi-region-access-point-2.png](images%2F17-multi-region-access-point-2.png)

- 버킷이 두 Region에 걸쳐 복제되어 있고, Multi-Region 액세스 포인트가 있다.
- 활성화된 버킷이 하나인 경우 요청은 최소 대기 시간으로 가지 않고 활성화된 곳으로 이동한다.
- 활성화된 버킷이 여러개인 경우 대기 시간이 가장 짧은 Region으로 요청이 전달된다.

![18-multi-region-access-point-failover-controls.png](images%2F18-multi-region-access-point-failover-controls.png)

- 이러한 장애 조치 컨트롤은 Active/Passive에서도 유용하지만 대기 시간 관점에서 Active/Active에서도 유효하다.

---

### VPC Endpoint Gateway

![19-vpc-endpoint-gateway.png](images%2F19-vpc-endpoint-gateway.png)

- 기본적으로 S3 버킷은 AWS 클라우드에 있다.
- 접속하기 위해서는 공용 인터넷을 거쳐야 한다.
- Public 인스턴스의 경우 Internet Gateway를 통해 접근할 수 있다.
- Private 인스턴스의 경우 인터넷 통신을 할 수 없다.
- 이러한 경우 VPC Endpoint Gateway를 통하여 공용 인터넷을 거치지않고 S3 버킷에 접근할 수 있다.
- **보안과 비용 측면에서 VPC Endpoint Gateway가 선호된다.**

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [S3 Access Log Format](https://docs.aws.amazon.com/AmazonS3/latest/dev/LogFormat.html)