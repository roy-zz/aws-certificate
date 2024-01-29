# Container Orchestration

- 이번 장에서는 **DevOps Engineer Professional (DOP)** 을 준비하며 "컨테이너 오케스트레이션 서비스들"에 대해서 알아보도록 한다.

---

### Amazon ECS

- ECS: Elastic Container Service

#### EC2 Launch Type

- AWS에서 도커 컨테이너 실행: ECS 클러스터에서 ECS 작업 실행
- EC2 Launch Type: 인프라스트럭처를 프로비저닝 및 유지 관리해야 한다.
- ECS 클러스터에 등록하려면 각 EC2 인스턴스에서 ECS 에이전트를 실행해야 한다.
- AWS에서 컨테이너 시작 및 정지를 관리할 수 있다.

![1-ec2-launch-type.png](images%2F1-ec2-launch-type.png)

- ECS 클러스터는 여러 개의 EC2 인스턴스로 구성된다.
  - 각각의 인스턴스는 ECS 에이전트를 실행하고 있다.
- 에이전트가 EC2 인스턴스를 Amazon ECS 서비스 및 지정된 ECS 클러스터에 등록한다.
- ECS 태스크를 시작하면 AWS가 컨테이너를 시작하거나 중지한다.
- 즉, 새로운 도커 컨테이너가 생길 때마다 그에 따라 배치된다.

#### Fargate Launch Type

- AWS에서 도커 컨테이너를 실행한다.
- 인프라를 프로비저닝하지 않으므로 관리할 EC2 인스턴스가 없다.
- 모두 서버리스로 작동한다.
- 작업 정의만 생성하면 된다.
- AWS는 필요한 CPU/RAM을 기반으로 ECS 태스크를 실행한다.
- 확장하기 위해서 태스크의 수를 증가시키면 된다. 더 이상 EC2 인스턴스는 사용되지 않는다.

![2-fargate-launch-type.png](images%2F2-fargate-launch-type.png)

- ECS 클러스터가 있는 경우 ECS 태스크를 정의하는 태스크 정의만 만들면 AWS가 CPU와 RAM의 수에 따라서 ECS 태스크를 실행한다.

#### IAM Roles for ECS

- EC2 Instance Profile(EC2 Launch Type만 해당)
    - ECS Agent에서 사용된다.
    - ECS 서비스로 API를 호출한다.
    - 컨테이너 로그를 CloudWatch 로그로 전송한다.
    - ECR에서 도커 이미지를 다운로드 한다.
    - Secrets Manager 또는 SSM 매개 변수 저장소에서 중요 데이터를 참조한다.
- ECS Task Role:
    - 각 Task가 특정 역할을 갖도록 허용한다.
    - 실행하는 여러 ECS 서비스에 대해 다른 역할을 사용한다.
    - Task 역할은 Task 정의(Definition)에 정의되어 있다.

![3-iam-role.png](images%2F3-iam-role.png)

- ECS 에이전트는 EC2 인스턴스 프로필을 사용하여 ECS 서비스에 API 호출을 생성하고 인스턴스를 복원한다.
- 그리고 CloudWatch Logs로 API를 호출하여 컨테이너로 로그를 전송한다.
- ECR에 API 호출을 사용해서 ECR에서 도커 이미지를 가져온다.
- 첫 번째 태스크 A에는 EC2 태스크 A 역할이 있다.
  - 태스크 A는 S3에 대해 API 호출을 실행할 수 있다.
- 두 번째 태스크 B에는 태스크 B 역할이 있다.
  - 태스크 B는 DynamoDB에 대한 API 호출을 실행할 수 있다.

#### Load Balancer Integrations

- **Application Load Balancer**가 지원되며 대부분의 사용 사례에서 작동한다.
- **Network Load Balancer**는 높은 처리량/높은 성능을 요구하는 경우에만 사용하거나, "AWS Private Link"와 함께 사용하는 것이 권장된다.
- **Elastic Load Balancer**는 사용이 가능하지만 사용이 권장되지 않는다.(고급 기능이 없고, Fargate 없음)

![4-load-balancer-integrations.png](images%2F4-load-balancer-integrations.png)

- 여러 개의 ECS 태스크가 실행 중이며 모두 ECS 클러스터에 있다.
- 이 태스크들은 HTTP나 HTTPS 엔드포인트로 노출해야 한다.
- 따라서 ALB를 그 앞에서 실행하여 사용자들이 ECS 태스크에 요청을 전달할 수 있도록 구축할 수 있다.

#### Data Volumes (EFS)

![5-data-volume.png](images%2F5-data-volume.png)

- ECS Task에 EFS 파일 시스템을 마운트할 수 있다.
- EC2 및 Fargate Launch Type 모두에서 작동한다.
- AZ에서 실행 중인 작업은 EFS 파일 시스템에서 동일한 데이터를 공유한다.
- Fargate + EFS = Serverless
- 활용 사례: 컨테이너를 위한 지속적인 Multi-AZ 공유 스토리지
- 단, Amazon S3를 파일 시스템으로 마운트할 수 없다.

#### ECS Service Auto Scaling

- 원하는 ECS Tasks의 수를 자동으로 늘리거나 줄인다.
- Amazon ECS Auto Scaling을 사용하여 AWS 애플리케이션을 자동 스케일링한다.
    - ECS 서비스의 평균 CPU 사용률
    - ECS 서비스의 평균 메모리 사용률 - RAM 스케일링
    - ALB Request Count Per Target - ALB에서 오는 메트릭
- **Target Tracking**: 특정 CloudWatch 메트릭의 목표 값을 기반으로 확장한다.
- **Step Scaling**: 지정된 CloudWatch 알람을 기반으로 확장한다.
- **Scheduled Scaling**: 지정된 날짜/시간을 기준으로 확장(예측 가능한 변경 사항)한다.
- ECS 서비스 자동 스케일링(Task Level)은 EC2 자동 스케일링(EC2 인스턴스 Level)과 동일하지 않다.
- Fargate는 Serverless이기 때문에 자동 스케일링을 훨씬 쉽게 설정할 수 있다.

#### EC2 Launch Type - Auto Scaling EC2 Instances

- 기본 EC2 인스턴스를 추가하여 ECS 서비스 확장을 수용한다.
- Auto Scaling Group
    - CPU 사용률을 기반으로 ASG 스케일링
    - 시간의 경과에 따라 EC2 인스턴스 추가
- ECS Cluster Capacity Provider
    - ECS Task에 필요한 인프라스트럭처를 자동으로 프로비저닝하고 확장하는데 사용한다.
    - 용량 제공자(Capacity Provider)가 자동 확장 그룹(Auto Scaling Group)과 쌍을 이룬다.
    - 용량(CPU, RAM...) 부족 시 EC2 인스턴스를 추가한다.

- 아래는 CPU 사용량을 CloudWatch Metric을 통해 확인하고 CloudWatch Alarm을 통해서 Auto Scaling하는 예시이다.
  - 서비스 A에는 두 개의 태스크가 있고 CPU 사용량을 확인할 수 있다.
  - AWS 애플리케이션 오토 스케일링에 의해 오토 스케일링된다.
  - CPU 사용량이 엄청나게 증가하는 경우 CloudWatch 메트릭, 즉 ECS 서비스 수준에서 CPU 사용량을 다시 모니터링하는 메트릭이 CloudWatch 알람을 트리거한다.
  - 알람이 트리거되면 스케일링 활동이 트리거된다.
  - ECS 서비스에서 원하는 용량이 증가하며, 새로운 태스크가 생성된다.
  - 선택적으로 이 서비스가 EC2 시작 유형에서 실행 중이라면 ECS 용량 공급자는 EC2 인스턴스가 지원하는 ECS 클러스터를 확장하는 것을 도와준다.

![6-service-cpu-usage-example.png](images%2F6-service-cpu-usage-example.png)

- 아래는 Event Bridge에서 ECS Task를 호출하는 예시이다.
  - EventBridge에서 호출되는 ECS 태스크다.
  - ECS 클러스터가 있고 Fargate에 의해 지원되고 있으며 S3를 사용하고 있다.
  - 사용자는 S3 버킷에 객체를 업로드하고 있으며 S3 버킷은 EventBridge와 통합되어 모든 이벤트를 전송하고 있다.
  - EventBridge는 규칙을 가질 수 있으며 규칙은 ECS 태스크를 준비 상태로 실행한다.
  - ECS 태스크가 생성되면 ECS 태스크 역할이 연결된다.
  - 태스크 자체가 할 수 있는 일은 객체를 가져와서 처리한 다음 결과를 DynamoDB에 전송하는 것이다.

![7-tasks-invoked-event-bridge.png](images%2F7-tasks-invoked-event-bridge.png)

- 아래는 Event Bridge Schedule에서 ECS Task를 호출하는 예시이다.
  - ECS 클러스터는 Fargate와 EventBridge로 지원된다.
  - 1시간마다 트리거되도록 규칙을 예약하고 규칙은 Fargate에서 ECS 태스크를 실행한다.
  - 즉, 1시간마다 Fargate 클러스터에 새로운 태스크가 생성된다.

![8-tasks-invoked-event-bridge-schedule.png](images%2F8-tasks-invoked-event-bridge-schedule.png)

- 아래는 SQS의 Message를 ECS Tasks들이 소비하는 예시이다.
  - ECS 태스크가 두 개 있는 서비스가 ECS 클러스터에서 실행되고 있다.
  - 서비스 자체는 SQS 큐에서 메시지를 처리하고 있다.
  - 서비스 위에 ECS 서비스 오토 스케일링을 활성화할 수 있다.
  - SQS 큐에 메시지가 많을수록 더 많은 태스크를 ECS 서비스에서 수행할 수 있다.

![9-sqs-queue.png](images%2F9-sqs-queue.png)

- 아래는 EventBridge를 사용하여 ECS 클러스터 내에서 이벤트를 가로채는 예시다.
  - 종료되는 태스크에 반응하고 싶은 경우다.
  - ECS 클러스터에서 종료되거나 시작되는 모든 태스크는 EventBridge의 이벤트로 트리거될 수 있다.
  - "ECS 태스크 상태 변경", "중지됨" 등의 이벤트가 트리거된다.
  - SNS 토픽에 알림을 보내고 관리자에게 이메일을 전송할 수 있다.
  - 결론적으로, EventBridge를 사용하면 ECS 클러스터의 컨테이너의 라이프사이클을 이해할 수 있다.

![10-intercept-stopped-task-eventbridge.png](images%2F10-intercept-stopped-task-eventbridge.png)

#### Logging with "awslogs" driver

- 컨테이너는 애플리케이션 로그를 CloudWatch Logs(CW Logs)에 직접 보낼 수 있다.
- CW Logs를 위해서 "awslogs" 로그 드라이버를 활성화해야 한다.
- 태스크 정의에서 `logConfiguration` 파라미터를 구성해야 한다.
- Fargate Launch Type
  - 태스크 실행 역할에는 필수 권한이 있어야 한다.
  - "awslogs", "splunk", "awsfirelens" 로그 드라이버를 지원한다.
- EC2 Launch Type
  - 로그가 컨테이너 EC2 인스턴스의 디스크 공간을 차지하는 것을 방지한다.
  - CloudWatch Unified Agent & ECS Container Agent를 사용한다.
  - `/etc/ecs/ecs.config` 경로의 "ECS_AVAILABLE_LOGGING_DRIVERS"를 사용하여 로깅을 활성화한다.
  - 컨테이너 EC2 인스턴스에는 권한이 있어야 한다.

![11-logging-awslogs-driver.png](images%2F11-logging-awslogs-driver.png)

#### Logging with Sidecar Container

- 파일 시스템의 다른 모든 컨테이너 및 파일에서 로그를 수집하고 로그를 로그 스토리지(예: CloudWatch Logs)로 보내는 역할을 담당하는 사이드카 컨테이너를 사용한다.
  - 사이드카 컨테이너는 ECS 태스크의 일부다.

![12-logging-sidecar-container.png](images%2F12-logging-sidecar-container.png)

- 태스크 자체 출력으로 로그를 만들지 않고 파일 시스템으로 로그를 생성한다.
- 사이드카 컨테이너의 역할은 파일 시스템에서 파일들을 꺼내어 CloudWatch Logs로 전송하는 것이다.

---

### Amazon ECR

- ECR: Elastic Container Registry
- AWS에서 도커 이미지를 저장하고 관리한다.
- 개인 및 공용 저장소(Amazon ECR Public Gallery)
- Amazon S3가 지원하는 ECS와 완벽하게 통합된다.
- IAM을 통해 접근을 제어한다.
- 이미지 취약성 검색, 버전 지정, 이미지 태그, 이미지 수명 주기 등을 지원한다.

![13-ecr-overview.png](images%2F13-ecr-overview.png)

- ECS 클러스터의 EC2 인스턴스는 도커 이미지가 필요한 경우 ECR에서 가져올 수 있다.
- 가져오기 위해서는 IAM 역할에 서명해야 한다.

#### Lifecycle Policy

- 기간이나 개수에 따라 오래되었거나 사용하지 않는 이미지를 자동으로 제거한다.
- 각 수명주기 정책에는 하나 이상의 규칙이 포함된다.
- 모든 규칙은 동시에 평가된 후 우선순위에 따라 적용된다.
- 이미지는 기준을 충족한 후 24시간 이내에 만료된다.
- 불필요한 저장 비용을 줄이는데 도움이 된다.

![14-ecr-lifecycle-policy.png](images%2F14-ecr-lifecycle-policy.png)

#### Uniform Pipeline

![15-ecr-uniform-pipeline.png](images%2F15-ecr-uniform-pipeline.png)

- 배포를 위해 동일한 파이프라인을 만들 수 있다.
- 코드가 CodeCommit에 있고 Python, Ruby, Node.js 등 어떤 언어라도 지원한다.
- CodeBuild를 통해서 생성된 도커 이미지는 Amazon ECR로 푸시할 수 있다.
- 도커 이미지는 ECS나 Fargate에 동일한 방식으로 배포된다.

---

### Amazon EKS

- Amazon EKS는 "Amazon Elastic Kubernetes Service"의 약자다.
- AWS에서 관리되는 Kubernetes 클러스터를 시작하는 방법이다.
- Kubernetes는 컨테이너형(보통 Docker) 애플리케이션의 자동 배포, 확장 및 관리를 위한 오픈 소스 시스템이다.
- ECS의 대안이며, 둘의 목표는 비슷하지만 API는 서로 다르다.
- EKS는 서버리스 컨테이너를 배포하기 위해 워커 노드 또는 Fargate를 배포하려는 경우 EC2를 지원한다.
- 사용 사례: 회사에서 이미 사내 또는 다른 클라우드에서 Kubernetes를 사용하고 있으며 Kubernetes를 사용하여 AWS로 마이그레이션하려는 경우에 사용된다.
- Kubernetes는 클라우드에 종속되지 않으며 Azure, GCP와 같은 모든 클라우드에서 사용 가능하다.
- 여러 리전의 경우 각 리전당 하나의 EKS 클러스터를 배포해야 한다.
- CloudWatch Container Insights를 사용하여 로그 및 메트릭을 수집할 수 있다.

#### Diagram

![16-eks-diagram.png](images%2F16-eks-diagram.png)

- 하나의 VPC에 세 개의 AZ가 공용 서브넷과 사설 서브넷으로 분리되어 있다.
- EKS는 워커 노드를 생성하고 이는 EC2 인스턴스가 된다.
- 이 노드들은 각각 EKS 파드를 실행하며 ECS 태스크와 아주 유사하다.
- EKS 파드들은 EKS 노드 내에서 실행되며 Auto Scaling Group으로 관리된다.
- EKS 서비스를 외부로 노출하고 싶다면 사설 로드 밸런서나 공용 로드 밸런서를 설정해 웹과 통신하도록 설정할 수 있다.

#### Node Type

- **Managed Node Group**
    - 사용자를 위한 노드(EC2 인스턴스) 생성 및 관리
    - 노드는 EKS가 관리하는 ASG의 일부다.
    - 온디맨드 또는 스팟 인스턴스를 지원한다.
- **Self-Managed Node**
    - 사용자가 생성하여 EKS 클러스터에 등록하고 ASG에서 관리하는 노드다.
    - 사전 구축된 AMI를 사용할 수 있다. - Amazon EKS Optimized AMI
    - 온디맨드 또는 스팟 인스턴스를 지원한다.
- **AWS Fargate**
    - 유지 관리가 필요없으며 관리가 필요한 노드가 없다.

#### Data Volume

- EKS 클러스터에 StorageClass 매니페스트를 지정해야 한다.
- 컨테이너 스토리지 인터페이스(CSI) 호환 드라이버를 활용해야 한다.
- 여러 종류의 스토리지를 지원한다.
    - Amazon EBS
    - Amazon EFS (work with Fargate)
    - Amazon FSx for Lustre
    - Amazon FSx for NetApp ONTAP

#### Control Plane Logging

- EKS 컨트롤 플레인 감사 및 진단 로그를 CloudWatch Logs로 보낸다.
- EKS 컨트롤 플레인은 아래와 같은 로그 유형을 가지고 있다.
  - API Server 
  - Audit 
  - Authenticator
  - Controller Manager
  - Scheduler
- CloudWatch Logs로 보낼 정확한 로그 유형을 선택하는 기능을 제공한다.

![17-control-plane-logging.png](images%2F17-control-plane-logging.png)

#### Nodes & Containers Logging

- 노드, 파드 및 컨테이너 로그를 캡처하여 CloudWatch Logs로 보낼 수 있다.
- CloudWatch 에이전트를 사용하여 CloudWatch에 메트릭을 보낼 수 있다.
- "Fluent Bit" 또는 "Fluentd" 로그 드라이버를 사용하여 CloudWatch Logs로 로그를 전송한다.
- 컨테이너 로그는 로그 대렉토리 `/var/log/containers`에 저장된다.
- CloudWatch Container Insights를 사용하여 노드, 파드, 태스크 및 서비스에 대한 대시보드 모니터링 솔루션을 얻는다.

![18-node-container-logging.png](images%2F18-node-container-logging.png)

---

### 참고한 강의

- [DevOps Engineer Professional](https://www.udemy.com/course/aws-certified-devops-engineer-professional-korean)
- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)