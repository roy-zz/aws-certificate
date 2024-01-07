# Databases

- 이번 장에서는 **Solutions Architect Professional (SAP)** 을 준비하며 "데이터베이스"에 대해서 알아보도록 한다.

---

### DynamoDB

- NoSQL 데이터베이스로 완전 관리되며 대규모로 확장 가능하다. (1,000,000 rps)
- Apache Cassandra와 유사하며 DynamoDB로 마이그레이션될 수 있다.
- 프로비저닝할 디스크 공간이 없으며 객체의 최대 크기는 400KB다.
- 용량: 프로비저닝(WCU, RCU & Auto Scaling) 또는 온디멘드
- CRUD(Create/Read/Update/Delete)를 지원한다.
- 읽기 작업은 강력한 일관성을 제공한다.
- 여러 테이블에 걸쳐 트랜잭션을 지원한다. (ACID를 지원)
- 백업이 가능하면 시점 복구를 지원한다.
- 테이블 클리스는 Standard와 Infrequent Access(IA)가 있다.

#### 기본

- DynamoDB는 테이블로 구성되어 있다.
- 각 테이블에는 기본 키(Primary Key)가 있다.
- 각 테이블은 무한히 많은 항목(행)을 가질 수 있다.
- 각 항목에는 속성이 있으며 시간에 따라 추가할 수 있고 값이 `null`일 수 있다.
- 항목의 최대 크기는 400KB다.
- 지원되는 데이터 유형은 다음과 같다.
  - Scalar 유형: String, Number, Binary, Boolean, Null
  - Document 유형: List, Map
  - Set 유형: String Set, Number Set, Binary Set

#### Primary Key

- **옵션 1: 파티션 키 (해시)**
- 파티션 키는 각 항목마다 고유해야 한다.
- 파티션 키는 데이터가 분산되도록 "다양"해야 한다.
- 만약 `user_id`가 있다면 훌륭한 파티션 키로 사용할 수 있다.

![1-dynamodb-primary-key.png](images%2F1-dynamodb-primary-key.png)

- 파티션 키가 있고 이름, 나이와 같은 속성이 있다.
- Jone은 `user_id` = `12broiu45` 파티션 키를 가지고 있다.

- **옵션 2: 파티션 키 + 정렬 키**
- 두 개의 키의 조합이 고유해야 한다.
- 파티션 키별로 데이터를 그룹화한다.
- 정렬 키를 범위 키라고 부르기도 한다.

![2-dynamodb-primary-key-sort.png](images%2F2-dynamodb-primary-key-sort.png)

- 예를 들어, 사용자 게임 테이블이 있을 수 있다.
  - `user_id`는 파티션 키가 된다.
  - `game_id`는 정렬 키가 된다.
  - 두 키를 조합하면 고유한 키가 생성된다.
- 정렬 키의 좋은 예시는 타임스탬프다.

#### Index

- 객체 = 파티션 키 + 정렬 키(선택사항) + 특성
- LSI - Local Secondary Index
  - 기본 키를 동일하게 유지해야 한다.
  - 대체 정렬 키를 선택해야 한다.
  - 테이블이 생성될 때 정의되야 한다.
- GSI - Global Secondary Index
  - 기본 키를 변경할 수 있고 옵션으로 정렬 키를 가질 수 있다.
  - 테이블이 생성된 후에 정의될 수 있다.

- **주 테이블 및 인덱스에서 PK + 정렬 키로만 쿼리할 수 있다.** 
  - 이러한 속성은 RDS와 다르다.

#### 주요 기능

- **TTL**: 지정된 Epoch 날짜가 지나면 행이 자동으로 만료된다.
- **DynamoDB Streams**:
  - DynamoDB 테이블의 변화에 실시간으로 대응할 수 있다.
  - Lambda, EC 인스턴스에서 데이터를 읽을 수 있다.
  - 24시간동안 데이터가 유지된다.

![3-important-feature.png](images%2F3-important-feature.png)

- **Global Tables**:
  - 여러 리전에 Active-Active 복제를 할 수 있다.
  - DynamoDB Streams가 활성화되어 있어야 한다.
  - 낮은 대기 시간과 DR 목적에 유용하다.

![4-important-feature.png](images%2F4-important-feature.png)

- `us-east-1` 리전의 테이블에 변경 사항이 발생하면 `ap-southeast-2`의 테이블에도 동일하게 반영된다.
- `ap-southeast-2` 리전의 테이블에 변경 사항이 발생하면 `us-east-1`의 테이블에도 동일하게 반영된다.

#### Amazon Kinesis Data Streams for DynamoDB

- Kinesis Data Streams를 사용하여 DynamoDB의 항목 수준 변화를 캡처할 수 있다.
- 사용자 정의 및 긴 데이터 보존 기간동안 저장이 가능하다.
  - DynamoDB Streams에서 보존되는 24시간보다 더 큰 시간으로 설정할 수 있다. (최대 365일)

![5-kinesis-data-streams-for-dynamodb.png](images%2F5-kinesis-data-streams-for-dynamodb.png)

- DynamoDB 테이블은 Kinesis Data Streams를 변경할 항목으로 푸시한다.
- 푸시된 데이터는 Kinesis Data Firehose로 전송되어 일반적인 저장소인 S3, Redshift 등에 저장될 수 있다.
- 푸시된 데이터는 Kinesis Data Analytics로 전송되어 실시간 분석 및 실시간 계산될 수 있다.
  - 처리가 완료된 데이터는 Kinesis Data Streams, Kinesis Data Firehose, Lambda 등으로 전송될 수 있다.

#### S3 파일의 인덱스로 활용

![6-indexing-object-in-dynamodb.png](images%2F6-indexing-object-in-dynamodb.png)

- S3는 인덱스나 정렬기능이 없기 때문에 객체를 검색하기 좋은 방법은 아니다.
- 메타데이터 정보를 DynamoDB에 저장할 수 있다.
- 사용자는 객체를 추가하거나 업데이트하기 위해서 S3 버킷에 쓰기작업을 한다.
  - S3 이벤트에 의해 트리거되는 람다 함수가 있다.
  - 메타데이터를 찾기 위한 API를 생성할 수 있다.
  - LSI와 GSI를 생성하여 DynamoDB 테이블에 저장한다.
- 저장된 데이터를 여러가지 작업을 수행할 수 있다.
  - 날짜를 기준으로 조회
  - 특정 사용자가 사용하는 저장소의 총량 조회
  - 일정한 특정을 가진 모든 객체를 조회
- 이러한 작업들은 S3에만 파일을 저장하면 수행할 수 없는 작업들이다.

#### DAX

- DAX는 DynamoDB Accelerator의 약자다.
- DynamoDB를 위한 원활한 캐시를 제공하며 사용을 위해서 애플리케이션 코드를 수정할 필요가 없다.
- DAX를 통해 DynamoDB에 쓰기 작업을 할 수 있다.
- 캐시된 읽기 및 쿼리에 대한 마이크로초 지연 시간을 제공한다.
- DynamoDB 특정 행에 읽기 작업이 너무 많이 발생하면 제한으로 인해 Hot Key 문제가 발생할 수 있다.
  - DAX를 활용하면 DynamoDB 클러스터를 생성하여 해당 행을 캐싱하여 문제를 해결할 수 있다.
- 기본적으로 5분간 지속되는 TTL을 가지고 있다.
- Multi AZ를 지원하며 운영 환경에서는 최소 3개의 노드를 생성할 것을 권장한다.
- KMS를 통한 저장중 암호화, VPC, IAM, CloudTrail 등을 지원한다.

![7-dynamodb-dax.png](images%2F7-dynamodb-dax.png)

#### DAX vs ElastiCache

![8-dax-vs-elasticache.png](images%2F8-dax-vs-elasticache.png)

- DynamoDB에 직접 접근하고 싶은 클라이언트가 있다면 DAX를 사용하는 것이 좋다.
- DAX에 객체가 바로 캐싱되고 심지어 쿼리 결과도 DAX에 저장된다.
- 사용자가 집계 결과를 저장하고자 하는 장소로 다른 클라이언트들이 ElastiCache를 살펴보도록 할 수 있다.

---

### Amazon OpenSearch

- 기존에는 ElasticSearch였으며 "Amazon OpenSearch"로 이름이 변경되었다.
- Kibana는 "OpenSearch Dashboard"라는 이름으로 제공된다.
- OpenSearch의 완전 관리 버전이다. (오픈 소스 프로젝트이며, ElasticSearch의 포크)
- 서버에서 실행되야 하며 서버리스 오퍼링을 제공하지 않는다.
- 여러 사례에서 사용된다.
  - 로그 분석, 실시간 애플리케이션 모니터링, 보안 분석, 전체 텍스트 검색, 클릭스트림 분석, 인덱싱

#### OpenSearch + OS Dashboard + Logstash

- OpenSearch(예: ElasticSearch): 검색 및 인덱싱 기능을 제공한다.
  - 인스턴스 유형, 다중 AZ 등을 지정해야 한다.
- OpenSearch Dashboard(예: Kibana)
  - OpenSearch에 있는 데이터 위에 실시간 대시보드를 제공한다.
  - CloudWatch 대시보드의 대안이며 고급 기능이다.
- Logstash
  - 로그 수집 메커니즘으로 "Logstash Agent"를 사용한다.
  - CloudWatch Logs를 대체할 수 있다. (유지 및 세분화 여부 결정)

#### OpenSearch Patterns - DynamoDB

- OpenSearch는 DynamoDB와 통합되어 사용될 수 있다.

![9-opensearch-pattern-dynamodb.png](images%2F9-opensearch-pattern-dynamodb.png)

- DynamoDB의 업데이트 및 삭제 이벤트는 DynamoDB Stream으로 전성된다.
- 람다 함수를 통해서 모든 데이터를 OpenSearch로 전송할 수 있다.
- 데이터가 Amazon OpenSearch 저장되면 자신만의 API를 만들 수 있다.
- ALB 뒤에서 실행되는 애플리케이션에서 API를 사용하여 아이템을 검색할 수 있다.
- 애플리케이션은 API를 활용하여 DynamoDB 테이블에서 아이템을 검색한다.

#### OpenSearch Patterns - CloudWatch Logs

![10-opensearch-pattern-cloudwatch-logs.png](images%2F10-opensearch-pattern-cloudwatch-logs.png)

- CloudWatch Logs에서 구독 필터로 데이터를 보낼 수 있다.
  - AWS가 관리하는 람다 함수에서 OpenSearch로 데이터를 실시간으로 전송할 수 있다.
- CloudWatch Logs에서 구독 필터로 데이터를 보낼 수 있다.
  - Kinesis Data Firehose에서 거의 실시간으로 OpenSearch로 데이터를 전송할 수 있다.

---

### RDS

- PostgreSQL, MySQL, MariaDB, Oracle, Microsoft SQL Server 엔진을 제공한다.
- 프로비저닝, 백업, 패치 적용, 모니터링을 제공한다.
- VPC 내에서 시작되며, 일반적으로 프라이빗 서브넷에서 보안 그룹을 사용하여 네트워크 액세스를 제어한다.
  - 람다 함수를 사용할 때 이러한 제어는 중요한 항목이다.
- EBS(gp2 또는 io1)를 스토리지로 사용하며, 자동 확장으로 볼륨 크기 증가가 가능하다.
- 특정 시점 복구를 통해 자동화된 백업을 제공한다.
- 수동 및 여러 리전에 걸쳐 스냅샷 복사본을 생성할 수 있다.
- 이벤트(운영, 운영 중단 등)에 대해서 SNS을 통해서 전달할 수 있다.

#### Multi AZ & Read Replicas

- **Multi-AZ**: 중단 시 페일오버를 위해 대기 인스턴스를 사용한다.

![11-rds-multi-az.png](images%2F11-rds-multi-az.png)

- 하나의 가용성 영역에 마스터 인스턴스가 있고 다른 가용성 영역에 대기 인스턴스가 있다.
- 둘은 하나의 DNS 이름으로 연동되어 있으며 동기 방식으로 복제된다.
- 마스터 인스턴스가 중단되는 경우 대기 인스턴스가 마스터로 승격된다.
- 처음부터 두 개의 인스턴스는 동일한 DNS 이름을 사용하고 있었기 때문에 자동화된 복구가 가능하다.

- **Read Replicas**: 읽기 처리량을 늘리고 궁극적인 일관성이 있으며 여러 리전에 복제될 수 있다.

![12-rds-read-replicas.png](images%2F12-rds-read-replicas.png)

- RDS 인스턴스는 여러 읽기 전용 복제본을 가질 수 있으며 비동기 방식으로 복제된다.
- 기본 RDS 인스턴스에서는 읽기와 쓰기 작업이 가능하지만 읽기 전용 복제본에서는 읽기 작업만 가능하다.

#### Distributing Read across Replicas

![13-distributing-read-across-replica.png](images%2F13-distributing-read-across-replica.png)

- 애플리케이션의 로직을 제거하기 위해서 Route53을 사용한다.
- Route53에서 상태 검사와 함께 가중치 레코드 세트를 생성한다.
  - 각각의 복제본에 25%의 가중치를 할당한다.
  - 일반적으로 읽기 전용 복제본들에 동일한 가중치를 할당하는 것이 좋다.
- 애플리케이션은 할당된 가중치에 맞게 읽기 전용 복제본에 요청을 보내게된다.

#### 보안

- EBS 볼륨에 저장되는 데이터와 스냅샷에 KMS 암호화를 지원한다.
- Oracle 및 SQL Server 전용으로 TDE(Transparent Data Encryption)을 지원한다.
- 모든 DB 엔진에서 RDS로의 SSL 전송 중 암호화를 제공한다.
- MySQL, PostgreSQL, MariaDB의 경우 IAM 인증을 지원한다.
- RDS 내부에서 인가는 여전이 수행해야 하며 IAM으로는 수행할 수 없다. 
  - 예를 들어, 특정 테이블에 대한 접근을 제한하는 등의 작업은 IAM을 통해서 수행할 수 없다.
- 암호화되지 않은 RDS 스냅샷을 암호화된 스냅샷으로 복사할 수 있다.
- CloudTrail을 사용하여 RDS 내에서 이루어진 쿼리를 추적할 수 없다.

#### IAM Authentication

- IAM 데이터베이스 인증은 MariaDB, MySQL, PostgreSQL과 함께 작동한다.
- 비밀번호는 필요없으며 IAM & RDS API 호출을 통해 얻은 인증 토크만 있으면 된다.
- 인증 토큰의 수명은 15분이다.
- 아래와 같은 이점을 얻을 수 있다.
  - SSL을 사용하여 네트워크 IN/OUT을 암호화해야 한다.
  - DB 대신 IAM 사용자를 중앙에서 관리할 수 있다.
  - IAM 역할 및 EC2 인스턴스 프로파일을 활용하여 손쉽게 통합할 수 있다.

![14-iam-authentication.png](images%2F14-iam-authentication.png)

- RDS 서비스에서 API 호출을 통해 인증 토큰을 얻을 수 있다.
- SSL 암호화된 연결에 이 인증 토큰을 사용하면 데이터베이스에 연결할 수 있다.
- 보안 그룹도 EC2 인스턴스가 연결할 수 있도록 구성되어 있어야 한다.

#### RDS for Oracle

- **Backups**
  - RDS 백업을 사용하여 Oracle용 Amazon RDS로 백업 및 복원할 수 있다.
  - Oracle RMAN (Recovery Manager)를 통해 RDS가 아닌 백업 및 복구를 제공한다. (RDS는 지원되지 않음)

![15-rds-for-oracle.png](images%2F15-rds-for-oracle.png)

- RDS 인스턴스의 경우 "RDS Backup"을 사용하여 백업하거나 복구할 수 있다.
- RDS 인스턴스에서 Oracle RMAN 백업을 사용하여 백업본을 S3에 저장한다.
  - 이후 AWS 외부에서 실행되는 Oracle DB에서 복구를 진행할 수 있다.

- **Real Application Cluster(RAC)**
  - Oracle용 RDS는 RAC를 지원하지 않는다.
  - EC2 인스턴스에서 실행 중인 Oracle의 경우 모든 제어를 할 수 있기 때문에 RAC를 지원한다.
- Oracle용 RDS는 TDE(Transparent Data Encryption)을 지원하여 데이터를 스토리지에 쓰기 전에 암호화한다.
- DMS는 Oracle RDS에서 작동한다.

![16-rds-for-oracle.png](images%2F16-rds-for-oracle.png)

- On-Premise에 Oracle DB가 실행되고 있다.
- AWS Cloud에는 RDS 오라클 인스턴스가 실행되고 있다.
- AWS Database Migration Service를 사용하여 데이터를 마이그레이션할 수 있다.

#### RDS for MySQL

- 네이티브 mysqldump를 사용하여 MySQL RDS DB를 RDS가 아닌 DB로 마이그레이션할 수 있다.
- 외부 MySQL 데이터베이스는 데이터 센터 또는 EC2 인스턴스에서 실행될 수 있다.

![17-rds-for-mysql.png](images%2F17-rds-for-mysql.png)

- DB 관리자로 mysqldump 도구를 이용해 데이터를 내보내고 원하는 곳으로 데이터를 가져올 수 있다.
- 데이터를 가져온 다음 복제를 실행한다.
- 완료되면 복제를 중단하고 기존의 RDS 인스턴스를 종료한다.

#### RDS Proxy for Lambda

- RDS와 함께 람다 함수를 사용하면 데이터베이스 연결이 열리고 유지된다.
- 이로 인해 람다 함수가 많이 실행되는 경우에 `TooManyConnections` 예외가 발생할 수 있다.
- RDS 프록시를 사용하면 유휴 연결을 정리 및 연결 풀 관리를 처리하는 코드가 더 이상 필요하지 않다.
- IAM 인증 또는 DB 인증을 지원하며 자동 스케일리을 지원한다.
- 람다 함수는 프록시에 대한 연결이 있어야 한다.
  - VPC에서 공개 프록시 -> 공개 람다
  - 비공개 프록시 -> 람다

![18-rds-proxy-for-aws-lambda.png](images%2F18-rds-proxy-for-aws-lambda.png)

- 프라이빗 서브넷에 오로라 DB 클러스터가 실행하고 있다.
- 공용 서브넷에는 RDS 프록시가 생성되어 있다.
- 공개적으로 작동하고 있는 람다 함수는 IAM을 통해 RDS 프록시에 접근하고 오로라 DB 클러스터에 접근한다.

#### Cross Region Failover

![19-cross-region-failover.png](images%2F19-cross-region-failover.png)

- `us-east-1`에 메인 RDS 인스턴스가 있고 `us-west-2`에 읽기 전용 복제본이 있으며 비동기로 복제되고 있다.
- 상태 검사를 위한 하나의 옵션은 Ec2 인스턴스를 설치하여 데이터베이스 상태를 모니터링하고 `/health-db` 경로로 노출하는 방법이다.
- 다른 하나의 옵션은 CW 알람을 정의하고 CloudWatch 알람을 상태 검사에 링크한다.
    - 상태 검사가 해제되면 CW 알람에도 링크할 수 있다.
- SNS 토픽이나 CloudWatch Event와 연결될 수 있다.
- 만약 문제가 있는 경우 람다 함수를 호출하여 Route 53 레코드를 변경하여 읽기 전용 복제본을 메인 DB로 승격할 수 있다.

---

### Aurora

- PostgreSQL, MySQL 엔진을 지원한다.
- 최대 128TB와 6개의 데이터 복제본, Multi-AZ 스토리지를 지원한다.
- 최대 15개의 읽기 전용 복제본과 모든 데이터에 액세스할 수 있는 리더 엔드포인트를 제공한다.
- 전체 데이터베이스가 다중 리전에 복제된다.
- S3에서 데이터를 직접 불러오거나 내보낼 수 있다.
  - 클라이언트의 앱을 사용해 오로라로부터 데이터를 읽은 후 S3에 추가할 필요가 없다.
  - 오로라에서 바로 S3로 데이터가 전송되어 리소스와 네트워크 비용이 절감된다.
- RDS와 동일하게 백업, 스냅샷, 복원이 제공된다.

#### High Availability & Read Scaling

- 3개의 AZ에 걸쳐 6개의 복사본을 가진다.
  - 쓰기에 필요한 6개 중 4개의 복사본이 있다.
  - 읽기에 필요한 6개 중 3개의 복사본이 있다.
  - Peer-to-Peer 복제를 통한 자가 치유를 제공한다.
  - 스토리지가 100개의 볼륨에 걸쳐 스트라이프 처리된다.
- 마스터를 위한 자동 페일오버를 30초 이내에 수행한다.
- 마스터 노드와 함께 최대 15개의 읽기 전용 복제본을 생성할 수 있다.
- 리전간 복제를 지원한다.

![20-ha-read-scaling.png](images%2F20-ha-read-scaling.png)

- 128TB까지 자동 확장 가능한 공유 볼륨이 있다.
- 오로라 데이터베이스에 데이터를 삽입하면 세 개의 다른 AZ 블록에서 6개의 블록에 걸쳐 복제된다.

#### Aurora DB Cluster

![21-aurora-db-cluster.png](images%2F21-aurora-db-cluster.png)

- 자동 확장되는 공유 저장소가 있다.
- 클라이언트는 라이터 엔드포인트를 통해 마스터 노드에 연결될 수 있다.
- 마스터 인스턴스는 여러 읽기 전용 복제본을 생성할 수 있으며 읽기 전용 복제본은 자동으로 확장된다.
  - 여러 읽기 전용 복제본은 리더 엔드포인트에 연결되어 로드 밸런싱된다.

#### Endpoint

- 엔드포인트 = 호스트 주소 + 포트
- **Cluster Endpoint (Writer Endpoint)**
  - 오로라 클러스터의 현재 기본 DB 인스턴스에 연결한다.
  - DB 클러스터의 모든 쓰기 작업(Insert, Update, Delete)에 사용된다.
- **Reader Endpoint**
  - 오로라 클러스터의 모든 복제본에 대한 읽기 전용 연결을 위한 로드 밸런싱을 제공한다.
  - 읽기 작업에만 사용된다.
- **Custom Endpoint**
  - 오로라 클러스터에서 선택한 DB 인스턴스의 집합을 나타낸다.
  - 용량 및 구성이 서로 다른 DB 인스턴스의 서로 다른 하위 집합(예: 서로 다른 DB 파라미터 그룹)에 연결하려는 경우에 사용된다.
- **Instance Endpoint**
  - 오로라 클러스터의 특정 DB 인스턴스에 연결한다.
  - 특정 DB 인스턴스를 진단하고 미세 조정하려는 경우에 사용된다.

#### Custom Endpoint

- 오로라 인스턴스의 하위 집합을 사용자 지정 엔드포인트로 정의한다.
- 예를 들어, 특정 복제본에 대한 분석 쿼리를 실행하는 등의 작업에서 사용된다.
- 일반적으로 사용자 지정 엔드포인트를 정의한 후에 리더 엔드포인트가 사용되지 않는다.

![22-custom-endpoint.png](images%2F22-custom-endpoint.png)

- 클라이언트는 쓰기 작업을 위해서 라이터 엔드포인트에 연결한다.
- 일반적인 읽기 쿼리를 실행하기 위해 리더 엔드포인트를 사용한다.
- 분석을 위한 쿼리를 실행할 때는 사용자 지정 엔드포인트에 연결한다.

#### Logs

- 다음 유형의 오로라 MySQL 로그 파일을 모니터링할 수 있다.
  - 에러 로그
  - 느린 쿼리 로그
  - 일반 로그
  - 감사 로그
- 이러한 로그 파일은 CloudWatch Logs에 다운로드되거나 게시된다.

![23-aurora-logs.png](images%2F23-aurora-logs.png)

#### Troubleshooting & Performance

- **Performance Insights**: 대기, SQL 문, 호스트 및 사용자별 문제를 찾을 수 있다.
  - SQL 쿼리가 데이터베이스에 로드하는 것을 볼 수 있고, 실시간으로 분석할 수 있다.
  - 데이터베이스에 로드를 가장 많이 하는 사용자를 분석할 수 있다.

![24-troubleshooting-rds-performance.png](images%2F24-troubleshooting-rds-performance.png)

- **CloudWatch Metrics**: CPU, Memory, Swap Usage 등의 지표를 제공한다.
- **Enhanced Monitoring Metrics**: 호스트 수준, 프로세스 확인, 초당 메트릭을 확인할 수 있다.
  - 예를 들어, 데이터베이스에서 실행되는 상위 100개의 프로세스와 같은 심화된 내용을 확인할 수 있다.
- **Slow Query Logs**

#### Aurora Serverless

- 실제 사용량에 따라 자동화된 데이터베이스 인스턴스화 및 자동 확장한다.
- 빈번하지 않거나 간헐적이거나 예측 불가능한 워크로드에 적합하다.
- 용량을 계획할 필요가 없다.
- 초당 비용을 지불하고 비용 효율성을 향상시킬 수 있다.

![25-aurora-serverless.png](images%2F25-aurora-serverless.png)

- 클라이언트는 오로라에 의해 관리다는 프록시 플릿에 액세스한다.
- 공유 스토리지 볼륨을 사용하고 있는 인스턴스들은 필요에 따라 자동으로 확장된다.

#### Serverless - Data API

- 애플리케이션은 간단한 API 엔드포인트로 오로라 서버리스 DB 액세스를 제공한다. 
  - JDBC 연겨링 필요하지 않다.
- SQL 문 실행을 위한 보안 HTTPS 엔드포인트를 제공한다.
- 프로그램이 복잡하고 영구적인 데이터베이스 연결 관리를 할 필요가 없다.
- 사용자는 Data API 및 Secrets Manager(자격 증명이 확인되는 경우)에 대한 권한이 부여되어야 한다.

![26-aurora-serverless-data-api.png](images%2F26-aurora-serverless-data-api.png)

- 오로라 서버리스 데이터베이스는 Data API가 활성화되어 있다.
- 애플리케이션은 HTTP API 호출을 하고 Data API를 통해 SQL 문을 전달하지만 데이터베이스 자격 증명을 제공하지는 않는다.
- Secret Manager에 있는 데이터베이스의 기본 자격 증명에 액세스할 충분한 권한을 갖고 있는지 확인한다.
- 사용자의 애플리케이션은 IAM 정책에 따르며 오로라 서버리스 Data API에 액세스해야 한다.
  - 추가로 Secret Manager의 비밀에도 액세스할 수 있는 권한이 있어야 한다.

#### RDS Proxy for Aurora

![27-rds-proxy-for-aurora.png](images%2F27-rds-proxy-for-aurora.png)

- 연결을 관리하고 싶다면 오로라를 위한 RDS 프록시를 사용할 수 있다.
- 오로라 주요 인스턴스는 읽고 쓸 수 있으며 인스턴스 앞에 RDS 프록시를 생성하는 것이 가능하다.
- 사용자의 요청은 ALB를 통해 람다 함수를 호출하고 람다 함수는 데이터 조회를 위해서 RDS 프록시에 연결된다.

![28-rds-proxy-for-aurora.png](images%2F28-rds-proxy-for-aurora.png)

- **오로라 읽기 복제본에만 연결되는 읽기 전용 엔드포인트를 추가로 작성하는 기능이 있다.**
- 주요 인스턴스가 있고 읽기 전용 복제본 1, 2가 있다.
- RDS 프록시를 생성할 때 읽기 전용 엔드포인트를 생성하여 읽기 전용 복제본에만 연결되는 RDS 프록시를 생성할 수 있다.

#### Global Aurora

- **Aurora Cross Region Read Replicas**
  - 재해 복구에 유용하게 사용된다.
  - 간단하게 설치할 수 있다.
- **Aurora Global Database (권장)**
  - 1개의 기본 리전이 있다. (읽기/쓰기)
  - 최대 5개의 읽기 전용 보조 리전을 사용할 수 있으며, 복제 지연 시간이 1초 미만이다.
  - 보조 리전당 최대 16개의 복제본 읽기를 생성할 수 있다.
  - 대기 시간을 줄이는 데 도움이 된다.
  - 하나의 리전에 문제가 생겼을 때, 다른 리전을 승격시키는데 1분 이내의 RTO를 보장한다.
  - PostgreSQL용 오로라에서 RPO를 관리하는 기능을 사용할 수 있다.

![29-global-aurora.png](images%2F29-global-aurora.png)

- 주요 리전(`us-east-1`)에 있는 애플리케이션과 오로라 인스턴스가 있다.
- 읽기 전용의 보조 리전(`us-west-1`)에는 애플리케이션과 읽기 전용 오로라 인스턴스가 있다.

#### Write Forwarding

- 보조 DB 클러스터가 쓰기 작업을 수행하는 SQL 문을 기본 DB 클러스터로 전달할 수 있도록 한다.

![30-write-forwarding.png](images%2F30-write-forwarding.png)

- 데이터는 항상 기본 DB 클러스터에서 먼저 변경된 다음 보조 DB 클러스터로 복제된다.
- 기본 DB 클러스터에는 항상 모든 데이터의 최신 복제본이 있다.
- 관리할 엔드포인트의 수가 감소한다.
  - 만약 `us-central-1`에 접속하는 사용자가 쓰기 작업을 하려면 직접 `us-east-2`의 클러스터에 접속해야 한다.
  - Write Forwarding을 사용하면 사용자는 실제로 어떤 인스턴스에서 데이터가 저장되는지 알 필요도 없으며 작업에 사용되는 엔드포인트가 감소하게 된다.

#### Convert RDS to Aurora

![31-convert-rds-to-aurora.png](images%2F31-convert-rds-to-aurora.png)

- RDS 인스턴스에서 DB 스냅샷을 생성하고 오로라 DB 인스턴스에서 복제한다.
  - 기존 애플리케이션에서 기존 DB에 지속적으로 쓰기 작업을 진행하고 있다면 데이터 정합성에 문제가 발생할 수 있다.
  - 하지만 빠르게 마이그레이션 된다는 장점이 있다.
- RDS 인스턴스에서 오로라 읽기 전용 복제본을 생성한다.
  - 이후 오로라 읽기 전용 복제본을 승격시킨다.
  - 0에 가까운 복제시간을 제공한다.

---

### 참고한 강의

- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)