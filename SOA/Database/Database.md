# Database in AWS

이번 장에서는 **SysOps Administrator**를 준비하며 **AWS의 데이터베이스 서비스**에 대해서 알아보도록 한다.

---

### Amazon RDS

- RDS는 관계형 데이터베이스 서비스다.
- 관리형 DB 서비스로 SQL 언어를 사용하는 DB를 위해 제공된다.
- AWS에서 관리하는 클라우드에 데이터베이스를 생성할 수 있다.
  - Postgres
  - MySQL
  - MariaDB
  - Oracle
  - Microsoft SQL Server
  - Aurora (AWS 독점 데이터베이스)

#### RDS vs EC2 인스턴스에 DB 설치

- RDS는 관리형 서비스다.
  - 자동화된 프로비저닝, OS 패치를 지원한다.
  - 지속적인 백업 및 특정 타임스탬프로 복원(특정 시점 복원)을 지원한다.
  - 모니터링을 위한 대시보드를 지원한다.
  - 향상된 읽기 성능을 위한 읽기 복제본을 제공한다.
  - DR(재해 복구)을 위한 Multi AZ 설정을 지원한다.
  - 업그레이드를 위한 유지 관리 기간을 제공한다.
  - 확작 기능(수직 및 수평)을 제공한다.
  - EBS 지원 스토리지(gp2 또는 io1)를 제공한다.
- 관리형 서비스이므로 인스턴스에 SSH로 접속할 수 없다.

#### Storage Auto Scaling

- RDS DB 인스턴스의 스토리지를 동적으로 늘리는 데 도움이 된다.
- RDS가 무료로 사용하는 데이터베이스 스토리지의 부족을 감지하면 자동으로 확장된다.
- 데이터베이스 스토리지를 수동으로 확장할 필요가 없다.
- 최대로 확장할 수 있는 스토리지 임계값을 설정해야 한다.
- 아래와 같은 상황에 RDS 스토리지를 자동으로 수정한다.
  - 무료 저장용량이 할당된 저장용량의 10% 미만인 경우
  - 낮은 저장용량이 5분 이상 지속되는 경우
  - 마지막 수정 후 6시간이 지난 경우
- 워크로드를 예측할 수 없는 애플리케이션에 유용하다.
- 모든 RDS 데이터베이스 엔진(MariaDB, MySQL, PostgreSQL, SQL Server, Oracle)을 지원한다.

![1-storage-auto-scaling.png](images%2F1-storage-auto-scaling.png)

#### Read Replica

- 최대 15개의 읽기 전용 복제본을 지원한다.
- "동일 AZ", "교차 AZ" 또는 "교차 Region"을 지원한다.
- 복제는 ASYNC이므로 읽기의 최종 일관성이 유지된다.
- 복제본을 자체 DB로 승격시킬 수 있다.
- 애플리케이션은 읽기 복제본을 활성화하기 위해 접속 URL을 변경해야 한다.

![2-read-replicas-for-read-scalability.png](images%2F2-read-replicas-for-read-scalability.png)

- 정상적인 부하를 처리하는 운영 데이터베이스가 있다.
- 일부 분석을 위해 보고용 애플리케이션을 실행해야 한다.
- 읽기 전용 복제본을 생성하여 그곳에서 새로운 워크로드를 실행한다.
- 운영 데이터베이스는 분석용 애플리케이션으로 인해 영향을 받지 않는다.
- 읽기 전용 복제본은 "SELECT"만 가능하며, "INSERT", "UPDATE", "DELETE"를 지원하지 않는다.

![3-read-replicas-use-cases.png](images%2F3-read-replicas-use-cases.png)

- AWS에서는 데이터가 한 AZ에서 다른 AZ로 이동할 때 네트워크 비용이 발생한다.
- **동일한 Region & 다른 AZ인 경우 비용이 발생하지 않고, 다른 Region인 경우 전송 비용이 발생**한다.

![4-read-replicas-network-cost.png](images%2F4-read-replicas-network-cost.png)

#### Multi AZ (Disaster Recovery)

- 동기식으로 복제가 이루어진다.
- 동일한 DNS 이름을 사용하므로 장애 발생시 애플리케이션 코드 수정없이 "standby" 상태의 인스턴스를 사용할 수 있다.
- 가용성을 향상시킬 수 있다.
- AZ 장애, 네트워크 손실, 인스턴스 장애 시 자동으로 장애 조치를 실행한다.
- 앱에 수동으로 개입할 필요가 없다.
- 성능 향상을 위한 확장 용도로 사용되지 않는다.
- 읽기 전용 복제본은 재해 복구(DR)을 위해 다중 AZ로 설정된다.

![5-multi-az-disaster-recovery.png](images%2F5-multi-az-disaster-recovery.png)

#### 단일 AZ에서 다중 AZ로 전환

- DB 정지가 불필요하므로 다운타임이 0이다.
- 데이터베이스에 대해 "수정"을 클릭하기만 하면 된다.
- 내부적으로 아래의 단계가 발생한다.
  - 기존 데이터베이스에서 스냅샷이 생성된다.
  - 새로운 AZ에서 스냅샷을 기반으로 복원이 이루어진다.
  - 두 데이터베이스 간에 동기화가 설정된다.

![6-single-az-multi-az.png](images%2F6-single-az-multi-az.png)

#### 장애 조치 조건

- 다중 가용지역인 경우 아래와 같은 상황에서 장애 조치가 이루어진다.
- 기본(Primary) DB 인스턴스
  - 실패
  - OS 소프트웨어 패치 진행
  - 네트워크 단절로 인한 접속 불가
  - DB 인스턴스 유형 변경과 같은 수정 작업
  - 부하가 늘고 무응답
  - 근본적인 스토리지 오류
- 가용 영역 장애
- 장애 조치가 포함된 재부팅을 사용하여 DB 인스턴스의 수동 장애 조치가 시작

---

### RDS Proxy

#### Lambda

- 기본적으로 Lambda Function은 AWS 소유의 VPC에서 실행된다.
- 사용자의 VPC에 포함되는 리소스인 RDS, ElastiCache, Internal ELB 등에는 접근할 수 없다.
- RDS를 Public Subnet에 생성하여 문제를 해결할 수 있지만 보안상 권장되지 않는 방법이다.

![7-lambda-by-default.png](images%2F7-lambda-by-default.png)

- Lambda의 경우 VPC ID, Subnet, Security Group을 정의해야 한다.
- Lambda는 서브넷에 ENI(Elastic Network Interface)를 생성하고 이 ENI를 통해 직접 연결을 설정한다.
- `AWSLambdaVPCAccessExecutionRole` 역할 추가가 필요하다.

![8-lambda-in-vpc.png](images%2F8-lambda-in-vpc.png)

#### Lambda를 위한 RDS Proxy

- RDS와 함께 Lambda Function을 사용하면 데이터베이스 연결을 열고 유지한다.
- Lambda Function이 계속 추가되면 연결 가능한 Connection이 고갈되어 `TooManyConnections` 오류가 발생할 수 있다.
- RDS Proxy를 사용하면 유휴 연결(idle connection) 정리 및 연결 풀(connection pool)을 처리하는 코드가 더 이상 필요하지 않다.
- IAM 인증 또는 DB 인증, Auto Scaling을 지원한다.
- Lambda Function은 프록시에 연결되어 있어야 한다.
  - Public Proxy => Public Lambda
  - Private Proxy => VPC의 Lambda

![9-rds-proxy-for-lambda.png](images%2F9-rds-proxy-for-lambda.png)

---

### DB Parameter Group

- 파라미터 그룹을 사용하여 DB 엔진을 구성할 수 있다.
- 동적 매개변수가 즉시 적용된다.
- 인스턴스 재부팅 후 정적 매개변수가 적용된다.
- DB와 연결된 파라미터 그룹을 수정할 수 있다.(재부팅 필요)
- DB 기술에 대한 파라미터 목록은 벤더사의 공식문서를 확인해야 한다.
- 아래는 반드시 기억해야 하는 매개변수다.
  - PostgreSQL / SQL Server: `rds.force_ssl=` => SSL 연결을 강제한다.
  - MySQL / MariaDB: `require_secure_transport=1` => SSL 연결을 강제한다.

---

### Backup vs Snapshot

#### Backup

- 백업은 "지속적"이며 특정 시점 복구가 가능하다.
- 유지 관리 기간 동안 백업이 수행된다.
- DB 인스턴스를 삭제해도 자동 백업을 유지할 수 있다.
- 백업된 데이터는 0~35일 사이로 보관 기간을 설정할 수 있다.
- 백업을 비활성화하려면 보관 기간을 0으로 설정하면 된다.

#### Snapshot

- 스냅샷은 IO 작업을 수행하며 몇 초에서 몇 분까지 데이터베이스를 중지할 수 있다.
- 다중 AZ DB에서 생성된 스냅샷은 마스터에는 영향을 미치지 않고 "standby"에만 영향을 미친다.
- 스냅샷은 첫 번째 스냅샷 이후 변경사항이 증분된다.
- DB 스냅샷을 복사 및 공유할 수 있다.
- 수동으로 생성한 스냅샷은 만료되어 삭제되지 않는다.
- DB 삭제 시 "최종 스냅샷"을 생성할 수 있다.

- **자동 백업 또는 DB 스냅샷에서 복원하면 새로운 DB 인스턴스가 생성된다.**

#### Snapshot 공유

- **수동 스냅샷**: AWS 계정간에 공유가 가능하다.
- **자동 스냅샷**: 공유할 수 없으며 먼저 복사한다.
- 암호화되지 않은 스냅샷과 고객 관리 키로 암호화된 스냅샷만 공유할 수 있다.

![10-snapshot-sharing.png](images%2F10-snapshot-sharing.png)

- 암호화된 스냅샷을 공유하는 경우 암호화에 사용된 고객 관리 키도 공유해야 한다.

![11-snapshot-sharing-kms-encryption.png](images%2F11-snapshot-sharing-kms-encryption.png)

---

### Event & Event Subscription

- RDS는 아래와 관련된 이벤트를 기록한다.
  - DB 인스턴스
  - 스냅샷
  - 파라미터 그룹, 보안 그룹 등..
- 예를 들어, DB 상태가 "Pending"에서 "Running"으로 변경된 경우 상태 변경 이벤트를 기록한다.
- **RDS 이벤트 구독**
  - SNS를 이용하여 이벤트 발생 시 알림을 받을 이벤트를 구독한다.
  - 이벤트 소스(인스턴스, 보안 그룹 등..) 및 이벤트 카테고리(생성, 장애 조치 등)를 저장한다.
- RDS는 EventBridge에 이벤트를 전달한다.

![12-rds-events-event-subscription.png](images%2F12-rds-events-event-subscription.png)

#### CloudWatch와 통합

- CloudWatch에는 아래와 같이 RDS와 관련되어 수집되는 지표들이 있다.(hypervisor에서 수집)
  - `DatabaseConnections`
  - `SwapUsage`
  - `ReadIOPS` / `WriteIOPS`
  - `ReadLatency` / `WriteLatency`
  - `ReadThroughPut` / `WriteThroughPut`
  - `DiskQueueDepth`
  - `FreeStorageSpace`
- 강화된 모니터링 (DB 인스턴스의 에이전트에서 수집)
  - 다양한 프로세스나 스레드가 CPU를 어떻게 사용하는지 확인해야 할 때 유용하다.
  - 50개 이상의 새로운 CPU, 메모리, 파일 시스템 및 디스크 I/O 지표에 액세스할 수 있다.

![13-database-log-file.png](images%2F13-database-log-file.png)

---

### Performance Insight

- 데이터베이스 성능을 시각화하고 데이터베이스 성능에 영향을 미치는 모든 문제를 분석한다.
- 성능 개선 도우미 대시보드를 사용하면 데이터베이스 로드를 시각화하고 로드를 필터링할 수 있다.
  - By Waits => 병목 현상이 발생한 리소스(CPU, IO, Lock 등..)을 찾는다.
  - By SQL statements => 문제가 되는 SQL 문을 찾는다.
  - By Hosts => DB를 가장 많이 사용하고 있는 서버를 찾는다.
  - By Users => DB를 가장 많이 사용하고 있는 사용자를 찾는다.
- DBLoad: DB 엔진의 활성 세션 수
- 데이터베이스에 부하를 주는 SQL 쿼리를 볼 수 있다.
- 아래는 여러 메트릭을 확인하는 예시이다.

![14-performance-insights-db-waits.png](images%2F14-performance-insights-db-waits.png)

- 아래와 같이 쿼리 실행 시간을 분석하여 튜닝이 필요한 쿼리를 파악할 수 있다.

![15-performance-insights-sql.png](images%2F15-performance-insights-sql.png)

- 아래와 같이 사용자 별로 사용량을 분석할 수 있다.

![16-performance-insights-user.png](images%2F16-performance-insights-user.png)

---

### Amazon Aurora

- Aurora는 AWS의 독점 기술이며 오픈 소스가 아니다.
- PostgreSQL과 MySQL은 모두 AuroraDB로 지원되므로 드라이버가 Aurora를 PostgreSQL, MySQL로 인식하고 작동한다.
- Aurora는 "AWS 클라우드에 최적화"되어 있으며 RDS의 MySQL보다 5배, RDS의 PostgreSQL보다 3배 이상 향상된 성능을 제공한다.
- Aurora 스토리지는 10GB 단위로 최대 128TB까지 자동으로 증가한다.
- Aurora는 최대 15개의 복제본을 보유할 수 있으며 복제 프로세스는 MySQL보다 빠르다.(10ms 미만의 복제 지연)
- Aurora의 장애 조치는 즉각적으로 이루어지며, HA(고가용성)는 기본적으로 제공된다.
- Aurora는 RDS보다 비용이 더 높지만(20% 정도 높음) 더 효율적이다.

#### 고가용성과 읽기 확장

- 3개의 가용지역에 걸쳐 6개의 복제본이 생성된다.
  - 쓰기에는 6개의 복제본 중 4개가 필요하다.
  - 읽기에는 6개의 복제본 중 3개가 필요하다.
  - peer-to-peer 복제를 통해서 자가 치유를 지원한다.
  - 스토리지는 수백 개의 볼륨에 걸쳐 스트라이핑된다.
- 하나의 Aurora 인스턴스가 쓰기를 수행한다.(마스터)
- 장애가 발생하는 경우 30초 이내에 마스터에 대한 자동 장애 복구가 실행된다.
- 마스터 + 최대 15개의 Aurora 읽기 전용 복제본이 읽기를 제공한다.
- **기본적으로 지역간 복제를 지원**한다.

![17-high-availability-read-scaling.png](images%2F17-high-availability-read-scaling.png)

- Aurora는 Writer Endpoint를 제공하며 항상 마스터 인스턴스를 가르키도록 되어 있다.
- 마스터 인스턴스에 문제가 발생하는 경우 자동으로 읽기 전용 복제본으로 연결하기 때문에 즉각적으로 장애 대응이 가능하다.
- 읽기 전용인 Reader Endpoint를 제공하며 읽기 전용 복제본은 최대 15개까지 지원한다.
- 부하 분산은 연결(Connection) 수준에서 이루어지며, 실행되는 SQL 단위가 아니다.

![18-aurora-db-cluster.png](images%2F18-aurora-db-cluster.png)

#### 특징

- 자동 장애 조치를 제공한다.
- 백업 및 복구를 제공한다.
- 격리 및 보안을 제공한다.
- 업계의 규정을 준수한다.
- Push-button 스케일링을 지원한다.
- 다운타임 없는 자동 패치를 지원한다.
- 고급 모니터링을 지원한다.
- 규칙적인 유지 보수를 지원한다.
- Backtrack: 백업을 사용하지 않고 언제든지 데이터를 복원한다.

#### Backup & Backtracking & Restore

- **자동 백업(Automatic Backup)**
  - 보관기간 1~35일(비활성화가 불가)
  - PITR, 현재 시간으로부터 5분 이내에 DB 클러스터를 복원한다.
  - **새로운 DB 클러스터로 복원**한다.
- **역추적(Backtracking)**
  - DB 클러스터를 시간에 맞춰 앞뒤로 되감을 수 있다.(최대 72시간)
  - **새 DB클러스터를 생성하지 않고 현재 위치에서 복원**한다.
  - Aurora MySQL만 지원한다.
- **복제(Cloning)**
  - 원래 클러스터와 동일한 DB 클러스터 볼륨을 사용하는 새 DB 클러스터를 생성한다.
  - Copy-On-Write 프로토콜을 사용한다.
    데이터의 원본/단일 복사본을 사용하고 데이터가 변경된 경우에만 스토리지를 할당한다.
  - 운영 데이터를 활용하여 개발 환경을 구축하는 등의 작업에 사용된다.

#### SysOps가 알아야 하는 Aurora 상식

- 각각의 읽기 전용 복제본에 우선순위 계층(0 ~ 15)을 연결할 수 있다.
  - 장애 조치 우선순위를 제어한다.
  - RDS는 우선 순위가 가장 높은(가장 낮은 번호) 읽기 전용 복제본을 승격시킨다.
  - 복제본의 우선 순위가 동일한 경우 RDS는 크기가 가장 큰 읽기 전용 복제본을 승격시킨다.
  - 복제본의 우선순위와 크기가 동일한 경우 RDS는 임의의 복제본을 승격시킨다.
- RDS MySQL 스냅샷을 사용하여 Aurora MySQL Cluster로 마이그레이션할 수 있다.

#### CloudWatch Metric

- `AuroraReplicaLag`: 기본 인스턴스에서 업데이트를 복제할 때 지연되는 정도
- `AuroraReplicaLagMaximum`: 클러스터에 있는 모든 DB 인스턴스의 지연 시간 중 최대값
- `AuroraReplicaLagMinimum`: 클러스터에 있는 모든 DB 인스턴스의 지연 시간 중 최소값
- 복제본 지연 시간이 높으면 사용자는 데이터를 가져오는 복제본에 따라 다른 결과를 볼 수 있다.(최종 일관성)
- `DatabaseConnections`: DB 인스턴스에 대한 현재 연결 수
- `InsertLatency`: Insert 작업의 평균 지속 시간

![19-cloudwatch-metric.png](images%2F19-cloudwatch-metric.png)

#### Security

- 저장 시 암호화
  - AWS KMS를 사용한 데이터베이스 마스터 및 복제본 암호화 - 시작 시점에 정의되어 있어야 햔다.
  - 마스터가 암호화되지 않으면 읽기 전용 복제본은 암호화할 수 없다.
  - 암호화되지 않은 데이터베이스를 암호화하려면 DB 스냅샷을 거쳐 암호화된 상태로 복원해야 한다.
- **In-flight encryption**: "TLS-ready"가 기본값으로, 클라이언트 측 AWS TLS 루트 인증서를 사용한다.
- **IAM Authentication**: 데이터베이스에 접속할 때, 사용자/비밀번호 입력 대신 사용할 IAM 역할
- **Security Group**: RDS/Aurora에 대한 네트워크 액세스 제어
- RDS Custom을 제외하고는 **SSH를 사용할 수 없다.**
- 감사(Audit) 로그를 활성화하고 CloudWatch Logs로 전송하여 더 오랜 기간 보관할 수 있다.

---

### Amazon ElastiCache

- 관계형 데이터베이스가 RDS로 제공되는 것과 같은 방식이다.
- ElastiCache는 관리형 Redis 또는 관리형 Memcached를 지원한다.
- 캐시는 성능이 매우 뛰어나고 대기 시간이 짧은 인메모리 데이터베이스다.
- 읽기 집약적인 워크로드에 대한 데이터베이스의 부하를 줄이는 데 도움이 된다.
- 애플리케이션을 Stateless로 만드는 데 도움이 된다.
- AWS는 OS 유지 관리/패치 적용, 최적화, 설정, 구성, 모니터링, 장애 복구 및 백업을 담당한다.
- **ElastiCache를 도입하기 위해서는 애플리케이션 코드의 변경이 많이 필요**하다.

#### Solution Architecture - DB Cache

- 애플리케이션은 ElastiCache를 쿼리하고, 사용할 데이터가 없는 경우(Cache miss) RDS에서 가져와서 ElastiCache에 저장한다.
- RDS의 부하를 완화하는 데 도움이 된다.
- 캐시에는 최신 데이터만 사용되도록 하는 무효화 전략이 있어야 한다.

![20-solution-architecture-db-cache.png](images%2F20-solution-architecture-db-cache.png)

#### Solution Architecture - User Session Store

- 사용자는 분산된 애플리케이션들 중 하나에 로그인을 시도한다.
- 애플리케이션은 ElastiCache에 세션 데이터를 저장한다.
- 사용자가 애플리케이션의 다른 인스턴스를 실행한다.
- 인스턴스가 ElastiCache에 저장되어 있는 정보를 조회하고 사용자가 이미 로그인되어 있는 것으로 판단한다.

![21-solution-architecture-user-session-store.png](images%2F21-solution-architecture-user-session-store.png)

#### Redis vs Memcached

- **Redis**
  - 다중 AZ를 통해 자동 장애 조치를 지원한다.
  - 읽기를 확장하고 고가용성을 제공하는 읽기 전용 복제본이 있다.
  - AOF 지속성을 이용하여 데이터 내구성을 항샹시킨다.
  - **백업 및 복원 기능을 제공**한다.
  - Set과 정렬된(Sorted) Set을 지원한다.

![22-elasticache-redis.png](images%2F22-elasticache-redis.png)

- **Memcached**
  - 데이터 분할을 위한 다중 노드를 지원한다. (Sharding)
  - 고가용성을 지원하지 않는다.
  - 데이터 지속성을 지원하지 않는다.
  - 백업과 복구를 지원하지 않는다.
  - Multi-Threaded 기반의 아키텍처로 구성되어 있다.

![23-elasticache-memcached.png](images%2F23-elasticache-memcached.png)

#### 복제: 클러스터 모드 비활성화

- 기본 노드는 1개, 최대 5개의 복제본을 가질 수 있다.
- 비동기식 복제가 이루어진다.
- 기본 노드는 읽기/쓰기에 사용된다.
- 기본 노드를 제외한 노드들은 읽기 전용으로 사용된다.
- **하나의 샤드에 있는, 모든 노드는 모든 데이터를 가지고 있다.**
- 노드 장애 시 데이터 손실을 방지할 수 있다.
- 장애 조치를 위해 기본적으로 다중 AZ가 활성화된다.
- 읽기 성능 확장에 도움이 된다.

![24-replication-cluster-mode-disabled.png](images%2F24-replication-cluster-mode-disabled.png)

#### 복제: 클러스터 모드 활성화

- **데이터가 여러 샤드로 분할되어 쓰기 확장에 도움**이 된다.
- 각 샤드에는 기본 노드와 최대 5개의 복제본 노드가 있습니다. (클러스터 비활성화와 동일)
- 다중 AZ 기능이 제공된다.
- 클러스터당 **최대 500개의 노드**를 가질 수 있다.
    - 단일 마스터 = 500개의 샤드
    - 1개의 마스터 & 1개의 복제본 = 250개의 샤드
    - 1개의 마스터 & 5개의 복제본 = 83개의 샤드

![26-replication-cluster-mode-enabled.png](images%2F26-replication-cluster-mode-enabled.png)

#### Redis 확장: 클러스터 모드 비활성화

- **Horizontal**:
  - 읽기 전용 복제본을 추가/제거하여 수평 확장/축소한다. (최대 5개의 복제본)
- **Vertical**:
  - 더 크거나 더 작은 노드 유형으로 확장/축소할 수 있다.
  - ElastiCache는 내부적으로 새 노드 그룹을 생성한 후 데이터 복제 및 DNS 업데이트를 수행한다.

![25-redis-scaling-cluster-mode-disabled.png](images%2F25-redis-scaling-cluster-mode-disabled.png)

#### Redis 확장: 클러스터 모드 활성화

- 두 가지 모드가 있다.
  - **Online Scaling**: 확장 프로세스 중에 요청을 계속 처리한다. (다운타임이 없고, 성능 저하 없음)
  - **Offline Scaling**: 확장 프로세스(백업 및 복원) 중에는 요청을 처리할 수 없다.
    추가 구성이 지원된다.(노드 유형 변경, 엔진 버전 업그레이드 등)
- **Horizontal**: 리샤딩 및 샤드 재분배
  - **Resharding**: 샤드를 추가/제거하여 규모를 확장/축소한다.
  - **Shard Rebalancing**: 샤드 간에 키스페이스를 최대한 균등하게 분배한다.
  - 온라인 및 오프라인 확장을 지원한다.
- **Vertical**: 읽기/쓰기 용량 변경
  - 더 크거나 더 작은 노드 유형으로 확장/축소한다.
  - 온라인 확장만 지원한다.

#### Redis Auto Scaling

- 원하는 샤드 또는 복제본을 자동으로 늘리거나 줄인다.
- 대상 추적 및 예약된 확장 정책을 모두 지원한다.
- 클러스터 모드가 활성화된 Redis에서만 작동한다.

![27-redis-auto-scaling.png](images%2F27-redis-auto-scaling.png)

#### Redis Connection Endpoint

- **Standalone Node**
  - 읽기 및 쓰기 작업을 위한 하나의 엔드포인트
- **Cluster Mode Disabled Cluster**
  - Primary Endpoint: 모든 쓰기 작업용
  - Reader Endpoint: 모든 읽기 전용 복제본에 읽기 작업을 균등하게 분할한다.
  - Node Endpoint: 읽기 작업용
- **Cluster Mode Enabled Cluster**
  - Configuration Endpoint: 클러스터 모드 활성화 명령을 지원하는 모든 읽기/쓰기 작업용
  - Node Endpoint: 읽기 작업용

![28-redis-connection-endpoint.png](images%2F28-redis-connection-endpoint.png)

#### Redis 모니터링을 위한 Metric

- `Evictions`: 새로운 쓰기 작업을 위한 공간을 허용하기 위해 캐시가 제거한 만료되지 않은 항목의 수다. (메모리가 초과된 경우)
  - 만료된 항목을 제거하는 제거 정책을 선택한다. (예. LRU(Least Recently Used) 항목 제거)
  - 더 큰 노드 유형(더 많은 메모리)으로 확장하거나 더 많은 노드를 추가하여 수평 확장한다.
- `CPUUtilization`: 전체 호스트의 CPU 사용률을 모니터링한다.
  - 더 큰 노드 유형(더 많은 메모리)으로 확장하거나 더 많은 노드를 추가하여 수평 확장한다.
- `SwapUsage`: 50MB를 초과할 수 없다.
  - 예약된 메모리가 충분히 구성되었는지 확인해야 한다.
- `CurrConnections`: 동시 및 활성 연결 수
  - 문제 해결을 위해 애플리케이션 동작을 조사한다.
- `DatabaseMemoryUsagePercentage`: 메모리 사용률(%)
- `NetworkByteIn/Out` & `NetworkPacketsIn/Out`
- `ReplicationBytes`: 복제되는 데이터의 양
- `ReplicationLag`: 복제본이 기본 노드에서 얼마나 뒤처져 있는 정도

---

#### Memcached Scaling

- Memcached 클러스터는 1 ~ 40개의 노드를 가질 수 있다.(Soft Limit)
- **Horizontal**:
  - 클러스터에서 노드 추가/제거
  - 자동 검색을 통해 앱에서 노드를 찾을 수 있다.
- **Vertical**:
  - 더 크거나 더 작은 노드 유형으로 확장/축소
  - 확장 프로세스
    - 새 노드 유형으로 새로운 클러스터를 생성한다.
    - 새 클러스터의 엔드포인트를 사용하도록 애플리케이션을 업데이트한다.
    - 기존 클러스터를 삭제한다.
  - Memcached의 클러스터와 노드는 캐싱된 데이터가 없는 상태로 시작되므로 애플리케이션에서 데이터를 채워주어야 한다.

![29-memcached-scaling.png](images%2F29-memcached-scaling.png)

#### Memcached Auto Discovery

- 일반적으로 DNS 엔드포인트를 사용하여 클러스터의 개별 캐시 노드에 수동으로 연결해야 한다.
- "Auto Discovery"은 모든 노드를 자동으로 식별한다.
- 클러스터의 모든 캐시 노드는 다른 모든 노드에 대한 메타데이터 목록을 유지한다.
- 고객 관점에서 원활하게 이루어진다.

![30-memcached-auto-discovery.png](images%2F30-memcached-auto-discovery.png)

#### Memcached 모니터링을 위한 Metric

- `Evictions`: 새로운 쓰기 작업을 위한 공간을 허용하기 위해 캐시가 제거한 만료되지 않은 항목의 수다. (메모리가 초과된 경우)
  - 만료된 항목을 제거하는 제거 정책을 선택한다. (예. LRU(Least Recently Used) 항목 제거)
  - 더 큰 노드 유형(더 많은 메모리)으로 확장하거나 더 많은 노드를 추가하여 수평 확장한다.
- `CPUUtilization`
  - 더 큰 노드 유형으로 수직 확장하거나 더 많은 노드를 추가하여 수평 확장한다.
- `SwapUsage`: 50MB를 초과할 수 없다.
- `CurrConnections`: 동시 및 활성 연결 수
  - 문제 해결을 위하여 애플리케이션 동작을 조사한다.
- `FreeableMemory`: 호스트의 여유 메모리 양

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [RDS Performance Insight](https://aws.amazon.com/blogs/database/tuning-amazon-rds-for-mysql-with-performance-insights)