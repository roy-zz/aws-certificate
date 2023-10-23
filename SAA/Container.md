# Container Section

이번 장에서는 SAA를 준비하며 **컨테이너(Container)**에 대해서 알아보도록 한다.

---

### 도커(Docker) 란

- 도커는 앱을 배포하는 소프트웨어 개발 플랫폼이다.
- 애플리케이션은 모든 OS에서 실행할 수 있는 컨테이너에 패키징된다.
- 애플리케이션 실행 위치에 상관없이 동일하게 실행된다.
  - 모든 머신에서 실행 가능.
  - 호환성 이슈 없음.
  - 예측 가능한 행동.
  - 작업량 감소.
  - 유지보수 및 구축이 보다 용이.
  - 모던 언어, OS, 모든 기술과 함께 작동.
- 사용 사례: 마이크로서비스 아키텍처(MSA), 사내에서 AWS 클라우드로 이동

#### Docker on an OS

![docker-on-os.png](images%2FContainer%2Fdocker-on-os.png)

#### 도커 이미지 저장

- 도커 이미지는 도커 이미지 저장소에 저장된다.
- **도커 허브(Docker Hub)**
  - Public 저장소
  - 많은 기술이나 OS(예: Ubuntu, MySQL)의 기본 이미지를 찾을 때 사용된다.
- **Amazon ECR(Amazon Elastic Container Registry)**
  - Private 저장소
  - "Amazon ECR Public Gallery"는 Public 저장소

#### Docker vs Virtual Machines

- 도커는 "가상화 기술의 일종"이지만 정확히는 아니다.
- 리소스가 하나의 서버에서 호스트 => 많은 컨테이너와 공유된다.

![docker-vs-vm.png](images%2FContainer%2Fdocker-vs-vm.png)

![start-with-docker.png](images%2FContainer%2Fstart-with-docker.png)

#### AWS에서 도커 컨테이너 관리

- **Amazon ECS(Amazon Elastic Container Service)**: Amazon 자체 컨테이너 플랫폼
- **Amazon EKS(Amazon Elastic Kubernetes Service)**: Amazon의 관리형 Kubernetes(오픈 소스)
- **AWS Fargate**:
  - Amazon 자체 Serverless 컨테이너 플랫폼
  - ECS 및 EKS와 함께 작동한다.
- **Amazon ECR**: 컨테이너 이미지 저장소

---

### Amazon ECS

- ECS: Elastic Container Service

#### EC2 Launch Type

- AWS에서 도커 컨테이너 실행: ECS 클러스터에서 ECS 작업 실행
- EC2 Launch Type: 인프라스트럭처를 프로비저닝 및 유지 관리해야 한다.
- ECS 클러스터에 등록하려면 각 EC2 인스턴스에서 ECS 에이전트를 실행해야 한다.
- AWS에서 컨테이너 시작 및 정지를 관리할 수 있다.

![ecs-ec2-launch-type.png](images%2FContainer%2Fecs-ec2-launch-type.png)

#### Fargate Launch Type

- AWS에서 도커 컨테이너를 실행한다.
- 인프라스트럭처를 프로비저닝하지 않으므로 관리할 EC2 인스턴스가 없다.
- 모두 Serverless로 작동한다.
- 작업 정의만 생성하면 된다.
- AWS는 필요한 CPU/RAM을 기반으로 ECS Task를 실행한다.
- 확장하기 위해서 Task의 수를 증가시키면 된다. 더 이상 EC2 인스턴스는 사용되지 않는다.

![ecs-fargate-launch-type.png](images%2FContainer%2Fecs-fargate-launch-type.png)

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

#### Load Balancer Integrations

- **Application Load Balancer**가 지원되며 대부분의 사용 사례에서 작동한다.
- **Network Load Balancer**는 높은 처리량/높은 성능을 요구하는 경우에만 사용하거나, "AWS Private Link"와 함께 사용하는 것이 권장된다.
- **Elastic Load Balancer**는 사용이 가능하지만 사용이 권장되지 않는다.(고급 기능이 없고, Fargate 없음)

#### Data Volumes (EFS)

![ecs-data-volume.png](images%2FContainer%2Fecs-data-volume.png)

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

![ecs-service-cpu-usage-example.png](images%2FContainer%2Fecs-service-cpu-usage-example.png)

- 아래는 Event Bridge에서 ECS Task를 호출하는 예시이다.

![ecs-tasks-invoked-event-bridge.png](images%2FContainer%2Fecs-tasks-invoked-event-bridge.png)

- 아래는 Event Bridge Schedule에서 ECS Task를 호출하는 예시이다.

![ecs-tasks-invoked-event-bridge-schedule.png](images%2FContainer%2Fecs-tasks-invoked-event-bridge-schedule.png)

- 아래는 SQS의 Message를 ECS Tasks들이 소비하는 예시이다.

![ecs-sqs-queue.png](images%2FContainer%2Fecs-sqs-queue.png)

---

### Amazon ECR

- ECR: Elastic Container Registry
- AWS에서 도커 이미지를 저장하고 관리한다.
- 개인 및 공용 저장소(Amazon ECR Public Gallery)
- Amazon S3가 지원하는 ECS와 완벽하게 통합된다.
- IAM을 통해 접근을 제어한다.
- 이미지 취약성 검색, 버전 지정, 이미지 태그, 이미지 수명 주기 등을 지원한다.

![ecr-overview.png](images%2FContainer%2Fecr-overview.png)

---

### Amazon EKS

- Amazon EKS: Amazon Elastic Kubernetes Service
- AWS에서 관리형 Kubernetes 클러스터를 시작하는 방법이다.
- Kubernetes는 컨테이너형(일반적으로 Docker) 애플리케이션의 자동 배포, 확장 및 관리를 위한 오픈 소스 시스템이다.
- ECS의 대안이며, 서비스의 목표는 비슷하지만 API는 다르다.
- EKS는 EC2를 지원하고, 워커 노드나 Fargate를 배포하는 경우에는 Serverless 컨테이너를 사용할 수 있다.
- 사용 사례: 사내 또는 다른 클라우드에서 Kubernetes를 사용하고 있으며, Kubernetes를 사용하여 AWS로 마이그레이션하려는 경우
- Kubernetes는 클라우드에 종속되지 않는다.(Azure, GCP 등 모든 클라우드에서 사용 가능)
- Multiple Region의 경우, Region당 하나의 EKS 클러스터를 구축한다.
- CloudWatch Container Insights를 사용하여 로그 및 메트릭을 수집한다.

![eks-diagram.png](images%2FContainer%2Feks-diagram.png)

#### Node Types

- **Managed Node Groups**:
  - 노드(EC2 인스턴스)를 생성하고 관리한다.
  - 노드는 EKS에서 관리하는 ASG의 일부이다.
  - 온-디멘드 또는 스팟 인스턴스를 지원한다.
- **Self-Managed Nodes**:
  - 사용자가 생성하여 EKS 클러스터에 등록하고 ASG에서 관리하는 노드다.
  - 사전에 구축된 AMI를 사용할 수 있다. (Amazon EKS Optimized AMI)
  - 온-디멘드 또는 스팟 인스턴스를 지원한다.
- **AWS Fargate**:
  - 유지보수가 필요없으며, 관리되는 노드가 없다.

#### Data Volumes

- EKS 클러스터에 StorageClass 매니페스트를 지정해야 한다.
- CSI(Container Storage Interface) 호환 드라이버를 활용한다.
- 아래와 같은 파일 시스템을 지원한다.
  - Amazon EBS
  - Amazon EFS (works with Fargate)
  - Amazon FSx for Lustre
  - Amazon FSx for NetApp ONTAP

---

### AWS App Runner

- 웹 애플리케이션 및 API를 규모에 맞게 쉽게 배포할 수 있도록 완벽하게 관리되는 서비스다.
- 인프라스트럭처에 대한 경험이 필요없다.
- 소스 코드 또는 컨테이너 이미지로 시작할 수 있다.
- 웹 앱을 자동으로 구축하고 배포한다.
- 자동 확장, 고가용성, 로드 밸런서, 암호화를 지원한다.
- VPC 액세스를 지원한다.
- 데이터베이스, 캐시 및 메시지 큐 서비스에 연결된다.
- 활용 사려: 웹 앱, API, 마이크로서비스, 신속한 운영 구축

![app-runner.png](images%2FContainer%2Fapp-runner.png)

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03