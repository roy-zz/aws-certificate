# Disaster Recovery & Migration

이번 장에서는 SAA를 준비하며 **재해 복구(Disaster Recovery)와 마이그레이션(Migration)** 에 대해서 알아보도록 한다.

---

### Overview

- 기업의 비즈니스 연속성(continuity)이나 재무에 부정적인 영향을 미치는 모든 사건은 재해라고 본다.
- 재해 복구(DR)는 재해에 대비하고 복구하는 것을 의미한다.
- On-premise to On-premise 재해 복구는 매우 비용이 많이 발생하는 작업이다.
- On-premise to AWS Cloud 재해 복구는 하이브리드 재해 복구다.
- 클라우드 환경을 사용한다면 "AWS Cloud Region A" to "AWS Cloud Region B" 재해 복구가 가능하다.
- 재해 복구에 대해서 두 가지 용어를 정의해야 한다.
  - RPO: Recovery Point Objective(복구 시점 목표)
  - RTO: Recovery Time Objective(복구 시간 목표)

![rpo-rto.png](images%2FDisasterRecoveryMigration%2Frpo-rto.png)

- RPO는 복구 시점 목표로 "재해 발생 시간"으로 부터 발생하는 데이터 유실을 의미한다.
- RTO는 복구 시간 목표로 "재해 발생 시간"으로 부터 서비스가 정상적으로 작동할 때까지의 시간을 의미한다.

#### 전략

- 재해 복구를 위한 여러가지 전략이 있다.
  - Backup & Restore
  - Pilot Light
  - Warm Standby
  - Hot Site / Multi Site Approach

![dr-strategy.png](images%2FDisasterRecoveryMigration%2Fdr-strategy.png)

#### Backup & Restore

- 데이터를 백업하고 재해가 발생하는 경우 백업된 데이터로 복원하는 전략이다.
- RPO가 높기 때문에 재해가 발생하는 경우 많은 양의 데이터가 유실된다.

![dr-backup-restore.png](images%2FDisasterRecoveryMigration%2Fdr-backup-restore.png)

#### Pilot Light

- 작은 버전의 앱이 항상 클라우드에서 실행된다.
- 핵심 기능을 유지하고 복구하는 데 유용하게 사용된다.
- "Backup & Restore"와 유사하게 작동한다.
- 아래의 이미지 기준으로 이미 "AWS Cloud"에서 시스템이 가동 대기 중이며, 데이터는 복제되고 있기 때문에 "Backup & Restore" 전략보다 복원 속도가 빠르다.

![dr-pilot-light.png](images%2FDisasterRecoveryMigration%2Fdr-pilot-light.png)

#### Warm Standby

- 전체 시스템이 최소한의 리소스로 가동 중이다.
- 아래의 이미지 기준으로 회사의 데이터센터에 재해 발생하여 "AWS Cloud"에 트래픽이 몰리는 경우 EC2가 Auto Scaling되어 트래픽을 처리할 수 있다.

![dr-warm-standby.png](images%2FDisasterRecoveryMigration%2Fdr-warm-standby.png)

#### Multi Site / Hot Site Approach

- 매우 낮은(분 또는 초단위) RTO를 제공하지만 매우 비싸다는 단점이 있다.
- 운영 수준의 시스템 전부를 AWS 및 On-premise 환경에서 실행시킨다.
- 한 쪽에 재해가 발생하더라도 다른 쪽에서 트래픽을 처리할 수 있다.

![dr-multi-site-hot-site-approach.png](images%2FDisasterRecoveryMigration%2Fdr-multi-site-hot-site-approach.png)

#### AWS DR Tip

- AWS Cloud 환경에서 재해 복구를 하기위한 몇 가지 유용한 팁이 있다.

![all-aws-multi-region.png](images%2FDisasterRecoveryMigration%2Fall-aws-multi-region.png)

- **Backup**
  - EBS 스냅샷, RDS 자동 백업/스냅샷 등을 활용할 수 있다.
  - S3 / S3 IA / Glacier로 정기적으로 푸시, 라이프사이클 정책, 지역(Region)간 복제를 활용할 수 있다.
  - On-premise 환경인 경우 Snowball 또는 Storage Gateway를 활용할 수 있다.
- **High Availability**
  - Route 53을 사용하여 DNS를 Region에서 다른 Region으로 마이그레이션한다.
  - RDS Multi-AZ, ElastiCache Multi-AZ, EFS, S3를 활용할 수 있다.
  - Direct Connect에서 Site-to-Site VPN 복구를 활용할 수 있다.
- **Replication**
  - RDS 복제(Cross Region), AWS Aurora + Global Databases를 활용할 수 있다.
  - On-premise 환경의 데이터베이스를 RDS로 마이그레이션할 수 있다.
  - Storage Gateway를 활용할 수 있다.
- **Automation**
  - CloudFormation / Elastic Beanstalk로 완전히 새로운 환경을 다시 구축할 수 있다.
  - CloudWatch를 사용하여 EC2 인스턴스를 복구 및 재부팅할 수 있다.
  - 자동화를 위하여 AWS Lambda Function을 활용할 수 있다.

---

### Database Migration Service(DMS)

- 데이터베이스를 AWS로 신속하고 안전하게 마이그레이션하고, 탄력적이며 자가 치유(Self-Healing)가 가능하다.
- 마이그레이션 중에도 원본 데이터베이스를 사용할 수 있다.
- 이전 버전의 오라클에서 다음 버전의 오라클로 전환하는 것을 지원한다.
- 이기종간의 마이그레이션을 지원한다.(예: MSSQL to Aurora)
- CDC(Change Data Capture, 변경 데이터 캡처)를 이용한 지속적인 데이터 복제가 가능하다.
- 복제 작업을 수행하는 EC2 인스턴스를 생성해야 한다.

![database-migration-service.png](images%2FDisasterRecoveryMigration%2Fdatabase-migration-service.png)

- DMS를 사용할 때, 복제를 원하는 Source는 아래의 항목이 될 수 있다.
  - On-premise 및 EC2 인스턴스의 데이터베이스: Oracle, MSSQL, MySQL, MariaDB, PostgreSQL, MongoDB, SAP, DB2
  - Azure: Azure SQL Database
  - Amazon RDS(Aurora 포함)
  - Amazon S3
  - DocumentDB
- DMS를 사용할 때, 복제가 될 Target은 아래의 항목이 될 수 있다.
  - On-premise 및 EC2 인스턴스의 데이터베이스: Oracle, MSSQL, MySQL, MariaDB, PostgreSQL, SAP
  - Amazon RDS
  - Redshift, DynamoDB, S3
  - OpenSearch Service
  - Kinesis Data Streams
  - Apache Kafka
  - DocumentDB & Amazon Neptune
  - Redis & Babelfish

#### AWS Schema Conversion Tool(SCT)

- 기존 데이터베이스의 스키마를 다른 엔진의 스키마로 변경한다.
- OLTP: MSSQL 또는 Oracle 데이터베이스 스키마를 MySQL, PostgreSQL, Aurora의 스키마로 변경할 수 있다.
- OLAP: Teradata 또는 Oracle 데이터베이스 스키마를 Amazon Redshift로 변경할 수 있다.
- 컴퓨팅 집약적(compute-intensive) 인스턴스를 선호하여 데이터 변환에 최적화되어 있다.

![schema-conversion-tool.png](images%2FDisasterRecoveryMigration%2Fschema-conversion-tool.png)

- 동일한 DB 엔진을 마이그레이션하는 경우 SCT를 사용할 필요가 없다. 
  - On-premise에서 실행 중인 PostgreSQL에서 RDS PostgreSQL로 마이그레이션하는 경우

- DMS를 활용하여 CDC를 이용한 지속적인 데이터 복제를 할 수 있다.

![dms-continuous-replication.png](images%2FDisasterRecoveryMigration%2Fdms-continuous-replication.png)

#### RDS & Aurora MySQL 마이그레이션

- RDS MySQL에서 Aurora MySQL로 마이그레이션.
  - 옵션 1: RDS MySQL의 DB 스냅샷이 MySQL Aurora DB로 복원된다.
  - 옵션 2: RDS MySQL에서 Aurora Read Replica를 생성하고, 복제 지연 시간이 0일 경우 자체 DB 클러스터로 승격시킨다.
- 외부 MySQL에서 Aurora MySQL로 마이그레이션.
  - 옵션 1: Percona XtraBackup을 사용하여 Amazon S3에서 파일 백업 생성한다. Amazon S3에서 Aurora MySQL DB를 생성한다.
  - 옵션 2: Aurora MySQL DB를 생성한다. mysqldump 유틸리티를 사용하여 MySQL을 Aurora로 마이그레이션한다.(S3 method보다 느림)
- 두 데이터베이스가 모두 가동 중인 경우 DMS를 사용할 수 있다.

![rds-aurora-mysql-migration.png](images%2FDisasterRecoveryMigration%2Frds-aurora-mysql-migration.png)

#### RDS & Aurora PostgreSQL 마이그레이션

- RDS PostgreSQL에서 Aurora PostgreSQL로 마이그레이션
  - 옵션 1: RDS PostgreSQL의 DB 스냅샷이 PostgreSQL Aurora DB로 복원된다.
  - 옵션 2: RDS PostgreSQL에서 Aurora Read Replica를 생성하고, 복제 지연 시간이 0인 경우 자체 DB 클러스터로 승격시킨다.
- 외부 PostgreSQL에서 Aurora PostgreSQL로 마이그레이션
  - 백업을 생성하여 Amazon S3에 저장한다.
  - aws_s3 Aurora 확장자를 사용하여 가져온다.(import)
- 두 데이터베이스가 모두 가동 중인 경우 DMS를 사용할 수 있다.

#### On-Premise와 AWS 전략

- Amazon Linux 2 AMI(.iso 형식)를 VM으로 다운로드할 수 있다.
  - VMWare, KVM, VirtualBox(Oracle VM), Microsoft Myper-V
- VM Import/Export
  - 기존 애플리케이션을 EC2로 마이그레이션한다.
  - On-premise VM에 대한 DR 저장소 전략을 수립한다.
  - VM을 EC2에서 On-premise로 다시 보낼 수 있다.
- AWS Application Discovery Service
  - 마이그레이션 계획을 위해 On-premise 서버에 대한 정보를 수집한다.
  - 서버 사용률과 종속성을 매핑한다.
  - AWS 마이그레이션 허브로 추적한다.
- AWS Database Migration Service(DMS)
  - On-premise to AWS, AWS to On-premise 복제가 가능하다.
  - 다양한 데이터베이스 기술(Oracle, MySQL, DynamoDB 등)과 연동이 가능하다.
- AWS Server Migration Service(SMS)
  - On-premise 운영 서버를 증분하여 AWS로 복제한다.

---

### AWS Backup

- AWS 완전관리 서비스다.
- AWS 서비스 전반에서 중앙 집중식으로 백업 관리 및 자동화가 가능하다.
- 맞춤형 스크립트 및 수동 프로세스로 생성이 필요없다.
- 지원되는 서비스 목록은 아래와 같다.
  - Amazon EC2 / Amazon EBS
  - Amazon S3
  - Amazon RDS (모든 종류의 엔진) / Amazon Aurora / Amazon DynamoDB
  - Amazon DocumentDB / Amazon Neptune
  - Amazon EFS / Amazon FSx (Lustre & Windows File Server)
  - AWS Storage Gateway (Volume Gateway)
- Cross-region 백업을 지원한다.
- Cross-account 백업을 지원한다.
- 지원되는 서비스에 대한 PITR을 지원한다.
- 온디멘드 및 예약 백업이 가능하다.
- 태그 기반(Tag-based) 기반의 백업 정책이 가능하다.
- 백업 계획(Backup Plans)라고 하는 백업 정책을 생성할 수 있다.
  - 백업 주기(Every 12 hours, Daily, Weekly, Monthly, Cron 방식)
  - 백업 윈도우
  - Cold Storage로 전환(Never, Days, Weeks, Months, Years)
  - 보존 기간(Always, Days, Weeks, Months, Years)

![aws-backup.png](images%2FDisasterRecoveryMigration%2Faws-backup.png)

#### Vault Lock

- "AWS Backup Vault"에 저장하는 모든 백업에 대해 WORM(Write Once Read Many) 상태가 적용된다.
- 백업을 보호하기 위한 추가 방어 계층이 있다.
  - 의도치 않거나 악의적인 삭제 작업
  - 보존 기간을 단축하거나 변경하는 업데이트
- 활성화된 경우 루트 사용자도 백업을 삭제할 수 없다.

![aws-backup-vault-lock.png](images%2FDisasterRecoveryMigration%2Faws-backup-vault-lock.png)

---

### AWS Application Discovery Service

- On-premise 데이터 센터에 대한 정보를 수집하여 마이그레이션 프로젝트 계획을 수립할 수 있다.
- 서버 활용률 데이터 및 종속성 매핑이 마이그레이션을 위해 중요하다.
- Agentless Discovery(AWS Agentless Discovery Connector)
  - CPU, 메모리, 디스크 사용량 등 VM 인벤토리, 구성 및 성능 기록
  - 에이전트 기반 탐색(AWS Application Discovery Agent)
- Agent-based Discovery(AWS Application Discovery Agent)
  - 시스템 구성, 시스템 성능, 실행 프로세스 및 시스템 간 네트워크 연결에 대한 세부 정보
- 결과 데이터는 "AWS Migration Hub" 내에서 볼 수 있다.

---

### AWS Application Migration Service(MGN)

- "AWS Server Migration Service(SMS)"를 대체하는 CloudEndure Migration의 "AWS 진화"다.
- AWS로의 애플리케이션 마이그레이션을 간소화하는 "Lift-and-shift (rehost)"다.
- 물리적, 가상 및 클라우드 기반 서버를 AWS에서 기본적으로 실행되도록 변환한다.
- 광범위한 플랫폼, 운영 체제 및 데이터베이스를 지원한다.
- 다운타임을 최소화하고 비용을 절감한다.

![application-migration-service.png](images%2FDisasterRecoveryMigration%2Fapplication-migration-service.png)

---

### VMware Cloud on AWS

- 일부 고객은 VMware Cloud를 사용하여 On-premise 데이터 센터를 관리한다.
- 데이터 센터 용량을 AWS로 확장하면서도 VMware Cloud 소프트웨어를 계속 사용하려는 경우에 사용된다.
- 아래는 대표적인 사용 사례다.
  - VMware vSphere 기반 워크로드를 AWS로 마이그레이션
  - VMware vSphere 기반 Private, Public 및 하이브리드 클라우드 환경 전반에서 프로덕션 워크로드 실행
  - 재해 복구 전략 수립

![vmware-cloud-aws.png](images%2FDisasterRecoveryMigration%2Fvmware-cloud-aws.png)

---

### 대용량 데이터를 AWS로 전송

- 200TB의 데이터를 클라우드에서 전송하며, 100Mbps의 인터넷 연결을 가지고 있다고 가정한다.
- 인터넷을 통한 Site-to-Site VPN
  - 즉시 설정이 가능하다.
  - 200(TB) * 1000(GB) * 1000(MB) * 8(MB) / 100Mbps = 16,000,000초 = 185일이 소요된다.
- Direct Connect를 통한 1Gbps
  - 한 번의 설정이 한달 이상 소요된다.
  - 200(TB) * 1000(GB) * 8(GB) / 1Gbps = 1,600,000초 = 18.5일이 소요된다.
- Snowball을 사용
  - 2~3개의 Snowball을 병렬적으로 가져간다.
  - end-to-end 전송에 약 1주일이 소요된다.
  - DMS와 결합이 가능하다.
- 지속적인 복제/전송을 위해 DMS 또는 DataSync와 함께, Site-to-Site VPN 또는 DX를 사용한다.

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03