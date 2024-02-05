# Other Services

- 이번 장에서는 **DevOps Engineer Professional (DOP)** 을 준비하며 "기타 AWS 서비스들"에 대해서 알아보도록 한다.

---

### AWS Tag Editor

- 여러 리소스의 태그를 한 번에 관리할 수 있다.
- 태그 추가/업데이트/삭제가 가능하다.
- 모든 AWS 리전에서 태그를 지정하거나 태그가 지정되지 않은 리소스를 검색할 수 있다.

![1-aws-tag-editor.png](images%2F1-aws-tag-editor.png)

---

### Amazon QuickSight

- 대화형 대시보드를 만들기 위한 서버리스 머신러닝 기반의 BI(Business Intelligence) 서비스다.
- 세션별 가격 정책을 통해 빠르고 자동으로 확장 가능하며 내장 가능하다.
- 비즈니스 분석, 빌딩 시각화, 임시 분석 수행, 데이터를 사용하여 BI 확보 등에 사용된다.
- RDS, AUrora, Athena, Redshift, S3 등과 통합된다.
- QuickSight로 데이터를 가져오는 경우 SPICE 엔진을 사용한 인메모리 계산을 제공한다.
- 엔터프라이즈 에디션의 경우 CLS(Column-Level Security)를 설정할 수 있다.

![2-amazon-quicksight.png](images%2F2-amazon-quicksight.png)

#### Integration

![3-amazon-quicksight-integration.png](images%2F3-amazon-quicksight-integration.png)

- QuickSight는 수많은 데이터 소스와 통합될 수 있다.
- **AWS Service**
  - RDS, Aurora, Redshift, Athena, S3, OpenSearch, Timestream
- **SaaS**
  - salesforce, Jira
- **3rd Party**
  - teradata
- **기타**
  - 온프레미스 데이터베이스(JDBC), XLSX, CSV, JSON, .TSV, TLF, CLF

#### Dashboard & Analysis

- 사용자(표준 버전) 및 그룹(엔터프라이즈 버전) 정의
  - 이러한 사용자 및 그룹은 QuickSight 내에만 존재하며 IAM과는 다르다.
- Dashboard
  - 공유할 수 있는 분석물의 읽기 전용 스냅샷이다.
  - 분석물의 구성(filtering, 파라미터, 컨트롤, 정렬)을 보존한다.
- 분석 또는 대시보드를 사용자 또는 그룹과 공유할 수 있다.
- 대시보드를 공유하려면 먼저 게시해야 한다.
- 대시보드를 보는 사용자도 기본 데이터를 볼 수 있다.

---

### AWS Glue

- 추출(Extract), 변형(Transform), 적재(Load) 즉, ETL을 담당하는 서비스다.
- 분석을 위한 데이터 준비 및 변환에 유용하게 사용된다.
- 완전환 서버리스 서비스다.

![4-aws-glue.png](images%2F4-aws-glue.png)

- S3 버킷이나 RDS에서 Glue를 사용하여 데이터를 추출할 수 있다.
- Glue에서 데이터를 변환하여 목적지인 Redshift에 적재하도록 구축할 수 있다.

- 아래의 이미지와 같이 CSV 형식의 데이터를 Parquet 형식의 데이터로 변환하여 "Amazon Athena"를 사용하여 분석할 수 있다.
  - Parquet 형식은 열 기반 데이터 형식이기 떄문에 Athena와 같은 서비스와 사용하기 좋다.
  - 예를 들어, S3 버킷에 삽입 작업을 하고 있고, 파일의 형식들은 CSV 형식이다.
  - Glue ELT 서비스를 사용해 CSV를 임포트하고 Glue 서비스 내에서 이것을 Parquet 형식으로 변환할 수 있다.
  - S3 버킷에 파일을 삽입할 때마다 람다 함수에 이벤트 알림을 전송해 Glue ETL 작업을 트리거할 수 있다.
  - 람다 함수를 EventBridge로 대체할 수 있다.

![5-glue-convert-data-parquet-format.png](images%2F5-glue-convert-data-parquet-format.png)

- 아래의 이미지와 같이 "AWS Glue Data Crawler"를 활용하여 메타데이터를 생성하고 Dataset의 카탈로그로 사용할 수 있다.
  - S3, RDS, DynamoDB나 온프레미스에서 실행되는 JDBC 호환가능한 데이터베이스와 호환된다.
  - Glue Data Crawler가 데이터베이스를 크롤링하면서 테이블, 열, 데이터 유형 등에 관한 모든 메타데이터를 Glue Data Catalog에 저장한다.
  - 전체 데이터베이스와 테이블 메타데이터를 확보하고 나면 Glue Jobs가 이를 활용해 ETL을 수행한다.
  - 뒷단에서 데이터 디스커버리와 스키마 디스커버리를 위해 Amazon Athena를 사용할 경우 Athena도 Glue Data Catalog를 활용한다.
  - Redshift Spectrum, EMR도 동일하게 작동한다.

![6-glue-data-catalog.png](images%2F6-glue-data-catalog.png)

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

### 참고한 강의

- [DevOps Engineer Professional](https://www.udemy.com/course/aws-certified-devops-engineer-professional-korean)
- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)