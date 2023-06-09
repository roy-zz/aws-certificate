## 데이터베이스(Database)

- 최초 작성 일자: 2023-03-21
- 수정 내역:
  - 2023-03-21: 최초 작성

---

### Amazon Aurora

- **MySQL 및 PostgreSQL과 완벽하게 호환되는 고성능 관리형 관계형 데이터베이스**
- 완벽한 MySQL 및 PostgreSQL 호환성과 함께 비할 데 없는 고성능과 고가용성을 글로벌 규모로 제공하도록 설계된 서비스.

#### 사용 이유

- 상용 데이터베이스의 1/10의 비용으로 완벽한 MySQL 및 PostgreSQL 호환성을 유지하면서 성능 집약적인 애플리케이션 및 중요한 워크로드를 지원할 수 있다.
- 99.99%의 가동 시간 SLA와 리전 간 재해 복구를 사용한 글로벌 복제로 지원되는 다중 AZ 가용성을 바탕으로 1분 안에 애플리케이션을 구축할 수 있다.
- 서버리스와 같은 혁신 기능이 포함된 완전관리형 데이터베이스를 통해 사용자 만족도를 높이는 애플리케이션 구축에 집중하여 생산성을 개선하고 총 소유 비용을 낮출 수 있다.
- MySQL 또는 PostgreSQL 데이터베이스를 표준 도구로 사용하여 Aurora로 간편하게 마이그레이션하거나 Babelfish for Aurora PostgreSQL을 통해 최소한의 코드 변경으로 레거시 SQL Server 애플리케이션을 실행할 수 있다.

#### 작동 방식

- Amazon Aurora는 기본 제공 보안, 연속적인 백업, 서버리스 컴퓨팅, 최대 15개의 읽기 전용 복제본, 자동 다중 리전 복제 및 다른 AWS 서비스와의 통합을 제공한다.
![](images/database_services/amazon_aurora.png)

#### 사용 사례

- **엔터프라이즈 애플리케이션 현대화**: 고객 관계 관리(CRM), 전사적 자원 관리(ERP), 공급망 및 결제 애플리케이션과 같은 엔터프라이즈 애플리케이션을 운영하는 데 필요한 고가용성과 성능을 제공한다.
- **SaaS 애플리케이션 구축**: 유연한 인스턴스 및 스토리지 크기 조정을 통해 안정적인 고성능 다중 테넌트 서비스형 소프트웨어(SaaS) 애플리케이션을 지원한다.
- **글로벌 분산 애플리케이션 배포**: 모바일 게임, 소셜 미디어 앱 및 온라인 서비스와 같이 다중 리전 확장 및 복원력을 필요로 하는 인터넷 규모 애플리케이션을 개발한다.
- **서버리스로 전환**: 용량 관리 없이 사용한 용량에 대한 요금만 지불하고 즉각적이고 세분화된 크기 조정을 통해 비용의 최대 90%를 절감할 수 있다.

---

### Amazon Aurora Serverless V2

- **초당 100,000건이 넘는 트랜잭션으로 즉시 확장**
- Amazon Aurora Serverless는 Amazon Aurora의 온디맨드 자동 크기 조정 구성 버전이다.
- 애플리케이션 요구 사항을 기반으로 자동으로 시작 및 종료되어 용량을 확장 또는 축소하며, 데이터베이스 용량을 관리할 필요 없이 AWS에서 데이터베이스를 실행할 수 있다.

#### 종류

- **Amazon Aurora Serverless v1**: 사용 빈도가 낮거나 간헐적이거나 예측할 수 없는 워크로드에 대한 간단하고 비용 효율적인 옵션이다.
- **Amazon Aurora Serverless v2**: Amazon Aurora Serverless v2는 수십만 건의 트랜잭션 처리도 가능하도록 1초 미만으로 즉시 확장한다. 확장을 통해 애플리케이션에 필요한 정확한 양의 데이터베이스 리소스를 제공하도록 세분화된 단위로 용량을 조정한다. 피크 로드에 대비한 프로비저닝 용량의 비용에 비해 데이터베이스 비용의 최대 90%까지 절약할 수 있다.

#### 사용 이유

- **뛰어난 확장성**: 수십만 건의 트랜잭선 처리도 가능하도록 1초 미만으로 즉시 확장한다.
- **고가용성**: 복제, 글로벌 데이터베이스, 다중 AZ, 읽기 전용 복제본 등을 포함한 광범위한 Aurora 기능으로 비즈니스 크리티컬 워크로드를 지원한다.
- **비용 효율성**: 정확한 양의 데이터베이스 리소스만을 제공하고 소비한 용량만큼만 비용을 지불하도록 세분화된 단위로 확장한다.
- **단순성**: 데이터베이스 용량 프로비저닝 및 관리의 복잡성을 없애준다. 데이터베이스가 애플리케이션의 요구에 맞춰 크기를 조정한다.
- **투명성**: 수신되는 애플리케이션 요청을 방해하지 않으면서 데이터베이스 용량을 즉시 크기 조정한다.
- **내구성**: 6방향 복제 기능과 내결함성을 갖춘 분산 및 자가 복구 Aurora 스토리지를 사용하여 데이터 손실을 방지한다.

#### 사용 사례

- **가변 워크로드**: 인사, 예산 정책, 운영 보고 애플리케이션 등과 같이, 최대 사용 시간이 30분에서 몇 시간으로 하루에 한두 번 또는 연간 몇 차례 사용하는 등 자주 사용되지 않는 애플리케이션을 실행할 때, 최대 사용량까지 프로비저닝하여 지속적으로 사용하지 않는 리소스에 대한 요금을 지불하거나, 평균 용량으로 프로비저닝하여 성능 문제 및 사용자 불만족의 위험을 감수하지 않아도 된다.
- **예측할 수 없는 워크로드**: 하루 종일 데이터베이스를 사용하는 워크로드를 실행하고 있으며, 최대 사용량을 예측하기 힘든 경우에도, 데이터베이스가 애플리케이션의 최대 로드 요구 사항을 충족할 수 있도록 용량이 자동으로 증가되고 사용량 증가가 둔화되면 다시 원래대로 조정된다.
- **기업 데이터베이스 플릿 관리**: Aurora Serverless v2를 사용하면 데이터베이스 용량이 애플리케이션의 요구 사항에 따라 자동으로 조정되기 때문에, 더 이상 데이터베이스 플릿에서 수천 개의 데이터베이스를 수동으로 관리할 필요가 없다.
- **서비스형 소프트웨어 애플리케이션**: Aurora Serverless v2를 사용하면 SaaS 공급 업체가 프로비저닝된 용량 비용을 걱정하지 않고 각 고객에 대해 Aurora 데이터베이스 클러스터를 프로비저닝할 수 있다.
- **여러 서버에 분할된 확장 데이터베이스**: Aurora Serverless v2를 사용할 경우 고객이 데이터베이스를 여러 Aurora 인스턴스로 분할하면 서비스가 필요에 따라 즉시 자동으로 용량을 조정한다.

---

### Amazon DocumentDB(MongoDB 호환)

- **완전관리형 문서 데이터베이스**
- 완전관리형 기본 JSON 도큐먼트 데이터베이스를 통해 손쉽게 엔터프라이즈 워크로드 크기 조정

#### 사용 이유

- 기존 애플리케이션에서 생성한 것과 동일하게 유연한 JSON 형식으로 데이터를 저장, 쿼리, 인덱싱, 집계하여 애플리케이션을 신속하게 개발하고 개선한다.
- 컴퓨팅 및 스토리지 크기를 독립적으로 조정하여 페타바이트 스토리지에서 문서 읽기 및 쓰기 요청을 초당 수백만 개로 확대한다.
- 라이선싱 수수료 없이 획일적인 수동 데이터베이스 관리 작업이 불필요한 완전관리형 데이터베이스로 생산성을 개선하고 총 소유 비용을 낮춘다.
- 기존 MongoDB API, 드라이버, 도구를 사용하면서도 고가용성, 내구성, 대기 시간이 짧은 글로벌 읽기, 내장된 보안 모범 사례와 같은 엔터프라이즈 기능을 사용한다.

#### 작동 방식

- Amazon DocumentDB는 완전관리형 기본 JSON 도큐먼트 데이터베이스로서 인프라를 관리하지 않고도 규모와 관계없이 중요한 문서 워크로드를 쉽고 비용 효율적으로 운영할 수 있게 해준다.
- Amazon DocumentDB는 내장 보안 모범 사례, 지속적인 백업, 다른 AWS 서비스와의 기본 통합을 제공하여 사용자의 아키텍처를 간소화한다.
- **Elastic Clusters**
![](images/database_services/amazon_documentdb_elastic_cluster.png)
- **자체 관리형 MongoDB 워크로드 마이그레이션**: 클러스터 관리 소프트웨어 실행, 백업 구성 또는 프로덕션 워크로드 모니터링에 대한 걱정 없이 MongoDB 호환 데이터베이스를 설정하고, 보안을 유지하고 크기를 조정한다.
![](images/database_services/amazon_documentdb_self_managed.png)

#### 사용 사례

- **콘텐츠 관리 데이터 저장 및 쿼리**: 콘텐츠 관리 시스템(CMS)에 저장된 검토, 이미지 및 기타 콘텐츠에 빠르고 안정적으로 액세스하여 고객 경험을 개선한다.
- **사용자 프로파일, 기본 설정 및 요청 관리**: 고객 추천을 생성하고 온라인 트랜잭션을 사용한다. 수백만 개의 사용자 프로파일 및 기본 설정을 관리한다.
- **모바일 및 웹 애플리케이션 크기 조정**: 짧은 대기 시간과 글로벌 읽기로 초당 수백만 개의 사용자 요청을 처리하도록 확장 가능한 애플리케이션을 구축한다.

---

### Amazon DynamoDB

- **관리형 NoSQL 데이터베이스**
- 모든 규모에서 10밀리초 미만의 성능을 제공하는 빠르고 유연한 NoSQL 데이터베이스 서비스

#### 사용 이유

- 일관된 10밀리초 미만의 성능, 거의 무제한의 처리량 및 스토리지와 자동 다중 리전 복제 기능을 사용하여 앱을 제공할 수 있다.
- 유휴 시 암호화, 자동 백업 및 복원과 최대 99.999% 가용성의 SLA로 보장되는 안정성을 바탕으로 데이터를 보호한다.
- 요구 사항에 따라 자동으로 확장되고 축소되는 완전관리형 서버리스 데이터베이스를 통해 혁신에 집중하고 비용을 최적화할 수 있다.
- AWS 서비스와 통합하면 데이터로 더 많은 것을 수행할 수 있다. 기본 제공되는 도구를 사용하여 분석을 수행하고 인사이트를 추출하며 트래픽 추세를 모니터링할 수 있다.

#### 작동 방식

- Amazon DynamoDB는 모든 규모에서 고성능 애플리케이션을 실행하도록 설계된 완전관리형의 서버리스 Key-Value NoSQL 데이터베이스다.
- DynamoDB는 기본 제공 보안, 지속적인 백업, 자동화된 다중 리전 복제, 인 메모리 캐시 및 데이터 가져오기/내보내기 도구를 제공한다.
![](images/database_services/amazon_dynamodb.png)

#### 사용 사례

- **소프트웨어 애플리케이션 개발**: 수백만 사용자와 초당 수백만 건의 요청에 대한 높은 동시성 및 연결이 요구되는 사용자 콘텐츠 메타데이터 및 캐시를 지원하는 인터넷 규모의 애플리케이션을 구축할 수 있다.
- **미디어 메타데이터 스토어 생성**: 실시간 동영상 스트리밍 및 대화형 컨텐츠와 같은 미디어 및 엔터테인먼트 워크로드의 처리량 및 동시성을 확장하고 AWS 리전에 걸친 다중 리전 복제를 짧은 대기 시간으로 제공할 수 있다.
- **원활한 소매 경험 제공**: 장바구니, 워크플로 엔진, 재고 추적 및 고객 프로필 배포를 위한 설계 패턴을 사용할 수 있다. DynamoDB는 높은 트래픽의 대규모 이벤트를 지원하며 초당 수백만 건의 쿼리를 처리할 수 있다.
- **게임 플랫폼 확장**: 운영 오버헤드 없이 혁신을 추진하는 데 집중할 수 있다. 수백만 동시 사용자의 플레이어 데이터, 세션 기록 및 순위표를 사용하여 게임 플랫폼을 구축할 수 있다.

---

### Amazon ElastiCache

- **인 메모리 캐싱 서비스**
- 실시간 애플리케이션의 성능을 실시간으로 개선

#### 사용 이유

- 초당 수억 개의 작업과 최대 1페비바이트(PiB)의 데이터에 대해 마이크로초의 응답 시간을 실현한다. (데이터 계층화 사용)
- 다중 AZ 배포로 99.99%의 SLA를 달성하고 교차 리전 복제를 통해 1분 이내에 재해 복구를 강화한다.
- 자주 읽는 데이터에 대한 캐시를 추가하여 리소스를 최적화하고 총 소유 비용을 낮춤으로써 비용을 최적화한다.
- 널리 사용되는 오픈 소스 기술인 Redis와 Memcached를 사용하여 애플리케이션을 빠르게 구축하고 다른 AWS 서비스와 쉽게 통합할 수 있다.

#### 작동 방식

- Amazon ElastiCache는 Redis 및 Memcached와 호환되는 완전관리형 서비스로서 현대적 애플리케이션의 성능을 최적의 비용으로 실시간으로 개선해준다.
- ElastiCache는 마이크로초의 응답 시간으로 초당 수억 개의 작업으로 확장되며 엔터프라이즈급 보안 및 신뢰성을 제공한다.
![](images/database_services/amazon_elasti_cache.png)

#### 사용 사례

- **총 소유 비용 절감**: 데이터를 캐시하고 데이터베이스 I/O를 오프로드하여 운영 부담을 줄이고 비용을 절감하며 데이터베이스와 애플리케이션의 성능을 개선한다.
- **실시간 애플리케이션 데이터 캐싱**: 마이크로초의 응답 시간과 높은 처리량을 위해 자주 사용하는 데이터를 메모리에 저장하여 초당 수억 개의 작업을 지원한다.
- **실시간 세션 스토어**: 임시 세션 데이터를 저장하여 마이크로초의 응답 시간으로 게임, 전자 상거래, 소셜 미디어 및 온라인 애플리케이션을 빠르게 개인화한다.
- **실시간 순위표**: ElastiCache의 기본 데이터 구조로 애플리케이션 개발을 간소화한다.

---

### Amazon Keyspaces(Apache Cassandra 호환)

- **관리형 Cassandra 호환 데이터베이스**
- 고가용성의 확장 가능한 관리형 Apache Cassandra 호환 데이터베이스 서비스
- Amazon Keyspaces를 사용하면 현재 사용 중인 것과 동일한 Cassandra 애플리케이션 코드 및 개발자 도구를 사용하여 AWS에서 Cassandra 워크로드를 실행할 수 있다.
- 서버를 프로비저닝, 패치 또는 관리할 필요가 없으며 소프트웨어를 설치, 유지 관리 또는 운영할 필요도 없다.

#### 사용 이유

- **Apache Cassandra와 호환**: Amazon Keyspaces에는 기존에 사용 중인 CQL(Cassandra Query Language) API 코드, Cassandra 드라이버 및 개발자 도구를 사용할 수 있다.
- **서버 관리 불필요**: 서버를 프로비저닝, 패치 또는 관리할 필요가 없어 더 나은 애플리케이션을 구축하는 데 집중할 수 있다. 테이블을 자동으로 확장 및 축소할 수 있으며, 온디맨드 또는 프로비저닝된 용량 모드를 선택하면 애플리케이션의 트래픽 패턴을 기준으로 읽기 및 쓰기 비용을 최적화할 수 있다.
- **큰 규모를 지원하는 성능**: 용량 계획 없이 초당 수천 건의 요청을 처리할 수 있는 사실상 무제한의 처리량과 스토리지로 애플리케이션을 구축할 수 있다.
- **높은 가용성 및 보안**: Amazon Keyspaces는 AWS 리전 내에서 99.99%의 SLA를 제공할 수 있다. 테이블은 기본적으로 암호화되고 고가용성을 위해 여러 AWS 가용 영역에서 세 번 복제된다.

#### 사용 사례

- **낮은 대기 시간이 요구되는 애플리케이션 구축**: 산업 장비 유지 보수, 거래 모니터링, 차량 관리 및 경로 최적화와 같이 10 밀리초 미만의 대기 시간이 요구되는 분야를 위해 고속으로 데이터를 처리한다.
- **오픈 소스 기술을 사용하여 애플리케이션 구축**: Java, Python, Ruby, .NET과 같은 광범위한 프로그래밍 언어에 사용 가능한 오픈소스 Cassandra API 및 드라이버를 사용하여 애플리케이션을 구축한다.
- **Cassandra 워크로드를 클라우드로 이전**: Amazon Keyspaces를 사용하면 추가적인 인프라를 관리하지 않고도 AWS 클라우드에서 Cassandra 테이블을 설정, 보호 및 확대할 수 있다.
- **애플리케이션용 데이터 스토어**: Amazon Keyspaces를 사용하여 IoT(사물 인터넷)를 위한 디바이스 또는 게임의 플레이어 프로필에 대한 정보를 저장할 수 있다. Amazon Keyspaces를 사용하여 로그 파일의 입력 항목 또는 채팅 애플리케이션의 메시지 기록과 같은 대량의 시계열 데이터를 저장할 수 있다.

---

### Amazon MemoryDB for Redis

- **내구성을 갖춘 Redis 호환 인 메모리 데이터베이스 서비스로, 초고속 성능을 제공**
- Redis 호환성 및 내구성을 갖춘 인 메모리 데이터베이스 서비스로, 초고속 성능을 제공

#### 사용 이유

- Stack Overflow의 5년 연속 '가장 사랑받는' 데이터베이스인 Redis를 사용하여 빠르게 애플리케이션을 구축한다.
- 매일 13조 개 이상의 요청과 매초 1억 6천만 개 이상의 요청을 처리하는 초고속 성능으로 데이터에 액세스한다.
- 빠른 데이터베이스 복구 및 재시작을 위해 다중 AZ 트랜잭션 로그를 사용하는 인메모리 스토리지를 사용하여 데이터를 안정적으로 저장한다.
- 클러스터마다 스토리지 수 기가바이트부터 100테라바이트 이상에 이르기까지 애플리케이션의 요구 사항에 맞게 원활하게 크기를 조정한다.

#### 작동 방식

![](images/database_services/amazon_memorydb_redis.png)

#### 사용 사례

- **웹 및 모바일 애플리케이션 구축**: 스트림, 목록, 세트 등의 다양한 Redis 데이터 구조를 사용하여 짧은 대기 시간 및 높은 처리량을 필요로 하는 까다롭고 데이터 집약적인 웹 및 모바일 애플리케이션을 위해 콘텐츠 데이터 스토어, 채팅 및 메시지 대기열, 지리공간 인덱스를 구축한다.
- **소매업을 위한 빠른 고객 데이터 액세스**: 마이크로초 읽기 및 10밀리초 미만의 쓰기 대기 시간으로 개인 맞춤형 서비스를 제공하고 사용자 프로필, 기본 설정, 재고 추적 및 주문 처리를 관리한다.
- **온라인 게임 개발**: 실시간 업데이트를 수행하기 위해 대용량, 짧은 대기 시간, 높은 동시성을 필요로 하는 게임 애플리케이션을 위해 플레이어 데이터 스토어, 세션 기록 및 리더보드를 구축한다.
- **미디어 및 엔터테인먼트 스트리밍**: 미디어 및 엔터테인먼트 애플리케이션을 위해 높은 동시성의 스트리밍 데이터 피드를 실행해 사용자 활동을 수집하고 매일 수백만 개의 요청을 지원한다.

---

### Amazon Neptune

- **완전관리형 그래프 데이터베이스 서비스**
- 탁월한 확장성과 가용성을 제공하도록 설계된 서버리스 데이터 그래프 데이터베이스

#### 사용 이유

- 무제한의 정점과 엣지로 그래프를 확장하고 가장 까다로운 애플리케이션에서 초당 10만 건 이상의 쿼리를 지원한다. 스토리지는 클러스터당 최대 128TiB까지 확장되고 읽기 작업은 클러스터당 최대 15개의 복제본까지 확장된다.
- 6개의 데이터 복사본을 3개의 가용 역역에 걸쳐 유지하므로 애플리케이션의 고가용성을 지원할 수 있다. 글로벌 데이터베이스는 단일 데이터베이스를 여러 AWS 리전에서 사용할 수 있도록 한다.
- 애플리케이션 보안을 위한 ACID 트랜잭션, 자동 백업, 스냅샷 및 특정 시점 복구와 같은 기능도 포함되어 있다. 전송/저장 암호화 및 IAM 지원과 같은 포괄적인 보안 기능을 제공한다.
- AWS 서비스와 통합하면 Amazon SageMaker의 추천 기능 및 Amazon Lambda의 이벤트 기반 함수 등을 사용하여 세계에서 가장 까다로운 데이터 과학 및 개발 관련 문제를 해결할 수 있다.

#### 작동 방식

- Amazon Neptune은 클라우드용으로 구축된 완전관리형 데이터베이스 서비스로 그래프 애플리케이션을 쉽게 구축하고 실행할 수 있도록 지원한다.
- Neptune은 기본 제공 보안, 연속 백업, 서버리스 컴퓨팅 및 다른 AWS 서비스와의 통합을 제공한다.
![](images/database_services/amazon_neptune.png)

#### 사용 사례

- **Customer 360을 통한 개인화 혁신**: 소셜 그래프 및 고객에 대한 360도 보기를 통해 ID 확인 솔루션을 위한 ID 그래프를 손쉽게 구축한다. 광고 타게팅, 개인화 및 분석을 위한 업데이트를 가속화한다.
- **사기 패턴 탐지**: 사람, 장소 및 트랜잭션 사이의 관계를 모델링하여 명백하지 않은 관계를 발견함으로써 사기 패턴을 거의 실시간으로 감지할 수 있는 그래프 쿼리를 구축한다.
- **기계 학습 예측 활용**: Amazon Neptune 기계 학습(ML)은 그래프 신경망(GNN)을 사용하여 그래프 이외의 방법을 사용한 예측과 비교할 때 대부분의 그래프 예측 정확도를 50% 이상 개선한다.
- **IT 보안 개선**: 계층형 보안 접근 방식을 사용하여 IT 인프라를 사전에 탐지하고 조사한다. 자산을 관계로 모델링하여 IT 환경의 다양한 측면이 상호 작용하는 방식을 확인한다.

---

### Amazon Quantum Ledger Database(QLDB)

- **완전관리형 원장 데이터베이스**
- 암호화 방식으로 검증되고 변경 불가능한 데이터 변경 로그 유지

#### 사용 이유

- 변경 불가능하고 투명한 저널을 사용하여 모든 애플리케이션 데이터의 변경에 대한 시퀀스 기록을 추적하고 유지할 수 있다.
- 데이터의 무결성을 신뢰할 수 있다. 기본 제공 암호화 검증을 통해 데이터 변경에 대해 서드 파티 검증을 지원할 수 있다.
- QLDB ACID 트랜잭션 및 Amazon Kinesis로의 실시간 스트리밍 지원을 통해 정확한 이벤트 기반 시스템을 구축할 수 있다.
- 자동 스토리지 및 리소스 크기 조정을 제공하는 서버리스 아키텍처를 통해 소규모로 시작하고 사용한 만큼만 요금을 지불할 수 있다.

#### 작동 방식

- Amazon QLDB는 완전관리형 원장 데이터베이스로, 투명하고, 변경 불가능하며, 암호화 방식으로 검증 가능한 트랜잭션 로그를 제공한다.
![](images/database_services/amazon_quantum_ledger_database.png)

#### 사용 사례

- **금융 거래 저장**: 신용 및 직불 거래와 같은 모든 금융 거래의 완전하고 정확한 레코드를 생성한다.
- **공급망 시스템 조정**: 각 거래 이력을 기록하고, 설비에서 제조되어 매장으로 배송, 저장 및 판매된 모든 배치의 세부 정보를 제공한다.
- **청구 기록 유지**: 청구의 전체 수명을 추적하고 암호화 방식으로 데이터 무결성을 확인하여 데이터 입력 오류 및 조작으로부터 애플리케이션의 복원력을 유지한다.
- **디지털 레코드 중앙 집중화**: 급여, 보너스 및 복지 등 직원 세부 정보에 대한 완전한 중앙 집중식 레코드를 생성하는 레코드 시스템 애플리케이션을 구현한다.

---

### Amazon RDS

- **MySQL, PostgreSQL, Oracle, SQL Server 및 MariaDB를 위한 관리형의 관계형 데이터베이스 서비스**
- 클릭 몇 번으로 클라우드에서 관계형 데이터베이스를 설정, 운영 및 확장할 수 있다.

#### 사용 이유

- 인프라를 프로비저닝하거나 소프트웨어를 유지 관리할 필요 없이 비효율적이고 시간 소모적인 데이터베이스 관리 태스크를 제거할 수 있다.
- 선택한 관계형 데이터베이스 엔진을 클라우드 또는 온프레미스에 배포하고 그 크기를 조정할 수 있다.
- Amazon RDS 다중 AZ 배포로 고가용성을 달성할 수 있다.
- 클라우드에서 탄생한 데이터베이스에서 10년이 넘는 시간에 걸쳐 쌓이고 검증된 운영 전문성, 보안 모범 사례 및 혁신을 활용할 수 있따.

#### 작동 방식

- Amazon RDS는 클라우드에서 간편하게 데이터베이스를 설치, 운영 및 확장할 수 있는 관리형 서비스 모음이다.
- Amazon Aurora(MySQL 호환, PostgreSQL 호환), MySQL, MariaDB, PostgreSQL, Oracle, SQL Server의 7가지 주요 엔진 중에서 선택하고 Amazon RDS on AWS Outposts를 통해 온프레미스에 배포할 수 있다.
- **Amazon RDS**
![](images/database_services/amazon_rds.png)

- **Amazon RDS Custom**
![](images/database_services/amazon_rds_custom.png)

- **Amazon RDS on AWS Outposts**
![](images/database_services/amazon_rds_on_outposts.png)

#### 사용 사례

- **웹 및 모바일 애플리케이션 구축**: 높은 가용성, 처리량 및 스토리지 확장성을 통해 꾸준히 증가하는 앱을 지원할 수 있다. 다양한 애플리케이션 사용량 패턴에 적합한 유연한 종량제 요금을 활용할 수 있다.
- **관리형 데이터베이스로의 이동**: Amazon RDS로 새 앱을 혁신하고 구축하면 데이터베이스를 직접 관리하지 않아도 된다. 데이터베이스를 직접 관리하는 작업은 복잡할 뿐 아니라 시간과 비용이 많이 소요된다.
- **레거시 데이터베이스에서 탈출하기**: Amazon Aurora로 마이그레이션하여 비용은 높고 조건은 나쁜 상용 데이터베이스에서 벗어날 수 있다. Aurora로 마이그레이션하면 1/10의 비용으로 상용 데이터베이스의 확장성, 성능 및 가용성을 얻을 수 있다.

---

### Amazon Redshift

- **빠르고 간단하며 비용 효율적인 데이터 웨어하우징**
- 클라우드 데이터 웨어하우징을 위한 최고의 가격 대비 성능

#### 사용 이유

- 데이터 이동이나 데이터 변환 없이 클릭 몇 번으로 모든 데이터에서 실시간 인사이트와 예측 인사이트를 얻고 데이터 사일로를 없앨 수 있다.
- 추가 비용 없이 바로 사용할 수 있는 획기적인 성능으로 다른 클라우드 데이터 웨어하우스보다 최대 5배 더 뛰어난 가격 대비 성능을 얻을 수 있다.
- 가장 안전하고 신뢰성이 높은 데이터 웨어하우스 서비스로 인프라를 관리해야 하는 부담없이 몇 초 만에 인사이트를 얻을 수 있다.

#### 작동 방식

- Amazon Redshift는 SQL을 사용하여 여러 데이터 웨어하우스, 운영 데이터베이스 및 데이터 레이크에서 정형 데이터 및 반정형 데이터를 분석하고 AWS가 설계한 하드웨어 및 기계 학습을 사용해 어떤 규모에서든 최고의 가격 대비 성능을 지원한다.
![](images/database_services/amazon_redshift.png)

#### 사용 사례

- **재무 및 수요 예측 개선**: 예측 인사이트를 위한 기계 학습 모델을 자동으로 생성, 훈련 및 배포할 수 있다.
- **협업 및 데이터 공유**: 서드 파티 데이터를 기반으로 애플리케이션을 구축하면서 계정, 조직 및 파트너 간에 데이터를 안전하게 공유한다.
- **비즈니스 인텔리전스(BI) 최적화**: Amazon QuickSight, Tableau, Microsoft PowerBI 또는 기타 비즈니스 인텔리전스 도구를 사용하여 인사이트 중심의 보고서 및 대시보드를 구축한다.
- **개발자 생산성 향상**: 드라이버를 구성하고 데이터베이스 연결을 관리하지 않고도 다양한 프로그래밍 언어 및 플랫폼에서 단순화된 데이터 액세스, 수집 및 송신이 가능하다.

----

### Amazon Timestream

- **완전관리형 시계열 데이터베이스**
- 고속의 확장 가능한 서버리스 시계열 데이터베이스

#### 사용 이유

- 평활화, 어림셈, 보간용 기본 제공 분석 기능을 갖춘 SQL을 사용하여 시계열 데이터를 빠르게 분석할 수 있다.
- 서버리스 데이터베이스가 매일 수백만 개의 쿼리를 처리하고 필요에 따라 자동으로 확장된다.
- 최근 데이터를 위한 메모리 스토어 및 과거 데이터를 위한 마그네틱 스토어를 비롯한 스토리지 티어를 통해 데이터 수명 주기 관리를 간소화할 수 있다.
- 기존 시계열 솔루션보다 적은 비용으로 데이터에서 인사이트를 도출하고 비즈니스 의사 결정을 내릴 수 있다.

#### 작동 방식

- Amazon Timestream은 고속의 확장 가능한 서버리스 시계열 데이터베이스 서비스로, 이 서비스를 사용하면 하루 수조 건의 이벤트를 1,000배 더 빠른 속도로 손쉽게 저장하고 분석할 수 있다.
- Amazon Timestream은 자동으로 스케일 업 또는 스케일 다운하여 용량 및 성능을 조정하므로 기본 인프라를 관리할 필요가 없다.
- **IoT 애플리케이션**
![](images/database_services/amazon_timestream_iot_application.png)

- **DevOps 애플리케이션**
![](images/database_services/amazon_timestream_devops_application.png)

- **분석 애플리케이션**
![](images/database_services/amazon_timestream_analytic_application.png)

#### 사용 사례

- **IoT 애플리케이션**: 내장된 분석 기능을 사용하여 IoT 애플리케이션에서 생성된 시계열 데이터를 빠르게 분석하고 추세와 패턴을 식별할 수 있다.
- **DevOps 애플리케이션**: 운영 지표를 수집하고 분석하여 상태 및 사용량을 모니터링하고, 실시간으로 데이터를 분석하여 성능과 가용성을 개선한다.
- **분석 애플리케이션**: 분석 및 인사이트용 추가 집계 기능을 사용하여 애플리케이션의 수신 및 발신 웹 트래픽 데이터를 저장하고 처리한다.

---

### AWS Database Migration Service

- **최소한의 가동 중단으로 데이터베이스 마이그레이션**
- 가동 중지 시간을 최소화하면서 80만 개 이상의 데이터베이스를 안전하게 마이그레이션할 수 있는 솔루션.

#### 사용 이유

- 데이터베이스 및 분석 워크로드를 검색, 평가, 변환하고 자동화 마이그레이션을 통해 AWS로 마이그레이션할 수 있다.
- 다중 AZ와 지속적인 데이터 복제 및 모니터링을 통해 마이그레이션 프로세스 중에 고가용성을 유지하고 가동 중단 시간을 최소화할 수 있다.
- Oracle, SQL Server, PostgreSQL, MySQL, MongoDB, MariaDB, 기타 데이터베이스 및 오픈 소스 데이터베이스 엔진에서 동종 및 이기종 데이터베이스 마이그레이션을 지원한다.
- 적은 비용으로 테라바이트 크기의 데이터베이스를 마이그레이션할 수 있다. 마이그레이션 프로세스 중에 사용한 컴퓨팅 리소스와 추가 로그 스토리지에 대한 비용만 지불하면 된다.

#### 작동 방식

- AWS Database Migration Service(AWS DMS)는 데이터베이스 및 분석 워크로드를 AWS로 빠르고 안전하게 이동하여 가동 중단 시간 및 데이터 손실을 방지하는 데 도움이 되는 관리형 마이그레이션 및 복제 서비스로, 20개 이상의 데이터베이스 및 분석 엔진 간의 마이그레이션을 지원한다.
- **다른 종류의 데이터베이스 마이그레이션**
![](images/database_services/aws_database_migration_service_not_same_type.png)

- **같은 종류의 데이터베이스 마이그레이션**
![](images/database_services/aws_database_migration_service_same_type.png)

#### 사용 사례

- **관리형 데이터베이스로의 이동**: 간소화된 마이그레이션 프로세스를 통해 레거시 또는 온프레미스 데이터베이스에서 관리형 클라우드 서비스로 마이그레이션하여 개발자에게 혁신할 수 있는 시간을 제공한다.
- **라이선스 비용에서 벗어나 비즈니스 성장 가속화**: 목적별 데이터베이스로 현대화하여 1/10의 비용으로 모든 사용 사례에 대해 더 빠르게 혁신하고 구축한다.
- **백업 파일 복제**: 가동 중지 시간 및 데이터 손실을 최소화하기 위해 비즈니스 크리티컬 데이터베이스 및 데이터 저장소의 중복 파일을 생성한다.
- **데이터 마트와의 통합 개선**: 데이터 레이크를 구축하고 데이터 저장소의 변경 데이터에 대한 실시간 처리를 수행한다.

---

### 참고한 자료

- [Amazon Aurora](https://aws.amazon.com/ko/rds/aurora/?nc2=h_ql_prod_db_aa)
- [Amazon Aurora Serverless](https://aws.amazon.com/ko/rds/aurora/serverless/?nc2=h_ql_prod_db_aav2)
- [Amazon DocumentDB](https://aws.amazon.com/ko/documentdb/?nc2=h_ql_prod_db_doc)
- [Amazon DynamoDB](https://aws.amazon.com/ko/dynamodb/?nc2=h_ql_prod_db_ddb)
- [Amazon ElastiCache](https://aws.amazon.com/ko/elasticache/?nc2=h_ql_prod_db_elc)
- [Amazon Keyspaces(Apache Cassandra 호환)](https://aws.amazon.com/ko/keyspaces/?nc2=h_ql_prod_db_mcs)
- [Amazon MemoryDB for Redis](https://aws.amazon.com/ko/memorydb/?nc2=h_ql_prod_db_memdb)
- [Amazon Neptune](https://aws.amazon.com/ko/neptune/?nc2=h_ql_prod_db_nep)
- [Amazon QLDB](https://aws.amazon.com/ko/qldb/?nc2=h_ql_prod_db_qldb)
- [Amazon RDS](https://aws.amazon.com/ko/rds/?nc2=h_ql_prod_db_rds)
- [Amazon Redshift](https://aws.amazon.com/ko/redshift/?nc2=h_ql_prod_db_rs)
- [Amazon Timestream](https://aws.amazon.com/ko/timestream/?nc2=h_ql_prod_db_ts)
- [AWS Database Migration Service](https://aws.amazon.com/ko/dms/?nc2=h_ql_prod_db_dbm)