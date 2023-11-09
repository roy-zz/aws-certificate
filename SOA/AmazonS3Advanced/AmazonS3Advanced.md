# Amazon S3 Advanced

이번 장에서는 **SysOps Administrator**를 준비하며 **S3 심화 기능**에 대해서 알아보도록 한다.

---

### S3 Lifecycle Rules

- 스토리지 클래스 간에 객체를 전환할 수 있다.
- 자주 액세스하지 않는 객체의 경우 Standard IA로 이동한다.
- 빠른 액세스가 필요하지 않은 아카이브 객체의 경우 Glacier 또는 Glacier Deep Archive로 이동시킬 수 있다.
- 수명 주기 규칙을 사용하여 객체 이동을 자동화할 수 있다.

![1-moving-between-storage-classes.png](images%2F1-moving-between-storage-classes.png)

- **Transition Actions**: 객체를 다른 스토리지 클래스로 전환하도록 구성한다.
  - 생성 60일 후에 객체를 Standard IA 클래스로 이동한다.
  - 6개월 후 아카이빙을 위해 Glacier로 이동한다.
- **Expiration Actions**: 일정 시간이 지나면 객체가 만료(삭제)되도록 구성한다.
  - 액세스 로그 파일은 365일 후에 삭제되도록 설정할 수 있다.
  - **이전 버전의 파일을 삭제하는 데 사용할 수 있다.(버전 관리가 활성화된 경우)**
  - 불완전환 Multi-Part 업로드를 삭제하는 데 사용할 수 있다.
- 특정 접두사에 대한 규칙을 생성할 수 있다.(예. `s3://mybucket/mp3/*`)
- 특정 객체 태그에 대한 규칙을 생성할 수 있다.(예. Department: Finance )

#### 예시

- **시나리오 1**
- EC2의 애플리케이션은 프로필 사진이 S3에 업로드된 후 이미지 썸네일을 생성한다.
  이러한 썸네일은 쉽게 다시 생성할 수 있으며 60일 동안만 보관하면 된다.
  소스 이미지는 60일 동안 즉시 검색할 수 있어야 하며, 그 후 사용자는 최대 6시간까지 기다릴 수 있다.
- S3 소스 이미지는 60일 후에 Glacier로 전환하기 위한 수명주기 구성을 갖춘 Standard 버전일 수 있다.
- S3 썸네일은 60일 후에 만료(삭제)되도록 수명 주기 구성을 사용하여 One-Zone IA에 있을 수 있다.

- **시나리오 2**
- 회사의 규칙에 따르면 삭제된 S3 객체를 30일 동안 즉시 복구할 수 있어야 하지만 이러한 일은 거의 발생하지 않는다.
  이 시간 이후 최대 365일 동안 삭제된 객체는 48시간 이내에 복구할 수 있다.
- 객체 버전을 보유하려면 S3 버전 관리를 활성화하여 "삭제된 객체"가 실제로 "삭제 마커"로 숨겨지고 복구될 수 있도록 한다.
- 객체의 "이전 버전"을 Standard IA로 전환한다.
- 이후 "이전 버전"을 Glacier Deep Archive로 전환한다.

#### Storage Class Analysis

- 객체를 올바른 스토리지 클래스로 전환할 시기를 결정하는 데 도움이 된다.
- Standard 및 Standard IA에 대한 권장 사항이다.
  - One-Zone IA 또는 Glacier에서는 작동하지 않는다.
- 보고서는 매일 업데이트된다.
- 데이터 분석을 보기 시작하는 데 24 ~ 48시간이 소요된다.
- 수명주기 규칙을 통합(또는 개선)하기 위한 좋은 첫 번째 단계다.

![2-storage-class-analysis.png](images%2F2-storage-class-analysis.png)

---

### S3 Event Notification

- S3:ObjectCreated, S3:ObjectRemoved, S3:ObjectRestore, S3:Replication...
- S3에 업로드된 이미지의 썸네일을 생성하는 것 같은 작업에 사용된다.
- **원하는 만큼 S3 이벤트를 생성할 수 있다.**
- S3 이벤트 알림은 일반적으로 몇 초 안에 이벤트를 전달하지만 경우에 따라 1분 이상 걸릴 수도 있다.

![3-event-notification.png](images%2F3-event-notification.png)

#### IAM Permission

- 이벤트 알림을 위해서 IAM Permission이 필요하다.

![4-event-notification-iam-permissions.png](images%2F4-event-notification-iam-permissions.png)

- S3 서비스는 SNS Topic로 데이터를 보내고 있다. 
- 이러한 작업을 가능하게 하려면 SNS 리소스 액세스 정책을 첨부해야 한다.
- 유사하게 SQS를 이용하려면 SQS 리소스 액세스 정책을 생성하고 첨부해야 한다.
- Lambda Function 또한 Lambda 리소스 정책을 첨부해야 S3가 Function을 호출할 권리가 생긴다.
- **S3에서는 IAM 역할을 사용하지 않고, 대신 SNS Topic, SQS, Lambda에 리소스 액세스 정책을 정의해야 한다.**

#### Amazon EventBridge

![5-event-notification-amazon-eventbridge.png](images%2F5-event-notification-amazon-eventbridge.png)

- Amazon EventBridge를 사용하면 18개가 넘는 AWS 서비스를 목적지로 이벤트를 전송할 수 있다.
- 결과적으로 S3 Event Notification 기능을 향상시킬 수 있다.
- JSON 규칙을 사용한 고급 필터링(메타데이터, 객체 크기, 이름 등)을 제공한다.
- 다중 목적지(Step Function, Kinesis Streams / Firehose)를 제공한다.
- 보관, 재생 이벤트, 안정적인 전달을 제공한다.

---

### Baseline Performance

- Amazon S3는 높은 요청 속도, 지연 시간 100~200ms에 맞춰 자동으로 확장된다.
- 애플리케이션은 버킷의 접두사당 초당 최소 3,500개의 PUT/COPY/POST/DELETE 또는 5,500개의 GET/HEAD 요청을 달성할 수 있다.
- 버킷의 접두사 수에는 제한이 없다.
- 예(객체 경로 => 접두사)
  - `bucket/folder1/sub1/file` => `/folder1/sub1/`
  - `bucket/folder1/sub2/file` => `/folder1/sub2/`
  - `bucket/1/file` => `/1/`
  - `bucket/2/file` => `/2/`
- 4개의 접두사 모두에 균등하게 읽기를 분산하면 GET 및 HEAD에 대해 초당 22,000개의 요청을 달성할 수 있다.

#### Multi-Part upload

- 100MB를 초과하는 파일에 권장되며 5GB를 초과하는 파일에는 필수로 사용해야 한다.
- 업로드 병렬화에 도움이 될 수 있다.(전송 속도 향상)

![6-s3-performance-multipart-upload.png](images%2F6-s3-performance-multipart-upload.png)

#### S3 Transfer Acceleration

- 대상 지역의 S3 버킷으로 데이터를 전달하는 AWS 엣지 로케이션으로 파일을 전송하여 전송 속도를 높인다.
- Multi-Part 업로드와 호환하여 사용할 수 있다.

![7-s3-performance-transfer-acceleration.png](images%2F7-s3-performance-transfer-acceleration.png)

#### Byte-Range Fetches

- 특정 바이트 범위를 요청하여 GET 요청을 병렬화한다.
- 장애 발생 시 복원력을 향상시킨다.
- 다운로드 속도를 높이는 데 사용할 수 있다.

![8-byte-range-fetches-1.png](images%2F8-byte-range-fetches-1.png)

- 부분 데이터(예. 파일 헤드)만 검색하는 데 사용할 수 있다.

![9-byte-range-fetches-2.png](images%2F9-byte-range-fetches-2.png)

---

### Select & Glacier Select

- 서버 측 필터링을 수행하여 SQL을 사용하여 더 적은 데이터를 검색할 수 있다.
- 간단한 SQL을 사용하여 행 및 열 기준으로 필터링이 가능하다.
- 네트워크 전송량이 적고 클라이언트측 CPU 비용도 적다.

![10-select-glacier-select.png](images%2F10-select-glacier-select.png)

- S3가 파일을 필터링하여 필요한 데이터만 검색할 수 있다.
- S3 Select를 사용하면 400% 빨리지며 80% 저렴하게 사용할 수 있다.
- S3가 서버 측에서 CSV 파일을 찾아 필터링하고, 필터링되어 축소된 데이터 세트를 내보낸다.
- Glacier Select도 동일하게 작동한다.

---

### S3 Batch Operation

- 단일 요청으로 기존 S3 객체에 대한 대량 작업을 수행한다.
  - 객체 메타데이터 및 속성 수정
  - S3 버킷 간 객체 복사를 제공한다.
  - **암호화되지 않은 객체를 암호화한다.**
  - ACL, 태그를 수정한다.
  - S3 Glacier에서 객체를 복원한다.
  - Lambda 함수를 호출하여 각 객체에 대해 사용자 지정 작업을 수행한다.
- Lambda 함수를 호출하여 각 객체에 대해 사용자 지정 작업을 수행한다.
- S3 배치 작업은 재시도를 관리하고, 진행 상황을 추적하고, 완료 알림을 보내고, 보고서를 생성한다.
- **S3 Inventory를 사용하여 객체 목록을 가져오고 S3 Select를 사용하여 객체를 필터링할 수 있다.**

![11-s3-batch-operation.png](images%2F11-s3-batch-operation.png)

---

### S3 Inventory

- 모든 객체를 나열할 수 있고 S3 버킷에서 그에 대응하는 메타데이터도 나열할 수 있다.
- S3 List API를 이용해 모든 객체를 나열하고 모든 관련 메타데이터를 얻는 것보다 좋은 방법이다.
- 대표적인 사용 예시는 아래와 같다.
  - 객체의 복제 및 암호화 상태에 대한 감사 및 보고를 생성한다.
  - S3 버킷의 객체 수를 가져올 때 사용할 수 있다.
  - 이전 객체 버전의 총 스토리지를 식별할 수 있다.
- 일일 또는 주간 보고서를 생성할 수 있다.
- CSV, ORC, Apache Parquet 형태로 데이터를 추출할 수 있다.
- Amazon Athena, Redshift, Presto, Hive, Spark 등을 사용하여 모든 데이터를 쿼리할 수 있다.
- S3 Select를 사용하여 생성된 보고서를 필터링할 수 있다.
- 비즈니스, 규정 준수, 규제 요구 사항 등에 사용될 수 있다.

---

### S3 Glacier

- 아카이빙/백업을 위한 저렴한 객체 스토리지다.
- 데이터가 장기간(10년) 보존된다.
- 온-프레미스 Magnetic Tape 스토리지의 대안이다.
- 연평균 내구성은 99.999999999%이다.
- 월별 스토리지당 비용($0.004/GB - Standard | $0.00099 / GB Deep Archive)
- Glacier의 각 항목을 "아카이브"라고 한다. (최대 40TB)
- 아카이브는 "Vaults"에 저장된다.
- 기본적으로 AES-256(AWS에서 관리하는 키로 암호화)을 사용하여 저장 데이터를 암호화한다.

#### Operations

- **Vault Operations**
  - Create & Delete: 아카이브가 없을 때만 삭제한다.
  - Retrieving Metadata: 생성 날짜, 아카이브 수, 모든 아카이브의 총 크기 등을 검색한다.
  - Download Inventory: Vault에 있는 아카이브 목록(아카이브 ID, 생성 날짜, 크기)를 다운로드한다.
- **Glacier Operations**
  - Upload: 대규모 아카이브의 경우 단일 작업 또는 부분별(Multi-Part upload) 작업
  - Download: 먼저 아카이브에 대한 검색 작업을 시작한 다음 Glacier가 아카이브 다운로드를 준비한다.
    그러면 사용자는 스테이징 서버에서 데이터를 다운로드하는 데 제한된 시간을 갖게 된다.(선택적으로 검색할 바이트 범위 또는 부분을 지정한다.)
  - Delete: 아카이브 ID를 지정하여 Glacier Rest API 또는 AWS SDK를 사용한다.
- 복원 링크에는 만료일이 있다.
- 아래와 같은 검색 옵션이 있다.
  - Expedited(1 ~ 5 minutes): $0.03 per GB, $10 per 1000 requests
  - Standard(3 ~ 5 hours): $0.01 per GB, $0.03 per 1000 requests
  - Bukl(5 ~ 12 hours): $0.0025 per GB, $0.025 per 1000 requests

#### Vault Policy & Vault Lock

- 각 Vault에는 아래의 항목이 포함된다.
  - 1개의 Vault Access 정책
  - 1개의 Vault Lock 정책
- "Vault 정책"은 JSON으로 작성된다.
- "Vault Access 정책"은 S3 버킷 정책과 유사하다. (사용자 제한/계정 권한)
- "Vault Lock 정책"은 규제 및 규정 준수 요구 사항에 대해 잠그는 정책이다.
  - 정책은 불변(Immutable)이며 절대 변경할 수 없다.
  - 예시 1: 1년 미만의 아카이브 삭제 금지
  - 예시 2: WORM 정책 구현

![12-vault-policies-vault-lock.png](images%2F12-vault-policies-vault-lock.png)

#### 복원 작업에 대한 알림

- **Vault Notification Configuration**
  - 작업이 완료되면 메시지가 SNS로 전송되도록 Valut를 구성할 수 있다.
  - 선택적으로 작업을 시작할 때, SNS Topic을 지정한다.
- **S3 Event Notification**
  - S3는 S3 Glacier 스토리지 클래스에 아카이브된 객체의 복원을 지원한다.
  - `s3:ObjectRestore:Post` => 객체 복원이 시작되면 알림을 제공한다.
  - `s3:ObjectRestore:Completed` => 객체 복원이 완료되면 알림을 제공한다.

![13-Notification-restore-opration.png](images%2F13-Notification-restore-opration.png)

#### Multi Part Upload

- 큰 객체를 순서 상관없이 부분적으로 업로드한다.
- 100MB를 초과하는 파일에는 권장되며, 5GB를 초과하는 파일은 필수로 사용해야 한다.
- 업로드 병렬화를 통하여 전송 속도 향상에 도움이 될 수 있다.
- 최대 Part의 수는 10,000이다.
- 실패하는 경우 **실패한 부분만 다시 업로드**하므로 성능을 향상시킬 수 있다.
- 네트워크 중단과 같은 이유로 업로드에 문제가 발생하였을 때, 수명 주기 정책을 사용하여 N일 후 완료되지 않은 업로드의 오래된 Part를 자동으로 삭제할 수 있다.
- AWS CLI 또는 AWS SDK를 사용하여 업로드한다.

![14-multipart-upload-deep-dive.png](images%2F14-multipart-upload-deep-dive.png)

---

### Amazon Athena

- S3에 저장된 데이터를 분석하는 서버리스 쿼리 서비스다.
- Presto 기반의 표준 SQL을 사용하여 파일을 쿼리한다.
- CSV, JSON, ORC, Avro 및 Parquet를 지원한다.
- 검색된 데이터 1TB당 $5.00을 지불해야 한다.
- 보고서/대시보드를 위해 Amazon Quicksight와 함께 일반적으로 사용된다.
- Business Intelligent/분석/보고, 분석 및 VPC Flow Logs 쿼리, ELB Logs, CloudTrail trails 등에 사용된다.

![15-amazon-athena.png](images%2F15-amazon-athena.png)

#### 성능 향상

- 비용 절감을 위해 열 형식 데이터를 사용하여 스캔 횟수를 감소시킬 수 있다.
  - Apache Parquet 또는 ORC가 권장된다.
  - 큰 성능 개선을 기대할 수 있다.
  - Glue를 사용하여 데이터를 Parquet 또는 ORC로 변환할 수 있다.
- 소규모 검색을 위한 **데이터 압축**(bzip2, gzip, lz4, snappy, zlip, zstd 등)을 지원한다.
- 가상 열에 대한 간편한 쿼리를 위한 S3의 데이터 세트 파티션을 제공한다.

![16-amazon-athena-partition-column.png](images%2F16-amazon-athena-partition-column.png)

- 더 큰 파일을 사용하여 오버헤드를 최소화한다.

#### Federated Query

- 관계형, 비관계형, 객체 및 사용자 지정 데이터 소스(AWS 또는 On-Premise)에 저장된 데이터에 대해 SQL 쿼리를 실행할 수 있다.
- AWS Lambda에서 실행되는 데이터 소스 커넥터를 사용하여 통합 쿼리(예: CloudWatch Logs, DynamoDB, RDS 등)를 실행한다.
- 결과를 Amazon S3에 다시 저장한다.

![17-federated-query.png](images%2F17-federated-query.png)

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [S3 Glacier Select](https://aws.amazon.com/blogs/aws/s3-glacier-select/)