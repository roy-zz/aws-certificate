# Disaster Recovery

이번 장에서는 **SysOps Administrator**를 준비하며 **재해 복구**에 대해서 알아보도록 한다.

---

### AWS DataSync

- 대용량의 데이터 이동을 지원한다.
  - 에이전트를 사용하여 온프레미스 / 기타 클라우드에서 AWS로 데이터를 이동한다. (NFS, SMB, HDFS, S3 API 등..)
  - 하나의 AWS 서비스에서 다른 AWS 서비스로 데이터를 이동한다. 에이전트가 필요하지 않다.
- 복제 작업을 매시간, 매일, 매주 예약할 수 있다.
- 파일 권한 및 메타데이터가 보존된다. (NFS POSIX, SMB 등)
- 하나의 에이전트 작업은 10Gbps를 사용할 수 있으며 대역폭 제한을 설정할 수 있다.

![1-datasync-nfs-smb.png](images%2F1-datasync-nfs-smb.png)

- 온프레미스와 DataSync가 실행되는 AWS 영역이 있다.
- 온프레미스에는 NFS, SMB 서버가 있고 DataSync 에이전트가 설치되어 있다.
- DataSync 에이전트는 연결을 설정하고 DataSync 서비스로 암호화된 방식으로 연결한다.
- DataSync를 사용하고 싶은데 그에 대한 네트워크 용량이 없는 경우에는 AWS Snowcone 장치를 사용해야 한다.
- AWS Snowcone 장치에는 DataSync 에이전트가 미리 설치되어 있다.

![2-datasync-trasfer-between-aws-service.png](images%2F2-datasync-trasfer-between-aws-service.png)

- DataSync를 통해 아래의 항목들과 동기화할 수 있다.
  - Amazon S3 (Glacier를 포함한 모든 스토리지 클래스)
  - Amazon EFS
  - Amazon FSx (Windows, Lustre, NetApp, OpenZFS 등..)

---

### AWS Backup

- 완전관리형 서비스다.
- AWS 서비스 전반에 거쳐 백업을 중앙에서 관리하고 자동화한다.
- 사용자 정의 스크립트 및 수동 프로세스를 생성할 필요가 없다.
- 지원되는 서비스 항목은 아래와 같다.
  - Amazon EC2 / Amazon EBS
  - Amazon S3
  - Amazon RDS (모든 엔진) / Amazon Aurora / Amazon DynamoDB
  - Amazon DocumentDB / Amazon Neptune
  - Amazon EFS / Amazon FSx (Lustre, Windows File Server)
  - AWS Storage Gateway (Volume Gateway)
- 교차 지역 백업을 지원한다.
- 교차 계정 백업을 지원한다.
- 지원되는 서비스에 대해 PITR을 지원한다.
- On-Demand 및 예약 백업을 지원한다.
- 태그 기반 백업 정책을 지원한다.
- "Backup Plan"이라는 백업 정책을 생성한다.
  - 백업 빈도(12시간 마다, 매일, 매주, 매월, cron 표현식)
  - 백업 Window
  - Cold 스토리지로 전환(없음, 일, 주, 월, 년)
  - 보유 기간(항상, 일, 주, 월, 년)

![3-backup.png](images%2F3-backup.png)

- "Backup Plan"을 생성하고 특정 AWS 리소스를 할당한다.
- 정상적으로 백업이 생성되면 데이터는 S3에 저장될 것이다. (AWS Backup에 특화된 내부 버킷)

#### AWS Backup Vault Lock

- AWS Backup Vault에 저장하는 모든 백업에 WORM(Write Once Read Many) 상태를 적용한다.
- 아래의 항목으로부터 백업을 보호하기 위한 추가 방어 계층이다.
  - 부주의하거나 악의적인 삭제 작업
  - 보존 기간을 단축하거나 변경하는 업데이트
- 활성화되면 루트 사용자도 백업을 삭제할 수 없다.

![4-backup-vault-lock.png](images%2F4-backup-vault-lock.png)

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)