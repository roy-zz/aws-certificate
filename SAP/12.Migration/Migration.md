# Migration

- 이번 장에서는 **Solutions Architect Professional (SAP)** 을 준비하며 "마이그레이션"에 대해서 알아보도록 한다.

---

### Cloud Migration: The 6R

- 클라우드 마이그레이션을 위한 [6가지 전략](https://aws.amazon.com/blogs/enterprise-strategy/6-strategies-for-migrating-applications-to-the-cloud/)에 대해서 살펴본다.

![1-cloud-migration-6r.png](images%2F1-cloud-migration-6r.png)

#### Rehosting (lift & shift)

- AWS(애플리케이션, 데이터베이스, 데이터 등)에서 재호스팅하여 간단하게 마이그레이션한다.
- 클라우드 최적화 작업을 수행하지 않고 애플리케이션을 그대로 마이그레이션한다.
- 비용을 30%까지 절감할 수 있다.
- 예를 들어, AWS VM 가져오기/내보내기, AWS Service Migration Service 등이 있다.

#### Replatforming

- 데이터베이스를 RDS로 마이그레이션한다.
- Java(Tomcat) 기반의 애플리케이션을 Elastic Beanstalk로 마이그레이션한다.
- 핵심 아키텍처를 변경하지 않고 일부 클라우드 최적화를 활용한다.

#### Repurchase (drop & shop)

- 클라우드로 이동하는 동안 다른 제품으로 이동한다.
- SaaS 플랫폼으로 이동하는 경우가 많다.
- 단기적으로는 비용이 많이 들지만 신속하게 구축할 수 있다.
- 예를 들어, CRM -> Selesforce, HR -> Workday, CMS -> Drupal 등이 있다.

#### Refactoring / Re-architecting

- Cloud Native 기능을 사용하여 애플리케이션을 설계하는 방법을 다시 그린다.
- 기능, 확장, 성능을 추가해야 하는 비즈니스 요구에 따라 추진할 수 있다.
- 예를 들어, 서버리스 아키텍처로 애플리케이션 이동, S3 사용 등이 있다.

#### Retire

- 필요하지 않은 것들을 종료한다.
  - 애플리케이션을 다시 설계한 결과로 필요없는 구성요소들이 발생할 수 있다.
- 공격을 위해 노출된 부분을 감소시키므로 보안성이 향상된다.
- 비용을 10 ~ 20% 절감할 수 있다.
- 필요하지 않은 구성요소가 제거되어 유지보수해야 하는 리소스에 집중할 수 있다.

#### Retain

- 여러 이용으로 인해 마이그레이션하지 않는다.
- 클라우드 마이그레이션에서는 여전히 결정해야 하는 사항이다.

---

### AWS Storage Gateway

![2-storage-gateway.png](images%2F2-storage-gateway.png)

- 온프레미스 데이터와 클라우드 데이터의 브릿지 역할을 한다.
- 재해 복구, 백업 및 복원, 계층형 스토리지, 온프래미스 캐시 및 저지연 파일 접근 등에 사용된다.
- 네 가지의 스토리지 게이트웨이가 있다.
  - S3 File Gateway
  - FSx File Gateway
  - Volume Gateway
  - Tape Gateway

#### S3 File Gateway

- NFS 및 SMB 프로토콜을 사용하여 구성된 S3 버킷에 액세스할 수 있다.
- 가장 최근에 사용된 데이터는 파일 게이트웨이에 캐시된다.
- S3 Standard, S3 Standard IA, S3 One Zone IA, S3 Intelligent Tiering을 지원한다.
- 라이프사이클 정책에 의한 S3 Glacier로의 이동을 지원한다.
- 각 파일 게이트웨이에 대해 IAM 역할을 사용하는 버킷 액세스 정책을 사용할 수 있다.
- 사용자 인증을 위해 Active Directory와 통합된 SMB 프로토콜을 지원한다.

![3-s3-file-gateway.png](images%2F3-s3-file-gateway.png)

#### FSx File Gateway

- Windows 파일 서버용 Amazon FSx에 대한 기본 액세스를 제공한다.
- 자주 액세스하는 데이터의 로컬 캐시를 제공한다.
- Windows 기본 호환성을 제공한다. (SMB, NTFS, Active Directory..)
- 그룹 파일 공유 및 홈 디렉토리에 유용하다.

![4-fsx-file-gateway.png](images%2F4-fsx-file-gateway.png)

#### Volume Gateway

- S3에서 지원하는 iSCSI 프로토콜을 사용하여 스토리지를 차단한다.
- 온프레미스 볼륨을 복원하는데 도움이 되는 EBS 스냅샷을 지원한다.
- Cached Volumes: 최신 데이터에 대한 낮은 지연 시간 액세스
- Stored Volumes: 전체 데이터셋이 전제되어 있으며, S3로 백업이 예약되어 있다.

![5-volume-gateway.png](images%2F5-volume-gateway.png)

#### Tape Gateway

- 물리적 테이브를 사용한 백업 프로세스가 있는 회사에서 사용된다.
- 기존과 동일한 프로세스를 사용하지만 테이프 게이트웨이를 사용하면 데이터는 클라우드에 저장된다.
- S3와 Glacier가 지원하는 Virtual Tape Library(VTL)을 사용한다.
- 기존 테이프 기반 프로세스(iSCSI 인터페이스)를 사용하여 데이터를 백업한다.
- 주요 백업 소프트웨어 공급업체와 협력할 수 있다.

![6-tape-gateway.png](images%2F6-tape-gateway.png)

#### Hardware appliance

- 스토리지 게이트웨이를 사용하면 온프레미스 가상화가 필요하다.
- 그렇지 않은 경우에 "Storage Gateway Hardware Appliance"를 사용할 수 있다.
- 구매는 amazon.com에서 할 수 있다.
- 파일 게이트웨이, 볼륨 게이트웨이, 테이프 게이트웨이와 함께 작동한다.
- 필요한 CPU, 메모리, 네트워크 SSD 캐시 리소스가 있다.
- 소규모 데이터 센터에서 매일 NFS 백업에 유용하게 사용된다.

![7-hardware-appliance.png](images%2F7-hardware-appliance.png)

#### Summary

![8-aws-storage-gateway.png](images%2F8-aws-storage-gateway.png)

- 온프레미스에서 스토리지 게이트웨이 VM과 Hardware Appliance를 배포하고 스토리지 게이트웨이 서비스를 배포한다.
- 로컬 캐시에 파일 게이트웨이를 사용하고 싶다면 "사용자/그룹 파일 공유"가 있고 NFS나 SMB 프로토콜을 통해 접근한다.
- S3 파일 게이트웨이에 직접 연결하여 S3에 데이터를 백업하도록 구축할 수 있다. 
  - 단, Glacier와 Glacier Deep Archive는 사용할 수 없다.
  - 두 클래스를 사용해야 한다면 S3 라이프사이클 정책을 만들어 이동되도록 할 수 있다.
- FSx 파일 게이트웨이를 사용하는 경우 Windows 파일 서버를 위해 FSx로 데이터를 보내면 S3에 자동으로 백업된다.
- 볼륨 게이트웨이의 경우 iSCSI 프로토콜에 볼륨을 탑재하고 볼륨 게이트웨이는 스토리지 게이트웨이를 통해 S3에 연결된다.
  - S3는 EBS 볼륨으로 변환되어 AWSDptj wpeofh qhrdnjsgkf tn dlTek.
- 백업 애플리케이션은 iSCSI VTL 프로토콜 테이프 게이트웨이에 연결된다.
  - 테이프 게이트웨이는 S3에 VTL로 연결되어 있다.
  - 테이프를 Glacier와 Glacier Deep Archive에 저장하도록 구축할 수 있다.

---

### AWS Storage Gateway - Advanced

#### File Gateway: Extensions

![9-file-gateway-extensions.png](images%2F9-file-gateway-extensions.png)

- 온프레미스의 서버는 NFS 또는 SMB 프로토콜을 이용해서 파일 게이트웨이 어플라이언스에 접근할 수 있다.
  - 파일은 S3의 백엔드에 저장된다.
- VPC를 사용해 다른 게이트웨이 어플라이언스를 생성하고 EC2 인스턴스를 NFS와 SMB에 연결할 수 있다.
  - EC2 인스턴스가 S3에 파일 시스템으로 접근하고 싶은 경우 파일 게이트웨이 어플라이언스를 사용할 수도 있다.
- 같은 프로토콜을 이용해 온프레미스에서 클라우드로 애플리케이션을 마이그레이션할 수 있다.
- S3 이벤트는 람다를 호출할 수 있고, 데이터를 Athena, Redshift Spectrum, EMR로 분석할 수 있다.
- 재해 복구를 위하여 S3의 데이터를 다른 리전으로 복제할 수 있다.

#### File Gateway: Read Only Replicas

![10-file-gateway-read-only-replicas.png](images%2F10-file-gateway-read-only-replicas.png)

- 하나의 온프레미스 데이터 센터에서 다른 데이터 센터로 파일 시스템을 복제해야 한다.
- 첫번째 온프레미스 데이터 센터에서는 파일을 게이트웨이 플랜1로 생성하고 데이터를 저장하면 데이터가 S3 버킷에 저장된다.
- 다른 온프레미스 데이터 센터에서는 읽기 전용 복제를 만들어서 파일 시스템, 애플리케이션 서버가 파일 게이트웨이 어플라이언스의 읽기 전용 복사본을 읽을 수 있도록 할 수 있다.

#### File Gateway: Backup and Lifecycle Policies

![11-file-gateway-backup-lifecycle-policies.png](images%2F11-file-gateway-backup-lifecycle-policies.png)

- 백업과 수명 주기 정책을 설정할 수 있다.
- 애플리케이션 서버가 파일 게이트웨이의 파일 시스템과 통신하고 그 파일들은 S3로 백업된다.
- 자주 사용되지 않는 파일들을 위해 S3 수명 주기 정책을 설정할 수 있다.
- 자주 액세스되지 않는 파일들은 S3 Standard - IA로 이동된다.
  - 일정 시간이 지나면 S3 Glacier로 이동된다.

#### File Architecture

- **Amazon S3 Object Versioning**
  - S3 Object Versioning을 활성화하면 수정된 여러 개의 객체의 버전을 저장할 수 있다.
  - 사용자들이 특정 파일을 이전 버전으로 복원해달라는 요청을 할 때 유용하게 사용된다.
  - 전체 파일 시스템을 이전 버전으로 복원할 수 있다.
  - 복원 알림을 받으려면 게이트웨이에서 `RefreshCache` API를 사용해야 한다.
- **Amazon S3 Object Lock**
  - WORM(Write Once Read Many) 데이터에 대한 파일 게이트웨이를 사용할 수 있도록 지원한다.
  - 파일 공유 클라이언트에 파일 수정이나 이름 변경이 있는 경우 파일 게이트웨이는 이전 버전에 영향을 주지 않고 객체의 새로운 버전을 생성하고 원래 잠금 버전은 변경되지 않는다.

---

### AWS Snow Family

- 엣지에서 데이터를 수집 및 처리하고 데이터를 AWS로 마이그레이션할 수 있는 보안이 뛰어난 휴대용 장치다.
- 데이터 마이그레이션을 위해서 "Snowcone", "Snowball Edge", "Snowmobile"이 사용된다.
- 엣지 컴퓨팅을 위해서 "Snowcone", "Snowball Edge"가 사용된다.

#### Data Migration

- 아래의 표는 데이터의 크기에 따라 마이그레이션에 소요되는 시간을 나타낸다.

![12-data-migration-snow-family.png](images%2F12-data-migration-snow-family.png)

- 데이터 마이그레이션을 위해서는 아래의 과제들을 풀어야 한다.
  - 제한된 연결
  - 제한된 대역폭
  - 높은 네트워크 비용
  - 공유 대역폭
  - 접속 안정성
- 만약 네트워크를 통해 데이터를 전송하는데 일주일 이상 소요된다면 스노우볼 장비를 사용하는 것이 권장된다.

#### Diagrams

- 클라이언트는 S3 버킷에 네트워크를 통해 직접 데이터를 전송할 수 있다.

![13-direct-upload-s3.png](images%2F13-direct-upload-s3.png)

- 스노우볼 장비를 이용하는 경우 장비를 요청하여 우편으로 전달받는다.
- 데이터를 로컬 기기에서 직접 스노우볼 장비에 저장한다.
- 데이터가 저장된 장비를 AWS로 전달하면 데이터가 S3 버킷으로 저장된다.

![14-with-snow-family.png](images%2F14-with-snow-family.png)

#### Snowball Edge

- 물리적 데이터 전송 솔루션으로 AWS 내부 또는 외부로 TB 또는 PB의 데이터를 이동시킬 수 있다.
- 네트워크를 통해 데이터를 이동하는 방식의 대안으로 사용된다.
- 데이터 전송 작업당 비용을 지불한다.
- 블록 스토리지 및 S3 호환 객체 스토리지를 제공한다.
- **Snowball Edge Storage Optimized**
  - 블록 볼륨 및 S3 호환 객체 스토리지를 위한 80TB의 HDD 용량을 제공한다.
- **Snowball Edge Compute Optimized**
  - 블록 볼륨 및 S3 호환 객체 스토리지를 위한 42TB의 HDD 또는 28TB의 NVMe 용량을 제공한다.
- 대규모 데이터 클라우드 마이그레이션, DC 해제, 재해 복구 등에 사용된다.

![15-snowball-edge.png](images%2F15-snowball-edge.png)

#### Snowcone & Snowcone SSD

- 어디서나 휴대가 가능한 간편한 소형 컴퓨팅이다. (2.1kg)
- 견고하고 안전하며 혹독한 환경에서도 견딜 수 있다.
- 엣지 컴퓨팅, 저장 및 데이터 전송에 사용된다.
- **Snowcone**: 8TB의 HDD 스토리지를 제공한다.
- **Snowcone SSD**: 14TB의 SSD 스토리지를 제공한다.
- 공간의 제약 등 Snowball을 사용할 수 없는 환경에 사용된다.
- 배터리 및 케이블을 사용자가 직접 제공해야 한다.
- 오프라인으로 AWS로 데이터를 전송하거나, 인터넷에 연결하여 AWS DataSync를 사용하여 데이터를 전송할 수 있다.

![16-snowcone-ssd.png](images%2F16-snowcone-ssd.png)

#### Snowmobile

![17-snowmobile.png](images%2F17-snowmobile.png)

- 엑사바이트 단위의 데이터를 전송할 수 있다. (1EB = 1,000PB = 1,000,000TB)
- 각각의 스노우모빌은 100PB의 데이터를 저장할 수 있으며 여러대를 사용하여 더 큰 데이터를 이동할 수 있다.
- 높은 보안을 제공하며, 온도 제어, GPS, 연중무휴 비디오 보안을 제공한다.
- 10PB 이상 데이터를 전송하는 경우 스노우볼보다 우수하다.

#### Snow Family 비교

- 아래의 표를 참고하여 각 Snow Family 장비의 차이점을 확인할 수 있다.

![18-snow-family-data-migration.png](images%2F18-snow-family-data-migration.png)

#### 사용 절차

1. AWS 콘솔에서 스노우볼 디바이스 전송을 요청한다.
2. 서버에 스노우볼 클라이언트와 AWS OpsHub를 설치한다.
3. Snowball을 서버에 연결하고 클라이언트를 사용하여 파일을 복사한다.
4. 작업이 완료되면 장비를 다시 AWS로 전송한다.
5. 데이터가 S3 버킷에 로드된다.
6. Snowball 장비의 데이터는 완벽하게 삭제된다.

#### Edge Computing

- 엣지 위치에서 생성되는 동안 데이터를 처리할 수 있다.
  - 길 위의 트럭, 바다 위의 배, 지하에 있는 광산 등이 있다.
- 이러한 장소들은 인터넷이 제한되거나 차단되고, 컴퓨팅 파워가 제한되거나 차단될 수 있다.
- 엣지 컴퓨팅을 수행하기 위해 Snowball Edge / Snowcone 장치를 설정한다.
- 엣지 컴퓨팅은 아래와 같은 사용 사례가 있다.
  - 데이터 전처리
  - 엣지에서 머신러닝
  - 미디어 스트림의 트랜스코딩
- 필요한 경우 디바이스를 AWS로 다시 전송할 수 있다.

- **Snowcone & Snowcone SSD**
  - 2 CPUs, 4GB Memory, 유선 또는 무선 액세스
  - 코드 또는 옵션 배터리를 사용하는 USB-C 전원
- **Snowball Edge - Compute Optimized**
  - 104 vCPUs, 416GiB Memory
  - 비디오 처리 및 머신러닝을 위해서 선택적으로 GPU를 사용할 수 있다.
  - 28TB NVMe 또는 42TB HDD의 스토리지를 사용할 수 있다.
- **Snowball Edge - Storage Optimized**
  - 최대 40 vCPUs, 80GiB Memory, 80TB의 스토리지를 제공한다.
- 모두 EC2 인스턴스, AWS 람다 함수를 실행할 수 있다. (AWS IoT Greengrass 사용)
- 장기 배포 옵션을 사용하여 1년 또는 3년 동안 할인된 금액으로 사용할 수 있다.

#### AWS OpsHub

- 이전에는 Snow Family 디바이스를 사용하려면 CLI가 필요했다.
- 오늘날에는 Snow Family 디바이스를 관리하기 위해 AWS OpsHub(컴퓨터/노트북에 설치한 소프트웨어)를 사용할 수 있다.
  - 단일 또는 클러스터된 디바이스 잠금 해제 및 구성할 수 있다.
  - 전송 중인 파일을 확인할 수 있다.
  - Snow Family Device에서 실행 중인 인스턴스를 시작 및 관리할 수 있다.
  - 디바이스 메트릭(스토리지 용량, 활성 인스턴스)을 모니터링할 수 있다.
  - 디바이스에서 호환되는 AWS 서비스를 시작할 수 있다. (예: EC2 인스턴스, AWS DataSync, NFS)

![19-aws-opshub.png](images%2F19-aws-opshub.png)

#### 전송 성능 향상

- 여러 터미널에서 한 번에 여러 번 쓰기 작업을 수행한다.
- 작은 용량의 파일을 일괄 전송한다. 
  - 최소 1MB까지 소규모 파일로 압축한다.
- 전송 중인 파일에 대해 다른 작업을 수행하지 않는다.
- 로컬 네트워크의 사용을 줄인다.
- 컴퓨터에 직접 연결하여 불필요한 홉을 제거한다.
- 파일 인터페이스를 사용하는 데이터 전송 속도는 일반적으로 25MB/s에서 40MB/s사이다.
  - 이보다 더 빨리 데이터를 전송해야 한다면 일반적으로 250MB/s에서 400MB/s 사이의 전송 속도를 가진 S3 Adapter for Snowball을 사용해야 한다.

---

### Database Migration Service (DMS)

- 데이터베이스를 AWS로 빠르고 안전하게 마이그레이션할 수 있으며, 복원력이 뛰어난 자가복구 기능을 제공한다.
- 마이그레이션하는 동안 소스 데이터베이스를 계속 사용할 수 있다.
- 동종 마이그레이션을 지원한다.
  - 예를 들어, Oracle에서 Oracle 데이터베이스로 마이그레이션할 수 있다.
- 이기종 마이그레이션을 지원한다.
  - 예를 들어, Microsoft SQL Server에서 Aurora로 마이그레이션할 수 있다.
- CDC를 사용하여 연속된 데이터 복제를 지원한다.
- 복제 작업을 수행하려면 EC2 인스턴스를 생성해야 한다.

![20-dms.png](images%2F20-dms.png)

- 소스 데이터베이스는 온프레미스 서버에 있다.
- EC2 인스턴스를 생성하여 DMS 소프트웨어를 실행하면 대상 데이터베이스에 데이터를 삽입한다.

#### Source & Target

- **Source**
  - 온프레미스와 EC2 인스턴스 데이터베이스: Oracle, MSSQL Server, MySQL, MariaDB, PostgreSQL, MongoDB, SAP, DB2
  - Azure: Azure SQL Database
  - Amazon RDS: Aurora를 포함한 모든 종류의 엔진
  - Amazon S3
  - DocumentDB
- **Target**
  - 온프레미스와 EC2 인스턴스 데이터베이스: Oracle, MSSQL Server, MySQL, MariaDB, PostgreSQL, SAP
  - Aurora를 포함한 모든 종류의 엔진
  - Amazon Redshift
  - Amazon DynamoDB
  - Amazon S3
  - OpenSearch Service (Source로 사용될 수 없음)
  - Kinesis Data Streams (Source로 사용될 수 없음)
  - DocumentDB

#### Schema Conversion Tool (SCT)

- 데이터베이스 스키마를 한 엔진에서 다른 엔진으로 변환할 때 사용된다.
- OLTP의 경우 SQL Server나 Oracle에서 MySQL로 변환하거나, PostgreSQL에서 Aurora로 변환할 수 있다.
- OLAP의 경우 Teradata나 Oracle에서 Redshift로 변환할 수 있다.

![21-schema-conversion-tool.png](images%2F21-schema-conversion-tool.png)

- SCT는 DMS와 함께 실행되어 소스의 데이터베이스에서 타겟으로 스키마를 변환한다.
- **동일한 DB 엔진을 마이그레이션하는 경우에는 SCT를 사용할 필요가 없다.**
  - 예를 들어, 온프레미스에서 사용하는 PostgreSQL DB를 RDS PostgreSQL로 마이그레이션하는 경우 동일한 엔진이므로 SCT를 사용할 필요가 없다.

#### Good to Know

- VPC 피어링, VPN(site to site, software), Direct Connect와 함께 작동한다.
- 전체 로드, 전체 로드 + CDC, CDC 조합을 모두 지원한다.
- **Oracle**
  - Source: "BinaryReader"를 사용하여 소스에 대한 TDE를 지원한다.
  - Target: 기본 키가 있는 테이블의 BLOB와 TDE를 지원한다.
- **OpenSearch**
  - Source: 지원하지 않는다.
  - Target: DMS를 사용하여 관계형 데이터베이스에서 마이그레이션할 수 있다.
  - 따라서 DMS를 사용하여 OpenSearch의 데이터를 복제할 수 없다.

#### Snowball & DMS

- 더 큰 데이터 마이그레이션에는 TB 단위의 정보가 포함될 수 있다.
- 네트워크 대역폭 또는 데이터 크기로 인해 제한될 수 있다.
- AWS DMS는 Snowball Edge & S3를 사용하여 마이그레이션 속도를 높일 수 있다.
- 아래의 단계를 따라 작동한다.
  1. AWS SCT를 사용하여 데이터를 로컬로 추출하고 엣지 디바이스로 이동한다.
  2. 엣지 디바이스 또는 디바이스를 AWS로 다시 전송한다.
  3. AWS가 디바이스를 받은 후 엣지 디바이스는 데이터를 자동으로 S3 버킷으로 로드한다.
  4. AWS DMS는 파일을 가져와 대상 데이터 저장소로 데이터를 마이그레이션한다. CDC(Change Data Capture)를 사용하는 경우 해당 업데이트는 S3 버킷에 기록된 후 대상 데이터 저장소에 적용된다.

---

### AWS Cloud Adoption Readiness Tool (CART)

- 조직이 클라우드 채택 및 마이그레이션을 위한 효율적이고 효과적인 계쇡을 수립할 수 있도록 지원한다.
- 클라우드로 전환하려는 아이디어를 AWS 모범 사례를 따르는 상세 계획으로 전환한다.
- 6가지 관점(비즈니스, 인력, 프로세스, 플랫폼, 운영, 보안)에 걸친 일련의 질문에 답변할 수 있다.
- 마이그레이션 준비 수준에 대한 사용자 정의 리포트를 생성할 수 있다.

![22-cloud-adoption-readiness-tool.png](images%2F22-cloud-adoption-readiness-tool.png)

---

### Disaster Recovery

- 회사의 사업 지속성 또는 재무에 부정적인 영향을 미치는 모든 사건을 재해(Disaster)로 정의할 수 있다.
- 재해 복구(DR)는 재해에 대비하고 복구하는 것을 의미한다.
- 다양한 종류의 재해 복구 방법이 있다.
  - 온프레미스 -> 온프레미스: 전통적인 DR이며 매우 많은 비용이 발생한다.
  - 온프레미스 -> AWS 클라우드: 하이브리드 재해 복구다.
  - AWS 클라우드 리전 A -> AWS 클라우드 리전 B
- 재해 복구를 위한 두 개의 용어를 기억해야 한다.
  - RPO: Recovery Point Objective
  - RTO: Recovery Time Objective

#### RPO & RTO

![23-rpo-rto.png](images%2F23-rpo-rto.png)

- RPO는 "복구 지점 목표"로 백업을 얼마나 자주 실행하는지에 따라 결정된다.
  - 재해가 발생하면 기본적으로 RPO와 재해 발생시점 사이의 시간은 데이터가 손실된다.
  - 예를 들어, 매시간 데이터를 백업했는데 재해가 발생하면 한 시간 전으로 돌아갈 수 있으므로 한 시간의 데이터가 손실된다.
  - 실제로는 요구 사항에 따라 다르지만 RPO는 재해 발생 시 데이터 손실을 어느 정도 수용할 의향이 있는지에 따라 결정된다.
- RTO는 재해가 발생한 시점부터 복구될 때까지의 시간이다.
  - 따라서 재해 발생 시점과 RTO 사이에는 애플리케이션의 가동 중지 시간이 있다.
- 기본적으로 RPO 최적화는 RTO 솔루션과 아키텍처 결정을 추진하며, 비용과 직결된다.

#### Strategy

- 일반적으로 사용되는 재해 복구 전략은 네 가지가 있다.
  - Backup and Restore
  - Pilot Light
  - Warm Standby
  - Hot Site / Multi Site Approach

![24-disaster-recovery-strategy.png](images%2F24-disaster-recovery-strategy.png)

#### Backup & Restore (High RPO)

![25-backup-and-restore.png](images%2F25-backup-and-restore.png)

- 회사에 데이터 센터가 있고 AWS 클라우드에는 S3 버킷이 있다.
- 데이터를 백업하는 경우 AWS 스토리지 게이트웨이를 사용할 수 있다.
  - 비용 최적화를 목적으로 S3 라이프사이클 정책을 통해 Glacier로 데이터를 전환할 수 있다.
- 스노우볼 패밀리를 사용하는 경우 일주일에 한 번씩 대용량의 데이터를 Glacier로 이전할 수도 있다.
  - 이러한 경우 RPO는 일주일정도가 된다.
- AWS 클라우드를 사용한다면 EBS 볼륨, Redshift, RDS를 사용하게 되고 RPO는 24시간 혹은 1시간정도 된다.
  - 얼마나 자주 스냅샷을 생성하는지에 따라 RPO가 결정된다.
- 재해가 발생하면 모든 데이터를 복원해야 하며 AMI를 이용해 EC2 인스턴스를 재생성하고 애플리케이션을 실행할 수 있다.
- 스냅샷을 사용하여 바로 복원하고 RDS 데이터베이스나 EBS 볼륨, Redshift 등을 재생성할 수 있다.
- 데이터를 복원하는데 시간이 많이 걸리며 RTO 또한 높지만, 비용이 저렴하다.

#### Pilot Light

- 작은 버전의 앱이 항상 클라우드에서 실행되고 있다.
- 백업 및 복원과 매우 유사하게 작동한다.
- 중요한 시스템이 이미 설치되어 있으므로 백업 및 복원보다는 속도가 빠르다.

![26-pilot-light.png](images%2F26-pilot-light.png)

- 데이터 센터에 서버와 데이터베이스가 있고 AWS 클라우드에는 EC2 인스턴스와 RDS가 있다.
- 데이터 센터의 데이터베이스는 지속적으로 RDS로 데이터가 복제되고 있다.
- 만약 재해가 발생하면 EC2 인스턴스를 실행하고 Route53은 트래픽을 EC2 인스턴스로 전환한다.

#### Warm Standby

- 전체 시스템이 가동 중이지만 최소한의 규모로 실행되고 있다.
- 재해가 발생하는 경우 운영 수준의 트래픽을 처리할 수 있을 정도로 확장될 수 있다.

![27-warm-standby.png](images%2F27-warm-standby.png)

- 데이터 센터에는 리버스 프록시, 앱 서버, 마스터 DB가 실행되고 있다.
- Route53은 회사의 데이터 센터로 트래픽을 전달하고 있다.
- 데이터 센터의 DB에서 클라우드의 RDS로 데이터 복제가 이루어지고 있다.
- AWS의 EC2 Auto Scaling Group은 최소한의 용량으로 실행되고 있으여 데이터 센터의 DB와 통신하고 있다.
- 재해가 발생하는 경우 Route53은 페일오버하여 트래픽을 AWS의 ELB로 전달하도록 수정한다.
- EC2 인스턴스 또한 데이터 센터의 DB가 아니라 RDS를 바라보도록 페일오버할 수 있다.
- 이전에 살펴본 전략보다 RPO와 RTO를 낮출 수 있지만 많은 비용이 발생한다.

#### Multi Site / Hot Site Approach

- 매우 낮은 RTO를 제공하지만 매우 높은 비용을 지불해야 한다.
- 운영 수준의 트래픽을 처리할 수 있는 시스템을 온프레미스 서버와 클라우드에서 실행하고 있다.

![28-multi-site-hot-site-approach.png](images%2F28-multi-site-hot-site-approach.png)

- 온프레미스 서버와 AWS 서버에서 운영 수준의 애플리케이션이 실행 중이므로 Route53은 양쪽으로 트래픽을 전달한다.
- 온프레미스에 재해가 발생하는 경우 EC2 인스턴스는 RDS Slave 데이터베이스에 페일오버할 수 있따.

#### AWS Multi Region

![29-multi-region.png](images%2F29-multi-region.png)

- 클라우드에서 다중 리전을 사용하는 경우에도 재해 복구 전략을 만들 수 있다.
- 하나의 리전에 Aurora Global 마스터 DB가 있고 다른 리전에 Aurora Global Slave DB가 있으며 데이터는 지속적으로 복제되고 있다.
- Route53은 두 리전으로 트래픽을 전달하고 있기 때문에 하나의 리전에 재해가 발생하더라도 사용자의 트래픽을 처리할 수 있다.

#### Tips

- Backup
  - EBS 스냅샷, RDS 자동화된 백업 및 스냅샷 등..
  - S3 / S3 IA / Glacier, 라이프사이클 정책, 지역 간 복제에 대한 정기적인 추진
  - 온프레미스 데이터를 스노우볼 또는 스토리지 게이트웨이를 통해 클라우드로 복제
- High Availability
  - Route53을 사용하여 DNS를 리전에서 리전으로 마이그레이션
  - RDS Multi-AZ, ElastiCache Multi-AZ, EFS, S3
  - 기본적으로 Direct Connect를 사용하는 경우 재해 복구 용도로 Site to Site VPN을 사용할 수 있다.
- Replication
  - RDS 복제(지역 간), Aurora + Global Database
  - 온프레미스에서 RDS로 데이터베이스 복제
  - 스토리지 게이트웨이
- Automation
  - 완전히 새로운 환경을 재현하기 위한 CloudFormation / Elastic Beanstalk
  - 알람이 실패하는 경우 CloudWatch를 사용하여 EC2 인스턴스 복구 및 재부팅
  - 맞춤형 자동화를 위한 AWS 람다 함수
- Chaos
  - 넷플릭스는 EC2를 무작위로 종료하는 "simian-army"를 가지고 있다.

---

### AWS Fault Injection Simulator (FIS)

- AWS 워크로드에서 오류 지입 실험을 실행하기 위해 사용되는 완전 관리형 서비스다.
- 카오스 엔지니어링 기반으로 CPU 또는 메모리의 급격한 증가와 같은 중단 이벤트를 생성하고 시스템의 반응을 관찰하며 개선 사항을 구현하여 애플리케이션을 강화한다.
- 숨겨진 버그와 성능 병목 현상을 찾아내는데 도움이 된다.
- EC2, ECS, EKS, RDS 등 다양한 AWS 서비스를 지원한다.
- 원하는 중단을 생성하는 미리 구축된 템플릿을 사용할 수 있다.

![30-fault-injection-simulator.png](images%2F30-fault-injection-simulator.png)

- FIS가 있고 미리 구축된 템플릿을 통해서 리소스에 장애를 발생시킨다.
- 장애를 발생시키면 앱이 어떻게 작동하는지 확인할 수 있다. 
  - CloudWatch 알람이 발생하는지 EventBridge, X-Ray 등이 예상한대로 작동하는지 확인할 수 있다.
- 성능, 관찰력, 회복력에 문제가 있었는지 애플리케이션을 개선해서 병목 지점을 개선할 수 있는지 확인할 수 있다.

---

### AWS Application Discovery Service

- 온프레미스 데이터 센터에 대한 정보를 수집하여 마이그레이션 프로젝트 계획을 생성한다.
- 마이그레이션을 위해 서버 활용률 데이터 및 종속성 매핑이 중요하다.
- **Agentless Discovery (AWS Agentless Discovery Connector)**
  - VMware 호스트에 배포할 수 있는 OVA(Open Virtual Appliance) 패키지다.
  - CPU, 메모리 및 디스크 사용량과 같은 VM 인벤토리, 구성 및 성능을 기록한다.
  - 특정 OS에 종속되지 않는다.
- **Agent-based Discovery (AWS Application Discovery Agent)**
  - 시스템 구성, 시스템 성능, 실행 프로세스 및 시스템 간 네트워크 연결 세부 정보를 파악한다.
  - Microsoft Server, Amazon Linux, Ubuntu, RedHat, CentOS, SUSE 등을 지원한다.
- 분석 결과 데이터를 CSV로 내보내거나 AWS Migration Hub에서 확인할 수 있다.
- Amazon Athena에서 사전 정의된 쿼리를 사용하여 탐색할 수 있다.

#### Migration Hub Data Exploration

- 검색 시 Athena를 사용하여 온프레미스 서버에서 수집된 데이터를 분석할 수 있다.
- 데이터는 S3 버킷에 정기적인 간격으로 자동으로 저장된다.
- Athena에서 사전 정의 또는 사용자 지정 쿼리를 사용하여 데이터를 분석한다.
- 예를 들어, 각 서버에서 실행되는 프로세스의 유형을 확인할 수 있다.
- CMDB(Configuration Management Database) 내보내기와 같은 추가 데이터 원본 업로드 기능을 제공한다.
- Athena와 QuickSight를 통합하여 데이터를 시각화할 수 있다.

![31-migration-hub-data-exploration.png](images%2F31-migration-hub-data-exploration.png)

#### Application Migration Service (MGN)

- AWS로 애플리케이션 마이그레이션을 단순화하는 Lift & Shift(재호스트) 솔루션이다.
- 물리적, 가상 및 클라우드 기반 서버를 AWS에서 기본적으로 실행할 수 잇도록 변환한다.
- 광범위한 플랫폼, 운영 체제 및 데이터베이스를 지원한다.
- 다운타임을 최소화하고 비용을 절감할 수 있다.

![32-application-migration-service.png](images%2F32-application-migration-service.png)

#### Elastic Disaster Recovery

- 물리적, 가상 및 클라우드 기반 서버를 쉽고 빠르게 AWS로 복구할 수 있다.
- 예를 들어, 가장 중요한 데이터베이스(오라클, MySQL, SQL Server 등), 엔터프라이즈 애플리케이션(SAP), 랜섬웨어 공격으로부터 데이터 보호 등을 할 수 있다.
- 서버에 대한 지속적인 블록 수준의 복제를 한다.

![33-elastic-disaster-recovery.png](images%2F33-elastic-disaster-recovery.png)

#### On-Premise Strategy

- **Ability to download Amazon Linux 2 AMI as a VM (`.iso` 형식)**
  - VMware, KVM, VirtualBox, Microsoft Hyper-V
- **AWS Application Discovery Service**
  - 마이그레이션을 계획하기 위해 온프레미스 서버에 대한 정보를 수집한다.
  - 서버 활용률 및 종속성을 매핑한다.
  - AWS Migration Hub로 데이터를 전송한다.
- **AWS Application Migration Service(MGN)**
  - AWS 서버 마이그레이션 서비스 및 클라우드 지속 마이그레이션을 교체한다.
  - 온프레미스 서버를 AWS로 점진적으로 복제한다.
  - 전체 VM을 AWS로 마이그레이션한다.
- **AWS Elastic Disaster Recovery(DRS)**
  - "CloudEndure Disaster Recovery"를 대체하는 서비스다.
  - 온프레미스 워크로드를 AWS로 복구한다.
- **AWS Database Migration Service(DMS)**
  - 온프레미스 -> AWS, AWS -> AWS, AWS -> 온프레미스 복제를 지원한다.
  - 다양한 데이터베이스 엔진을 지원한다. (Oracle, MySQL, DynamoDB 등..)

---

### AWS Migration Evaluator

- AWS로의 마이그레이션을 위한 데이터 기반 비즈니스 사례 구축을 지원한다.
- 현재 조직에서 실행 중인 작업에 대한 명확한 기준을 제공한다.
- 광범위한 기반 검색을 수행할 에이전트 없는 컬렉터를 설치한다.
- 온프레미스 풋프린트, 서버 종속성에 대한 스냅샷을 생성한다.
- 현재 상태를 분석하고 대상 상태를 정의한 다음 마이그레이션 계획을 수립한다.

![34-migration-evaluator.png](images%2F34-migration-evaluator.png)

- 컬렉터를 설치하면 데이터를 수집하거나 데이터를 가져올 수 있다.
- 데이터 소스는 Migration Evaluator 서비스를 실행하게 하여 빠른 통찰력을 제공하고 비용에 대한 통찰력을 제공한다.
- 필요한 경우 Business Case를 사용하여 전문적으로 지도받을 수 있다.

---

### AWS Backup

- 완전관리형 서비스다.
- AWS 서비스 전반에 걸쳐 백업을 중앙 집중식으로 관리하고 자동화한다.
- 맞춤형 스크립트 및 수동 프로세스를 생성할 필요가 없다.
- 지원되는 여러 AWS 서비스가 있다.
  - EC2, EBS, S3, RDS, Aurora, DynamoDB, DocumentDB, Neptune, EFS, FSx, Storage Gateway(Volume Gateway)
- 교차 리전 백업을 지원한다.
- 교차 계정 백업을 지원한다.
- 지원되는 서비스들에 대해서 PITR을 지원한다.
- 온디멘드와 예약된 백업을 제공한다.
- 태그 기반의 백업 정책을 생성할 수 있다.
- 백업 정책으로 알려져 있는 백업 플랜을 생성할 수 있다.
  - 백업 주기 (일, 주간, 월간, Cron 표현식)
  - 백업 창
  - Cold Storage로 이동 (일, 몇주, 몇 개월, 몇 년)
  - 보존 기간 (항상, 일, 주, 월, 년)

![35-aws-backup.png](images%2F35-aws-backup.png)

#### Backup Vault Lock

- AWS Backup Vault에 저장한 모든 백업에 WORM(Write Once Read Many) 상태를 적용한다.
- 백업을 보호하기 위한 추가적인 방어 계층을 제공한다.
  - 의도하지 않거나 악의적은 삭제 작업으로부터 보호한다.
  - 보존 기간을 단축하거나 변경하는 업데이트 작업으로부터 보호한다.
- 활성화된 경우에는 루트 사용자도 백업을 삭제할 수 없다.

![36-vault-lock.png](images%2F36-vault-lock.png)

---

### 참고한 강의

- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)