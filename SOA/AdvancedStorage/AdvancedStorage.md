# Advanced Storage Solutions

이번 장에서는 **SysOps Administrator**를 준비하며 **심화 스토리지**에 대해서 알아보도록 한다.

---

### Snow Family

- 데이터를 수집하고 처리하는 보안이 뛰어난 휴대용 장치를 엣지에 구축하고 AWS 안팎으로 데이터를 마이그레이션한다.
  - 데이터 마이그레이션: Snowcone, Snowball Edge, Snowmobile
  - 엣지 컴퓨팅: Snowcone, Snowball Edge

![1-data-migration-snow-family.png](images%2F1-data-migration-snow-family.png)

- 100TB를 1Gbps로 전송하고 싶다면 12일이 걸린다.
- 많은 양의 데이터를 AWS로 마이그레이션하고 싶지만 네트워크 전송량, 연결 제한, 대역폭 제한이라는 문제로 인해 많은 돈과 시간이 소요된다.
- 네트워크로 데이터를 전송하는 데 일주일 이상 소요되는 경우 snowball 장치를 사용하는 것이 권장된다.

![2-direct-upload-s3.png](images%2F2-direct-upload-s3.png)

- 일반적으로 S3에 데이터를 업로드할 때는 공용 인터넷을 통한다.

![3-with-snow-family.png](images%2F3-with-snow-family.png)

- AWS에서 Snowball 장비를 신청하면 우편으로 장비를 받을 수 있다.
- 사용자는 로컬 기기에서 직접 데이터를 로드하고 AWS로 기기를 전송하여 데이터를 S3로 저장할 수 있다.

#### Snowball Edge

- 물리적 데이터 전송 솔루션: AWS 내부 또는 외부로 TB 또는 PB 규모의 데이터를 이동한다.
- 네트워크를 통한 데이터 이동(및 네트워크 요금 지불)의 대안이다.
- 데이터 전송 작업당 비용을 지불한다.
- 블록 스토리지 및 S3 호환 객체 스토리지를 제공한다.
- **Snowball Edge Storage Optimized**
  - 블록 볼륨 및 S3 호환 객체를 위한 80TB의 HDD 스토리지를 제공한다.
- **Snowball Edge Compute Optimized**
  - 블록 볼륨 및 S3 호환 객체 스토리지를 위한 42TB HDD 또는 28TB NVMe 용량을 제공한다.
- 대규모 데이터 클라우드 마이그레이션, DC 서비스 해제, 재해 복구 등에 사용된다.

#### Snowcone & Snowcone SSD

- 어디에서나 사용할 수 있는 소형 휴대용 컴퓨팅, 견고하고 안전하며 열악한 환경에서도 견딜 수 있다.
- 4.5 파운드, 2.1kg으로 가볍다.
- 엣지 컴퓨팅, 저장, 데이터 전송에 사용되는 장치다.
- **Snowcone**: 8TB의 HDD 스토리지를 제공한다.
- **Snowcone SSD**: 14TB의 SSD 스토리지를 제공한다.
- Snowball이 맞지 않는 곳(공간적으로 제한된 환경)에서는 Snowcone을 사용한다.
- 배터리/케이블은 사용자가 직접 준비해야 한다.
- 오프라인으로 AWS로 다시 보내거나 인터넷에 연결하고 AWS DataSync를 사용하여 데이터를 보낼 수 있다.

#### Snowmobile

- 엑사바이트 규모의 데이터를 전송할 때 사용한다.(1EB = 1,000PB = 1,000,000TB)
- 각 Snowmobile의 용량은 100PB다. (여러 대를 병렬로 사용할 수 있음)
- 높은 보안: 온도 제어, GPS, 연중무휴 비디오 감시를 제공한다.
- 10PB가 넘는 데이터를 전송하는 경우 Snowball보다 우수하다.

![4-snow-familiy-compairson.png](images%2F4-snow-familiy-compairison.png)

#### 사용 절차

1. AWS 콘솔에서 Snowball 디바이스 배송을 요청한다.
2. 서버에 Snowball 클라이언트를 설치하고 AWS에는 OpsHub를 설치한다.
3. Snowball을 서버에 연결하고 클라이언트를 사용하여 파일을 복사한다.
4. 작업이 완료되면 디바이스를 반송한다.(AWS 시설로)
5. 데이터가 S3 버킷에 저장된다.
6. Snowball의 데이터가 완벽하게 제거된다.

#### Edge Computing

- 엣지 로케이션에서 데이터가 생성되는 동안 데이터를 처리한다.
  - 도로 위의 트럭, 바다 위의 배, 지하 광산 스테이션 등
  - 이러한 위치 들은 인터넷이 제한되거나 불가능하고 컴퓨팅 성능이 제한되거나 불가능 할 수 있다.
- 엣지 컴퓨팅을 수행하기 위해 Snowball Edge/Snowcone 디바이스를 설정했다.
  - 데이터 전처리, 머신러닝, 미디어 스트림 트랜스코딩과 같은 작업에 사용된다.
- 데이터 전송과 같이 사용자가 필요한 경우 장치를 AWS로 반송할 수 있다.

- **Snowcone & Snowcone SSD**
  - 2 CPU, 4GB 메모리, 유선 또는 무선 액세스를 제공한다.
  - 코드 또는 배터리 옵션을 사용하는 USB-C 전원을 제공한다.
- **Snowball Edge - Compute Optimized**
  - 104 vCPU, 416GIB 램을 제공한다.
  - 선택적으로 GPU를 사용하여 비디오 처리 또는 머신러닝에 사용할 수 있다.
  - 28TB NVMe 또는 42TB HDD 사용 가능 스토리지를 제공한다.
  - 최대 16개 노드까지 스토리지 클러스터링이 사용 가능하다.
- **Snowball Edge - Storage Optimized**
  - 최대 40 vCPU, 80GiB 램, 80TB의 스토리지를 제공한다.
- 모두 EC2 인스턴스 및 Lambda Function을 실행할 수 있다.(AWS IoT Greengrass를 활용하여)
- 장기 배포 옵션을 사용하여 1년 또는 3년 약정하는 경우 할인된 금액에 사용할 수 있다.

#### OpsHub

![5-opshub.png](images%2F5-opshub.png)

- 지금까지 Snow Family 장치를 사용하려면 CLI(Command Line Interface 도구)가 필요했다.
- AWS OpsHub(컴퓨터/노트북에 설치하는 소프트웨어)를 사용하면 Snow Family 장치를 관리할 수 있다.
  - 단일 또는 클러스터링된 장치의 잠금 해제 및 구성할 수 있다.
  - 전송 중인 파일을 확인할 수 있다.
  - Snow 제품군 장치에서 실행되는 인스턴스를 시작하거나 관리할 수 있다.
  - 장치의 지표(저장 용량, 장치의 활성 인스턴스)를 모니터링할 수 있다.
  - 장치에서 호환 가능한 AWS 서비스를 실행(EC2 인스턴스, DataSync, NFS)할 수 있다.

---

### Amazon FSx

- AWS에서 실행되는 3rd part 고성능 파일 시스템이다.
- 완전 관리형 서비스다.
  - FSx for Lustre
  - FSx for Windows File Server
  - FSx for NetApp ONTAP
  - FSx for OpenZFS

#### FSx for Windows (File Server)

- Windows용 FSx는 완벽하게 관리되는 Windows 파일 시스템 공유 드라이브다.
- SMB 프로토콜 및 Windows NTFS를 지원한다.
- Microsoft Active Directory 통합, ACL, 사용자 할당량을 지원한다.
- **Linux EC2 인스턴스에 마운트할 수 있다.**
- **Microsoft의 분산 파일 시스템(DFS) 네임스페이스를 지원**한다.(여러 FS에 걸쳐 파일을 그룹화)
- 최대 10GB/s, 수백만 IOPS, 100PB의 데이터 확장을 제공한다.
- 스토리지 옵션
  - SSD: 대기 시간에 민감한 워크로드(데이터베이스, 미디어 처리, 데이터 분석 등)
  - HDD: 광범위한 워크로드(홈 디렉터리, CMS 등)
- 온프레미스 인프라(VPN 또는 Direct Connect)에서 액세스할 수 있다.
- 다중 AZ로 구성하여 고가용성을 구성할 수 있다.
- 데이터는 매일 S3에 백업된다.

#### FSx for Lustre

- Lustre는 대규모 컴퓨팅을 위한 일종의 병렬 분산 파일 시스템이다.
- Lustre라는 이름은 "Linux"와 "Cluster"에서 유래되었다.
- 머신러닝, **고성능 컴퓨팅(HPC)**에 사용된다.
- 비디오 처리, 재무 모델링, 전자 설계자동화와 같은 작업에 사용된다.
- 최대 100GB/s, 수백만 IOPS, 밀리초 미만의 지연 시간까지 확장할 수 있다.
- 스토리지 옵션
  - SSD: 짧은 지연 시간, IOPS 집약적인 워크로드, 소규모 및 무작위 파일 작업
  - HDD: 처리량 집약적인 워크로드, 대규모 및 순차 파일 작업
- **S3와 원활한 통합**
  - 파일 시스템으로 S3를 읽을 수 있다.(FSx를 통해)
  - 계산 결과를 S3에 저장할 수 있다.(FSx를 통해)
- **온프레미스 서버(VPN 또는 Direct Connect)에서 사용할 수 있다.**

#### Lustre - 파일 시스템 배포 옵션

- **Scratch 파일 시스템**
  - 임시 저장
  - 데이터가 복제되지 않으므로 파일 서버에 장애가 발생하면 데이터가 유실된다.
  - 높은 버스트(6배 더 빠름, TiB당 200Mbps)
  - 단기 처리 프로세스, 비용 최적화 등에 사용된다.
- **Persistent 파일 시스템**
  - 장기 보관
  - 데이터는 동일한 AZ 내에서 복제된다.
  - 실패한 파일을 몇 분 안에 교체한다.
  - 장기 처리, 민감한 데이터 저장 등에 사용된다.

![6-lustre-file-system-deployment-options.png](images%2F6-lustre-file-system-deployment-options.png)


#### FSx for NetApp ONTAP

- AWS에서 관리되는 NetApp ONTAP이다.
- **NFS, SMB, iSCSI 프로토콜과 호환되는 파일 시스템**이다.
- ONTAP 또는 NAS에서 실행되는 워크로드를 AWS로 이동한다.
- 호환되는 제품은 아래와 같다.
  - Linux, Windows, macOS, VMware Cloud on AWS, Amazon Workspaces & AppStream 2.0, EC2, ECS, EKS
- 스토리지는 자동으로 축소되거나 증가한다.
- 스냅샷, 복제, 저비용, 압축 및 데이터 중복을 제거한다.
- 특정 시점의 즉각적인 복제를 통해 새 워크로드 테스트에 유용하다.

![7-fsx-netapp-ontap.png](images%2F7-fsx-netapp-ontap.png)

#### FSx for OpenZFS

- AWS의 관리형 Open ZFS 파일 시스템이다.
- NFS(v3, v4, v4.1, v4.2)와 호환되는 파일 시스템이다.
- ZFS에서 실행되는 워크로드를 AWS로 이동한다.
- 호환되는 제품은 아래와 같다.
  - Linux, Windows, macOS, VMware Cloud on AWS, Amazon Workspaces & AppStream 2.0, EC2, ECS, EKS
- 0.5ms 미만의 지연 시간으로 최대 1,000,000 IOPS를 지원한다.
- 스냅샷, 압축 및 저렴한 비용을 제공한다.
- 특정 시점의 즉각적인 복제를 통해 새 워크로드 테스트에 유용하다.

![8-fsx-for-openzfs.png](images%2F8-fsx-for-openzfs.png)

#### FSx for Windows AZ

- **FSx for Windows - Single-AZ**
  - AZ 내에서 데이터를 자동으로 복제한다.
  - 2세대: 단일 AZ 1(SSD), 단일 AZ 2(SSD & HDD)

![9-fsx-for-windows-single-az.png](images%2F9-fsx-for-windows-single-az.png)

- **FSx for Windows - Multi-AZ**
  - AZ 전반에 걸쳐 데이터를 자동으로 복제한다.(동기식)
  - 다른 AZ에는 대기(Standby) 파일 서버가 있다.(자동 장애 조치)

![10-fsx-for-windows-multi-az.png](images%2F10-fsx-for-windows-multi-az.png)

---

### AWS Storage Gateway

#### Hybrid Cloud for Storage

- AWS는 "하이브리드 클라우드"를 추진하고 있다.
  - 인프라의 일부는 클라우드에 있다.
  - 인프라의 일부는 온프레미스에 있다.
- 아래와 같은 여러 이유 때문에 이러한 구성을 유지해야 할 수 있다.
  - 장기간의 클라우드 마이그레이션
  - 보안 요구 사항
  - 규정 준수 요구 사항
  - IT 전략
- S3는 EFS/NFS와 달리 독점 스토리지 기술이다. 이러한 S3 데이터를 온프레미스에 노출하기 위해서 **AWS Storage Gateway**가 사용된다.

#### Storage Cloud Option

![11-storage-cloud-native-options.png](images%2F11-storage-cloud-native-options.png)

- EBS나 EC2 인스턴스 스토어는 블록 스토리지다.
- EFS나 FSx같은 파일 시스템도 있고, S3나 Glacier같은 객체 수준의 저장소도 있다.

#### Storage Gateway

- 온프레미스 데이터와 클라우드 데이터 간의 연결을 셍성한다.
- 재해 복구, 백업 및 복원, 계층형 스토리지, 온프레미스 캐시 및 지연 시간이 짧은 파일 액세스와 같은 작업에 사용된다.
- 총 4개의 Storage Gateway가 있다.
  - S3 File Gateway
  - FSx File Gateway
  - Volume Gateway
  - Tape Gateway

![12-storage-gateway.png](images%2F12-storage-gateway.png)

#### S3 File Gateway

- 구성된 S3 버킷은 NFS 및 SMB 프로토콜을 사용하여 액세스할 수 있다.
- **가장 최근에 사용한 데이터는 파일 게이트웨이에 캐시**된다.
- S3 Standard, S3 Standard-IA, S3 One Zone-IA, S3 Intelligent-Tiering을 지원한다.
- **수명 주기 정책을 사용하여 S3 Glacier로 전환**할 수 있다.
- 각 파일 게이트웨이에 대한 IAM 역할을 사용한 버킷 액세스를 생성할 수 있다.
- SMB 프로토콜은 사용자 인증을 위해 Active Directory(AD)와 통합되어 있다.

![13-s3-file-gateway.png](images%2F13-s3-file-gateway.png)

#### FSx File Gateway

- Windows 파일 서버용 Amazon FSx에 대한 기본 액세스를 제공한다.
- 자주 액세스하는 데이터를 위한 로컬 캐시를 지원한다.
- Windows 기본 호환성(SMB, NTFS, Active Directory...)을 제공한다.
- 그룹 파일 공유 및 홈 디렉터리에 유용하다.

![14-fsx-file-gateway.png](images%2F14-fsx-file-gateway.png)

#### Volume Gateway

- S3가 지원하는 iSCSI 프로토콜을 사용하는 블록 스토리지다.
- 온프레미스 볼륨을 복원하는 데 도움이되는 EBS 스냅샷이 지원된다.
- **캐시 볼륨**: 최신 데이터에 대한 짧은 지연 시간 액세스를 제공한다.
- **저장된 볼륨**: 전체 데이터 세트가 온프레미스에 있으며 S3에 백업이 예약되어 있다.

![15-volume-gateway.png](images%2F15-volume-gateway.png)

#### Tape Gateway

- 일부 회사에서는 물리적 테이프를 사용하여 백업 프로세스를 수행한다.
- 테이프 게이트웨이를 통해 기업은 동일한 프로세스를 클라우드에서 사용한다.
- S3 및 Glacier는 Virtual Tape Library(VTL)을 지원한다.
- 기존 테이프 기반 프로세스와 iSCSI 인터페이스를 사용하여 데이터를 백업한다.
- 주요 백업 소프트웨어와 공급업체와 협력할 수 있다.

![16-tape-gateway.png](images%2F16-tape-gateway.png)

#### Hardware Appliance

![17-storage-gateway-hardware-appliance.png](images%2F17-storage-gateway-hardware-appliance.png)

- Storage Gateway를 사용한다는 것은 온프레미스 가상화가 필요하다는 것을 의미한다.
- 그렇지 않으면 Storage Gateway 하드웨어 어플라이언스를 사용할 수 있따.
- `amazon.com`에서 구매할 수 있다.
- File Gateway, Volume Gateway, Tape Gateway와 함께 작동한다.
- 필요한 CPU, 메모리, 네트워크, SSD 캐시 리소스가 있다.
- 소규모 데이터 센터의 일일 NFS 백업에 유용하다.

#### 요약

![18-storage-gateway-summary.png](images%2F18-storage-gateway-summary.png)

- 온프레미스에서 Storage Gateway VM과 하드웨어 어플라이언스를 배포하고 Storage Gateway 서비스를 배포한다.
- 로컬 캐시에 File Gateway가 있다면 이 사용 사례는 사용자 그룹 파일 공유가 있고 NFS나 SMB 프로토콜을 통해 접근하고자 하는 경우다.
- 첫 번째 옵션은 S3를 File Gateway에 연결하는 것이다.
  - S3가 사용자의 데이터를 백업한다.
  - 많은 저장 단계를 포함하지만, Glacier와 Glacier Deep Archive는 지원하지 않는다.
  - Glacier와 Glacier Deep Archive에 데이터를 보내고 싶다면 사용자가 직접 라이프사이클 정책을 생성해야 한다.
- FSx File Gateway를 사용할 경우 Windows 파일 서버를 위해 Amazon FSx로 데이터를 보내면 때때로 S3에 자동으로 백업된다.
- Volume Gateway의 또 다른 사용 사례는 응용 프로그램 서버가 iSCSI 프로토콜에 볼륨을 탑재하는 것이다.
  - Volume Gateway는 Storage Gateway를 통해 S3에 연결되며 여기에 볼륨 데이터가 저장된다.
  - S3는 이 데이터를 AWS EBS 볼륨으로 변환하여 AWS에서 복원할 수 있다.
- 백업 어플리케이션이 iSCSI VTL 프로토콜로 Tape Gateway와 연결된다.
  - Tape를 Glacier와 Glacier Deep Archive에 저장한다.

---

### 참고

- File Gateway는 POSIX 규격(Linux 파일 시스템)이다.
  - S3의 객체 메타데이터에 저장된 POSIX 메타데이터 소유권, 권한 및 타임스탬프
- Storage Gateway VM 재부팅(예. 유지 관리)
  - File Gateway: Storage Gateway VM을 다시 시작하면 된다.
  - Volume Gateway & Tape Gateway
    - Storage Gateway 서비스를 중지한다.(AWS 콘솔, VM 로컬 콘솔, Storage Gateway API)
    - Storage Gateway VM을 재부팅한다.
    - Storage Gateway 서비스를 시작한다. (AWS 콘솔, VM 로컬 콘솔, Storage Gateway API)

#### Activation

![19-storage-gateway-activation.png](images%2F19-storage-gateway-activation.png)

- 활성화 키를 얻는 두 가지 방법
  - Gateway VM CLI를 사용한다.
  - Gateway VM(Port 80)에 웹 요청을 한다.
- 활성화 실패 문제 해결
  - Gateway VM에 80번 포트가 열려있는지 확인한다.
  - Gateway VM의 시간이 올바른지 확인하고 시간이 자동으로 NTP(Network Time Protocol) 서버와 동기화되는지 확인한다.

![20-storage-gateway-activation-2.png](images%2F20-storage-gateway-activation-2.png)

#### Volume Gateway Cache

- 캐시 모드: 가장 최근의 데이터만 저장한다.
- 캐시 효율성 검토
  - `CacheHitPercent` 지표를 확인한다. (높을수록 좋음)
  - `CachePercentUsed` 지표를 확인한다. (너무 높으면 안됨)
- 더 큰 캐시 디스크를 생성한다.
  - 캐시된 볼륨을 사용하여 더 큰 크기의 새 볼륨을 복제한다.
  - 캐시된 볼륨으로 새 디스크를 선택한다.

![21-volume-gateway-cache.png](images%2F21-volume-gateway-cache.png)

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [Snowball Edge Update](https://aws.amazon.com/blogs/aws/aws-snowball-edge-update/)