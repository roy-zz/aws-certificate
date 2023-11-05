# Amazon S3

이번 장에서는 **SysOps Administrator**를 준비하며 **Simple Storage Service의 약자인 S3**에 대해서 알아보도록 한다.

---

### Simple Storage Service (S3)

- S3는 AWS의 주요 구성 요소 중 하나다.
- '무한 확장' 가능한 스토리지다.
- 많은 웹 사이트에서 S3를 백본으로 사용하고 있다.
- 많은 AWS 서비스에서도 S3와 통합하는 기능을 제공하낟.

- S3는 많은 사용 사례를 가지고 있다.
  - 백업 및 저장
  - 재해 복구
  - 보관소
  - 하이브리드 클라우드 스토리지
  - 애플리케이션 호스팅
  - 미디어 호스팅
  - 데이터 레이크 및 빅데이터 분석
  - 소프트웨어 제공
  - 정적 웹사이트
- "Nasdaq"에서는 7년간 저장되어 있는 데이터를 S3 Glacier로 이전하였다.
- "Sysco"는 데이터에 대한 분석을 실행하고 비즈니스 통찰력(Business Insights)을 얻고 있다.

#### Bucket

- S3를 사용하면 사용자는 버킷(디렉터리)에 객체(파일)을 저장할 수 있다.
- 버킷은 전역적으로 고유한 이름을 가져야 한다.(모든 Region, 모든 Account)
- 버킷은 Region 수준에서 정의된다.
- S3는 글로벌 서비스처럼 보이지만 버킷은 한 Region에 생성된다.
- 따라야 하는 명명 규칙은 아래와 같다.
  - 대문자와 밑줄(`_`)을 사용할 수 없다.
  - 3~63 길이의 이름을 가질 수 있다.
  - IP를 사용할 수 없다.
  - 소문자 또는 숫자로 시작해야 한다.
  - 접두사를 `xn--`사용할 수 없다.
  - 접미사를 `-s3alias`사용할 수 없다.

#### Object

- 객체(파일)에는 키가 있다.
- 키는 전체 경로다.
  - `s3://my-bucket/my_file.txt`
  - `s3://my-bucket/my_folder1/another_folder/my-file.txt`
- 키는 접두사 + 객체 이름으로 구성된다.
  - `s3://my-bucket/my_folder1/another_folder/my_file.txt`
- **버킷 내에 '디렉터리'라는 개념이 없다.** (UI가 사용자를 속여서 다르게 생각하도록 유도할 수 있다.)
- 키는 슬래시(`/`)를 포함하는 아주 긴 이름이고 키는 접두사와 객체 이름으로 이루어져 있다.

- 객체의 값은 본문의 내용이다.
  - 최대 객체의 크기는 5TB(5,000GB)다.
  - 5GB크기 이상의 파일을 업로드하는 경우 '멀티파트 업로드'를 사용해야 한다.
- 메타데이터(텍스트 키/값 쌍 목록 - 최대 10개) - 보안/수명 주기에 유용하다.
- 버전 관리가 활성화된 경우 "Version ID"를 가지고 있다.

---

### S3 Security

- User-Based
  - IAM Policy: IAM의 특정 사용자에게 허용되는 API 호출이 허용되는지 승인한다.
- Resource-Based
  - Bucket Policy: S3 콘솔의 버킷 전체 규칙을 설정한다. 교차 계정 액세스를 허용할 수 있다.
  - Object Access Control List (ACL): 더 세부적인 내용(비활성화 가능)
  - Bucket Access Control List (ACL): 덜 일반적임 (비활성화 가능)
- 참고: IAM 주체는 다음과 같은 경우 S3 객체에 액세스할 수 있다.
  - 사용자 IAM 권한이 이를 허용하거나 리소스 정책이 이를 허용한다.
  - 명시적인 `DENY` 정책이 없다.
- 암호화: 암호화 키를 사용하여 S3의 객체를 암호화한다.

#### S3 Bucket Policy

- JSON 기반 정책
  - Resources: 버킷 또는 객체
  - Effect: 허용(Allow) / 거부(Deny)
  - Actions: 허용 또는 거부를 위한 API 집합
  - Principal: 정책을 적용할 계정 또는 사용자
- 정책에 S3 버킷을 사용하여 아래의 항목을 수행할 수 있다.
  - 버킷에 대한 공개 액세스 권한을 부여한다.
  - 업로드 시 객체를 강제로 암호화한다.
  - 다른 계정에 대한 액세스 권한을 부여한다.(교차 계정)

![1-s3-bucket-policy.png](images%2F1-s3-bucket-policy.png)

- Principal은 `*`이므로 누구든지 `GetObject`를 할 수 있다.

#### Public Access - 버킷 정책 사용

![2-public-access-use-bucket-policy.png](images%2F2-public-access-use-bucket-policy.png)

- S3 버킷 정책을 이용해 버킷에 Public Access를 부여할 수 있다.
- 오른쪽에 있는 버킷에서 업로드하는 객체를 암호화하거나 다른 계정에 액세스 권한을 부여할 수 있다.
- 익명의 `www` 사용자가 S3 버킷 내의 파일에 액세스하기를 원하는 경우 S3 버킷 정책을 사용하여 공용 액세스를 허용할 수 있다.

#### User Access - IAM 정책 사용

![3-user-access-iam-permission.png](images%2F3-user-access-iam-permission.png)

- 계정에 IAM 사용자가 버킷에 접근하는 것을 원하는 경우가 있을 수 있다.
- IAM 정책을 통해서 권한을 할당할 수 있다.
- 정책이 S3 액세스할 수 있는 권한을 할당하기 때문에 IAM 사용자는 S3 버킷에 접근할 수 있다.

#### Instance Access - IAM 역할 사용

![4-instance-access-use-iam-role.png](images%2F4-instance-access-use-iam-role.png)

- EC2 인스턴스의 S3 버킷 액세스를 허용하는 경우 IAM 사용자는 적합하지 않다.
- 대신 IAM 역할을 사용해야 한다.
- 올바른 IAM 사용 권한으로 EC2 인스턴스 역할을 만들고 역할을 인스턴스에 연결하여 S3 버킷에 액세스할 수 있도록 설정할 수 있다.

#### Cross-Account Access - 버킷 정책 사용

![5-cross-account-access-bucket-policy.png](images%2F5-cross-account-access-bucket-policy.png)

- 다른 계정의 IAM 사용자에게 S3 버킷 접근 권한을 할당하고 싶다면 버킷 정책을 사용해야 한다.
- 다른 계정의 IAM 사용자의 접근을 허용하는 버킷 정책을 생성하면 다른 계정의 IAM 사용자는 버킷에 접근할 수 있고 API를 호출할 수 있다.

#### Block Public Access

![6-block-public-access.png](images%2F6-block-public-access.png)

- AWS가 회사 데이터 유출을 방지하기 위해 추가로 개발한 설정이다.
- 공개적인 S3 버킷 정책을 설정하더라도 이 설정이 되어 있다면 Public 접근을 할 수 없다.
- 계정 수준에서 설정할 수 있다.

---

### S3 Bucket Policy Advanced

- 정책에 S3 버킷을 사용하여 다음을 수행한다.
  - 버킷에 대한 공개 액세스 권한을 부여한다.
  - 업로드 시 객체를 강제로 암호화한다.
  - **다른 계정에 대한 액세스 권한을 부여(교차 계정)한다.**
- 선택 조건이 있다.
  - Public IP 또는 Elastic IP (Private IP는 지원하지 않음)
  - Source VPC 또는 Source VPC Endpoint - VPC 엔드포인트에서만 작동한다.
  - CloudFront 원본 ID
  - MFA
- 여러 가지 버킷 정책은 [공식 홈페이지](https://docs.aws.amazon.com/AmazonS3/latest/dev/example-bucket-policies.html)에서 확인할 수 있다.

#### 예시

- 아래는 예시는 `aws:PrincipalOrgID` 조컨 키를 사용하여 AWS 조직 내의 AWS 계정의 보안 주체로만 액세스를 제한한다.

![7-bucket-policy-advanced-1.png](images%2F7-bucket-policy-advanced-1.png)

- 아래는 예시는 `s3:x-amz-server-side-encryption` 조건 키를 사용하여 암호화되지 않은 객체가 S3 버킷에 업로드되는 것을 방지한다.

![8-bucket-policy-advanced-2.png](images%2F8-bucket-policy-advanced-2.png)

- 아래의 예시는 `NotIpAddress` 조건 키를 사용하여 특정 IP 주소에 대한 액세스를 제한한다.

![9-bucket-policy-advanced-3.png](images%2F9-bucket-policy-advanced-3.png)

- 아래는 S3 버킷의 모든 객체를 나열하고 다운로드할 수 있는 사용자 액세스 권한을 부여한다.

![10-bucket-policy-advanced-4.png](images%2F10-bucket-policy-advanced-4.png)

- 아래는 `MultiFactorAuthPresent` 조건 키를 사용하여 MFA를 사용하여 인증된 사용자로 액세스를 제한한다.

![11-bucket-policy-advanced-5.png](images%2F11-bucket-policy-advanced-5.png)

---

### Static Website Hosting

- S3는 정적 웹사이트를 호스팅하고 인터넷에 액세스할 수 있도록 한다.
- 웹사이트 URL은 아래와 같으며 Region에 따라 달라진다.
  - `http://bucket-name.s3-website-aws-region.amazonaws.com`
  - `http://bucket-name.s3-website.aws-region.amazonaws.com`
- 403 Forbidden 오류가 발생하는 경우 버킷 정책이 공개 읽기를 허용하는지 확인해야 한다.

![12-static-website-hosting.png](images%2F12-static-website-hosting.png)

---

### Versioning

- Amazon S3에서 파일의 버전을 지정할 수 있다.
- 버킷 수준에서 활성화된다.
- 동일한 키를 덮어쓰면 "버전"이 1, 2, 3...과 같이 변경된다.
- 버킷의 버전을 지정하는 것이 모범 사례다.
  - 의도하지 않은 삭제로부터 보호(버전 복원 기능)할 수 있다.
  - 이전 버전으로 쉽게 롤백할 수 있다.
- 참고
  - 버전을 활성화하기 전에 버전이 관리되지 않은 모든 파일은 버전이 `null`이다.
  - 버전 관리를 일시 중단해도 이전 버전은 삭제되지 않는다.

![13-versioning.png](images%2F13-versioning.png)

---

### Replication (CRR & SRR)

- 소스 및 대상 버킷에서 버전 관리를 활성화해야 한다.
- CRR은 Cross-Region Replication의 약자로 교차 지역 복제를 의미한다.
- SRR은 Same-Region Replication의 약자로 동일 지역 복제를 의미한다.
- 버킷은 다른 AWS 계정에 있을 수 있다.
- 복사는 비동기 방식으로 이루어진다.
- S3에 적절한 IAM 권한을 부여해야 한다.
- 사용 사례
  - CRR: 규정 준수, 짧은 지연 시간 액세스, 교차 계정 복제
  - SRR: 로그 집계, 운영 계정과 테스트 계정 간의 실시간 복제

![14-replication-crr-srr.png](images%2F14-replication-crr-srr.png)

- 복제를 활성화한 후에는 새롭게 생성되는 객체만 복제된다.
- 선택적으로 S3 배치 복제를 사용하여 기존 객체를 복제할 수 있따.
  - 기존 객체 및 복제에 실패한 객체를 복제한다.
- DELETE 작업의 경우
  - 소스에서 타겟으로 삭제 마커를 복제할 수 있다.(선택적으로 설정 가능)
  - 버전 ID가 있는 삭제는 복제되지 않는다.(악의적인 삭제를 방지하기 위해)
- **복제 체인 연결이 없다.**
  - 버킷 1이 버킷2로 복제되고 버킷 3으로 복제되는 경우
  - 버킷 1에서 생성된 객체는 버킷 3에 복제되지 않는다.

---

### S3 Storage Classes

- S3에는 여러 종류의 스토리지가 있다.
  - Amazon S3 Standard - General Purpose
  - Amazon S3 Standard-Infrequent Access (IA)
  - Amazon S3 One Zone-Infrequent Access
  - Amazon S3 Glacier Instant Retrieval
  - Amazon S3 Glacier Flexible Retrieval
  - Amazon S3 Glacier Deep Archive
  - Amazon S3 Intelligent Tiering
- 수동으로 또는 S3 수명 주기 구성을 사용하여 클래스 간에 이동할 수 있다.

#### Durability and Availability

- Durability:
  - 여러 AZ에 걸쳐 객체의 높은 내구성(99.999999999%)을 제공한다.
  - S3에 10,000,000개의 객체를 저장하는 경우 평균적으로 10,000년에 한 번씩 단일 객체의 손실이 발생할 것으로 예상할 수 있다.
  - 모든 스토리지 클래스에 동일하다.
- Availability:
  - 서비스를 얼마나 쉽게 사용할 수 있는지 측정한다.
  - 스토리지 등급에 따라 다르다.
  - 예를 들어, S3 Standard는 99.99%의 가용성을 제공하며 1년에 53분동안 사용할 수 없음을 의미한다.

#### Standard - General Purpose

- 99.99% 가용성을 제공한다.
- 자주 액세스하는 데이터에 사용된다.
- 짧은 지연 시간과 높은 처리량을 제공한다.
- 동시 설비 장애 2개를 유지한다.
- 빅 데이터 분석, 모바일 및 게임 애플리케이션, 콘텐츠 배포 등에 사용된다.

#### Infrequent Access

- 자주 액세스하지 않지만 필요할 때 빠른 액세스가 필요한 데이터의 경우 사용한다.
- S3 Standard보다 비용이 저렴하다.
- Amazon S3 Standard-Infrequent Access (S3 Standard-IA)
  - 99.9%의 가용성을 제공한다.
  - 재해 복구 및 백업과 같은 작업에 사용된다.
- Amazon S3 One Zone-Infrequent Access (S3 One Zone-IA)
  - 단일 AZ에서 높은 내구성(99.999999999%)을 제공하지만 AZ가 파괴되면 데이터가 손실된다.
  - 99.5%의 가용성을 제공한다.
  - 온프레미스 데이터 또는 다시 생성할 수 있는 데이터의 보조 백엡 복사본을 저장하는 작업에 사용된다.

#### Glacier Storage Classes

- 아카이빙/백업을 위한 저렴한 객체 스토리지다.
- 가격: 스토리지 가격 + 객체 검색 비용
- Amazon S3 Glacier Instant Retrieval
  - 밀리초 단위 검색 기능으로 분기에 한 번 액세스하는 데이터에 적합하다.
  - 최소 저장 기간은 90일이다.
- Amazon S3 Glacier Flexible Retrieval (Amazon S3 Glacier의 새로운 이름)
  - 신속(1~5분), 표준(3~5시간), 대량(5~12시간)
  - 최소 저장 기간은 90일이다.
- Amazon S3 Glacier Deep Archive - 장기 보관용
  - Standard(12시간), 대량(48시간)
  - 최소 보관 기간은 180일이다.

#### Intelligent-Tiering

- 소액의 월간 모니터링 및 자동 계층화 비용을 지불하면 된다.
- 사용량에 따라 액세스 계층 간에 객체를 자동으로 이동한다.
- S3 Intelligent-Tiering의 경우 검색 비용이 발생하지 않는다.
- Frequent Access Tier (자동): 기본 계층
- Infrequent Access Tier (자동): 30일 동안 액세스되지 않은 객체
- Archive Instant Access Tier (자동): 90일 동안 액세스되지 않은 객체
- Archive Access Tier (선택): 90일에서 700일 이상 구성 가능
- Deep Archive Access Tier (선택): 180일에서 700일 이상 구성 가능

#### 비교

- 아래는 스토리지 클래스 간 차이를 비교할 수 있는 표다.

![15-storage-classes-comparison.png](images%2F15-storage-classes-comparison.png)

- 아래는 스토리지 클래스 간 가격 차이를 비교할 수 있는 표다. (us-east-1 기준)

![16-storage-classes-price-comparison.png](images%2F16-storage-classes-price-comparison.png)

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [Bucket Policy](https://docs.aws.amazon.com/AmazonS3/latest/dev/example-bucket-policies.html)
- [Storage Classes](https://aws.amazon.com/s3/storage-classes/)
- [Storage Pricing](https://aws.amazon.com/s3/pricing/)