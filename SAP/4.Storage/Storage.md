# Storage

- 이번 장에서는 **Solutions Architect Professional (SAP)** 을 준비하며 "스토리지"에 대해서 알아보도록 한다.

---

### EBS

- EBS 볼륨은 네트워크 드라이브로 한 번에 하나의 인스턴스에만 연결될 수 있다.
  - EBS 멀티태스킹 기능은 제외된다.
- 특정 AZ에 연결되어 있고 다른 AZ들 사이에서 전송하고 싶다면 먼저 스냅샷을 생성하고 다른 AZ에서 복원해야 한다.
- 볼륨을 늘릴 수는 있지만 줄일 수는 없다.
- EBS 볼륨을 최대한 활용해서 EBS 최적화를 통해 최대한의 처리량을 얻을 수 있는 쉬운 인스턴스 유형을 선택해야 한다.

![1-ebs.png](images%2F1-ebs.png)

- 두 개의 EC2 인스턴스가 있고 좌측의 인스턴스는 10GB의 EBS 볼륨을 가지고 있다.
- 우측의 인스턴스는 100GB와 50GB 두 개의 EBS 볼륨이 연결되어 있다.

#### 볼륨 유형

- EBS 볼륨은 6가지 유형으로 제공된다.
  - gp2 / gp3 (SSD): 다양한 워크로드에 대해 가격과 성능의 균형을 유지하는 범용 SSD 볼륨이다.
  - io1 / io2 (SSD) / io2 Block Express: 미션 크리티컬 저지연 또는 고처리량 워크로드를 위한 최고 성능의 SSD 볼륨이다.
  - st1 (HDD): 자주 액세스하고 처리량이 많은 워크로드를 위해 설계된 저비용 HDD 볼륨이다.
  - sc1 (HDD): 적은 빈도로 액세스하는 워크로드를 위해 설계되 ㄴ최저 비용의 HDD 볼륨이다.
- EBS 볼륨의 특징은 Size, Throughput, IOPS(I/O Ops Per Sec)가 특징이 된다.
- 부팅 볼륨 측면에서 부팅 볼륨은 EC2 인스턴스를 위한 운영체제가 있는 곳이다.
- 부팅 볼륨은 gp2, gp3, io1, io2 중에서 선택되어야 한다.

#### 스냅샷

- 증분 방식으로 변경된 블록만 백업된다.
- EBS 백업은 IO를 사용하므로 애플리케이션이 많은 트래픽을 처리하는 동안에는 실행하면 안된다.
- 스냅샷은 S3에 저장되지만 사용자가 직접 볼 수는 없다.
- 스냅샷을 수행할 볼륨을 분리할 필요는 없지만 권장된다.
- 여러 리전(DR용)에 걸쳐 스냅샷을 복사할 수 있다.
- 스냅샷에서 AMI(이미지) 생성이 가능하다.
- 스냅샷으로 복원된 EBS 볼륨을 미리 예열해야 한다.
  - Fast Snapshot Restore FSR 기능 또는 `fio`/`dd` 명령을 사용하여 전체 볼륨을 읽는다.

#### Amazon Data Lifecycle Manager

- EBS 스냅샷 및 EBS 지원 AMI의 생성, 보존 및 삭제를 자동화한다.
- 백업 예약, 계정 간 스냅샷 복사본, 오래된 백업 삭제 등의 작업이 자동화된다.
- 리소스 태그를 사용하여 리소스를 식별한다. (EC2 인스턴스, EBS 볼륨)
- Data Lifecycle Manager 외부에서 생성된 스냅샷/AMI를 관리하는 데 사용할 수 없다.
- 인스턴스 스토어에서 지원되는 AMI를 관리하는 데 사용할 수 없다.

![2-data-lifecycle-manager.png](images%2F2-data-lifecycle-manager.png)

- `Environment:Prod`로 EBS 볼륨을 태그해서 Amazon Data Lifecycle Manager가 자동으로 백업하도록 할 수 있다.
- EC2 인스턴스의 태그에 `Environment:Prod`를 추가하여 인스턴스 자체에서 EBS 볼륨을 백업하도록 할 수 있다.

#### Amazon Data Lifecycle Manager vs AWS Backup

- **Data Lifecycle Manager 사용**
  - EBS 스냅샷의 작성, 보존 및 삭제를 자동화하려는 경우에 사용된다.
- **AWS Backup 사용**
  - EBS 볼륨을 포함하여 사용하는 AWS 서비스 전반에 걸쳐 단일 장소에서 백업을 관리하고 모니터링한다.

#### EBS 암호화 - 계정 수준 설정

- 기본적으로 새로운 Amazon EBS 볼륨은 암호화되지 않는다.
- 자동으로 새로운 EBS 볼륨과 스냅샷을 암호화하는 계정 수준의 설정이 있다.
- 이 설정은 리전별로 사용하도록 설정해야 한다.

![3-ebs-encryption-account-level-setting.png](images%2F3-ebs-encryption-account-level-setting.png)

#### EBS Multi-Attach - io1/io2 family

- 동일한 AZ에 있는 여러 EC2 인스턴스에 동일한 EBS 볼륨을 연결한다.
- 각 인스턴스에는 볼륨에 대한 전체 읽기 및 쓰기 권한이 있다.
- 대표적인 사용 사례
  - Clustered Linux 애플리케이션에서 더 높은 애플리케이션 가용성을 달성한다. (예: Teradata)
  - 애플리케이션은 동시 쓰기 작업을 관리해야 한다.
- 클러스터 인식(cluster-aware) 파일 시스템을 사용해야 한다. (XFS, EXT4 등은 지원 X)

![4-ebs-multi-attach-io1io2-family.png](images%2F4-ebs-multi-attach-io1io2-family.png)

#### Local EC2 Instance Store

- EC2가 있는 물리적 서버에 연결된 물리적 디스크다.
- 물리적으로 연결되어 있기 때문에 매우 높은 IOPS를 제공한다.
- 최대 7.5TiB의 디스크 용량과 병렬도 나뉘어 있는 경우 최대 60TiB의 용량을 제공한다.
  - 수치는 시간이 지남에 따라 변경될 수 있다.
- EBS와 동일한 블록 스토리지다.
- 크기를 늘릴 수 없다.
- 하드웨어에 장애가 발생할 경우 데이터 손실의 위험이 있다.

![5-local-ec2-instance-store.png](images%2F5-local-ec2-instance-store.png)

#### Instance Store vs EBS

- 인스턴스는 저장소가 시스템에 물리적으로 연결된다. (일시적 스토리지)
- EBS는 네트워크 드라이브다. (지속적 스토리지) 
- 장점
  - I/O 성능 향상 (EBS gp2는 최대 IOPS는 16,000, io1은 64,000, io2 블록 익스프레스는 256,000)
  - 버퍼/캐시/스크래치 데이터/임시 컨텐츠에 적합하다.
  - 재부팅 시에도 데이터가 지속된다.
- 단점
  - 중지 또는 종료 시 인스턴스 스토어가 손실된다.
  - 인스턴스 저장소의 크기를 조정할 수 없다.
  - 사용자가 백업 작업을 수행해야 한다.

---

### EFS - Elastic File System

- 다수의 EC2에 마운트할 수 있는 관리형 NFS(Network File System)다.
- EFS는 다중 AZ & On-Premise(EX or VPN)의 EC2 인스턴스와 함께 작동한다.
- 고가용성, 확장성을 제공하지만 gp2의 3배 정도의 비싼 비용을 지불해야 한다.
  - 사용된 GB당 비용을 지불한다.

![6-elastic-file-system.png](images%2F6-elastic-file-system.png)

- 컨텐츠 관리, 웹 서비스, 데이터 공유, WordPress와 같은 서비스에서 사용된다.
- Linux 기반 AMI(Windows X)와 호환되고, POSIX와 호환된다.
- NFSv4.1 프로토콜을 사용한다.
- 보안 그룹을 사용하여 EFS에 대한 액세스를 제어할 수 있다.
- KMS를 이용하여 저장할 때 암호화한다.
- 하나의 VPC에만 연결할 수 있으며, AZ당 하나의 ENI(마운트 대상)를 생성할 수 있다.
- 표준 파일 API를 가진 POSIX 파일 시스템이다. (Linux)
- 파일 시스템은 용량 계획 없이 자동으로 확장된다.

#### Performance & Storage Classes

- **EFS Scale**
  - 동시 NFS 클라이언트 1,000개, 처리량 10GB 이상을 지원한다.
  - 자동으로 페타바이트 규모의 네트워크 파일 시스템으로 확장한다.
- **Performance Mode (EFS가 생성될 때 설정)**
  - **General Purpose (기본값)**: 지연 시간에 민감한 사용 사례에 사용된다. (웹 서버, CMS 등)
  - **Max I/O**: 지연 시간, 처리량, 높은 병렬 처리가 필요할 때 사용된다. (빅데이터, 미디어 처리 등)
- **Throughput Mode**
  - **Bursting**: 1TB = 50MiB/s + 최대 100MiB/s 버스트
  - **Provisioned**: 스토리지 크기에 관계없이 처리량을 설정한다. (예: 1TB 스토리지의 경우 1Gb/s)
  - **Elastic**: 워크로드에 따라 자동으로 처리량 증가 또는 감소
    - 읽기의 경우 최대 3Gb/s, 쓰기의 경우 최대 1Gb/s
    - 예측 불가능한 워크로드에 사용된다.

#### Storage Classes

- **Storage Tiers** (라이프사이클 관리 기능 - N일 후 파일 이동)
  - Standard: 자주 액세스하지 않는 파일의 경우
  - Infrequent access (EFS-IA): 파일 검색 비용, 저장 비용을 절감한다. 라이프사이클 정책을 통해 EFS-IA를 활성화한다.
- **Availability and Durability**
  - Regional: 다중 AZ, 운영 환경에 적합하다.
  - One Zone: 단일 AZ, 개발 환경에 적합하며 기본적으로 백업이 가능하며 IA(EFS One Zone-IA)와 호환된다.

![7-efs-storage-classes.png](images%2F7-efs-storage-classes.png)

- 파일을 모니터링하고 만약 60간 액세스가 없었다면 IA 계층으로 이동된다.

#### On-Premises & VPC Peering

![8-onpremise-vpc-peering.png](images%2F8-onpremise-vpc-peering.png)

- EFS와 ENI가 있고 여러 AZ에 걸쳐 중복된다.
- 다른 VPC의 EC2 인스턴스를 위해 VPC Peering이 되어있다.
- On-Premise 서버와는 DX 또는 VPN을 통해서 연결되어 있다.
- On-Premise 서버에서 EFS 드라이브를 마운트해야 할 때, ENI의 IPv4를 이용해 마운트해야 한다.
  - DNS는 사용할 수 없다.
  - 반면 EC2 인스턴스는 DNS를 사용할 수 있다.
- On-Premise 서버에서 ENI IP를 회수하여 NFS를 설치하는 데 사용해야 한다.

#### Access Point

- NFS 환경에 대한 애플리케이션 액세스를 쉽게 관리할 수 있다.
- 파일 시스템에 액세스할 때 POSIX 사용자 및 그룹을 사용하도록 강제한다.
- 파일 시스템 내의 디렉토리에 대한 액세스를 제한하고 선택적으로 다른 루트 디렉토리를 지정한다.
- IAM 정책을 사용하여 NFS 클라이언트의 액세스를 제한할 수 있다.

![9-efs-access-point.png](images%2F9-efs-access-point.png)

- EFS 파일 시스템이 있고 `/data`, `/secret`, `/config` 경로가 있다.
- UID 1001과 GID 1001로 액세스 포인트 1을 설정하고 경로는 `/config`로 설정한다.
  - 개발자 사용자 그룹의 사용자들에게 루트 경로(`/`)는 실제로 `/config`가 된다.
- UID 1002와 GID 1002로 액세스 포인트 2를 설정하고 경로는 `/data`로 설정한다.
  - 분석 사용자 그룹의 사용자들에게 루트 경로(`/`)는 실제로 `/data`가 된다.

#### File System Policy

- EFS 파일 시스템에 대한 액세스를 제어하는 리소스 기반 정책으로 S3 버킷 정책과 동일하다.
- 기본적으로 모든 클라이언트에 대한 전체 액세스 권한을 부여한다.

![10-file-system-policy.png](images%2F10-file-system-policy.png)

- Stephane 사용자는 Mount와 Write를 할 수 있으며 전송 중 암호화를 활성화해야 한다.

#### Cross-Region Replication

- EFS 파일 시스템의 객체를 다른 AWS 리전으로 복제한다.
- 새 EFS 파일 시스템 또는 기존 EFS 파일 시스템에 대한 설정을 변경해야 한다.
- 분 단위의 RPO 및 RTO를 제공한다.
- EFS 파일 시스템의 프로비저닝된 처리량에 영향을 주지 않는다.
- 컴플라이언스 및 비즈니스 연속성 목표 달성을 위해 사용된다.

![11-cross-region-replication.png](images%2F11-cross-region-replication.png)

---

### S3 - Simple Storage Service

- 객체 스토리지, 서버리스, 무제한 스토리지, 종량제
- 정적 컨텐츠(이미지, 비디오 파일)를 저장하기에 좋다.
- 인덱싱 시설없이 키로 객체에 엑세스한다.
- 파일 시스템이 아니므로 EC2에 기본적으로 마운트할 수 없다.
- 안티 패턴
  - 소량의 많은 파일들을 저장
  - POSIX 파일 시스템, 파일 잠금
  - 기능, 쿼리, 급변하는 데이터 검색
  - 동적 컨텐츠가 포함된 웹 사이트

#### Storage Classes Comparison

![12-storage-classes-comparison.png](images%2F12-storage-classes-comparison.png)

- S3 라이프사이클 정책을 사용하여 계층 간에 객체를 전환 또는 삭제할 수 있다.
- 스토리지 클래스에 대한 자세한 내용은 [공식문서](https://aws.amazon.com/s3/storage-classes/)를 확인한다.

#### Replication (Versioning Enabled)

- Cross Region Replication (CRR)을 지원한다.
- Same Region Replication (SRR)을 지원한다.
- 라이프사이클 규칙과 결합할 수 있다.
- 대기 시간 단축, 재해 복구, 보안에 도움을 줄 수 있다.
- **S3 Replication Time Control (S3 RTC)**
  - Amazon S3에 업로드하는 대부분의 객체를 몇 초만에 복제하고, 해당 객체를 99.99%를 15분 이내에 복제한다.
  - 컴플라이언스, DR 등에 도움이 된다.

![13-replication-versioning-enabled.png](images%2F13-replication-versioning-enabled.png)

#### Event Notifications

- `S3:ObjectCreated`, `S3:ObjectRemoved`, `S3:ObjectRestore`, `S3:Replication` 등의 작업이 있다.
- 객체 이름으로 필터링이 가능하다. (예: `*.jpb`)
- S3에 업로드된 이미지의 썸네일 생성과 같은 작업에 사용될 수 있다.
- 원하는 만큼의 "S3 이벤트"를 만들 수 있다.
- S3 에빈트 알림은 일반적으로 몇 초 안에 이벤트를 전달하지만 때로는 1분 이상 걸릴 수 있다.

![14-event-notifications.png](images%2F14-event-notifications.png)

- S3 이벤트는 S3에서 발생하며 SNS, SQS, 람다 함수를 작동시킬 수 있다.

![15-event-notification-amazon-eventbridge.png](images%2F15-event-notification-amazon-eventbridge.png)

- S3 이벤트 알림은 EventBridge와 통합될 수 있다.
- JSON 규칙을 사용한 고급 필터링 옵션 (메타데이터, 객체 크기, 이름 등..)을 제공한다.
- Step Function, Kinesis Streams, Firehose와 같은 많은 종류의 목적지를 지원한다.
- EventBridge Capabilities: 아카이브, 이벤트 재실행, 신뢰할 수 있는 제공과 같은 기능을 제공한다.

#### Baseline Performance

- Amazon S3는 높은 요청 속도, 지연 시간 100-200ms로 자동 확장된다.
- 애플리케이션은 버킷의 접두사당 초당 최대 3,500개의 PUT/COPY/POST/DELETE 요청을 지원하고, 5,500개의 GET/HEAD 요청을 지원한다.
- 버킷의 접두사 수에는 제한이 없다.
- 예시 (객체 경로 -> 접두사)
  - `bucket/folder1/sub1/file` -> `/folder1/sub1/`
  - `bucket/folder1/sub2/file` -> `/folder1/sub2/`
  - `bucket/1/file` -> `/1/`
  - `bucket/2/file` -> `/2/`
- 4개의 접두사 모두에 읽기를 균등하게 적용하면 GET 및 HEAD에 대한 초당 22,000건의 요청을 달성할 수 있다.

#### Performance

- **Multi-Part upload**
  - 100MB가 넘는 파일에는 사용이 권장되며, 5GB가 넘는 파일에는 반드시 사용해야 한다.
  - 업로드 병렬화(전송 속도 향상)에 도움이 될 수 있다.

![16-multi-part-upload.png](images%2F16-multi-part-upload.png)

- **S3 Transfer Acceleration**
  - 대상 리전의 S3 버킷에 데이터를 전달할 AWS 엣지 로케이션으로 파일을 전송하여 전송 속도를 향상한다.
  - Multi-Part 업로드와 동시에 사용될 수 있다.

![17-s3-transfer-acceleration.png](images%2F17-s3-transfer-acceleration.png)

#### S3 Byte-Range Fetches

- 특정 바이트 범위를 요청하여 GET 요청을 병렬화한다.
- 장애 발생 시 복원력이 향상된다.
- 다운로드 속도를 높이는 데 사용할 수 있다.

![18-byte-range-fetches.png](images%2F18-byte-range-fetches.png)

- 일부 데이터(예: 파일의 헤드)만 검색하는 데 사용할 수 있다.

![19-byte-range-fetches.png](images%2F19-byte-range-fetches.png)

#### Multi-Part Upload - Remove Incomplete Parts

![20-remove-incomplete-parts.png](images%2F20-remove-incomplete-parts.png)

- 용량이 큰 파일이 있을 때 Multi-Part 업로드 기능을 이용해 부분별로 S3 버킷에 업로드할 수 있다.
- 따라서 S3 버킷에 병렬로 파일이 업로드될 때 경우에 따라 일부 파일이 업로드되지 않을 수 있다.
- S3 수명 주기 정책을 설정하여 병렬 업로드에 문제가 있고 7일동안 완료되지 않은 경우 업로드된 파일을 삭제하도록 구성할 수 있다.
- 또한 CLI API호출로 목록을 만들고 Multi-Part 업로드를 할 수도 있다.

#### S3 Select & Glacier Select

- 서버 측 필터링을 수행하여 SQL을 사용하여 데이터를 검색한다.
- 단순 SQL문을 사용하여 행과 열을 기준으로 필터링할 수 있다.
- 네트워크 전송이 감소하고, 클라이언트 사이드의 CPU 비용이 절감된다.

![21-s3-select-glacier-select.png](images%2F21-s3-select-glacier-select.png)

- S3 SELECT를 사용한 후에는 S3 SELECT가 스스로 파일을 필터링하게 해서 400% 빠른 성능을 제공하고 80%까지 저렴하게 사용할 수 있게 해준다.

#### S3 Analytics - Storage Class Analysis

- 시험에서 "Storage Class Analysis"로 표시될 수 있다.
- 객체를 올바른 스토리지 클래스로 전환할 시기를 결정하는 데 도움이 된다.
- Standard 및 Standard IA에 대한 전환 권장 사항을 제공한다.
  - One-Zone IA나 Glacier에서는 지원하지 않는다.
- 보고서가 매일 업데이트된다.
- 데이터 분석을 시작하는 데 24 ~ 48시간이 소요된다.
- Amazon QuickSight에서 데이터를 시각화한다.
- 라이프사이클 규칙을 구성(또는 개선)하기 위한 좋은 첫 단계다.

![22-s3-analytics-storage-class-analysis.png](images%2F22-s3-analytics-storage-class-analysis.png)

- S3 버킷이 있고 그 위에 "Storage Class Analysis"가 있다.
- 잠시 후에 `.csv` 형식의 리포트가 생성되어 어떤 스토리지 클래스가 있는지 어떤 객체에 대한 통찰력을 제공하고 객체의 나이도 제공한다.

---

### S3 - Storage Lens

- 전체 AWS Organizations의 스토리지를 이해하고 분석 및 최적화하는 데 사용된다.
- 이상 징후 발견, 비용 효율성 파악, AWS Organizations 전체에 걸쳐 데이터 보호 모범 사례를 적용한다.
  - 30일 사용량 및 활동 메트릭을 분석한다.
- 조직, 특정 계정, 영역, 버킷 또는 접두사에 대한 데이터를 집계한다.
- 기본 대시보드 또는 자체 대시보드를 생성한다.
- 매일 메트릭을 S3 버킷(CSV, Parquet)으로 보내도록 구성할 수 있다.

![23-s3-storage-lens.png](images%2F23-s3-storage-lens.png)

- Storage Lens는 "Organization", "Account", "Region", "Bucket" 모든 것을 계정에 담는다.
- 모든 데이터를 리포트로 집계하여 분석한다.
- 마지막으로 요약하거나, 데이터 보호 및 비용 최적화를 할 수 있다.

#### Default Dashboard

- 무료 지표와 고급 지표 모두에 대한 요약된 통찰력과 동향을 시각화한다.
- 기본 대시보드에 다중 리전 및 다중 계정 데이터를 표시한다.
- Amazon S3에 의해 사전 구성된다.
- 삭제할 수는 없지만 사용하지 않도록 설정할 수 있다.

![24-default-dashboard.png](images%2F24-default-dashboard.png)

#### 무료 지표 vs 유료 지표

- Storage Lens에는 무료(Free) 지표와 유료(Paid) 지표가 있다.
- **Free Metrics**
  - 모든 고객이 자동으로 사용 가능하다.
  - 약 28개의 사용 메트릭을 포함한다.
  - 14일 동안 조회 가능한 데이터를 제공한다.
- **Advanced Metrics and Recommendations**
  - 추가 유료 메트릭 및 기능을 제공한다.
  - 고급 메트릭: 활동, 고급 비용 최적화, 고급 데이터 보호, 상태 코드
  - CloudWatch Publishing: 추가 비용 없이 CLoudWatch에서 메트릭 액세스를 한다.
  - Prefix Aggregation: Prefix 수준에서 메트릭을 수집한다.
  - 15개월 동안 조회할 수 있는 데이터를 제공한다.

![25-free-paid.png](images%2F25-free-paid.png)

---

### S3 Solution Architecture

#### Exposing Static Objects

![26-exposing-static-objects.png](images%2F26-exposing-static-objects.png)

- Case1
  - 공개적으로 액세스 가능한 EC2 인스턴스를 생성한다.
  - 저렴한 비용으로 구축할 수 있지만 확장이 불가능하고 가용성이 낮다.
- Case2
  - EC2 인스턴스 앞에 CloudFront를 추가한다.
  - EC2 인스턴스가 EBS 스토리지를 사용하도록 구성한다.
  - CloudFront를 통한 캐시를 사용하여 고객은 더 빠르게 응답받을 수 있고 EC2 인스턴스의 부하는 줄어든다.
  - 하지만 EC2 인스턴스에 장애가 발생하는 경우 서비스는 단절된다.
- Case3
  - CloudFront 뒤에 ALB를 추가하고 ALB의 대상 그룹으로 ASG를 추가한다.
  - ASG의 인스턴스들이 동시에 사용할 수 있는 EFS를 추가한다.
  - 가용성이 높고 확장 가능하지만 많은 비용이 발생한다.
- Case4
  - S3 앞에 CloudFront를 사용한다.
  - S3에 정적 웹사이트가 있는 경우에 사용할 수 있다.

#### Indexing object in DynamoDB

![27-indexing-object-dynamodb.png](images%2F27-indexing-object-dynamodb.png)

- S3에는 색인 설비가 없기 때문에 S3 버킷을 검색하여 객체를 찾을 수 없다.
- 올바른 방법은 DynamoDB의 인덱스 객체를 이용하는 것이다.
- S3 이벤트 알림을 통해 람다 함수가 트리거된다.
  - 트리거된 람다 함수는 파일의 메타데이터 정보를 DynamoDB 테이블에 삽입한다.
  - 추후 API를 생성해 객체의 메타데이터를 검색할 수 있다.
  - 예를 들어, 올바른 인덱스가 있다면 날짜별로 검색할 수 잇고, 고객이 사용하는 전체 검색도 볼 수 있다.
  - 특성을 가진 모든 객체를 나열할 수 있고 날짜 범위 내에서 업로드된 모든 객체를 찾을 수 있다.

#### Dynamic vs Static Content

![28-dynamic-static-content.png](images%2F28-dynamic-static-content.png)

- Route53에 DNS 요청을 하면 동적 콘텐츠를 위해 리디렉션될 수 있다.
  - 예를 들어, REST API나 HTTP 서버가 있다.
  - 예시에서는 ALB, EC2, API 게이트웨이, 람다 함수다.
  - DynamoDB와 데이터베이스 레이어에서 동적 데이터를 얻을 수 있다.
  - Caching/Session 레이어도 사용이 가능하다.
- 사용자가 DNS 쿼리를 했을 때 만약 정적 컨텐츠를 요청했다면 CDN 레이어로 요청이 전달된다.
  - CloudFront에서는 정적 컨텐츠를 조회하기 위해 S3 버킷에 접근한다.
  - 정적 파일을 엣지 로케이션에 위치시켜 엄청나게 비용을 절감하고 성능을 향상시킬 수 있다.
- 만약 OAI가 설정되지 않았다면 CloudFront를 거치지 않고 S3 버킷에 접근할 수 있다.
  - 이때는 Pre-signed URL을 사용하여 접근하게 된다.

---

### Amazon FSx

- AWS에서 실행되는 3rd Party 고성능 파일 시스템이다.
- AWS에서 완전하게 관리해준다.
  - FSx for Lustre
  - FSx for Windows File Server
  - FSx for NetApp ONTAP
  - FSx for OpenZFS

#### Amazon FSx for Windows (File Server)

- Windows 용 FSx는 완벽휴ㅏ게 관리되는 Windows 파일 시스템 공유 드라이브다.
- SMB 프로토콜 및 Windows NTFS를 지원한다.
- Microsoft Active Directory 통합, ACL, 사용자 할당량을 지원한다.
- Linux EC2 인스턴스에 마운트할 수 있다.
- 마이크로소프트의 DFS(Distributed File System) 네임스페이스(여러 FS에 걸쳐 그룹 파일)를 지원한다.
- 최대 초당 10GB/s, 수백만 IOPS, 100초 PB 데이터까지 확장 가능하다.
- Storage Option
  - SSD: 지연 시간에 민감한 워크로드(데이터베이스, 미디어 처리, 데이터 분석 등..)
  - HDD: 광범위한 워크로드(홈 디렉토리, CMS 등..)
- 사내 인프라(VPN 또는 Direct Connect)에서 액세스 가능하다.
- Multi-AZ로 구성하여 가용성을 높일 수 있다.
- 데이터는 매일 S3로 백업된다.

#### Amazon FSx for Lustre

- Lustre는 대규모 컴퓨팅을 위한 병렬 분산 파일 시스템의 한 종류다.
- Lustre라는 이름은 "Linux"와 "Cluster"에서 유래되었다.
- 머신 러닝, 고성능 컴퓨팅(HPC)등의 작업에 사용된다.
- 비디오 처리, 재무 모델링, 전자 설계 자동화등의 작업에 사용된다.
- 최대 100초 GB/s, 수백만 IOPS, ms 미만의 지연 시간까지 확장 가능하다.
- Storage Option
  - SSD: 저지연, IOPS 집약적인 워크로드, 소규모 및 랜덤 파일 작업등에 적합하다.
  - HDD: 처리량 집약적인 워크로드, 대규모 및 순차적 파일 작업에 적합하다.
- S3와의 원할한 통합을 지원한다.
  - 파일 시스템으로 "S3" 읽기가 가능하다. (FSx를 통해)
  - 계산 결과를 S3로 다시 쓸 수 있따. (FSx를 통해)
- On-Premise 서버에서 사용할 수 있다. (VPN 또는 Direct Connect)

#### FSx Lustre - File System Deployment Options

- **Scratch File System**
  - 임시 보관 용도로 사용된다.
  - 데이터가 복제되지 않기 때문에 파일 서버에 장애가 발생하는 경우 데이터가 지속되지 않는다.
  - 높은 버스트(TiB당 200MBps, 6배 빠른 속도)
  - 사용 사례: 단기 처리, 비용 최적화
- **Persistent File System**
  - 장기간 보관 용도로 사용된다.
  - 데이터가 동일한 AZ 내에서 복제된다.
  - 몇 분 안에 실패한 파일을 교체한다.
  - 사용 사례: 장기간 처리, 민감한 데이터

![29-file-system-deployment-options.png](images%2F29-file-system-deployment-options.png)

#### Amazon FSx for NetApp ONTAP

- AWS에서 관리되는 NetAPp ONTAP이다.
- NFS, SMB, iSCSI 프로토콜과 호환되는 파일 시스템이다.
- ONTAP 또는 NAS에서 실행 중인 워크로드를 AWS로 이동한다.
- 사용 가능한 환경은 아래와 같다.
  - Linux
  - Windows
  - MacOS
  - VMware Cloud on AWS
  - Amazon Workspaces & AppStream 2.0
  - Amazon EC2, ECS, EKS
- 스토리지가 자동으로 축소되거나 확장된다.
- 스냅샷, 복제, 저비용, 압축 및 데이터 중복을 제거한다.
- 시점별 즉각적인 복제를 제공하여 새로운 워크로드 테스트에 유용하게 사용된다.

![30-fsx-for-netapp-ontap.png](images%2F30-fsx-for-netapp-ontap.png)

#### Amazon FSx for OpenZFS

- AWS에서 관리되는 OpenZFS 파일 시스템이다.
- NFS(v3, v4, v4.1, v4.2)와 호환되는 파일 시스템이다.
- ZFS에서 실행 중인 워크로드를 AWS로 이동한다.
- 사용 가능한 환경은 아래와 같다.
  - Linux
  - Windows
  - MacOS
  - VMware Cloud on AWS
  - Amazon Workspaces & AppStream 2.0
  - Amazon EC2, ECS, EKS
- 0.5ms 미만의 지연 시간으로 최대 1,000,000 IOPS를 제공한다.
- 스냅샷, 압축, 저비용이 필요한 환경에 적합하다.
- 시점별 즉각적인 복제를 제공하여 새로운 워크로드 테스트에 유용하게 사용된다.

![31-fsx-for-openzfs.png](images%2F31-fsx-for-openzfs.png)

#### Single-AZ에서 Multi-AZ로 마이그레이션

![32-migration-singleaz-multiaz.png](images%2F32-migration-singleaz-multiaz.png)

- 새로운 Multi-AZ 환경의 FSx for Windows File Server를 생성한다.
  - AWS DataSync를 사용하여 데이터를 동기화한다.
  - 이 방식의 장점은 데이터 동기화가 백그라운드에서 일어난다는 점이다.
  - 속도는 느리지만 가용성은 유지된다.
- 새로운 Multi-AZ 환경의 FSx for Windows File Server를 생성한다.
  - 기존의 파일 서버를 중단하고 백업을 생성한다.
  - 새로운 Multi-AZ 환경에서 복구한다.
  - 서비스는 중단되지만 빠르게 복제가 가능하다.

#### FSx 볼륨 사이즈 축소

- 백업을 수행하는 경우 동일한 크기로만 복원할 수 있다.
- 파일 시스템의 저장 용량은 늘릴 수만 있고, 줄일 수는 없다.
- 대신 작은 크기의 새로운 파일 서버를 생성하고 DataSync를 사용하여 데이터를 동기화한 다음 애플리케이션을 마이그레이션할 수 있다.

![33-decrease-fsx-volume-size.png](images%2F33-decrease-fsx-volume-size.png)

#### FSx for Lustre - Data Lazy Loading

- S3를 입력 데이터 소스로 사용하는 Lustre의 모든 데이터 처리 작업은 Lustre가 먼저 데이터 세트를 완전히 다운로드하지 않고도 시작할 수 있다.
- 데이터가 늦게 로드된다: 실제로 처리된 데이터만 로드되므로 비용과 대기 시간을 줄일 수 있다.
- 데이터도 한 번만 로드되므로 S3에서 요청을 줄일 수 있다.

![34-data-lazy-loading.png](images%2F34-data-lazy-loading.png)

- 클라이언트는 아주 구체적인 데이터셋을 요구한다.
- Lustre를 위한 FSx에 로드되고 클라이언트가 처리한다.

---

### AWS DataSync

- 많은 양의 데이터를 이동할 수 있다.
  - On-Premise 또는 기타 클라우드에서 AWS로 이동할 수 있으며 에이전트가 필요하다. 
  - NFS, SMB, HDFS, S3 API 등을 지원한다.
  - AWS의 스토리지 서비스에서 다른 AWS 스토리지 서비스로 이동할 때 사용되며, 에이전트가 필요하지 않다.
- 동기화 가능 대상은 아래와 같다.
  - Amazon S3 (Glacier를 포함한 모든 스토리지 클래스)
  - Amazon EFS 
  - Amazon FSx (Windows, Lustre, NetApp, OpenZFS 등..)
- 복제 작업을 시간별, 일별, 주별로 예약할 수 있다.
- **파일 사용 권한 및 메타데이터(보안 등) 보존할 수 있다.**
  - 이 말은 NFS POSIX 파일 시스템과 SMB 권한을 준수한다는 의미다.
- 에이전트 작업 하나로 10Gbps를 사용할 수 있으며, 대역폭 제한 설정이 가능하다.

#### NFS / SMB to AWS

![35-nfs-smb-aws.png](images%2F35-nfs-smb-aws.png)

- On-Premise 영역과 "AWS DataSync"가 실행되는 AWS 리전이 있다.
- On-Premise 영역에는 NFS, SMB 서버가 있고, AWS DataSync 에이전트가 설치되어 있다.
- DataSync 에이전트는 연결을 설정하고 DataSync 서비스로 암호화된 방식으로 연결한다.
  - 전송된 데이터는 S3 버킷으로 이동될 수 있고, EFS 또는 FSx에 저장될 수 있다.
- On-Premise에서 DataSync 에이전트를 통해 데이터를 전송할만큼의 대역폭을 지원하지 않을 수 있다.
  - 이러한 경우 에이전트가 미리 설치된 AWS Snowcone을 통해서 데이터 마이그레이션이 가능하다.

#### AWS 스토리지 서비스 간 데이터 전송

![36-transfer-aws-storage-service.png](images%2F36-transfer-aws-storage-service.png)

- DataSync를 사용해서 서로 다른 AWS 스토리지 서비스들을 동기화할 수도 있다.
- 데이터를 동기화할 때 메타데이터도 스토리지 서비스 사이에 동기화된다.
- DataSync는 거의 모든 것과 동기화할 수 있지만 지속적이지는 않고 예약된 작업, 시간에 발생한다.

#### Private VIF through Direct Connect

![37-private-vif-direct-connect.png](images%2F37-private-vif-direct-connect.png)

- 회사의 데이터 센터와 AWS 클라우드가 Direct Connect로 연결되어 있고 데이터 동기화를 할 때, 비공개 통신이 필요한 상황이다.
- 데이터는 VPC를 통과해야 하며, DataSync 에이전트가 Direct Connect에 연결되어 있다.
- 첫번째 옵션으로 공용 VIF를 사용하여 VPC를 통하지 않고 AWS DataSync 서비스로 이동할 수 있다.
- 두번째 옵션으로 사설 VIF를 사용하여 VPC내의 PrivateLink, 인터페이스 VPC 엔드포인트를 경유하여 AWS DataSync로 이동될 수 있다.

---

### AWS Transfer Family

- FTP 프로토콜을 사용하여 Amazon S3 또는 Amazon EFS 간에 파일 전송 및 전송을 위한 완벽하게 관리되는 서비스다.
- 지원되는 프로토콜은 아래와 같다.
  - AWS Transfer for FTP (File Transfer Protocol (FTP))
  - AWS Transfer for FTPS (File Transfer Protocol over SSL (FTPS))
  - AWS Transfer for SFTP (Secure File Transfer Protocol (SFTP))
- 관리형 인프라, 확장성, 신뢰성, 고가용성(Multi-AZ)을 지원한다.
- 프로비저닝된 엔드포인트당 시간당 비용을 지불하며, 추가로 전송된 데이터 GB당 비용을 지불해야 한다.
- 서비스 내 사용자 자격 증명을 저장하고 관리한다.
- 기존 인증 시스템 (Microsoft Active Directory, LDAP, Okta, Amazon Cognito, 사용자 정의)과 통합된다.
- 파일 공유, 공용 데이터셋, CRM, ERP 등의 작업에 사용된다.

![38-aws-transfer-familiy.png](images%2F38-aws-transfer-familiy.png)

- Transfer Family는 세 종류가 있고 FTP는 엔드포인트를 이용해 직접 접속할 수 있다.
- Route53의 호스트 이름을 등록하여 FTP 사용자들이 접속하도록 설정할 수 있다.
- FTP 서비스에 대한 IAM 역할을 가질 것이며 S3 또는 EFS로부터 파일을 보내거나 받게 된다.
- 전송 서비스를 안전하게 하고자 한다면 MS Active Directory 같은 외부 인증 시스템을 이용해 사용자를 인증할 수 있다.

#### Endpoint 유형

- **Public Endpoint**
  - AWS에서 관리하는 IP를 사용한다. 
  - IP는 시간이 지나면서 변경될 수 있으므로 DNS 이름을 사용하는 것이 권장된다.
  - IP가 변경될 수 있기 때문에 네트워크 보안으로부터 소스 IP 주소로 리스트를 필터링하는 걸 허용할 수 없다. 

![39-public-endpoint.png](images%2F39-public-endpoint.png)

- **VPC Endpoint with Internal Access**
  - SFTP는 VPC 안에 배포된다.
  - VPC 내의 EC2 인스턴스는 비밀리에 엔드포인트와 회사 데이터 센터에 액세스할 수 있다.
  - VPN이나 DX를 통해 연결되면 내부 엔드포인트에 액세스할 수 있다.
  - 엔드포인트에 접근하기 위한 고정 개인 IP를 얻을 수 있고 허용 리스트를 설정할 수 있다.

![40-vpc-endpoint-internal-access.png](images%2F40-vpc-endpoint-internal-access.png)

- **VPC Endpoint with Internet-facing Access**
  - VPC 엔드포인트라는 하이브리드 버전은 인터넷에서 접속할 수 있따.
  - 동일한 VPC 배포하고 VPC나 회사 데이터센터의 어떤 서버에서든 EC2 인스턴스를 확보해 엔드포인트에 은밀히 액세스할 수 있다.
  - 엔드포인트를 위한 고용 IP를 설정할 수 있고 IP는 Elastic IP일 수 있다.
  - 보안 그룹을 설정하여 SFTP에 접속할 수 있는 인터넷 사용자를 제한할 수 있따.

![41-vpc-endpoint-internet-facing-access.png](images%2F41-vpc-endpoint-internet-facing-access.png)

---

### 스토리지 가격 비교

![42-price-of-storage.png](images%2F42-price-of-storage.png)

- SAP 시험을 볼 때 상황에 따른 가장 저렴한 스토리지를 선택할 수 있어야한다.
- 일반적으로 S3의 비용이 가장 저렴하며, EBS는 중간이고 EFS가 가장 비싼 비용을 지불해야 한다.
  - 하지만 EBS 스토리지라 하더라도 io1, io2 유형은 다른 어떤 스토리지보다 비용이 비싸다.

---

### 참고한 강의

- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)