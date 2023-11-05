# Amazon EC2 Storage & Data Management

이번 장에서는 **SysOps Administrator**를 준비하며 **EC2 스토리지와 스토리지 관리**에 대해서 알아보도록 한다.

---

### EBS Volume

- EBS(Elastic Block Store) 볼륨은 인스턴스가 실행되는 동안 인스턴스에 연결할 수 있는 네트워크 드라이브다.
- 인스턴스가 종료된 후에도 데이터를 유지할 수 있다.(`DeleteOnTermination` 값에 따라 결정)
- 한 번에 하나의 인스턴스에만 마운트할 수 있다.(CCP 수준에서, io-시리즈의 경우 여러 인스턴스에 장착 가능)
- 특정 가용성 영역에 바인딩되어 있다.
- 네트워크 USB로 생각하면 이해하기 쉽다. 
  한 번에 하나의 머신에 장착할 수 있고 영구적이며 다른 머신에 장착 가능
- 프리 티어에서는 General Purpose(SSD) 유형의 스토리지를 30GB의 용량을 사용할 수 있다. 또는 Magnetic을 매달 무료로 사용할 수 있다.

- 네트워크 드라이브이므로 물리적 드라이브가 아니다.
  - 네트워크를 사용하여 인스턴스와 통신하므로 약간의 지연 시간이 발생할 수 있다.
  - EC2 인스턴스에서 분리하고 다른 인스턴스에 빠르게 연결할 수 있다.
- AZ(가용 영역)에 잠겨 있다.
  - `us-east-1a`의 EBS 볼륨은 `us-east-1b`에 연결할 수 없다.
  - 볼륨을 이동하려면 먼저 스냅샷을 생성해야 한다.
- 프로비저닝된 용량(GB 단위 크기 및 IOPS)이 있어야 한다.
  - 프로비저닝된 모든 용량에 대해 요금이 청구된다.
  - 시간이 지남에 따라 드라이브 용량을 늘릴 수 있다.

#### 사용 예시

![1-ebs-volume-example.png](images%2F1-ebs-volume-example.png)

- 먼저 원하는 용량을 프로버저닝하고 초당 I/O 연산인 IOPS를 설정한다.
- 프로비전 용량에 대해 고지서를 받게 되고 더 나은 성능이나 크기를 원한다면 시간에 따라 용량을 증가시킬 수 있다.
- 이미지를 보면 EBS 볼륨은 하나의 인스턴스에만 장착될 수 있지만, 하나의 인스턴스에 여러개의 EBS 볼륨이 장착될 수 있다.
- EBS 볼륨은 생성할 때 연결하지 않아도 된다.
- EBS 볼륨을 생성할 때, `DeleteOnTermination` 설정 값을 결정할 수 있고 인스턴스가 종료되었을 때, 볼륨이 삭제될 것인지 결정할 수 있다.
- EBS 볼륨을 생성할 때, 기본값은 루트 볼륨에 체크가 된다. **루트 EBS 볼륨은 인스턴스가 종료되는 것과 동시에 삭제된다.**
  루트 EBS 볼륨을 제외한 이외의 볼륨은 EC2 인스턴스가 종료되도 기본적으로 삭제되지 않는다.

---

### EC2 Instance Store

- EBS 볼륨은 양호한 성능을 제공하지만 네트워크 드라이브이므로 성능이 "제한"된다.
- **고성능 하드웨어 디스크가 필요한 경우 "EC2 Instance Store"를 사용해야 한다.**
- EBS 볼륨 대비 I/O 성능이 높다.
- EC2 Instance Store가 중지되면 스토리지가 손실된다. 임시 저장장치로 봐야한다.
- 버퍼/캐시/스크래치 데이터/임시 콘텐츠를 저장하기에 적합하다.
- 하드웨어 장애 시 데이터 손실의 위험이 있다.
- 백업 및 복제는 "EC2 Instance Store"를 사용하는 사용자의 책임이다.
- 아래는 EC2 Instance Store IOPS를 정리한 표이며, EBS 볼륨에 비해서 높은 성능을 제공하는 것을 확인할 수 있다.

![2-local-ec2-instance-store.png](images%2F2-local-ec2-instance-store.png)

- **32,000IOPS라는 수치를 기억하고 이상의 성능을 원할 때는 EBS 볼륨이 아니라 EC2 Instance Store를 사용해야 한다.

---

### EBS 볼륨 유형

- EBS 볼륨은 6가지 종류가 있다.
  - gp2 / gp3 (SSD): 다양한 워크로드에 대해 가격과 성능의 균형을 맞춘 범용 SSD 볼륨이다.
  - io1 / io2 (SSD): 미션 크리티컬한 짧은 지연 시간 또는 높은 처리량 워크로드를 위한 최고 성능의 SSD 볼륨이다.
  - st1(HDD): 자주 액세스하고 처리량 집약적인 워크로드를 위해 설계된 HDD 볼륨이다.
  - sc1(HDD): 자주 액세스하지 않는 워크로드를 위해 설계된 최저 비용의 HDD 볼륨이다.
- EBS 볼륨은 크기 | 처리량 | IOPS(초당 I/O Ops Per Sec) 특성화되어 있다.
- 확실하지 않은 경우 항상 AWS 설명서를 참고하는 것이 좋다.
- **gp2 / gp3 및 io1 / io2만 부팅 볼륨으로 사용할 수 있다.**

---

#### General Purpose SSD

- 비용 효율적인 스토리지, 짧은 지연 시간을 제공한다.
- 시스템 부팅 볼륨, 가상 데스크탑, 개발 및 테스트 환경에 적합하다.
- 1GiB ~ 16TiB 범위의 용량을 제공한다.
- gp3
  - 기본 3,000 IOPS 및 125 MiB/s의 처리량을 제공한다.
  - 독립적으로 IOPS를 최대 16,000까지 처리량을 최대 1,000MiB/s까지 늘릴 수 있다.
- gp2
  - 작은 gp2 볼륨은 IOPS를 3,000까지 버스트할 수 있다.
  - 볼륨 크기와 IOPS가 연결되어 있으며 최대 IOPS는 16,000이다.
  - "3 IOPS per GB"의 의미는 5,334GB가 최대 IOPS에 있음을 의미한다.

#### Provisioned IOPS (PIOPS) SSD

- 지속적인 IOPS 성능을 갖춘 중요한 비즈니스 애플리케이션에 사용된다.
- 16,000 IOPS 이상이 필요한 애플리케이션에 적합하다.
- **스토리지 성능 및 일관성에 민감한 데이터베이스와 같은 워크로드에 적합**
- io1 / io2 (4GiB ~ 16TiB)
  - 최대 PIOPS: Nitro EC2 인스턴스의 경우 64,000, 기타 인스턴스의 경우 32,000
  - 스토리지의 크기에 관계없이 PIOPS를 늘릴 수 있다.
  - io2는 더 높은 내구성과 GiB당 더 높은 IOPS를 제공한다. (io1과 동일한 가격)
- io2 Block Express (4GiB ~ 64TiB)
  - 밀리초 미만의 지연 시간을 제공한다.
  - 최대 PIOPS: 256,000 (IOPS:GiB 비율 1,000:1)
- **EBS 다중 연결을 지원한다.**

#### Hard Disk Drives (HDD)

- 부팅 볼륨이 될 수 없다.
- 125GiB ~ 16TiB 범위의 용량을 제공한다.
- 처리량 최적화 HDD (st1)
  - 빅데이터, 데이터 웨어하우스, 로그 처리
  - 최대 처리량 500 MiB/s - 최대 IOPS 500
- Cold HDD (sc1)
  - 자주 액세스하지 않는 데이터의 경우에 사용된다.
  - 최저 비용이 중요한 시나리오에 사용된다.
  - 최대 처리량 250 MiB/s - 최대 IOPS 250

#### 볼륨 유형 요약

![3-volume-types-summary.png](images%2F3-volume-types-summary.png)

- **32,000 IOPS를 넘기고 싶다면 EC2 Nitro와 io1 또는 io2가 필요하다.**

---

### 다중 연결 - io1/io2

- **동일한 EBS 볼륨을 동일한 AZ의 여러 EC2 인스턴스에 연결할 수 있다.**
- 각 인스턴스에는 고성능 볼륨에 대한 전체 읽기 및 쓰기 권한이 있다.
- 대표적인 사용 사례는 아래와 같다.
  - 클러스터링된 Linux 애플리케이션에서 더 높은 애플리케이션 가용성을 달성할 수 있다.(예. Teradata)
  - 애플리케이션은 동시 쓰기 작업을 관리해야 한다.
- **한 번에 최대 16개의 EC2 인스턴스가 연결할 수 있다.**
- 클러스터를 인식하는 파일 시스템을 사용해야 한다. (XFS, EX4 등은 지원하지 않음)

![4-io1-io2-multi-attach.png](images%2F4-io1-io2-multi-attach.png)

---

### Volume Resizing

- EBS 볼륨은 늘릴 수 있다.
  - 크기 (모든 유형)
  - IOPS (io1만 해당)
- EBS 볼륨 크기를 조정한 후 드라이브를 다시 파티션해야 한다.
- 크기를 늘린 후에도 볼륨이 "최적화"(Optimisation) 단계에서 오랜 시간 동안 있을 수 있다. 
  볼륨은 여전히 사용할 수 있다.
- **EBS 볼륨의 크기를 줄일 수 없다.**
  크기를 줄이려는 경우 더 작은 볼륨을 생성한 후 데이터를 마이그레이션 해야 한다.

![5-volume-resizing.png](images%2F5-volume-resizing.png)

---

### Snapshots

- 특정 시점에 EBS 볼륨을 백업(스냅샷)한다.
- 스냅샷을 수행하기 위해 볼륨을 분리할 필요는 없지만 **권장된다.**
  볼륨이 인스턴스에 연결되어 있으면 데이터 일관성 문제가 있기 때문이다.
- AZ 또는 Region 전체에 걸쳐 스냅샷 복제가 가능하다.

![6-snapshots.png](images%2F6-snapshots.png)

#### Amazon Data Lifecycle Manager (DLM)

- EBS 스냅샷 및 EBS 지원 AMI의 생성, 보존 및 삭제를 자동화한다.
- 백업 예약, 계쩡 간 스냅샷 복사, 오래된 백업 삭제 등을 제공한다.
- 리소스 태그를 사용하여 리소스(EC2 인스턴스, EBS 볼륨)를 식별한다.
- DLM 외부에서 생성된 스냅샷/AMI를 관리하는 데 사용할 수 없다.
- 인스턴스 스토어 지원 AMI를 관리하는 데 사용할 수 없다.

![7-data-lifecycle-manager.png](images%2F7-data-lifecycle-manager.png)

#### Fast Snapshot Restore (FSR)

- **분당 가격이 청구되며 매우 비싼 비용을 지불해야 한다.**
- S3에 저장된 EBS 스냅샷이다.
- 기본적으로 각 블록에 처음 액세스할 때, I/O 작업 지연 시간이 있다.(블록은 S3에서 가져와야 함)
- 전체 볼륨을 강제로 초기화하거나 (`dd` 또는 `fio` 명령 사용) FSR을 활성화할 수 있다.
- FSR은 생성 시 완전히 초기화된 (I/O 대기 시간 없음) 스냅샷에서 볼륨을 생성하는 데 도움이 된다.
- 특정 AZ의 스냅샷에 대해 활성화된다.
- Data Lifecycle Manager가 생성한 스냅샷에서 활성화할 수 있다.

![8-fast-snapshot-restore.png](images%2F8-fast-snapshot-restore.png)

#### Features

- **EBS Snapshot Archive**
  - 스냅샷을 75% 더 저렴한 "아카이브 계층"으로 이동한다.
  - 아카이브를 복원하는 데 24 ~ 72 범위의 시간이 소요된다.
- **Recycle Bin for EBS Snapshots**
  - 실수로 삭제한 후 복구할 수 있도록 삭제된 스냅샷을 유지하는 규칙을 설정할 수 있다.
  - 보존 기간을 지정할 수 있다. (1일 ~ 365일)

![9-snapshots-features.png](images%2F9-snapshots-features.png)

---

### Migration

- EBS Volume은 동일한 AZ에서만 사용할 수 있도록 잠겨있다.
- 다른 AZ나 Region으로 마이그레이션하려면 아래의 단계를 진행해야 한다.
  - 볼륨 스냅샷을 생성한다.
  - (선택) 볼륨을 다른 Region으로 복사한다.
  - 원하는 AZ의 스냅샷에서 볼륨을 생성한다.

---

### Encryption

- 암호화된 EBS 볼륨을 생성하면 아래와 같은 기능이 제공된다.
  - 저장된 데이터는 볼륨 내부에서 암호화된다.
  - 인스턴스와 볼륨 간에 이동 중인 모든 데이터가 암호화된다.
  - 모든 스냅샷이 암호화된다.
  - 스냅샷에서 생성된 모든 볼륨은 암호화된다.
- 암호화 및 복호화는 투명하게 처리되므로 사용하는 할 일이 없다.
- 암호화는 지연 시간에 최소한의 영향을 미친다.
- EBS 암호화는 KMS(AES-256)의 키를 활용한다.
- 암호화되지 않은 스냅샷을 복사하면서 암호화가 가능하다.
- 암호화된 볼륨의 스냅샷은 암호화된다.

- 암호화되지 않은 볼륨을 암호화하려면 아래의 단계를 따르면 된다.
  - 볼륨의 EBS 스냅샷을 생성한다.
  - EBS 스냅샷을 암호화한다. (복사 사용)
  - 스냅샷에서 새로운 EBS 볼륨을 생성한다. (볼륨도 암호화)
  - 이제 암호화된 볼륨을 원본 인스턴스에 연결할 수 있다.

---

### Elastic File System (EFS)

- 다수의 EC2에 탑재할 수 있는 관리형 NFS(Network File System)이다.
- EFS는 다중 AZ의 EC2 인스턴스와 작동한다.
- 고가용성, 확장성, gp2대비 3배의 비용, 사용한만큼 비용을 지불한다.

![10-elastic-file-system.png](images%2F10-elastic-file-system.png)

- 콘텐츠 관리, 웹 서비스, 데이터 공유, WordPress와 같은 작업에 사용된다.
- NFSv4.1 프로토콜을 사용한다.
- 보안 그룹을 사용하여 EFS에 대한 액세스를 제어한다.
- **Linux 기반 AMI와 호환 가능하다. (Windows는 지원하지 않음)**
- KMS를 사용하여 저장 시 암호화를 지원한다.
- 표준 파일 API를 갖춘 POSIX 파일 시스템(~Linux)
- 파일 시스템은 자동으로 확장되고 종량제 요금이 부과되며 용량 계획이 필요하지 않다.

#### Performance & Storage Classes

- **EFS 규모**
  - 동시 NFS 클라이언트 1,000대와 처리량 10GB 이상을 지원한다.
  - 자동으로 페타바이트 규모의 네트워크 파일 시스템으로 확장할 수 있다.
- **Performance Mode (EFS 생성 시 결정)**
  - **General Purpose(default)**: 대기 시간에 민감한 사용 사례 (웹 서버, CMS 등)
  - **Max I/O**: 더 높은 대기 시간, 처리량, 고도의 병렬 처리 (빅 데이터, 미디어 처리)
- **Throughput Mode**
  - **Bursting**: 1TB = 50MiB/s + 최대 100MiB/s의 버스트
  - **Provisioned**: 스토리지 크기에 관계없이 처리량을 설정한다. 예를 들어, 1TB 스토리지의 경우 1GiB/s
  - **Elastic**: 워크로드에 따라 자동으로 처리량을 늘리거나 줄인다.
    - 읽기의 경우 최대 3GiB/s, 쓰기의 경우 1GiB/s
    - 예측할 수 없는 워크로드에 사용된다.

#### Storage Classes

- 스토리지 계층 (수명 주기 관리 기능 - N일 후 파일 이동)
  - Standard: 자주 액세스하는 파일용
  - Infrequent Access(EFS-IA): 파일 검색 비용이 저렴하고 저장 비용이 저렴하다.
    수명 주기 정책을 통해 EFS-IA를 활성화할 수 있다.
- 가용성 및 내구성
  - Standard: Multi-AZ, 운영 환경에 적합하다.
  - One Zone: One AZ, 개발에 적합한 하나의 AZ, 기본적으로 백업 활성화, IA와 호환(EFS One Zone-IA)
- 90% 이상의 비용을 절감할 수 있다.

![11-efs-storage-classes.png](images%2F11-efs-storage-classes.png)

---

### EBS vs EFS

#### Elastic Block Storage (EBS)

- EBS 볼륨
  - 인스턴스 1개에 연결된다. (io1/io2는 multi-attach 지원)
  - 단일 가용지역에 종속된다.
  - gp2: 디스크 크기가 증가하면 IO가 증가한다.
  - io1: IO를 독립적으로 늘릴 수 있다.
- 여러 AZ에 걸쳐 EBS 볼륨을 마이그레이션 하려면 아래의 단계를 따라야 한다.
  - 스냅샷을 생성한다.
  - 스냅샷을 다른 AZ로 복원한다.
  - EBS 백업은 IO를 사용하므로 애플리케이션이 많은 트래픽을 처리하는 동안에는 백업을 실행하면 안된다.
- EC2 인스턴스가 종료되면 인스턴스의 루트 EBS 볼륨도 기본적으로 종료된다. (비활성화할 수 있다.)

![12-ebs-vs-efs-1.png](images%2F12-ebs-vs-efs-1.png)

#### Elastic File System (EFS)

- 여러 AZ의 수백개의 인스턴스에 마운트될 수 있다.
- EFS 공유 웹사이트 파일(WordPress)에 사용될 수 있다.
- Linux 인스턴스(POSIX)만 지원한다.
- EFS는 EBS보다 가격대가 높다.
- 비용 절감을 위해 EFS-IA를 활용할 수 있다.

- **참고**: FES, EBS, Instance Store의 차이를 기억해야 한다.

![13-ebs-vs-efs-2.png](images%2F13-ebs-vs-efs-2.png)

---

### EFS Access Point

- NFS 환경에 대한 애플리케이션 액세스를 쉽게 관리한다.
- 파일 시스템에 액세스할 때 POSIX 사용자 및 그룹을 사용하도록 강제한다.
- 파일 시스템 내의 디렉토리에 대한 액세스를 제한하고 선택적으로 다른 루트 디렉토리를 지정한다.
- IAM 정책을 사용하여 NFS 클라이언트의 액세스를 제한할 수 있다.

![14-access-point.png](images%2F14-access-point.png)

---

### EFS Operations

- 현장에서 수행할 수 있는 작업은 아래와 같다.
  - 수명 주기 정책(IA 활성화 또는 IA 설정을 변경)을 설정할 수 있다.
  - 처리량 모드 및 프로비저닝된 처리량 수치를 설정할 수 있다.
  - EFS 액세스 포인트를 설정할 수 있다.
- DataSync를 사용한 마이그레이션이 필요한 작업을 할 수 있다.
  - 모든 파일 속성 및 메타데이터 복제
  - 암호화된 EFS로 마이그레이션
  - Performance Mode 설정(예. Max IO)

![15-operations.png](images%2F15-operations.png)

---

### EFS CloudWatch Metrics

- **PercentIOLimit**
  - 파일 시스템이 I/O 제한에 얼마나 근접했는지 확인할 수 있다. (General Purpose)
  - 100%인 경우 최대 I/O(마이그레이션)로 이동한다.
- **BurstCreditBalance**
  - 파일 시스템이 더 높은 처리량 수준을 달성하기 위해 사용할 수 있는 버스트 크레딧의 수를 확인할 수 있다.
- **StorageBytes**
  - 파일 시스템 크기(Byte, 15분 간격)
  - 크기(Dimensions): 표준, IA, 전체(Standard + IA)를 확인할 수 있따.

![16-efs-cloudwatch-metrics.png](images%2F16-efs-cloudwatch-metrics.png)

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [Solid State Drives](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html#solid-state-drives)