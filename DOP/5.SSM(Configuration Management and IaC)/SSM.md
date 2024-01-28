# SSM

- 이번 장에서는 **DevOps Engineer Professional (DOP)** 을 준비하며 "System Manager의 약자인 SSM"에 대해서 알아보도록 한다.

---

### Systems Manager 개요

- EC2 및 On-Premise 시스템을 대규모로 관리하도록 지원한다.
- 인프라 상태에 대한 운영 통찰력을 얻을 수 있는 많은 도구가 있다.
- 문제를 쉽게 감지하게 해주고 자동화된 패칭과 규정 준수를 향상시킨다.
    - 인스턴스를 패치해야 할 때마다 일반적으로 "System Manager"를 사용하게 된다.
    - 실행할 모든 자동화도 "System Manager"를 사용하게 된다.
- Windows와 Linux OS에서 작동하고 CloudWatch Metric 및 대시보드와 완전히 통합된다.
- 무료로 사용할 수 있으며, 생성한 리소스에 대한 비용만 지불하면된다.

#### Features

- "System Manager"에는 아래와 같이 수없이 많은 기능들이 있다.
- **Resource Groups**
- Operations Management
    - Explorer
    - OpsCenter
    - CloudWatch Dashboard
    - PHD
    - Incident Manager
- Shared Resources
    - **Documents**
- Change Management
    - Change Manager
    - **Automation**
    - Change Calendar
    - **Maintenance Windows**
- Application Management
    - Application Manager
    - AppConfig
    - **Parameter Store**
- Node Management
    - Fleet Manager
    - Compliance
    - **Inventory**
    - Hybrid Activations
    - **Session Manager**
    - **Run Command**
    - **State Manager**
    - **Patch Manager**
    - Distributer

#### 작동 방식

- 우리가 제어하는 시스템에 "SSM 에이전트"를 설치해야 한다.
- Amazon Linux 2 AMI 및 일부 Ubuntu AMI에 기본적으로 설치되어 있다.
- **SSM으로 인스턴스를 제어할 수 없다면 "SSM 에이전트"에 문제가 있을 가능성이 높다.**
- SSM 작업을 허용하려면 EC2 인스턴스에 적절한 IAM 역할이 있는지 확인해야 한다.

![1-system-manager-works.png](images%2F1-system-manager-works.png)

---

### AWS Tags

- 태그라는 텍스티 키-값 쌍을 여러 AWS 리소스에 추가할 수 있다.
- 일반적으로 EC2에 사용되며 이외에도 많은 리소스들에 사용된다.
- 자유로운 네이밍, 공통 태그인 이름(Name), 환경(Environment), 팀 등으로 구성할 수 있다.
- 리소스 그룹화, 자동화, 보안 및 비용 할당 등에 사용된다.

#### Resource Groups

- 태그를 이용해서 리소스 그룹을 생성할 수 있다.
- 같은 태그를 공유한다면 두 개의 리스소를 함께 그룹화할 수 있다.
- 예를 들어, "애플리케이션을 그룹화"하거나 "동일한 애플리케이션 스택의 여러 레이어를 그룹화"하거나 혹은 "생산 환경과 개발 환경을 구분"할 수 있게 해준다.

![2-resource-groups.png](images%2F2-resource-groups.png)

- 예시에서는 3개의 인스턴스 중에 2개는 `Environment = Dev` 태그를 추가하였고, 나머지 하나는 `Environment = Prod` 태그를 추가하였다.
- 처음 두 인스턴스는 논리적 그룹을 생성한다. (Region 단위에서)
- EC2 인스턴스 이외에 S3, DynamoDB, Lambda 등에서도 사용될 수 있다.
- **이러한 리소스 그룹을 생성하는 이유는 SSM을 그룹 수준에서 직접 운영하기 위해서다.**
- 예를 들어, 운영 체제를 패치하고 몇가지 작업을 수행할 수 있다.

---

### SSM Document

![3-ssm-documents-1.png](images%2F3-ssm-documents-1.png)

- "Document"는 JSON 또는 YAML 형식으로 작성할 수 있다.
- 매개 변수를 정의하여 "Document"가 하는 일을 정의한다.
- "Document"는 특정 서비스에 의해 실행된다.
- 이미 많은 "Document"가 AWS에 존재하기 때문에 우리는 그것을 활용하여 작업을 빠르게 진행할 수 있다.

![4-ssm-documents-2.png](images%2F4-ssm-documents-2.png)

- SSM을 많이 사용하게 되면 많은 SSM 문서를 작성하게 되며 "CloudFormation"과 개념이 비슷하다.
- 문서에는 "State Manager", "Patch Manager", "Automation", "SSM Parameter Store"와 같은 데이터를 검색할 수 있다.
- 모듈성과 동적인 기능으로 문서들이 작동하도록 할 수 있다.

#### Run Command

![5-ssm-run-command.png](images%2F5-ssm-run-command.png)

- 문서(= 스크립트)를 실행하거나 명령만 실행한다.
- 리소스 그룹을 사용하여 여러 인스턴스에 걸쳐 명령을 실행한다.
- 속도 제어/오류 제어를 사용할 수 있다.
- IAM 및 CloudTrail과 통합된다.
- 명령을 실행시키기 위해서 SSH가 필요하지 않다.
- 명령 출력은 콘솔에 표시되고 S3 버킷 또는 CloudWatch Logs로 전송될 수 있다.
- 명령 상태(진행 중, 성공, 실패 등)에 대해 SNS에 알림을 보낸다.
- EventBridge를 사용하여 호출이 가능하다.

---

### Automation

- EC2 인스턴스 및 기타 AWS 리소스의 일반적인 유지 관리 및 배포 작업을 단순화한다.
- 예를 들어, 인스턴스를 재시작하거나, AMI, EBS 스냅샷을 생성하는 작업을 할 수 있다.
- **Automation Runbook**
    - 자동화 유형의 SSM 문서
    - EC2 인스턴스나 AWS 리소스에 미리 정보를 주고 수행되는 작업을 정의한다.
    - AWS로 만든 사전 정의된 "Runbook"을 사용할 수도 있다.

![6-ssm-automations.png](images%2F6-ssm-automations.png)

- EBS 볼륨, AMI, RDS의 스냅샷을 생성하는 작업 등에 사용된다.
- "AWS Console", "AWS CLI", "SDK"를 통해 수동으로 실행할 수 있다.
- "EventBridge"의 규칙으로 자동화할 수 있다.
- 유지 관리 기간을 사용하는 일정에 따라 규칙 교정으로 사용한다.

---

### SSM Parameter Store

- 구성 및 비밀을 위한 안전한 저장소다.
- 선택적으로 KMS를 통해 구성들을 암호화할 수 있따.
- Serverless 방식의 확장 가능하며 내구성이 좋으며 SDK를 통해 간편하게 사용할 수 있다.
- 매개 변수를 업데이트하면 버전 트래킹을 할 수 있다.
- 보안은 IAM을 통해서 제공된다.
- "EventBridge"로 알림을 받을 수 있다.
- "CloudFormation"과 완벽하게 통합된다.
  "CloudFormation"이 매개 변수 저장소의 매개 변수를 입력 매개 변수로 활용할 수 있다.

![7-ssm-parameter-store.png](images%2F7-ssm-parameter-store.png)

- 응용 프로그램과 SSM 매개 변수 저장소가 있다.
- 이미지에서 처럼 일반 텍스트 구성을 저장할 수 있다.
- 응용 프로그램의 IAM 사용 권한이 확인된다.
- EC2 인스턴스 역할이나 암호화된 구성을 가질 수 있다.
- 이러한 경우 SSM 매개 변수 저장소는 KMS로 암호화되고 복호화될 것이다.
- 우리가 실행시키는 프로그램이 기본 KMS 키에 액세스할 수 있는 권한이 있어야 암호화와 복호화를 할 수 있다.

#### Hierarchy

![8-ssm-parameter-store-hierarchy.png](images%2F8-ssm-parameter-store-hierarchy.png)

- 계층 구조와 함께 매개 변수를 저장할 수 있다.
- 예를 들어, `/my-department` 안에 `/my-app`, 안에 `/dev`, `/prod` 경로를 나누어 원하는 정보를 넣을 수 있다.
- 필요에 따라 특정 부서에 액세스할 수 있도록 정책을 생성할 수 있다.
- **Parameter Store를 통해 Secret Manager에 접근할 기회도 있다.**
- AWS가 발행하는 Public 파라미터도 사용할 수 있다.
- 우리가 사용하는 특정 지역에서 최신 AMI를 찾으려면 `/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2` 경로의 매개 변수를 활용하면 된다.

#### Standard vs Advanced

![9-standard-advanced-parameter-tiers.png](images%2F9-standard-advanced-parameter-tiers.png)

- Standard와 Advanced 계층이 있다.
- **가장 큰 차이는 매개 변수 값의 크기다.**
- 가용성 또한 Advanced에는 있지만 Standard에는 없다.

#### Policies

![10-parameters-policies.png](images%2F10-parameters-policies.png)

- "Advanced Parameter"에서만 지원한다.
- 매개 변수에 TTL을 할당할 수 있도록하여 패스워드 같은 민감한 데이터를 업데이트하거나 삭제하도록 강요할 수 있다.
- 한 번에 여러가지 정책을 할당할 수 있다.
- 매개 변수 삭제가 필요한 경우 "EventBridge"와 통합하여 만료되기 15일 전에 알람을 받을 수 있다.

---

### SSM Patch Manager

- 관리형 인스턴스 패치 프로세스를 자동화한다.
- OS 업데이트, 응용 프로그램 업데이트, 보안 업데이트 등이 포함된다.
- EC2 인스턴스와 On-Premise 서버를 지원하고 Linux, macOS, Windows를 지원한다.
- **Maintenance Windows(유지 관리 기간)**를 사용하여 주문형 또는 일정에 따라 패치를 적용한다.
- 인스턴스를 스캔하고 패치 규정 준수 보고서(누락된 패치)를 생성한다.
- 패치 규정 준수 보고서를 S3로 보낼 수 있다.
- "Patch Manager"는 두 가지 구성 요소를 가지고 있다.
- **Patch Baseline**
    - 인스턴스에 설치해야 할 패치와 설치하지 말아야 할 패치를 정의한다.
    - 인스턴스에서 Approved/Rejected 패치를 지정하고 싶다면 "Patch Baseline"을 만드는 기능은 사용자의 몫이다.
    - 패치는 출시하면 며칠 안에 자동으로 승인(auto-approved)받을 수 있다.
    - 기본값으로 "Patch Baseline"은 SSM에서 관리되는 인스턴스에 보안과 관련된 중요한 패치만을 설치한다.
- **Patch Group**
    - 인스턴스 세트를 특정 "Patch Baseline"과 연결한다.
    - 예를 들어, 개발, 테스트, 운영을 위한 패치 그룹을 생성할 수 있다.
    - **인스턴스는 패치 그룹 태그 키를 사용하여 정의되어야 한다.**
    - 인스턴스는 하나의 패치 그룹에만 속할 수 있다.
    - 패치 그룹은 하나의 "Patch Baseline"에만 등록할 수 있다.

#### Patch Baselines

- "Patch Manager"에는 두 종류의 "Patch Baseline"이 있다.
- **Pre-Defined Patch Baseline**
    - 다양한 운영 체제에 대해 AWS에서 관리하며 수정할 수 없다.
    - **AWS-RunPatchBaseline(SSM Document)**: 운영 체제와 애플리케이션 패치(Linux, macOS, Windows Server)를 모두 적용한다.
- **Custom Patch Baseline**
    - 자신만의 패치 기준을 만들고 자동 승인할 패치를 선택한다.
    - 운영 체제, 허용된 패치, 거부된 패치 등을 포함한다.
    - 사용자 정의 및 대체 패치 저장소를 지정하는 기능을 제공한다.

![11-patch-manager.png](images%2F11-patch-manager.png)

- 위의 이미지에는 "SSM Agent"에서 실행되는 3가지 유형의 2개의 EC2 인스턴스가 있다.
- 첫 번째는 `OS = Windows` 태그와 `Patch Group = Dev`에 태그된다.
- 두 번째는 `OS = Windows`이며, 세 번째는 `OS = Windows` 이다.
- 첫 번째 "Patch Baseline"은 기본값 "Patch Group"에 연결된다. "Patch Group"이 기본값으로 정의되지 않았을 때다.
  특정 "Patch Group"을 갖지 않는 인스턴스는 첫 번째 패치 기준 ID를 갖게 된다.
- 두 번째 "Patch Group"은 Dev를 실행하며 기본 "Patch Baseline"이 아니다. 특정 Patch Baseline ID가 있다.
- 명령은 "AWS Run Patch Baseline"이라는 문서를 실행할 것이다.
  콘솔, SDK, Maintenance Windows에서 실행할 수 있다.
- **Patch Manager는 인스턴스를 패치하는 용도로만 사용된다.**

#### Maintenance Windows

- 인스턴스에서 작업을 수행할 시기에 대한 일정을 정의한다.
- OS 패치, 드라이버 업데이트, 소프트웨어 설치 등을 할 수 있다.
- "Maintenance Windows"에는 Schedule, Duration, 등록된 인스턴스 집합, 등록된 작업 집합이 포함된다.

---

### SSM Session Manager

- EC2 및 On-Premise 서버에서 보안 Shell 환경을 시작할 수 있다.
- 콘솔, CLI, Session Manager SDK를 통해서 액세스할 수 있다.
- **SSH 액세스, Bastion Host 또는 SSH 키를 사용하지 않는다는 장점이 있다.**

![12-session-managers.png](images%2F12-session-managers.png)

- "SSM Agent"를 실행하는 EC2 인스턴스가 어디 있는지, SSM 서비스로 등록할 수 있는 올바른 권한을 가지고 있는지만 확인하면 된다.
- 사용자는 올바른 IAM 권한을 가지고 "Session Manager" 서비스에 연결할 수 있다.
- 인스턴스에 대한 연결 및 실행된 명령을 기록한다.
- 세션 로그 데이터는 S3 또는 "CloudWatch Logs"로 전송될 수 있다.
  이를 통해 더 많은 통제와 안전이 보장된다.
- CloudTrail은 "StartSession" 이벤트도 가로챌 수 있다.

![13-session-managers-2.png](images%2F13-session-managers-2.png)

- IAM 권한
    - "Session Manager"에 액세스할 수 있는 사용자/그룹과 인스턴스를 제어한다.
    - 태그를 사용하여 특정 EC2 인스턴스에만 액세스하도록 제한할 수 있다.
    - 이미지를 예로 들어, 리소스 유형 환경이 `dev`라면 SSM 접근, S3 쓰기, CloudWatch 쓰기가 허용된다.
- 선택적으로 사용자가 실행할 수 있는 명령도 제한할 수 있다.

#### SSH vs Session Manager

- SSH 접속과 Session Manager를 통한 접속은 아래와 같은 차이가 있다.

![14-ssh-vs-session-manager.png](images%2F14-ssh-vs-session-manager.png)

- SSH를 사용하면 특정 IP 주소를 인스턴스로 허용하기 위한 보안 그룹이 필요하다.
  반면에 "Session Manager"를 사용하면 인바운드 규칙이 필요없다.
- "SSM Agent"와 올바른 IAM 역할이 있는 인스턴스만 있으면 올바른 권한을 가진 사용자가 "Session Manager"를 이용해 EC2 인스턴스에 대한 명령을 실행할 수 있다.
- 세션의 모든 데이터는 S3나 "CloudWatch Logs"에 저장될 수 있다.

#### Default Host Management Configuration

- 활성화되면 EC2 인스턴스 프로필을 사용하지 않고도 EC2 인스턴스를 관리형 인스턴스로 자동 구성한다.
- Instance Identity Role: AWS 서비스에 대해 EC2 인스턴스를 식별하는 것 이상의 권한이 없는 IAM 역할 유형이다.

![15-default-host-management-configuration.png](images%2F15-default-host-management-configuration.png)

- EC2 인스턴스에는 IMDSv2가 활성화되고 SSM 에이전트가 설치되어 있어야 한다.
  - IMDSv1은 지원하지 않는다.
- Session Manager, Patch Manager 및 Inventory를 자동으로 활성화한다.
- SSM 에이전트를 자동으로 최신 상태로 유지한다.
- AWS 리전별로 활성화해야 한다.

#### Hybrid Environments

- 온프레미스 서버, IoT 디바이스, 엣지 디바이스 및 가상 머신(예: 다른 클라우드 제공업체의 VM)을 관리하도록 Systems Manager를 설정할 수 있다.
- Systems Manager 콘솔에서 EC2 인스턴스는 접두사 "i-"를 사용하고 하이브리드 관리형 노드는 접두사 "mi-"를 사용한다.

![16-hybrid-environment.png](images%2F16-hybrid-environment.png)

- 온프레미스 데이터 센터의 서버를 등록하려면 Systems Manager에 하이브리드 인증을 생성하고 인증 코드와 ID를 가져와야 한다.
- 대상 서버에 SSM 에이전트를 설치하고 이전에 가져온 인증 코드와 ID로 Systems Manager에 SSM Agent를 등록한다.
- "mi"가 뒤에 인스턴스 ID가 붙은 관리형 인스턴스가 표시된다.

![17-automating-ssm-hybrid-activations-creation.png](images%2F17-automating-ssm-hybrid-activations-creation.png)

- API 게이트웨이, 람다, Systems Manager를 사용하여 아키텍처로 자동화할 수 있다.
- 서버가 시작되면 API 게이트웨이에 GET 요청을 전송한다.
- 페이로드가 람다로 전달되고 API 호출을 통해 Systems Manager에 하이브리드 인증을 생성한다.
- 인증 코드와 ID를 가져와 API 게이트웨이를 거쳐 서버에 전달되며 이를 사용해 서버가 스스로 등록할 수 있다.

#### IoT Greengrass Instance Activation

- SSM을 사용하여 IoT Greengrass Core 디바이스를 관리할 수 있다.
- Greengrass Core 디바이스에 SSM 에이전트를 설치하고 SSM에 관리형 노드로 등록해야 한다.
- SSM 에이전트는 수동으로 설치하거나 Greengrass 구성 요소로 배포할 수 있따.
  - Greengrass 구성 요소는 Greengrass 코어 디바이스에 직접 배포하는 사전 구축된 소프트웨어 모듈이다.
- System Manager와 통신하려면 토큰 교환 역할(IoT Core 디바이스의 IAM 역할)에 권한을 추가해야 한다.
- 모든 SSM 기능을 지원한다.
  - Patch Manager, Session Manager, Run Command 등..
- Greengrass Core 디바이스 전체에서 OS 및 소프트웨어 업데이트를 쉽게 업데이트하고 유지 관리할 수 있다.

![18-iot-greengrass-instance-activation.png](images%2F18-iot-greengrass-instance-activation.png)

- Greengrass 코어 디바이스는 IoT 장치로 SSM 에이전르를 설치한 다음 코어 디바이스의 IAM 역할인 토큰 교환 역할을 정의하고 올바른 권한을 부여하면 Systems Manager에 등록된다.
- Systems Manager도 코어 디바이스를 관리하기 위해 필요한 서비스 역할이 할당되어 있어야 한다.

#### Automation - Use Cases

![19-automation-use-case.png](images%2F19-automation-use-case.png)

- 상단의 예시는 EC2 인스턴스 및 RDS DB 인스턴스를 자동으로 시작 및 중지하여 비용을 절감하고 있다.
  - QA 환경이 있을 때 EventBridge로 스케줄을 설정해 SSM Automation이 EC2 인스턴스나 RDS 인스턴스를 자동으로 시작하도록 만들 수 있다.
  - 오후 5시에 EventBridge의 스케줄을 설정해 SSM Automation으로 EC2 인스턴스와 RDS 인스턴스를 중단할 수 있다.
- 하단의 예시는 EC2 인스턴스 및 RDS DB 인스턴스의 크기를 자동으로 축소하여 비용을 절감하고 있다.
  - 시작하고 중단하는 대신 밤에 용량을 줄이고 낮에 용량을 늘리려면 SSM Automation에서 AWS-ResizeInstance를 사용할 수 있다.
  - 낮에는 m5.2xlarge로 크기를 늘리고 밤에는 m5.large로 크기를 줄인다.
  
![20-build-golden-ami.png](images%2F20-build-golden-ami.png)

- EventBridge가 SSM Automation에서 매주 AWS-CreateImage 자동화를 실행해 AMI를 생성한다.
- AMI가 생성되면 람다 함수를 호출해 AMI ID를 전달하고 Parameter Store에 AMI ID를 저장한다.
- EC2 Image Builder와 유사하지만 SSM Automation을 사용하는 방식이다.

![21-automation-config.png](images%2F21-automation-config.png)

- SSM Automation은 AWS Config와 긴밀하게 통합되어 있다.
- 예를 들어, Config에서 버킷의 버전관리 활성화 여부를 감지하는 규칙이 있을 때, 버킷 버전 관리가 비활성화되어 있으면 규정이 준수되지 않고 있음을 알 수 있다.
- Config 문제 해결을 위해 Automation을 실행하도록 설정하면 SSM Automation에서 AWS-ConfigureS3BucketVersioning이 S3 버킷의 버전 관리를 활성화한다.

#### Compliance

- 패치 규정 준수 및 구성 불일치가 있는 관리형 노드 전체를 검사한다.
- 아래에 대한 현재 데이터를 표시한다.
  - Patches in Patch Manager
  - Associations in State Manager
- 리소스 데이터 동기화를 사용하여 데이터를 S3 버킷에 동기화할 수 있으며 Athena 및 QuickSight를 사용하여 분석할 수 있다.
- 여러 계정 및 리전에서 데이터를 수집하고 집계할 수 있다.
- 규정 준수 데이터를 Security Hub로 보낼 수 있다.

![22-compliance.png](images%2F22-compliance.png)

- S3 버킷에 데이터를 연동해 여러 리전에 걸쳐 규정 준수 데이터를 수집할 수 있다.
  - 이러한 수집 작업을 리소스 데이터 동기화라고 한다.
- S3 버킷에 모든 데이터를 수집하고 Athena나 QuickSight 등을 사용해 데이터를 분석할 수 있다.

#### OpsCenter

- 한 곳에서 문제를 확인, 조사 및 해결할 수 있다.
  - 다른 AWS 서비스를 탐색할 필요가 없다.
- 보안 문제 (Security Hub), 성능 문제 (DynamoDB 쓰로틀), 실패 (ASG 인스턴스 시작 실패) 등..
- 문제 해결을 위한 대기 시간이 단축된다.
- OpsItems
  - 조사 및 해결이 필요한 운영 문제 또는 중단을 해결한다.
  - 이벤트, 리소스, AWS Config 변경, CloudTrail 로그, EventBridge 등 다양한 문제가 있을 수 있다.
  - **문제 해결을 위한 권장 Runbook을 제공하거나 자동화를 사용해 문제를 해결할 수 있는지 알려준다.**
- EC2 인스턴스와 온프레미스 관리형 노드를 모두 지원한다.

![23-opscenter.png](images%2F23-opscenter.png)

- CloudWatch & Application Insights, EventBridge, Config, Security Hub, DevOps Guru, SSM Incident Manager 등의 모든 데이터가 OpsCenter에 수집된다.
- 이를 바탕으로 알림을 보내고 자동화를 구축할 수 있다.

![24-reduce-cost-deleting-orphaned-ebs-volumes.png](images%2F24-reduce-cost-deleting-orphaned-ebs-volumes.png)

- 오랫동안 사용되지 않은 EBS 볼륨을 스캔할 때, OpsCenter에서 이러한 볼륨을 OpsItem으로 등록한 다음 삭제하면 된다.
- EventBridge에서 매일 람다 함수를 호출해 EC2 서비스를 스캔한다.
- 예를 들어, 45일이 지난 EBS 볼륨을 찾는 상황에서 만약 해당하는 볼륨을 찾는다면 OpsCenter에서 직접 OpsItem을 생성한다.
- OpsCenter에서 Document를 실행하거나 스냅샷을 생성하거나 스냅샷이나 볼륨을 삭제하도록 설정해 SSM Automation에서 실행할 수 있다.

#### SSM Session Manager (VPC Endpoint)

![25-ssm-session-manager.png](images%2F25-ssm-session-manager.png)

- Systems Manager 서비스에서 Private 서브넷에 있는 EC2 인스턴스에 접근해야 하는 상황이다.
- 예를 들어, Session Manager로 EC2 인스턴스에 연결하려고 하는데 인터넷에 연결되어 있지 않는 상황이다.
- 이럴 때는 VPC 엔드포인트를 사용하여 연결해야 한다.
- 먼저 SSM 서비스에 VPC 엔드포인트가 필요하다.
- 보안 그룹에서 인바운드 포트 443을 연다.
  - SSM Session Manager 서비스에 인터페이스 엔드포인트가 필요하다.
  - 엔드포인트가 Session Manager 서비스에서 EC2 인스턴스에 비공개로 연결할 수 있도록 만들어준다.
- 인바운드 포트 443을 열어준다.
  - EC2 인스턴스에서는 이 서비스에 대해 아웃바운드 포트 443을 연다.
  - KMS 암호화를 사용한다면 KMS에도 VPC 인터페이스 엔드포인트를 생성해야 한다.
- CloudWatch Logs를 사용해 Session Manager의 세션에서 CloudWatch에 로그를 전송하려면 CloudWatch에도 VPC 엔드포인트를 생성해야 한다.
- S3에 로그를 전송한다면 S3의 VPC 게이트웨이 엔드포인트를 사용해야 한다.
- VPC 게이트웨이 엔드포인트를 사용한다면 VPC에서 라우팅 테이블을 업데이트해야 한다.

---

### AWS OpsWorks

- "Chef & Puppet"을 사용하면 서버 구성을 자동으로 수행하거나 반복 작업을 수행할 수 있다.
- EC2 및 On-Premise VM과 잘 작동한다.
- "AWS OpsWorks"는 "관리형 Chef" 또는 "관리형 Puppet"라고 볼 수 있다.
- "AWS SSM"의 대안으로 사용되며 더 많은 기능을 제공한다.
- 구성을 코드로 관리하는 데 도움이 된다.
- 일관된 배포에 도움이 된다.
- Linux/Windows 에서 작동된다.
- 사용자 계정, cron, ntp, packages, services 등을 자동화할 수 있다.
- "Recipes" 또는 "Manifests"라는 개념을 사용한다.
- "Chef / Puppet"은 SSM / Beanstalk / CloudFormation과 유사하다.
  하지만 Cloud 전반에서 작동하는 오픈 소스 도구다.

- OpsWorks Stacks에 대한 자세한 내용은 [공식 홈페이지](https://docs.aws.amazon.com/opsworks/latest/userguide/welcome_classic.html)를 확인하도록 한다.
- OpsWorks 라이프사이클 이벤트에 대한 자세한 내용은 [공식 홈페이지](https://docs.aws.amazon.com/opsworks/latest/userguide/workingcookbook-events.html)를 확인하도록 한다.

---

### 참고한 강의

- [DevOps Engineer Professional](https://www.udemy.com/course/aws-certified-devops-engineer-professional-korean)
- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)