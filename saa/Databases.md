# Databases

이번 장에서는 SAA를 준비하며 **AWS의 여러 Database**에 대해서 알아보도록 한다.

---

### Overview

- AWS에는 선택할 수 있는 많은 관리형 데이터베이스가 있다.
- 아키텍처에 따라 적합한 데이터베이스를 선택하기 위한 여러 질문들이 있다.
  - 읽기 작업량이 많거나, 쓰기 작업량이 많거나, 균형잡힌 작업량이 많거나, 처리량이 필요한가요?
    낮 동안 확장 또는 변동이 필요한가요?
  - 얼마나 많은 데이터를 저장하고 얼마나 오래 저장할 것인가? 데이터는 증가할 것인가? 객체의 평균 크기는? 어떻게 액세스할 것인가?
  - 데이터 내구성? 데이터에 대한 인증 출처?
  - 대기 시간 요구사항? 동시 사용자?
  - 데이터 모델? 데이터를 어떻게 조회할 것인가? 결합(Join)?, 구조화(Structured)?, 반구조화(Semi-Structured)
  - 강력한 스키마? 유연성 향상? 보고? 검색? RDBMS vs NoSQL
  - 라이선스 비용? Aurora와 같은 클라우드 네이티브 DB로 전환?

#### 유형

- **RDBMS(= SQL/OLTP)**: RDS, Aurora - Join을 사용할 때 유용하다.
- **NoSQL - No Joins, no SQL**: DynamoDB (~JSON), ElastiCache (Key/Value), Neptune (Graphs), DocumentDB(for MongoDB), Keyspaces(for Apache Cassandra)
- **Object Store**: S3(큰 객체를 위해 사용) / Glacier (백업이나 아카이빙을 위해 사용)
- **Data Warehouse (SQL Analytics/BI)**: Redshift(OLAP), Athena, EMR
- **Search**: OpenSearch(JSON) - 비정형 텍스트, 비정형 검색
- **Graphs**: Amazon Neptune - 데이터 간의 관계 표시
- **Ledger**: Amazon Quantum Ledger Database
- **Time series**: Amazon Timestream

#### Amazon RDS - 요약

- 관리형 PostgreSQL / MySQL / Oracle / SQL Server / MariaDB
- 프로비저닝된 RDS의 인스턴스의 크기와 EBS 볼륨 유형과 크기
- 스토리지 자동 확장 기능을 지원한다.
- 읽기 복제본 및 Multi AZ를 지원한다.
- IAM, Security Groups, KMS, SSL을 통해 정송 중 보안을 지원한다.
- 시점 복원 기능을 통한 자동 백업을 지원하며 최대 35일간의 데이터를 저장할 수 있다.
- 장기적인 복구를 위한 수동 DB 스냅샷을 제공한다.
- 관리 및 스케줄링된 유지보수(다운타임 포함)
- IAM 인증 지원, Secrets Manager와 통합된다.
- 기본 인스턴스(Oracle & SQL Server)에 대한 액세스 및 사용자 지정을 위한 RDS Custom을 지원한다.
- 사용 사례: 관계형 데이터셋(RDBMS/OLTP) 저장, SQL 쿼리 수행, 트랜잭션 처리

#### Amazon Aurora - 요약

- PostgreSQL/MySQL 호환 API, 스토리지와 컴퓨팅이 분리된다.
- Storage: 데이터가 3개의 AZ에 걸쳐 6개의 복제본에 저장된다. - 고가용성(Highly Available), 자가 복구(Self-Healing), 자동 확장(Auto-Scaling)
- Compute: 여러 AZ에 걸쳐 DB 인스턴스의 클러스터, 읽기 복제본이 자동 확장된다.
- Cluster: Writer 및 Reader DB 인스턴스에 대한 사용자 지정 엔드포인트를 지정한다.
- RDS와 동일한 보안/모니터링/유지보수 기능을 제공한다.
- Aurora에 대한 백업 및 복원 옵션을 파악해야 한다.
- **Aurora Serverless**: 예측할 수 없는/간헐적 워크로드, 용량 계획 없음
- **Aurora Multi-Master**: 지속적인 쓰기 Failover(높은 쓰기 가용성)
- **Aurora Global**: 각 지역에서 최대 16개의 DB 읽기 인스턴스, 1초 미만의 스토리지 복제
- **Aurora Machine Learning**: SageMaker & Comprehend on Aurora를 이용한 ML을 수행한다.
- **Aurora Database Cloning**: 스냅샷 복원보다 빠른 기존 클러스터에서 새로운 클러스터로 전환한다.
- 사용 사례: RDS와 동일하지만 유지 보수성이 낮거나 유연성이 높거나 성능이 높거나 기능이 늘어난다.

#### Amazon ElastiCache - 요약

- 관리형 "Redis/Memcached 데이터베이스"로 RDS와 유사하지만 캐시다.
- 인메모리 데이터 저장소로, 밀리초 미만의 대기 시간을 제공한다.
- ElastiCache 인스턴스 유형(예: cache.m6g.large)을 선택한다.
- 클러스터링(Redis) 및 Multi AZ 지원, 복제본 읽기(공유)를 제공한다.
- IAM, Security Groups, KMS, Redis Auth를 통한 보안을 제공한다.
- 백업/스냅샷/특정 시점 복원 기능을 제공한다.
- 관리 및 스케줄링된 유지보수 기능을 제공한다.
- 일부 애플리케이션 코드 변경 사항을 활용해야 한다.
- 사용 사례: Key/Value 저장, 비번한 읽기, 적은 쓰기, DB 쿼리에 대한 결과 캐시, 웹 사이트에 대한 세션 데이터 저장 등에 사용된다.

#### Amazon DynamoDB - 요약

- AWS 독점 기술로 관리형 Serverless NoSQL 데이터베이스로 밀리초 대기 시간 내에 데이터를 제공한다.
- 용량 모드(Capacity Modes): 프로비저닝된 용량(옵션인 자동 확장 또는 주문형 용량)
- ElastiCache를 Key/Value 저장소(예: TTL 기능을 사용하여 세션 데이터 저장)로 대체할 수 있다.
- 고가용성, 기본적으로 MultiAZ, 읽기 및 쓰기 이중화, 트랜잭션 기능을 제공한다.
- 읽기 캐시, 마이크로초 읽기 대기 시간을 위한 DAX 클러스터를 제공한다.
- 보안, 인증 및 권한 부여는 IAM을 통해 이루어져야 한다.
- 이벤트 처리: DynamoDB Streams는 AWS Lambda 또는 Kinesis Data Stream과 통합된다.
- GlobalTable 기능: Active-Active 설정
- PITR(새로운 테이블로 복원) 또는 주문형(On-Demand) 백업을 통해 최대 35일까지 자동 백업이 가능하다.
- PITR 창 내에서 RCU를 사용하지 않고 S3으로 내보내기를 사용할 수 있다. WCU를 사용하지 않고 S3에서 데이터를 가져올 수 있다.
- 스키마를 빠르게 진화시키는 데 유용하게 사용될 수 있다.
- 사용 사례: Serverless 애플리케이션 개발(소규모 문서 100s KB), 분산형 Serverless 캐시 등에 사용된다.

#### Amazon S3 - 요약

- S3는 객체에 대한 Key/Value 저장소다.
- 거대한 객체를 저장하는데 유용하게 사용되며, 많고 작은 객체를 저장하는 데는 적합하지 않다.
- Serverless, 무한 확장, 최대 객체의 크기 5TB, 버전 관리 기능을 제공한다.
- Tiers: S3 Standard, S3 Incrective Access, S3 Intelligent, S3 Glacier + 라이프사이클 정책
- Features: 버전 설정, 암호화, 복제, MFA-삭제, 액세스 로그 등을 제공한다.
- Security: IAM, 버킷 정책, ACL, Access Points, Object Lambda, CORS, Object/Vault Lock을 지원한다.
- Encryption: SSE-S3, SSE-KMS, SSE-C, Client-Side, 전송 중 TLS, 기본 암호화를 제공한다.
- Batch Operation: "S3 Batch"를 사용하여 객체에 대한 배치 작업, "S3 Inventory"를 사용하여 파일 나열을 제공한다.
- Performance: Multi-part 업로드, "S3 Transfer Acceleration", S3 Select를 제공한다.
- Automation: S3 이벤트 알림(SNS, SQS, Lambda, EventBridge)을 제공한다.
- 사용 사례: 정적 파일, 대용량 파일의 키/값 저장, 웹 사이트 호스팅 등에 사용된다.

#### DocumentDB

- Aurora는 AWS에서 PostgreSQL, MySQL의 구현체를 만든 것이다.
- DocumentDB는 NoSQL 데이터베이스인 MongoDB에서도 동일하다.
- MongoDB는 JSON 데이터를 저장, 쿼리 및 인덱싱 하는 데 사용된다.
- Aurora와 유사한 배포 컨셉(Deployment Concept)를 가지고 있다.
- 완전 관리형으로 3개의 AZ에 걸친 복제를 통해 고가용성을 제공한다.
- DocumentDB 스토리지는 자동으로 10GB씩, 최대 64TB까지 증가할 수 있다.
- 초당 수백만 건의 요청을 통해 워크로드로 자동 확장한다.

#### Amazon Neptune

- 완전 관리형 그래프 데이터베이스다.
- 인기있는 그래프 데이터 세트로는 소셜 네트워크(Social Network)가 있다.
  - 사용자는 친구가 있다.
  - 게시물에 댓글이 있다.
  - 댓글에 사용자의 좋아요가 있다.
  - 사용자가 게시물을 공유하고 좋아요를 누른다.
- 최대 15개의 읽기 복제본으로 3개의 AZ로 확장되어 고가용성을 제공한다.
- 고도로 연결된 데이터 세트를 사용하여 복잡하고 어려운 쿼리를 사용하는 애플리케이션 구축 및 실행에 유용하게 사용된다.
- 최대 수십억 개의 관계를 저장하고 밀리초 지연 시간으로 그래프를 쿼리할 수 있다.
- 잘 알려진 지식 그래프(Wikipedia), 사기 탐지, 추천 엔진, 소셜 네트워크 등에 사용된다.

![neptune.png](images%2FDatabase%2Fneptune.png)

#### Amazon Keyspaces (for Apache Cassandra)

- Apache Cassandra는 오픈 소스 NoSQL 분산 데이터베이스다.
- Amazon Keyspaces는 관리형 Apache Cassandra 호환 데이터베이스 서비스다.
- Serverless, 확장성, 고가용성 등 AWS에서 완벽하게 관리된다.
- 애플리케이션의 트래픽에 따라 테이블을 자동으로 업/다운한다.
- 테이블은 여러 AZ에 걸쳐 3번 복제된다.
- CQL(Cassandra Query Language)를 사용한다.
- 규모에 상관없이 밀리초 단위의 단일 지연 시간, 초당 1000개의 요청을 처리할 수 있다.
- 용량: 자동 확장 기능이 있는 "On-Demand 모드" 또는 "프로비저닝 모드"를 제공한다.
- 최대 35일까지 암호화, 백업, PITR(Point-In-Time Recovery)를 제공한다.
- 사용 사례: IoT 장비 정보 저장, 시계열 데이터 등을 저장할 때 사용된다.

#### Amazon QLDB

- QLDB는 "Quantum Ledger Database"의 약자다.
- 원장(Ledger)은 금융거래를 기록한 장부다.
- 3개의 AZ에 걸쳐 완벽하게 관리되며, Serverless, 고가용성(HA), 복제를 제공한다.
- 시간 경과에 따라 응용프로그램 데이터에 대해 변경된 모든 내용을 검토하는 데 사용된다.
- 불변(Immutable) 시스템: 어떤 항목도 제거 또는 수정할 수 없으며, 암호화로 확인할 수 있다.

![amazon-qldb.png](images%2FDatabase%2Famazon-qldb.png)

- 공통 원장 블록체인 프레임워크보다 2~3배 향상된 성능을 제공하며 SQL을 사용하여 데이터를 조작할 수 있다.
- "Amazon QLDB"의 경우 "Amazon Managed Blockchain"과 다르게 금융 규제 규칙에 대한 분산 구성 요소가 없다.

#### Amazon Timestream

- 완벽하게 관리되고 빠르고 확장성이 뛰어난 Serverless 시계열 데이터베이스다.
- 용량 조정을 위해 자동으로 업/다운된다.
- 하루에 수조 개의 이벤트를 저장 및 분석할 수 있다.
- 관계형 데이터베이스 비용의 1000배 빠르며, 1/10의 비용으로 사용할 수 있다.
- 스케줄링된 쿼리, 다중 측정 레코드, SQL과 호환된다.
- 데이터 스토리지 계층화: 메모리에 저장된 최신 데이터와 비용 최적화된 스토리지에 저장된 과거 데이터
- 내장된 시계열 분석 기능: 데이터 내 패턴을 거의 실시간으로 파악할 수 있도록 지원한다.
- 전송 중 및 정지 중에 암호화를 제공한다.
- 사용 사례: IoT 애플리케이션, 운영 애플리케이션, 실시간 분석 등에 사용된다.

![amazon-timestream.png](images%2FDatabase%2Famazon-timestream.png)

#### Amazon Timestream Architecture

![amazon-timestream-architecture.png](images%2FDatabase%2Famazon-timestream-architecture.png)

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03