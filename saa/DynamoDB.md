# Amazon DynamoDB

이번 장에서는 SAA를 준비하며 **Amazon DynamoDB**에 대해서 알아보도록 한다.

---

### Overview

- 완벽하게 관리되고 Multi-AZ에 걸쳐 복제가 가능한 고가용성(HA)
- NoSQL 데이터베이스로 RDB가 아니지만 트랜잭션을 지원한다.
- 대규모 워크로드, 분산형 데이터베이스로 확장할 수 있다.
- 초당 수백만 개의 요청, 수조 개의 행, 100TB 스토리지를 지원한다.
- 빠르고 일관된 성능(single-digit MS)
- 보안, 승인 및 관리를 위해 IAM과 통합된다.
- 저비용 및 자동 확장된다.
- 유지보수 또는 패치 적용이 없으므로 항상 사용할 수 있다.
- Standard & Infrequent Access(IA) 테이블 클래스를 지원한다.

#### Basics

- DynamoDB는 테이블로 구성되어 있다.
- 각 테이블에 Primary Key(기본키)가 있으며 생성 시 결정해야 한다.
- 각 테이블에는 무한한 수의 항목(rows)이 있을 수 있다.
- 각 항목에는 속성이 있으며 시간 경과에 따라 추가할 수 있고 값은 `null`일 수 있다.
- 항목의 최대 크기는 400KB다.
- 지원되는 데이터 유형은 아래와 같다.
  - **Scalar Types**: String, Number, Binary, Boolean, Null
  - **Document Types**: List, Map
  - **Set Types**: String Set, Number Set, Binary Set
- 따라서, DynamoDB에서는 스키마를 빠르게 진화시킬 수 있다.

![table-example.png](images%2FDynamoDB%2Ftable-example.png)

#### Read/Write Capacity Modes

- 테이블의 용량(읽기/쓰기 처리량)을 관리하는 방법을 제어한다.
- **Provisioned Mode (default)**
  - 초당 읽기/쓰기 횟수를 지정한다.
  - 용량을 미리 계획해야 한다.
  - RCU(Provisioned Read Capacity Units) 및 WCU(Write Capacity Units)에 대해 지불해야 한다.
  - RCU 및 WCU에 대한 Auto-Scaling 모드를 추가할 수 있다.
- **On-Demand Mode**
  - 워크로으데 따라 읽기/쓰기가 자동으로 확장한다.
  - 용량을 계획할 필요가 없다.
  - 사용하는 제품에 대한 비용을 더 많이 지불한다.
  - 예측할 수 없는 워크로드, 급격한 워크로드 급증에 적합하다.

#### Accelerator (DAX)

- DynamoDB를 위한 완벽하게 관리되고 가용성이 뛰어난 원활한 메모리 내 캐시다.
- 캐싱을 통해 읽기 혼잡을 해결할 수 있다.
- 캐시된 데이터에 대한 MS 단위의 대기시간을 보장한다.
- 애플리케이션의 로직 수정없이 기존 DynamoDB API와 호환된다.
- 캐시에 대한 5분(기본값) TTL을 가진다.

![dax.png](images%2FDynamoDB%2Fdax.png)

#### Accelerator (DAX) vs ElastiCache

![dax-vs-elasticache.png](images%2FDynamoDB%2Fdax-vs-elasticache.png)

#### Stream Processing

- 테이블의 항목 수준 수정(생성/수정/삭제) 순서가 매겨진 스트림
- 사용 사례
  - 실시간으로 변화에 대응(예: 사용자 환영 이메일)
  - 실시간 사용량 분석
  - 파생 테이블에 삽입
  - Region 간 복제 구현
  - DynamoDB 테이블 변경 시 AWS Lambda 호출
- **DynamoDB Streams**:
  - 24시간 보존
  - 한정된 소비자(Consumer) 수를 가진다.
  - AWS Lambda 트리거 또는 DynamoDB Stream Kinesis 어댑터를 사용한 프로세스
- **Kinesis Data Streams (신규)**:
  - 1년 보존
  - 높은 소비자(Consumer) 수를 가진다.
  - AWS Lambda, Kinesis Data Analytics, Kinesis Data Firehose, AWS Glue Streaming ETL을 통한 프로세스...

![streams.png](images%2FDynamoDB%2Fstreams.png)

#### Global Tables

![global-tables.png](images%2FDynamoDB%2Fglobal-tables.png)

- 여러 Region에 짧은 대기 시간으로 DynamoDB 테이블에 액세스할 수 있다.
- Active-Active 복제가 이루어진다.
- 애플리케이션은 모든 Region에서 테이블을 읽고 쓸 수 있다.
- 필수적으로 DynamoDB Streams를 사용하도록 설정해야 한다.

#### Time To Live (TTL)

- 만료 타임스탬프 이후 항목을 자동으로 삭제한다.
- 활용 사례: 최신 항목만 유지하여 저장된 데이터를 줄이고, 규제 의무 준수, 웹 세션 처리 등이 있다.

![time-to-live.png](images%2FDynamoDB%2Ftime-to-live.png)

#### 재해 복구(Disaster Recovery)를 위한 백업

- **PITR(Point-In-Time Recovery)을 사용한 지속적인 백업**
  - 지난 35일 동안 옵션으로 활성화할 수 있다.
  - 백업 윈도우 내에서 언제든지 시점 단위로 복구가 가능하다.
  - 복구 프로세스를 통해 새 테이블을 생성할 수 있다.
- **On-Demand 백업**
  - 명시적으로 삭제될 때까지 장기 보존을 위한 전체 백업을 진행한다.
  - 성능이나 대기 시간에 영향을 주지 않는다.
  - AWS Backup에서 구성 및 관리가 가능하다.(지역 간 복사 가능)
  - 복구 프로세스가 새 테이블을 생성한다.

#### Amazon S3와 통합

- **S3로 데이터 내보내기(Export, PITR 활성화 필수)**
  - 지난 35일 동안 모든 시점에서 작업이 가능하다.
  - 테이블의 읽기 용량에 영항을 주지 않는다.
  - DynamoDB 위에서 데이터 분석 수행이 가능하다.
  - 감사(Auditing)용 스냅샷을 보관할 수 있다.
  - JSON, ION 형식으로 데이터를 내보낼 수 있다.

- **S3에서 데이터 가져오기(Import)**
  - CSV, DynamoDB JSON, ION 형식으로 데이터를 가져올 수 있다.
  - 쓰기 용량을 전혀 사용하지 않는다.
  - 새로운 테이블을 생성한다.
  - 오류가 발생하는 경우 CloudWatch Logs에 기록된다. 

![integration-s3.png](images%2FDynamoDB%2Fintegration-s3.png)

- 아래의 이미지와 같이 DynamoDB와 AWS Lambda를 통합하여 Serverless API를 생성할 수 있다.

![building-serverless-api.png](images%2FDynamoDB%2Fbuilding-serverless-api.png)

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03