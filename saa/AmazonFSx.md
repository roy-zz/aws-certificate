# Amazon FSx

이번 장에서는 SAA를 준비하며 **Amazon FSx**에 대해서 알아보도록 한다.

---

## Overview

- AWS상에서 AWS가 아닌 타사의 고성능 파일 시스템을 **완전 관리형**으로 사용할 수 있다.

![fsx-overview.png](images%2FAmazonFSx%2Ffsx-overview.png)

### Amazon FSx for Windows (File Server)

- Windows용 FSx는 완벽하게 관리되는 Windows 파일 시스템 공유 드라이브다.
- SMB 프로토콜 및 Windows NTFS를 지원한다.
- Microsoft Active Directory 통합, ACL을 지원하면 사용자 당 할당된다.
- Linux EC2 인스턴스에 마운트 가능하다.
- Microsoft DFS(Distributed File System) 네임스페이스(여러 File System에 걸친 그룹 파일)를 지원한다.
- 최대 10초의 GB/s, 수백만 IOPS, 100초의 PB 데이터까지 확장 가능하다.
- 스토리지 옵션
  - SSD: 지연 시간에 민감한 워크로드(데이터베이스, 미디어 프로세싱, 데이터 분석 등)
  - HDD: 광범위한 워크로드(홈 디렉토리, CMS 등)
- 사내 인프라(VPN 또는 Direct Connect)에서 액세스할 수 있다.
- Multi-AZ로 구성이 가능하다.
- 데이터가 매일 S3로 백업된다.

### Amazon FSx for Lustre

- Lustre는 대규모 컴퓨팅을 위한 병렬 분산 파일 시스템의 일종이다.
- Lustre라는 이름은 "Linux"와 "Cluster"에서 유래되었다.
- 머신 러닝, 고성능 컴퓨팅(HPC)에 사용된다.
- 비디오 프로세싱, 재무 모델링, 전자 설계 자동화
- 최대 100s GB/s, 수백만 IOPS, 서브 ms 지연 시간까지 확장 가능하다.
- 스토리지 옵션
  - SSD: 저지연, IOPS 집약적 워크로드, 소규모 및 랜덤 파일 작업
  - HDD: 처리량 집약적 워크로드, 대규모 및 순차적 파일 작업
- S3와 원활하게 통합된다.
  - 파일 시스템으로 "S3 읽기"가 가능하다. (FSx를 통해)
  - 계산된 출력물을 다시 S3로 쓸 수 있다. (FSx를 통해)
- 사내 서버(VPN 또는 Direct Connect)에서 사용할 수 있다.

### FSx Lustre - File System Deployment Options

- Scratch(스크래치) 파일 시스템
  - 임시 보관
  - 데이터가 복제되지 않음 (파일 서버에 장애가 발생해도 지속되지 않음)
  - 높은 버스트(6배 더 빨라짐, TiB당 200MBps)
  - 용도: 짧은 기간의 프로세싱, 비용 최적화
- Persistent File System
  - 영구 파일 스토리지
  - 데이터가 동일한 AZ내에서 복제된다.
  - 몇 분 내에 실패한 파일을 교체한다.
  - 용도: 장기적인 프로세싱, 민감한 데이터 처리

![file-system-deployment-option.png](images%2FAmazonFSx%2Ffile-system-deployment-option.png)

### Amazon FSx for NetApp ONTAP

- AWS에서 관리형 NetApp ONTAP이다.
- NFS, SMB, iSCSI 프로토콜과 호환되는 파일 시스템이다.
- ONTAP 또는 NAS에서 실행 중인 워크로드를 AWS로 이동한다.
- 작업 대상
  - Linux
  - Windows
  - MacOS
  - VMware CLoud on AWS
  - Amazon Workspaces & AppStream 2.0
  - Amazon EC2, ECS and EKS
- 스토리지가 자동으로 축소되거나 확장된다.
- 스냅샷, 복제, 저비용, 압축 및 데이터
- 시점 단위의 즉각적인 복제가 가능하여 새로운 워크로드 테스트에 도움이 된다.

![netapp-ontap.png](images%2FAmazonFSx%2Fnetapp-ontap.png)

### Amazon FSx for OpenZFS

- AWS에서 관리되는 OpenZFS 파일 시스템이다.
- NFS와 호환되는 파일 시스템(v3, v4, v4.1, v4.2)
- ZFS에서 실행 중인 워크로드를 AWS로 이동할 수 있다.
- 작업 대상
  - Linux
  - Windows
  - MacOS
  - VMware Cloud on AWS
  - Amazon Workspaces & AppStream 2.0
  - Amazon EC2, ECS and EKS
- 최대 1,000,000 IOPS까지 지원하며 0.5ms 미만의 대기 시간을 포함한다.
- 스냅샷, 압축을 낮은 비용에 제공한다.
- 시점 단위의 즉각적인 복제가 가능하여, 새로운 워크로드 테스트에 도움이 된다.

![openzfs.png](images%2FAmazonFSx%2Fopenzfs.png)

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03