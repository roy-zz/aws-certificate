# Amazon Machine Image

이번 장에서는 **SysOps Administrator**를 준비하며 **Amazon Machine Image의 약자인 AMI**에 대해서 알아보도록 한다.

---

### Overview

- AMI는 "Amazon Machine Image"의 약자로, EC2 인스턴스의 사용자 정의다.
  - 자신만의 소프트웨어, 구성, 운영 체제, 모니터링을 추가할 수 있다.
  - 모든 소프트웨어가 사전 패키지되어 있으므로 부팅/구성 시간이 더 빨라진다.
- AMI는 특정 지역에 한정되도록 설계되었으며, 지역간 이동은 불가능하고 복제를 해야한다.
- 다양한 AMI로부터 EC2 인스턴스를 내보낼 수 있다.
  - AWS가 제공하는 "Public AMI"를 사용할 수 있다.
  - 사용자가 직접 만들어서 AMI를 사용할 수 있다. AMI를 만드는 작업을 자동화할 수 있지만 이것은 클라우드 사용자가 해야 하는 작업니다.
  - Marketplace에서 다른 사람이 만든 AMI를 구매하여 EC2 인스턴스를 실행할 수 있다.

- EC2 인스턴스를 시작 및 사용자 지정할 수 있다.
- 데이터 무결성을 위하여 인스턴스를 중지한다.
- AMI를 생성하면 EBS 스냅샷도 생성된다.
- 다른 AMI의 인스턴스를 실행할 수 있다.

![1-ami-process.png](images%2F1-ami-process.png)

- US-EAST-1A에서 인스턴스를 만들고 AMI를 생성한다.
- US-EAST-1B에서는 AMI를 이용해 EC2 인스턴스를 복사할 수 있다.

#### No-Reboot 옵션

- 인스턴스를 종료하지 않고도 AMI를 생성할 수 있다.
- 기본적으로 선택되어 있지 않다. AWS는 파일 시스템의 무결성을 유지하기 위해 AMI를 생성하기 전에 인스턴스를 종료한다.

![2-ami-no-reboot-disabled.png](images%2F2-ami-no-reboot-disabled.png)

- "No-Reboot"이 비활성화되어 있으면 인스턴스는 먼저 중단(Stop)된다.
- 중단(Stop) 후 첨부된 EBS 볼륨은 EBS 스냅샷으로 스냅샷을 갖게 된다.
- EBS 스냅샷이 AMI로 변환된다.
- "No-Reboot"을 활성화하면 현재 실행 중인 인스턴스는 연결된 EBS 볼륨에서 실행되는 스냅샷을 직접 얻게 된다.
- 이후 AMI 이미지가 생성된다.

![3-ami-no-reboot-enabled.png](images%2F3-ami-no-reboot-enabled.png)

#### AWS Backup Plans

![4-aws-backup-plans.png](images%2F4-aws-backup-plans.png)

- AWS에서 백업 서비스를 사용할 때 백업 플랜을 생성하여 AMI를 생성할 수 있다.
  - AWS 백업 서비스는 EBS 스냅샷을 생성하는 동안 인스턴스를 재부팅하지 않는다.
  - **이러한 이유로 백업 플랜을 사용할 때 파일 시스템의 완전성을 보장할 수 없다.**
  - 데이터 무결성을 유지하려면 AMI를 생서하는 동안 재부팅 매개변수를 제공해야 한다. (EventBridge + Lambda + CreateImage API with Reboot)

---

### 가용지역간 EC2 Instance Migration

![5-instance-migration-between-az.png](images%2F5-instance-migration-between-az.png)

- 인스턴스를 하나의 AZ에서 다른 AZ로 마이그레이션하고 싶다면 AMI를 이용하는 방법이 있다.
- 이미지를 예시로 "us-east-1a"에서 "us-east-1b"로 옮길 수 있다.
- 먼저 AMI를 EC2 인스턴스에서 가져오고 AMI를 다른 AZ로 새 EC 인스턴스로 복원한다.
- AMI가 원본에서 옮겨졌기 때문에 기본값으로 같은 데이터와 파일 시스템 애플리케이션을 가지게 된다.

#### Cross-Account AMI 공유

![6-cross-account-ami-sharing.png](images%2F6-cross-account-ami-sharing.png)

- AMI를 다른 AWS 계정과 공유할 수 있다.
- AMI 공유는 AMI 소유권에 영향을 주지 않는다.
- 암호화되지 않은 볼륨과 고객 관리형 키로 암호화된 볼륨이 있는 AMI만 공유할 수 있다.
- 암호화된 볼륨과 AMI를 공유하는 경우 이를 암호화하는 데 사용되는 고객 관리형 키도 공유해야 한다.

#### KMS로 암호화된 AMI 공유

![7-ami-sharing-with-kms-encryption.png](images%2F7-ami-sharing-with-kms-encryption.png)

- AMI가 CMK로 암호화되어 있다.
- AMI를 "Account B"와 공유하기 위해서는 KMS 키도 공유해야 한다.
- 대상 계정에 복호화와 재암호화 등을 위하여 키에 대한 권한을 제공해야 한다.

#### Cross-Account AMI 복제

![8-ami-sharing-with-copy.png](images%2F8-ami-sharing-with-copy.png)

- 사용자의 계정과 공유된 AMI를 복사하면 사용자의 계정에 있는 AMI의 소유자가 된다.
- 소스 AMI의 소유자는 AMI(EBS 스냅샷)를 지원하는 스토리지에 대한 읽기 권한을 부여해야 한다.
- 공유 AMI에 암호화된 스냅샷이 있는 경우 사용자는 키를 공유해야 한다.
- 복사하는 동안 자체 CMK로 AMI를 암호화할 수 있다.

![9-ami-copy-kms-encryption.png](images%2F9-ami-copy-kms-encryption.png)

- 공유된 AMI를 암호화하면서 복사를 할 수 있다.
- 기본 EBS 스냅샷을 공유하고 KMS 키에 대상 계정에 권한을 부여한다.
- 대상 계정은 복사 명령을 실행할 수 있다.
  - CMK-A를 이용해 복호화하여 EBS 스냅샷을 다시 암호화하고 CMK-B와 그 고유 계정으로 다시 암호화한다.
- 공유된 AMI가 있다면 AMI를 복사할 수 있다. 그럼 나중에 다른 것을 공유하지 않고도 자신만의 AMI를 갖게 된다.

---

### EC2 Image Builder

- "Virtual Machine"이나 컨테이너 이미지 생성을 자동화하는데 사용된다.
  - EC2 AMI 생성, 유지, 검증 및 테스트를 자동화한다.
- "주단위", "패키지 업데이트가 있을 때"와 같이 특정 스케쥴에 맞춰 실행될 수 있다.
- **무료 서비스로 사용한 리소스에 대한 비용만 지불하면 된다.**

![10-ec2-image-builder.png](images%2F10-ec2-image-builder.png)

---

### 운영 환경에서의 AMI

![11-ami-production-1.png](images%2F11-ami-production-1.png)

- IAM 정책을 사용하여 사용자가 사전 승인된 AMI(특정 태그로 태그가 지정된 AMI)에서만 EC2 인스턴스를 시작하도록 할 수 있다.

![12-ami-production-2.png](images%2F12-ami-production-2.png)

- AWS Config와 결합하여 비준수 EC2 인스턴스(승인되지 않은 AMI로 시작된 인스턴스)를 찾는다.

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [Image Builder 공식문서](https://docs.aws.amazon.com/imagebuilder/)