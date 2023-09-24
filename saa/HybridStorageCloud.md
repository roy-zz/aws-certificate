# Hybrid Cloud for Storage

이번 장에서는 SAA를 준비하며 **Hybrid Cloud**에 대해서 알아보도록 한다.

---

## Hybrid Cloud for Storage

- AWS는 하이브리드 클라우드를 추진하고 있다.
    - 인프라스트럭처의 일부가 클라우드에 있다.
    - 인프라스터럭처의 일부가 사내에 있다.
- Hybrid Cloud를 구성하는 이유는 아래와 같다.
    - 클라우드 마이그레이션 시간이 길어지는 경우
    - 보안 요구사항
    - 컴플라이언스 요구사항
    - IT 전략
- S3는 EFS/NFS와 다르게 독점 스토리지 기술이다.
- 이런 S3의 데이터를 사내에 노출하기 위해서는 **AWS Storage Gateway**가 필요하다.

![storage-cloud-native-options.png](images%2FHybridStorageCloud%2Fstorage-cloud-native-options.png)

### AWS Storage Gateway

- 사내 데이터와 클라우드 데이터 간의 Bridge 역할을 한다.
- 사용 사례는 아래와 같다.
  - 재해 복구(Disaster Recovery)
  - 백업 및 복원
  - 계층 스토리지
  - 온 프레미스 캐시 및 저지연 파일 액세스
- 스토리지 게이트웨이의 유형은 아래와 같다.
  - S3 파일 게이트웨이
  - FSx 파일 게이트웨이
  - Volume 게이트웨이
  - Tape 게이트웨이

### Amazon S3 File Gateway

- 구성된 S3 버킷은 NFS 및 SMB 프로토콜을 사용하여 액세스할 수 있다.
- 가장 최근에 사용한 데이터가 파일 게이트웨이에 캐싱된다.
- S3 Standard, S3 Standard IA, S3 One Zone A, S3 Intelligent Tiering을 지원한다.
- Lifecycle 정책에 의해 S3 Glacier로 전환된다.
- 각 File Gateway에 대해 IAM 역할을 사용하여 버킷에 액세스할 수 있다.
- 사용자 인증을 위해 SMB 프로토콜이 AD(Active Directory)와 통합된다.

![s3-file-gateway.png](images%2FHybridStorageCloud%2Fs3-file-gateway.png)

### Amazon FSx File Gateway

- Windows 파일 서버용 Amazon FSx 기본 액세스를 제공한다.
- 자주 액세스하는 데이터에 대한 로컬 캐시를 제공한다.
- Windows 기본 호환성(SMB, NTFS, Active Directory..)을 제공한다.
- 그룹 파일 공유 및 홈 디렉토리에 유용하다.

![fsx-file-gateway.png](images%2FHybridStorageCloud%2Ffsx-file-gateway.png)

### Volume Gateway

- S3에서 지원하는 iSCSI 프로토콜을 사용하여 스토리지를 차단한다.
- 사내 볼륨 복원에 도움이 되는 EBS 스냅샷을 제공한다.
- 캐시된 볼륨(Cached Volumes): 최신 데이터에 대한 지연 시간이 짧은 액세스를 제공한다.
- 저장된 볼륨(Stored Volumes): 온프레미스의 모든 데이터가, S3로 백업이 예약된다.

![volume-gateway.png](images%2FHybridStorageCloud%2Fvolume-gateway.png)

### Tape Gateway

- 일부 기업에서는 물리적 테이프를 사용한 백업 프로세스를 실행하고 있다.
- Tape Gateway를 사용하면 기업은 동일한 프로세스를 클라우드에서 사용할 수 있다.
- Amazon S3 및 Glacier가 지원하는 VTL(Virtual Tape Library)를 사용할 수 있다.
- 기존 테이브 기반 프로세스(및 iSCSI 인터페이스)를 사용하여 데이터 백업이 가능하다.
- 업계 최고의 백업 소프트웨어 공급업체와 협력하고 있다.

![tape-gateway.png](images%2FHybridStorageCloud%2Ftape-gateway.png)

### Storage Gateway - Hardware Appliance

- 스토리지 게이트웨이를 사용하면 사내 가상화가 필요하다.
- 그렇지 않으면, Storage Gateway Hardware Appliance를 사용할 수 있다.
- `amazon.com`에서 구입할 수 있다.
- File Gateway, Volume Gateway, Tape Gateway를 지원한다.
- 필요한 CPU, Memory, Network, SSD 캐시 리소스를 보유하고 있다.
- 소규모 데이터 센터에서 매일 NFS 백업을 수행하는 데 도움이 된다.

![hardware-appliance.png](images%2FHybridStorageCloud%2Fhardware-appliance.png)

![storage-gateway-overview.png](images%2FHybridStorageCloud%2Fstorage-gateway-overview.png)

---

## AWS Transfer Family

- FTP 프로토콜을 사용하여 Amazon S3 또는 Amazon EFS로 파일을 주고 받는 완전 관리형 서비스다.
- 지원되는 프로토콜 목록은 아래와 같다.
  - AWS Transfer for FTP (File Transfer Protocol(FTP))
  - AWS Transfer for FTPS (File Transfer Protocol over SSL(FTPS))
  - AWS Transfer for SFTP (Secure File Transfer Protocol (SFTP))
- 관리형 인프라, 확장성, 안정성, 고가용성(multi-AZ)
- 프로비저닝된 엔드포인트 및 시간당, 추가로 전송된 데이터(GB 단위)마다 비용을 지불한다.
- 서비스 내에서 사용자 자격 증명 저장 및 관리한다.
- 기존 인증 시스템과 통합(Microsoft Active Directory, LDAP, Okta, Amazon Cognito, Custom)
- 대표적인 사용 예시는 아래와 같다.
  - 공용 데이터셋
  - CRM
  - ERP

![transfer-familiy.png](images%2FHybridStorageCloud%2Ftransfer-familiy.png)

## AWS DataSync

- 많은 양의 데이터를 이동할 때 사용된다.
  - 사내/기타 클라우드 to AWS(NFS, SMB, HDFS, S3 API등) - 에이전트가 필요하다.
  - AWS to AWS(다양한 스토리지 서비스) - 에이전트가 필요없다.
- 동기화할 수 있는 위치는 아래와 같다.
  - Amazon S3(Glacier를 포함한 모든 스토리지 클래스)
  - Amazon EFS
  - Amazon FSx(Windows, Lustre, NetApp, OpenZFS...)
- 복제 작업을 매시간, 매일, 매주 예약할 수 있다.
- 파일 사용 권한 및 메타데이터 보존(NFS POSIX, SMB...)
- 에이전트 작업 하나로 10Gbps 사용 가능하며, 대역폭 제한을 설정할 수 있다.

![datasync-nfssmb-aws.png](images%2FHybridStorageCloud%2Fdatasync-nfssmb-aws.png)

![datasync-transfer-aws-storage-service.png](images%2FHybridStorageCloud%2Fdatasync-transfer-aws-storage-service.png)

## AWS Storage Comparison

- S3: 객체 스토리지
- S3 Glacier: 객체 보관소
- EBS Volumes: 한번에 하나의 EC2 인스턴스에 할당되는 네트워크 스토리지
- Instance Storage: EC2 인스턴스(높은 IOPS)를 위한 물리적 스토리지
- EFS: Linux 인스턴스용 네트워크 파일 시스템, POSIX 파일 시스템
- FSx for Windows: 윈도우 서버용 네트워크 파일 시스템
- FSx for Lustre: 고성능 컴퓨팅 리눅스 파일 시스템
- NetApp ONTAP용 FSx: 높은 OS 호환성
- OpenZFS용 FSx: 관리형 ZFS 파일 시스템
- Storage Gateway: S3 & FSx File Gateway, Volume Gateway, Tape Gateway
- Transfer Family: Amazon S3 또는 Amazon EFS 위에 FTP, FTPS, SFTP 인터페이스
- DataSync: 사내에서 AWS, 또는 AWS에서 AWS로 스케줄링된 데이터 동기화
- Snowcone/Snowball/Snowmobile: 많은 양의 데이터를 클라우드로 이동하기 위한 물리적 장치
- Database: 특정 워크로드의 경우, 대개 인덱싱 및 쿼리를 사용한다.

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03