# Deployment & Instance Management

- 이번 장에서는 **Solutions Architect Professional (SAP)** 을 준비하며 "배포와 인스턴스 관리"에 대해서 알아보도록 한다.

---

### AWS Elastic Beanstalk

- Elastic Beanstalk는 AWS에 애플리케이션을 배포하는 개발자 중심적인 관점이다.
- AWS의 많은 요소들을 사용한다.
  - EC2, Auto Scaling Group, Elastic Load Balancer, RDS 등..
- 많은 요소들을 하나의 관점으로 볼 수 있기 때문에 이해하기 쉽다.
- 하나의 관점을 바라보지만 각 구성 요소들을 완전히 제어할 수 있다.
- Beanstalk에 대한 사용료는 지불할 필요가 없으며 사용되는 서비스들에 대한 비용만 지불하면 된다.
- 다양한 형식의 플랫폼을 지원한다.
  - Go, Java SE, Java with Tomcat, .NET on Windows Server with IIS, Node.js, PHP, Python, Ruby, Packer Builder
  - Single Container Docker, Multi Container Docker, Preconfigured Docker
- 지원되지 않는 경우 사용자 지정 플랫폼(advanced)을 작성할 수 있다.
- Beanstalk는 온프레미스에서 클라우드로 애플리케이션을 "리플랫폼"할 때 유용하게 사용된다.

- **Managed service**
  - "인스턴스 구성 및 OS"는 Beanstalk에서 처리한다.
  - 배포 전략은 구성 가능하지만 Beanstalk에 의해 수행된다.
- 애플리케이션 코드만 개발자의 책임이다.

#### Environment

- 세 가지 아키텍처 모델을 제공한다.

![1-beanstalk-environment.png](images%2F1-beanstalk-environment.png)

- 단일 인스턴스 배포
  - 개발 환경에 적합하다.
  - EC2 인스턴스가 있고 EC2 인스턴스에 연결된 EIP가 있다.
  - EC2 인스턴스가 대체되는 경우 EIP를 새로운 인스턴스로 옮길 수 있으므로 업데이트할 때 유용하게 사용될 수 있다.
  - RDS를 구축하여 EC2 인스턴스와 통신되도록 할 수 있다.
- LB + ASG
  - 운영 또는 사전 운영 애플리케이션에 적합하다.
  - 예시에서는 두 개의 가용지역을 사용하고 있지만 필요한 경우 세 개의 가용지역도 사용하도록 구축할 수 있다.
  - Auto Scaling Group을 통해서 더 많거나 적은 트래픽을 처리하기 위해 시간에 따라 스케일링할 수 있다.
- ASG Only
  - 운영 중인 웹이 아닌 애플리케이션에 적합하다.
  - SQS 큐가 Beanstalk에 의해 관리되고 워커 계층은 SQS 큐의 부하에 따라 스케일링된다.

#### Web Server vs Worker Environment

- 애플리케이션에서 완료해야 할 작업을 수행하는 경우 이러한 작업을 전용 작업자 환경으로 오프로드한다.
- 애플리케이션을 두 계층으로 분리하는 것은 일반적으로 사용되는 방식이다.
- 예를 들어, 동영상 처리, zip 파일 생성등의 작업에 사용될 수 있다.
- `cron.yaml` 파일에서 주기적으로 실행되는 태스크를 정의할 수 있다.

![2-web-server-worker-environment.png](images%2F2-web-server-worker-environment.png)

- Web 계층에는 ELB와 EC2가 구성되어 있으며 고객의 요청을 받는 역할을 한다.
- 자동 확장을 위하여 Auto Scaling Group을 사용하고 있다.
- Worker 계층에는 SQS와 EC2가 구성되어 있으며 Web 계층에서 SQS 큐에 넣은 데이터를 처리한다.
- SQS 큐의 크기에 맞추어 EC2 인스턴스가 Auto Scaling된다.

#### Beanstalk Deployment Blue / Green

- Beanstalk만의 기능은 아니며 전통적인 배포 방식이다.
- 새로운 애플리케이션을 배포하기 위한 가동 중지 시간이 0이다.
- 새로운 "Stage" 환경을 생성하고 새로운 버전을 배포할 수 있다.
  - 즉, 새로운 Beanstalk 환경을 배포한다.
- Route53 가중치 정책을 사용하여 새로운 환경으로 트래픽을 조금씩 리디렉션하도록 설정할 수 있다.
- 새로운 환경의 테스트가 완료되면 "swap URLs" (DNS Swap)을 사용하여 모든 트래픽이 새로운 버전을 향하도록 구현할 수 있다.

![3-beanstalk-blue-green.png](images%2F3-beanstalk-blue-green.png)

---

### AWS CodeDeploy

![4-codedeploy.png](images%2F4-codedeploy.png)

- 많은 EC2 인스턴스에 애플리케이션을 자동으로 배포하는 경우에 사용된다.
- 인스턴스들은 Elastic Beanstalk에 의해 관리되지 않는다.
- 오픈 소스 툴(Ansible, Terraform, Chef, Puppet 등)을 사용하여 배포를 처리하는 여러 가지 방법이 있다.
- AWS 관리형 서비스인 AWS CodeDeploy를 사용하여 인스턴스에 새로운 애플리케이션을 배포할 수 있다.
- CodeDeploy는 EC2, ASG, ECS, Lambda를 배포할 수 있다.

#### EC2

- `appspec.yml` + 배포 전략을 사용하여 애플리케이션을 배포하는 방법을 정의할 수 있다.
- EC2 인스턴스 플릿에 대한 현장 업데이트를 수행할 수 있다.
- 후크를 사용하여 각 배포 단계 후 배포를 확인할 수 있다.

![5-codedeploy-to-ec2.png](images%2F5-codedeploy-to-ec2.png)

- V1인 EC2 인스턴스가 4개 있다.
- 그중 두 개의 인스턴스는 오프라인 상태로 변경되고, V2 애플리케이션으로 변경된다.
- 남은 두 개의 V1 인스턴스가 오프라인 상태로 변경되고, V2 애플리케이션으로 변경된다.
- 결과적으로 모든 인스턴스의 애플리케이션 버전이 V2로 변경된다.

#### ASG

- **In place update**
  - 기존 EC2 인스턴스를 업데이트한다.
  - ASG에서 새로 생성한 인스턴스도 자동화된 배포 기능을 제공한다.
- **Blue / green deployment**
  - 새로운 Auto Scaling Group이 생성된다. (설정 복사)
  - 이전 인스턴스를 유지할 기간을 선택한다.
  - ELB를 사용해야 한다.

![6-codedeploy-to-asg.png](images%2F6-codedeploy-to-asg.png)

- 트래픽을 처리하는 ALB가 있고 시작 템플릿 v1을 사용하여 실행된 EC2 인스턴스가 있다.
- CodeDeploy는 Blue/Green 배포를 위해 시작 템플릿 v2를 사용하여 새로운 EC2 인스턴스들을 생성한다.
- 시작 템플릿 v2를 사용하여 생성된 인스턴스가 정상이라고 판단되면 v1을 사용하여 생성된 인스턴스들을 종료한다.

#### Lambda

- 트래픽 시프트 기능을 제공한다.
- 배포를 검증하기 위한 트래픽 후크 사전 및 사후 기능을 제공한다.
  - 트래픽 시프트 시작 전 및 종료 후
- CloudWatch 알람을 사용하여 간편하고 자동화된 롤백을 제공한다.
- SAM 프레임워크는 기본적으로 CodeDeploy를 사용한다.

![7-codedeploy-to-lambda.png](images%2F7-codedeploy-to-lambda.png)

- v1 람다 함수가 실행되고 있다.
- CodeDeploy를 통해 v2 람다 함수가 배포된다.
- 별칭을 통해서 v1과 v2 람다 함수에 트래픽이 시프트된다.
- 사전 트래픽 후크와 사후 트래픽 후크를 통하여 테스트를 진행하고 람다 함수의 정상 여부를 판단할 수 있다.

#### ECS

- Amazon ECS 및 AWS Fargate를 위한 Blue/Green 배포를 지원한다.
- 설정은 ECS 서비스 정의 내에서 수행된다.
- 새로운 태스크 세트가 생성되고 트래픽이 새로운 태스크 테스트로 라우팅된다.
- X분 동안 모든 것이 안정적이라고 판단되면 이전 태스크 세트는 종료된다.
  - 새로운 버전의 문제를 파악할 수 있는 시간이 제공된다.
- 카나리 배포를 지원한다. (`Canary10Percent5Minute`)

![8-codedeploy-ecs.png](images%2F8-codedeploy-ecs.png)

---

### AWS CloudFormation

- AWS IaC(Infrastructure as Code) 서비스다.
- 여러 계정 및 리전에 걸쳐 스택을 생성할 수 있다.
  - 인프라를 재사용하고 신속하게 원하는 곳으로 재배포할 수 있다.
- Elastic Beanstalk 서비스를 중추로 사용한다.
- Service Catalog 서비스를 중추로 사용한다.
- SAM(Serverless Application Model) 프레임워크를 중추로 사용한다.

#### Retaining Data

- CloudFormation 템플릿이 삭제될 때 발생하는 작업을 제어하기 위해 리소스에 삭제 정책을 적용할 수 있다.
- **DeletionPolicy=Retain**
  - CloudFormation 삭제 시 보존할 리소스 및 백업을 지정한다.
  - 리소스를 유지하도록 지정하면 되고, 모든 리소스나 중첩된 스택에서 작동한다.
- **DeletionPolicy=Snapshot**
  - 리소스를 삭제할 수 있지만 그 전에 스냅샷이 생성된다.
  - EBS Volume, Elastic Cache Cluster, ElastiCache ReplicationGroup 등에 적용된다.
  - RDS DB 인스턴스, RDS DB Cluster, Redshift Cluster 등에 적용된다.
- **DeletionPolicy=Delete** (기본 정책)
  - CloudFormation 스택을 삭제하면 모든 것이 삭제된다.
  - `AWS::RDS:DBCluster` 리소스를 사용할 경우 기본 정책은 스냅샷이다.
  - S3 버킷을 삭제하려면 모든 버킷이 완전히 비어 있어야 하며 그렇지 않은 경우 실패한다.

#### Custom Resources

- CloudFormation은 람다를 사용하여 사용자 지정 리소스를 작업할 수 있고 많은 사례에 사용된다.
  - AWS에 새로운 서비스가 출시되었는데 CloudFormation에서 지원하지 않는 경우
  - CloudFormation을 통해 관리하고 싶은 온프레미스 리소스가 있는 경우
  - S3 버킷을 삭제하기 이전에 버킷을 비우고 싶은 경우
  - CloudFormation 템플릿에 적용하기 전에 AMI ID를 얻고 싶은 경우

![9-custom-resource.png](images%2F9-custom-resource.png)

#### StackSets

- 단일 작업으로 여러 계정 및 리전에 걸쳐 스택을 생성, 업데이트 또는 삭제할 수 있다.
- StackSets을 생성하기 위한 관리자 계정이 있다.
- StackSets에서 스택 인스턴스를 생성, 업데이트, 삭제할 수 있는 신뢰할 수 있는 계정이 있다.
- StackSets를 업데이트하면 모든 계정 및 리전에서 관련된 모든 스택 인스턴스가 업데이트된다.
- "자동 배포(Automatic Deployment)" 기능을 사용하여 AWS Organization 또는 OU의 계정에 자동으로 배포할 수 있다.

![10-stacksets.png](images%2F10-stacksets.png)

#### Drift

- CloudFormation을 통해 인프라를 구축할 수 있지만 수동 변경으로부터 보호되지는 않는다.
- 리소스가 중간에 변경되었다면 "드리프트"라고 한다.
- CloudFormation Drift를 사용하여 리소스가 수동으로 변경되었는지 확인할 수 있다.
- 전체 스택 또는 스택 내의 개별 리소스에서 드리프트를 탐지한다.

![11-drift.png](images%2F11-drift.png)

- CloudFormation Drift는 모든 리소스를 평가하고 CloudFormation이 제공하는 구성을 살펴본다.
- 비교 알고리즘을 통하여 누군가 수동으로 수정한 리소스를 확인할 수 있다.

#### Secret Manager 통합

![12-integration-secret-manager.png](images%2F12-integration-secret-manager.png)

- 첫 부분에서 Secret Manager로부터 시크릿을 생성한다.
- 두 번째 부분에서는 참조를 사용하여 시크릿을 찾아 RDS DB 인스턴스로 넘긴다.
- Secret를 찾아 RDS DB 인스턴스에 연결한다.

#### Resource Import

- 기존에 존재하거나 새로운 스택으로 리소스를 가져올 수 있다.
- CloudFormation 스택의 일부로 리소스를 삭제하고 다시 생성할 필요가 없다.
- 가져오기 작업 중에는 다음의 항목들이 필요하다.
  - 전체 스택(원본 스택 리소스 및 가져올 대상 리소스)을 설명하는 템플릿
  - 각 대상 리소스에 대한 고유 식별자 (예: S3 버킷의 이름)
- 가져올 각 리소스에는 `DeletePolicy` 속성 값과 Identifier가 있어야 한다.
- 동일한 리소스를 여러 스택으로 가져올 수 없다.

![13-resource-import.png](images%2F13-resource-import.png)

- `MyBucket`이라는 S3 버킷이 있고 스택에서 가져오기를 원한다.
- 리소스로 템플릿을 생성하여 S3 버킷을 가져온다.
- CloudFormation으로 리소스를 가져와 스택을 생성한다.
- 스택은 이미 존재하는 S3 버킷을 스택 리소스에 포함시켜 CloudFormation을 통해 관리하게 된다.
- **해당 리소스를 삭제하거나 다시 생성하지 않고 불러오기만 한다.**

---

### AWS Service Catalog

- AWS를 처음 접하는 사용자는 옵션이 너무 많기 때문에 규정을 준수하지 않는 스택을 생성할 수 있는 가능성이 있다.
- 관리자에 의해 미리 정의된 승인된 서비스만 실행할 수 있는 "셀프 서비스 포털"(self-service portal)을 제공할 수 있다.
- VM, 데이터베이스, 스토리지 옵션 등이 포함된다.

#### Diagram

![14-service-catalog-diagram.png](images%2F14-service-catalog-diagram.png)

- 관리자 태스크와 사용자 태스크가 있다.
- 관리자는 파워 유저이기 때문에 CloudFormation 템플릿을 작성할 수 있다.
- 모든 제품을 합쳐서 각각의 포트폴리오를 생성하고 포트폴리오에는 IAM 역할이 할당되어 컨트롤할 수 있다.
- 사용자는 Service Catalog를 통해 제품 목록을 확인할 수 있다.
- IAM에서 승인된 제품들만 확인할 수 있다. 

#### Summary

- AWS에서 승인된 IT 서비스 카탈로그를 생성 및 관리한다.
- "제품"(Product)은 CLoudFormation 템플릿이다.
- 예를 들어, VM 이미지, 서버, 소프트웨어, 데이터베이스, 리전, IP 주소 범위 등을 포함할 수 있다.
- CloudFormation을 통해 관리자가 일관성을 확보하고 표준화할 수 있다.
- 이러한 템플릿들은 포트폴리오(팀)에 할당된다.
- 팀에게는 제품을 출시할 수 있는 셀프 서비스 포털이 제공된다.
- 구축된 모든 제품은 중앙에서 관리되는 구축된 서비스다.
- 거버넌스, 컴플라이언스 및 일관성을 지원한다.
- 깊은 AWS 지식 없이 제품 출시에 사용자가 액세스할 수 있다.
- "ServiceNow"와 같은 "셀프 서비스 포털"과의 통합을 지원한다.

---

### AWS SAM - Serverless Application Model

- 서버리스 애플리케이션 개발 및 구축을 위한 프레임워크다.
- 모든 구성은 YAML 기반의 코드로 작성된다.
  - 람다 함수(`AWS::Serverless::Function`)
  - DynamoDB 테이블(`AWS::Serverless::SimpleTable`)
  - API 게이트웨이(`AWS::Serverless:API`)
  - Step Function - State Machine (`AWS::Serverless:StateMachine`)
- SAM은 람다, API 게이트웨이, DynamoDB를 로컬로 실행하는 데 도움을 줄 수 있다.
- SAM은 CodeDeploy를 사용하여 람다 함수(트래픽 시프트)를 구현할 수 있다.
- 백엔드에서 CLoudFormation을 활용할 수 있다.

#### CICD Architecture

![15-cicd-architecture-sam.png](images%2F15-cicd-architecture-sam.png)

- 소스 코드는 CodeCommit에 저장되고 CodeBuild로 전달되어 빌드, 테스트, 패키징한다.
- 이후 CodePipeline의 CloudFormation 단계가 적용된다.
- CodeDeploy는 자동으로 SAM에 의해 호출되어 버전 1 람다 함수에서 버전 2 람다 함수로 트래픽을 시프트한다.
- CloudFormation은 API 게이트웨이를 배포하고 업데이트할 수 있다.
- 필요한 경우 DynamoDB도 사용할 수 있다.

---

### AWS Cloud Development Kit (CDK)

- 친숙한 언어를 사용하여 클라우드 인프라를 정의한다.
  - Javascript, Typescript, Python, Java, .NET
- 코드는 CloudFormation 템플릿(JSON/YAML)으로 컴파일된다.
- 인프라와 애플리케이션 런타임 코드를 함께 배포할 수 있다.
  - 람다 함수에 적합하다.
  - ECS/EKS의 도커 컨테이너에 적합하다.

![16-cloud-development-kit.png](images%2F16-cloud-development-kit.png)

#### Example

![17-cdk-example.png](images%2F17-cdk-example.png)

- 예시에서는 Javascript가 사용되고 있다.
- 정의된 VPC와 ECS 클러스터와 Fargate 서비스와 함께 ALB가 정의되어 있다.
- 세 가지 서비스는 CDK CLI에 의해 컴파일되어 CloudFormation 템플릿에 사용되고 업로드 및 배포할 수 있다.

---

### AWS System Manager

- EC2 및 온프레미스 시스템을 대규모로 관리할 수 있도록 지원한다.
- 인프라 상태에 대한 운영 통찰력을 확보하여 문제를 쉽게 발견할 수 있다.
- 컴플라이언스 향상을 위한 패치 자동화를 지원한다.
- Windows 및 Linux OS 모두 지원한다.
- CloudWatch 메트릭/대시보드 및 AWS Config와 통합될 수 있다.
- 무료로 사용가능하다.

#### 작동 방식

- 사용자가 통제하는 시스템에 SSM 에이전트를 설치해야 한다.
- Amazon Linux AMI 및 일부 Ubuntu AMI에 기본적으로 설치된다.
- 인스턴스를 System Manager로 제어할 수 없는 경우 SSM 에이전트의 문제일 수 있다.
- EC2 인스턴스에 System Manager 작업을 허용할 수 있는 적절한 IAM 역할이 있는지 확인이 필요하다.

![18-systems-manager-works.png](images%2F18-systems-manager-works.png)

#### Run Command

- 문서(스크립트)를 실행하거나 커맨드만 실행할 수 있다.
- 여러 인스턴스에서 커맨드를 실행한다. (리소스 그룹 사용)
- 속도 제어 / 오류 제어를 지원한다.
- IAM & CloudTrail과 통합된다.
- SSH를 통한 접속이 필요하지 않다.
- 실행 결과를 콘솔에서 확인할 수 있다.

![19-run-command.png](images%2F19-run-command.png)

#### Send Command

- ASG 인스턴스가 종료되기 전에 커맨드를 전송할 수 있다.
- ASG가 EC2 인스턴스를 종료하기 전에 작업이 수행된다.
- 인스턴스를 `Terminating:Wait` 상태로 만드는 ASG Lifecycle Hook를 생성한다.
- `Terminating:EventBridge`를 사용하여 `Terminating:Wait` 상태를 모니터링한다.
- 종료 전에 인스턴스에 대한 작업을 수행하도록 SSM 자동화 문서를 트리거한다.

![20-send-command-asg-instance-terminated.png](images%2F20-send-command-asg-instance-terminated.png)

- ASG가 있고 EC2 인스턴스 중 하나를 종료시킨다.
- 종료를 의미하는 이벤트를 EventBridge로 전송하고 인스턴스는 한동안 이 상태에 머물게된다.
- EventBridge는 이벤트를 가로채서 규칙을 생성할 수 있다.
- 규칙은 SSM 자동화 문서를 실행하고 `SendCommand`를 통해서 인스턴스를 종료하도록 사용할 수 있다.

#### Systems Manager Patch Managers - Steps

1. 사용할 패치 기준선을 정의한다. 환경이 여러 개인 경우 여러 개를 정의할 수 있다.
2. "Patch Group" 정의: 태그를 기반으로 정의한다. 태그 패치 그룹을 사용한다.
3. "Maintenance Windows" 정의: 스케줄, 기간, 등록된 대상/패치 그룹 및 등록된 작업
4. "AWS-RunPatchBaselineRun" 명령을 "Maintenance Windows"(플랫폼 리눅스 & 윈도우즈 간 작업)의 등록된 작업의 일부로 추가한다.
5. 작업에 대한 속도 제어(통화 및 오류 임계값)를 정의한다.
6. SSM 인벤토리를 사용하여 패치 컴플라이언스를 모니터링한다.
- 보다 자세한 내용은 [공식 홈페이지](https://aws.amazon.com/blogs/mt/patching-your-windows-ec2-instances-using-aws-systems-manager-patch-manager/)를 확인한다.

![21-patch-manager-step.png](images%2F21-patch-manager-step.png)

#### Systems Manager Session Manager

- EC2 및 온프레미스 서버에서 보안 쉘을 시작할 수 있다.
- AWS 콘솔, AWS CLI 또는 Session Manager SDK를 통해서 액세스한다.
- SSH 액세스, bastion host 또는 SSH 키가 필요하지 않다.
- Linux, macOS 및 Windows를 지원한다.
- 인스턴스 및 실행된 명령에 대한 연결을 기록한다.
- 세션 로그 데이터를 S3 또는 CloudWatch 로그로 전송할 수 있다.
- CloudTrail이 `StartSession` 이벤트를 가로챌 수 있다.

![22-session-manager.png](images%2F22-session-manager.png)

#### Systems Manager OpsCenter

- AWS 리소스와 관련된 운영 문제를 해결한다.
- OpsItems: 이슈, 이벤트 및 경고
- 다음과 같은 각 OpsItem의 문제를 해결하기 위한 정보를 집계한다.
  - AWS Config 변경사항 및 관계
  - CloudTrail 로그
  - CloudWatch 알람
  - CloudFormation 스택 정보
- 문제를 해결하는 데 사용할 수 있는 Automation Runbook을 제공한다.
- EventBridge 또는 CloudWatch 알람으로 OpsItem을 생성할 수 있다.

---

### AWS Cloud Map

- 완벽하게 관리되는 리소스 검색 서비스다.
- 애플리케이션이 의존하는 백엔드 서비스/리소스 맵을 작성한다.
- 애플리케이션 구성 요소, 위치, 속성 및 상태를 AWS Cloud Map에 등록한다.
- 통합 상태 검사를 제공한다.
  - 불완전한 엔드포인트로 트래픽 전송을 중지할 수 있다.
- 애플리케이션은 AWS SDK, API 또는 DNS를 사용하여 AWS Cloud Map을 쿼리할 수 있다.

![23-cloudmap.png](images%2F23-cloudmap.png)

- Cloud Map을 사용하지 않는 경우 프론트엔드 서비스가 새로운 버전의 백엔드 서비스를 요청해야 할 때 코드 변경이 필요하다.
- Cloud Map을 사용하는 경우 Cloud Map의 엔드포인트를 구버전에서 새로운 버전으로 변경하면 되기 때문에 코드의 변경이 필요하지 않다.
  - 프론트엔드 서비스는 동적으로 새로운 버전의 위치를 찾게된다.

---

### 참고한 강의

- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)