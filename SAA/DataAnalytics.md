# Data & Analytics

이번 장에서는 SAA를 준비하며 **데이터와 분석**에 대해서 알아보도록 한다.

---

### Amazon Athena

- Amazon S3에 저장된 데이터를 분석하는 Serverless 쿼리 서비스다.
- 표준 SQL 언어를 사용하여 파일을 쿼리한다. (Presto 기반)
- CSV, JSON, ORC< Avro 및 Parquet를 지원한다.
- 가격: 스캔한 데이터 TB당 $5.00
- 보고/대시보드를 위해 "Amazon Quicksight"와 함께 일반적으로 사용된다.
- BI(Business Intelligence)/Analytics/Reporting, VPC FLow Logs, ELB Logs, Cloud Trail 분석 등을 위해 사용된다.
- Serverless SQL을 사용하여 S3에서 데이터를 분석해야 하는 경우 "Amazon Athena"를 사용하면 된다.

![amazon-athena.png](images%2FDataAnalytics%2Famazon-athena.png)

#### 성능 개선

- 비용 절감을 위해 **"컬럼 데이터(Columnar data)"**를 사용하여 스캔을 감소시킨다.
  - Apache parquet 또는 ORC가 권장된다.
  - 엄청난 성능 향상을 할 수 있다.
  - "Glue"를 사용하여 데이터를 Parquet 또는 ORC로 변환할 수 있다.
- 작은 검색(bzip2, gzip, lz4, snappy, zlip, zstd)을 위해 데이터를 압축할 수 있다.
- S3의 파티션 데이터 세트로 가상 컬럼에 대한 쿼리가 용이하다.
- 대용량 파일(128MB 이상)을 사용하여 오버헤드를 최소화할 수 있다.

#### 연합(Federated) 쿼리

- 관계형, 비관계형, 객체 및 사용자 지정 데이터 소스(AWS 또는 On-Premise)에 저장된 데이터 전반에 걸쳐 SQL 쿼리를 실행할 수 있다.
- "AWS Lambda"에서 실행되는 "Data Source Connector"를 사용하여 "Federated" 쿼리(예: CloudWatch Logs, DynamoDB, RDS 등)를 실행할 수 있다. 
- 결과를 Amazon S3에 다시 저장한다.

![athena-federated-query.png](images%2FDataAnalytics%2Fathena-federated-query.png)

---

### Redshift

- "Redshift"는 "PostgreSQL"을 기반으로 하지만 OLTP에는 사용되지 않는다.
- OLAP 데이터베이스로 온라인 분석 프로세싱(분석 및 데이터 웨어하우징)을 위해 사용된다.
- 다른 데이터 웨어하우스보다 10배 뛰어난 성능을 가지고 있으며, PB 단위로 데이터 확장이 가능하다.
- 데이터의 컬럼 스토리지(행 기반이 아닌) 및 병렬 쿼리 엔진을 가지고 있다.
- 프로비저닝된 인스턴스에 따라 비용을 지불하면 된다.
- 쿼리를 수행하기 위한 SQL 인터페이스가 있다.
- Amazon Quicksight 또는 Tableau와 같은 BI 도구와 통합된다.
- 인덱스 덕분에 Query/Join/Aggregation 속도가 "Amazon Athena"에 비해서 빠르다.

#### Cluster

![redshift-cluster.png](images%2FDataAnalytics%2Fredshift-cluster.png)

- Leader Node: 쿼리 계획, 결과 집계를 위해 사용된다.
- Compute Node: 쿼리를 수행하기 위해 Leader에게 결과를 보낸다.
- 노드의 크기를 미리 프로비저닝한다.
- 비용 절감을 위해 예약된 인스턴스를 사용할 수 있다.

#### Snapshots & DR

- 일부 클러스터에 대해 Redshift에 "Multi-AZ" 모드가 있다.
- 스냅샷은 S3에 내부적으로 저장된 클러스터의 시점 단위 백업이다.
- 스냅샷은 변경된 내용만 저장되어 증분된다.
- 스냅샷을 새 클러스터로 복원할 수 있다.
- 자동화: 8시간마다, 5GB마다, 또는 일정에 따라 보존 기간을 1일에서 35일 사이로 설정할 수 있다.
- 수동: 스냅샷은 수동으로 삭제할 때까지 유지된다.
- 클러스터의 스냅샷(자동 또는 수동)을 다른 "AWS Region"에 자동으로 복사하도록 Amazon Redshift를 구성할 수 있다.

![redshift-snapshots-dr.png](images%2FDataAnalytics%2Fredshift-snapshots-dr.png)

- Redshift에 데이터를 저장할 때, 단건 저장보다는 한 번에 많은(예: 배치) 데이터를 Insert하는 것이 유리하다.

![redshift-large-insert.png](images%2FDataAnalytics%2Fredshift-large-insert.png)

#### Spectrum

- 이미 S3에 있는 데이터를 로드하지 않고 쿼리할 수 있다.
- 쿼리를 시작하려면 Redshift 클러스터를 사용할 수 있어야 한다.
- 이후 다음 수천 개의 "Redshift Spectrum" 노드에 쿼리가 전송된다.

![redshift-spectrum.png](images%2FDataAnalytics%2Fredshift-spectrum.png)

---

### Amazon OpenSearch Service

- "Amazon OpenSearch"는 "Amazon ElasticSearch"의 후속 제품이다.
- DynamoDB에서 쿼리는 기본 키 또는 인덱스로만 존재한다.
- OpenSearch를 사용하면 부분적으로 일치하는 필드라도 검색할 수 있다.
- OpenSearch는 다른 데이터베이스를 보완하는 기능으로 사용하는 것이 일반적이다.
- 관리되는 인스턴스 클러스터 프로비저닝 또는 OpenSearch Serverless를 사용할 수 있다.
- SQL은 기본적으로 지원하지 않으며 플러그인을 통해서 추가할 수 있다.
- Kinesis Data Firehose, AWS IoT 및 CloudWatch 로그에서 데이터를 수집할 수 있다.
- Cognito & IAM, KMS 암호화, TLS를 통한 보안을 제공한다.
- OpenSearch Dashboard와 함께 시각화를 제공한다.
  
#### OpenSearch 패턴 - DynamoDB

![opensearch-patterns-dynamodb.png](images%2FDataAnalytics%2Fopensearch-patterns-dynamodb.png)

#### OpenSearch 패턴 - CloudWatch Logs

![opensearch-patterns-cloudwatch.png](images%2FDataAnalytics%2Fopensearch-patterns-cloudwatch.png)

#### OpenSearch 패턴 - Kinesis

![opensearch-patterns-kinesis.png](images%2FDataAnalytics%2Fopensearch-patterns-kinesis.png)

---

### Amazon EMR

- EMR은 "탄력적 지도 축소(Elastic Map Reduce)"의 약자이다.
- EMR을 통해 하둡 클러스터(Big Data)를 생성하여 방대한 데이터를 분석 및 처리할 수 있다.
- 클러스터는 수백 개의 EC2 인스턴스로 구성할 수 있다.
- EMR은 아파치 스파크, HBase, Presto, Flink와 함께 제공된다.
- EMR은 모든 프로비저닝 및 구성을 담당한다.
- 자동으로 확장되며 스팟 인스턴스와 통합될 수 있다.
- 데이터 처리, 머신러닝, 웹 인덱싱, 빅데이터 등에 사용된다.

#### Node Types & Purchasing

- Master Node: 클러스터 관리, 조정, 상태 관리 - 장기 실행
- Core Node: 작업 실행 및 데이터 저장 - 장기 실행
- Task Node(Optional): 작업 실행을 위한 목적으로만 사용되며 일반적으로 스팟 인스턴스를 사용한다.
- 구매 옵션:
  - On-demand: 신뢰성 있고 예측 가능하며 종료되지 않는다.
  - Reserved(최소 1년): 비용 절감(사용 가능한 경우 EMR이 자동으로 사용)
  - Spot Instances: 가격이 저렴하고 종료될 수 있으며 신뢰성이 떨어진다. 
- 장기 실행 중인 클러스터 또는 일시적(임시) 클러스터를 가질 수 있다.

---

### Amazon QuickSight

![amazon-quicksight.png](images%2FDataAnalytics%2Famazon-quicksight.png)

- Serverless 머신러닝 기반 비지니스 인텔리전스 서비스로 대화형 대시보드를 생성할 수 있다.
- 빠르고 자동으로 확장 가능하며 내장 가능하며 세션별 가격 정책을 가지고 있다.
- 비즈니스 분석, 건물 시각화, 임시 분석 수행, 데이터를 사용한 비즈니스 통찰력 확보 등에 사용된다.
- RDS, Aurora, Athena, Redshift, S3와 통합된다.

![quicksight-integrations.png](images%2FDataAnalytics%2Fquicksight-integrations.png)

- QuickSight로 데이터를 가져오는 경우 SPICE 엔진을 사용하여 메모리 내에서 계산한다.
- 엔터프라이즈 에디션의 경우 CLS(Column-Level Security) 설정이 가능하다.

#### Dashboard & Analysis

- **사용자**(Standard Version) 및 **그룹**(Enterprise Version) 정의
  - 사용자 및 그룹은 QuickSight 내에서만 존재하며 IAM과는 다른 개념이다.
- **대시보드**는 공유할 수 있는 분석을 위한 읽기 전용 스냅샷이다.
  - 분석을 위한 구성(필터링, 파라미터, 컨트롤, 정렬)을 저장할 수 있다.
- 분석 또는 대시보드를 사용자 또는 그룹과 공유할 수 있다.
- 대시보드를 공유하려면 먼저 게시해야 한다.
- 대시보드를 보는 사용자는 기본 데이터도 조회할 수 있다.

---

### AWS Glue

- ETL(Extract, Transform, Load)을 위한 관리형 서비스다.
- 분석을 위한 데이터 준비 및 변환에 유용하게 사용된다.
- 완전한 서버리스 서비스다.

![aws-glue.png](images%2FDataAnalytics%2Faws-glue.png)

- 아래의 이미지와 같이 CSV 형식의 데이터를 Parquet 형식의 데이터로 변환하여 "Amazon Athena"를 사용하여 분석할 수 있다.

![glue-convert-data-parquet-format.png](images%2FDataAnalytics%2Fglue-convert-data-parquet-format.png)

- 아래의 이미지와 같이 "AWS Glue Data Crawler"를 활용하여 메타데이터를 생성하고 Dataset의 카탈로그로 사용할 수 있다.

![glue-data-catalog.png](images%2FDataAnalytics%2Fglue-data-catalog.png)

#### High-level

- **Glue Job Bookmarks**: 오래된 데이터를 다시 처리하지 못하도록 한다.
- **Glue Elastic Views**:
  - SQL을 사용하여 여러 데이터 저장소에 걸쳐 데이터 결합 및 복제를 한다.
  - Custom 코드가 없고, Glue가 소스 데이터의 변경 사항을 모니터링 하며, Serverless로 작동한다.
  - "가상 테이블"을 활용한다. (구체화된 보기)
- **Glue DataBrew**: 사전 구축된 변환을 사용하여 데이터를 정리하고 정규화할 수 있다.
- **Glue Studio**: Glue에서 ETL 작업을 생성, 실행 및 모니터링할 수 있는 GUI 서비스다.
- **Glue Streaming ETL**(Apache Spark Structured Streaming 기반): "Kinesis Data Stream", "Kafka", "MSK(관리형 Kafka)"와 호환된다.

---

### AWS Lake Formation

- **Data Lake**: 분석을 목적으로 모든 데이터를 저장할 수 있는 중앙 장소
- 며칠 만에 데이터 레이크를 쉽게 설정할 수 있도록 완벽하게 관리되는 서비스다.
- 데이터를 검색, 정제, 변환 및 데이터 레이크로 수집할 수 있다.
- 복잡한 수동 단계(수집, 정리, 이동, 데이터 카탈로그 작성 등)와 중복제거(ML Transforms 사용)를 자동화한다.
- 데이터 레이크에 정형 데이터와 비정형 데이터를 결합한다.
- S3, RDS, RDB, NoSQL 에서 데이터를 가져올 수 있다.
- 응용프로그램에 대한 세분화된 액세스 제어(행 및 열 수준)를 제공한다.
- "AWS Glue"위에 구축해야 한다.

![aws-lake-formation.png](images%2FDataAnalytics%2Faws-lake-formation.png)

- 아래의 이미지와 같이 접근 제어와, 컬럼 수준의 보안을 적용할 수 있다.

![lake-formation-centralized-permissions.png](images%2FDataAnalytics%2Flake-formation-centralized-permissions.png)

---

### Kinesis Data Analytics

- SQL을 사용하는 "Kinesis Data Streams" & "Firehose"는 실시간 데이터 분석을 위해 사용될 수 있다.
- Amazon S3의 참조 데이터를 추가하여 스트리밍 데이터를 풍부하게 생성할 수 있다.
- 완전 관리형 서비스이기 때문에 프로비저닝할 서버가 없다.
- 자동으로 스케일링이 된다.
- 실제 소비율에 대해서만 지불한다.
- **Kinesis Data Streams**: 실시간 분석 쿼리에서 스트림을 생성한다.
- **Kinesis Data Firehose**: 분석 쿼리 결과를 대상으로 전송한다.
- 시계열 분석, 실시간 대시보드, 실시간 메트릭 등을 분석하는 데 사용된다.

![kinesis-data-analytics-for-sql.png](images%2FDataAnalytics%2Fkinesis-data-analytics-for-sql.png)

#### Kinesis Data Analytics for Apache Flink

- Flink(Java, Scala, SQL)를 사용하여 스트리밍 데이터를 처리 및 분석할 수 있다.

![kinesis-data-analytics-for-flink.png](images%2FDataAnalytics%2Fkinesis-data-analytics-for-flink.png)

- AWS의 관리 클러스터에서 "Apache Flink" 애플리케이션을 실행할 수 있다.
  - 컴퓨팅 리소스 프로비저닝, 병렬 계산, 자동 확장을 지원한다.
  - 애플리케이션 백업(체크포인트 및 스냅샷)을 지원한다.
  - "Apache Flink" 프로그래밍 기능을 사용할 수 있다.
  - "Flink"는 "Firehose"로 부터 데이터를 읽지 않으며, "Kinesis Analytics for SQL"을 사용해야 한다.

---

### Amazon Managed Streaming for Apache Kafka (Amazon MSK)

- "Amazon Kinesis"의 대안이다.
- AWS에서 작동하는 완전 관리형 "Apache Kafka" 서비스다.
  - 클러스터를 생성, 업데이트, 삭제할 수 있다.
  - MSK는 사용자를 위해 Kafka 브로커 노드 및 Zookeeper 노드를 생성 및 관리한다.
  - VPC, Multi-AZ에 MSK 클러스터를 구축할 수 있고, HA를 위해 최대 3개까지 배포할 수 있다.
  - 일반적인 Apache Kafka 장애로부터 자동으로 복구한다.
  - 데이터는 원하는 기간 동안 EBS 볼륨에 저장된다.
- **MSK Serverless**
  - 용량을 관리하지 않고 MSK에서 "Apache Kafka"를 실행한다.
  - MSK는 리소스를 자동으로 프로비저닝하고 컴퓨팅 및 스토리지를 확장한다.

- 아래의 이미지는 "Apache Kafka"를 사용하는 대표적인 아키텍처다.

![kafka-high-level.png](images%2FDataAnalytics%2Fkafka-high-level.png)

#### Kinesis Data Streams vs Amazon MSK

- **Kinesis Data Streams**
  - 1MB 메시지 크기 제한이 있다.
  - 샤드(Shard)와 함께 데이터가 스트리밍 된다.
  - 샤드가 분할 및 병합된다.
  - TLS를 통해서 전송중 암호화를 지원한다.
  - KMS를 통해 저장된 데이터의 암호화를 지원한다.

- **Amazon MSK**
  - 기본값은 1MB이며, 더 높은 값으로 메시지 크기를 설정할 수 있다.
  - 파티션(Partitions)과 함께 Kafka Topic을 사용한다.
  - Topic에 Partitions만 추가할 수 있다.
  - PLAINTEXT 또는 TLS를 통해 전송 중 암호화를 지원한다.
  - KMS를 통해 저장된 데이터의 암호화를 지원한다.

- 아래의 이미지와 같이 "Amazon MSK"는 여러 종류의 소비자(Consumers) 유형을 지원한다.

![amazon-msk-consumers.png](images%2FDataAnalytics%2Famazon-msk-consumers.png)

---

### 빅데이터 수집 파이프라인

- 데이터 수집 파이프라인을 서버가 완전히 필요없는 상태로 만드는 상황을 가정한다.
- 우리는 실시간으로 데이터를 수집해야 한다.
- 데이터가 전송될 때 우리는 데이터를 변형할 수 있어야 한다.
- SQL을 사용하여 변환된 데이터를 쿼리할 수 있어야 한다.
- 쿼리를 사용하여 생성된 보고서는 S3에 있어야 한다.
- 해당 데이터를 Data Warehouse로 이동하고 대시보드를 생성해야 한다.
- 이러한 요구사항이 있을 때, 아래의 이미지와 같은 아키텍처를 설계할 수 있다.

![bigdata-ingestion-pipeline.png](images%2FDataAnalytics%2Fbigdata-ingestion-pipeline.png)

- "IoT Core"를 통해 IoT 기기에서 데이터를 수집할 수 있다.
- "Kinesis"는 실시간 데이터 수집에 적합한 서비스다.
- "Firehose"를 통해 거의 실시간으로 S3에 데이터를 제공할 수 있다.
- "Lambda"는 데이터 변환을 통해 "Firehose"에 도움을 줄 수 있다.
- "Amazon S3"는 "SQS"에 대한 알림을 트리거할 수 있다.
- "Lambda"는 "SQS"를 구독할 수 있다.(S3와 Lambda Connecter를 사용)
- "Athena"는 서버리스 SQL 서비스이며 분석 결과는 S3에 저장된다.
- Reporting 버킷에는 분석된 데이터가 포함되어 있으며 "AWS QuickSight", "Redshift"등의 리포팅 도구에서 사용할 수 있다.

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03