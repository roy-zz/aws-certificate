# Compute & Load Balancing

- 이번 장에서는 **Solutions Architect Professional (SAP)** 을 준비하며 "컴퓨팅"과 "로드밸런싱"에 대해서 알아보도록 한다.

---

### Solution Architecture on AWS

![1-solution-architecture-aws.png](images%2F1-solution-architecture-aws.png)

- 사용자는 애플리케이션에 액세스하기 위해서 가장 먼저 DNS 계층과 통신한다.
  - DNS 계층은 일반적으로 Route 53이 된다.
- 사용자는 정적(static) 컨텐츠 또는 동적(dynamic) 컨텐츠를 요청하게 된다.
- 정적 컨텐츠의 경우 전세계에 배포되고 CDN(Content Delivery Network) 계층을 통해 캐싱되며 일반적으로 CloudFront가 된다.
- CloudFront는 이 데이터를 S3나 Glacier로부터 공급받지만 사용자 지정 소스에서도 얻을 수 있다.
- 사용자는 동적 컨텐츠를 조회하기 위해 웹 계층(CLB, ALB, NLB, API Gateway, Elastic IP)에 요청을 전송한다.
- 웹 계층은 컴퓨팅 계층과 통신해야 하고 컴퓨팅 계층은 EC2 인스턴스, ASG, Lambda 등이 될 수 있다.
  - 컴퓨팅 계층은 데이터베이스 계층을 통해서 데이터를 조회한다.
  - 자주 조회되는 데이터를 위하여 캐싱/세션 계층의 데이터를 조회한다.
  - 정적 데이터를 조회하기 위해서 스토리지 계층을 사용한다.
  - MSA 구조라면 솔루션을 분리하기 위해서 디커플링 오케스트레이션 계층을 사용한다.

---

### EC2

#### 인스턴스 유형

- R: 많은 RAM이 필요한 애플리케이션에 사용된다. 
  - 인메모리 캐시
- C: 우수한 CPU가 필요한 애플리케이션에 사용된다.
  - 컴퓨팅 / 데이터베이스
- M: 균형 잡힌 애플리케이션에 사용된다.
  - 일반적인 웹/앱
- I: 우수한 로컬 I/O가 필요한 애플리케이션에 사용된다.
  - 데이터베이스 / 스토리지
- G: GPU가 필요한 애플리케이션에 사용된다.
  - 비디오 렌더링 / 머신러닝
- T2 / T3: 버스트 가능 인스턴스 (최대 용량까지)
- T2 / T3 무제한: 무제한 버스트 가능
- 더 많은 EC2 인스턴스 유형을 확인하려면 [여기](https://www.ec2instances.info)에서 확인한다.

#### Placement Group

- 기본값으로 EC2 인스턴스를 실행하면 AWS 데이터 센터에 무작위로 배치된다.
- 사용자가 AWS에 인스턴스를 어디에 둘지 힌트를 제공할 수 있으며 배치 그룹(Placement Group)이라고 한다.
- 배치 그룹을 사용하여 EC2 인스턴스 배치 전략을 제어할 수 있다.
- 배치 그룹 전력
  - Cluster: 단일 가용성 영역에서 인스턴스를 저지연 그룹으로 클러스터링한다. 
  - Spread: 기본 하드웨어(AZ당 그룹당 최대 7개의 인스턴스)에 걸쳐 인스턴스를 배치한다.
  - Partition: AZ 내의 여러 파티션에 인스턴스를 분산한다. 그룹별로 100개의 EC2 인스턴스로 확장할 수 있다. (Hadoop, Cassandra, Kafka)
- 인스턴스를 배치 그룹으로 또는 배치 그룹에서 이동할 수 있다.
  - 인스턴스를 중단해야 한다.
  - CLI를 사용하여 "수정 -> 인스턴스 -> 배치" 작업을 수행해야 한다.
  - 인스턴스를 다시 실행한다.

#### Placement Group - Cluster

- 모든 인스턴스는 동일한 가용지역의 동일한 랙에 위치한다.

![2-placement-group-cluster.png](images%2F2-placement-group-cluster.png)

- 장점(Pros): 뛰어난 네트워크를 제공한다. (향상된 네트워킹을 사용하도록 설정된 인스턴스 간 10Gbps 대역폭 - 권장)
- 단점(Cons): 랙에 장애가 발생하면 모든 인스턴스가 동시에 장애가 발생한다.
- 사용 사례:
  - 빠르게 완료해야 하는 빅 데이터 작업
  - 매우 낮은 지연 시간과 높은 네트워크 처리량을 필요로하는 애플리케이션

#### Placement Group - Spread

- 서로 다른 AZ에 걸쳐 다른 하드웨어를 가지고 있고 각 인스턴스는 다른 하드웨어에서 실행된다.

![3-placement-group-spread.png](images%2F3-placement-group-spread.png)

- 장점(Pros): 
  - 가용성 전반에 걸쳐 확장이 가능하다.
  - 여러 AZ를 사용하여 동시에 장애가 발생할 확률이 낮아진다.
  - 인스턴스는 다른 물리적 하드웨어에 있다.
- 단점(Cons):
  - AZ당 7개의 인스턴스로 제한된다.
- 사용 사례:
  - 고가용성(HA)를 극대화해야 하는 애플리케이션
  - 각 인스턴스가 서로 장애로부터 격리되어야 하는 중요한 애플리케이션

#### Placement Group - Partition

![4-placement-group-partition.png](images%2F4-placement-group-partition.png)

- AZ당 최대 7개의 파티션을 생성할 수 있다.
- 최대 100개의 EC2 인스턴스를 사용할 수 있다.
- 파티션의 인스턴스가 다른 파티션의 인스턴스와 랙을 공유하지 않는다.
- 파티션 장애는 많은 EC2에 영향을 줄 수 있지만 다른 파티션에는 영향을 주지 않는다.
- EC2 인스턴스는 EC2 메타데이터 서비스 덕분에 어느 파티션에 속해 있는지 액세스할 수 있다.
- 사용 사례:
  - HDFS, HBase, Cassandra, Kafka

#### Launch Type

- On Demand Instances: 짧은 워크로드, 예측 가능한 가격, 신뢰성을 제공한다.
- Spot Instances: 짧은 워크로드, 저렴한 비용으로 인해 인스턴스가 손실될 수 있다. 신뢰성을 제공하지 않는다.
- Reserved: (최소 1년)
  - Reserved Instances: 긴 워크로드
  - Convertible Reserved Instances: 유연한 인스턴스를 사용하는 긴 워크로드
  - Highest to lowest discount: 모든 선불, 부분 선불, 선불 없음
- Dedicated Instances: 다른 사용자와 하드웨어를 공유하지 않는다.
- Dedicated Hosts: 전체 물리적 서버를 제공하고 인스턴스 배치를 제어할 수 있다.
  - 코어 또는 CPU 소켓 레벨에서 작동하는 소프트웨어 라이센스에 적합하다.
  - 인스턴스 재부팅이 동일한 호스트에 유지되도록 호스트 선호도를 정의할 수 있다.

#### EC2 Graviton

- AWS Graviton 프로세서는 최고의 가격 성능을 제공한다.
- 많은 Linux OS, Amazon Linux 2, RedHat, SUSE, Ubuntu에서 지원한다.
- Windows 인스턴스는 지원하지 않는다.
- Graviton2: 동급의 5세대 x86 기반 인스턴스에 비해 가격 성능 40% 향상
- Graviton3: Graviton2 대비 최대 3배 향상된 성능 제공
- 사용 사례: 앱 서버, 마이크로서비스, HPC, CPU 기반 ML, 비디오 인코딩, 게임, 인메모리 캐시 등..

#### EC2 지표

- CPU: CPU 사용률 + Credit 사용률 / 잔량
- Network: Network In / Out
- Status Check
  - Instance status: EC2 VM 확인
  - System status: 기반 하드웨어 확인
- Disk: Ops/Bytes에 대한 읽기/쓰기(인스턴스 스토어만 해당)
- **RAM은 EC2 지표에 포함되지 않는다.**
  - EC2 인스턴스에서 고객 맞춤형 지표로 CloudWatch로 직접 푸시해야 한다.

#### EC2 Instance Recovery

- Status Check
  - Instance status: EC2 VM 확인
  - System status: 기반 하드웨어 확인

![5-ec2-instance-recovery.png](images%2F5-ec2-instance-recovery.png)

- EC2 인스턴스가 CloudWatch 알람으로 모니터링된다.
  - CloudWatch 알람은 시스템의 상태를 확인하기 위해 설치되어 있다.
  - 상태 확인이 실패하는 경우 CloudWatch 알람으로 EC2 인스턴스 복구 작업을 정의할 수 있다.
- 일부 데이터는 사라지지만 **Private IP, Public IP, Elastic IP, 메타데이터, 배치 그룹은 유지**된다.
- SNS 토픽와 연결되어 슬랙과 같은 메신저를 통해 사용자에게 알람을 전송할 수 있다.

---

### High Performance Computing (HPC)

- 클라우드는 HPC를 수행하기에 완벽한 장소다.
- 매우 많은 수의 리소스를 즉시 생성할 수 있다.
- 더 많은 리소스를 추가하여 결과 도출 시간을 단축할 수 있다.
- 사용한 시스템에 대해서 사용한 시간만큼의 비용을 지불한다.
- 유전체학, 전산화학, 재무위험 모델링, 기상예측, 기계학습, 딥러닝, 자율주행 수행같은 작업에 사용된다.

#### Data Management & Transfer

- AWS Direct Connect: Private 보안 네트워크를 통해 GB/s의 데이터를 클라우드로 이동한다.
- Snowball & Snowmobile: PB 단위의 데이터를 클라우드로 이동한다.
- AWS DataSync: Windows용 EFS, FSx 및 On-Premise와 S3, EFS 간에 대용량 데이터를 이동한다.

#### Compute & Networking

- EC2 인스턴스
  - CPU 최적화, GPU 최적화 인스턴스를 가지고 있다.
  - Spot 인스턴스나 Spot Fleets를 사용하여 엄청난 비용 절감과 함께 Auto Scaling을 통해 우리가 필요로하는 연산의 크기에 기반해 자동으로 확장할 수 있다.
- EC2 배치 그룹: 클러스터 유형을 사용하면 최고의 네트워크 성능을 얻을 수 있다.
  - 아래의 예시에서는 10 Gbps의 네트워크 대시 시간을 확보하고 있다.

![6-compute-and-networking.png](images%2F6-compute-and-networking.png)

- EC2 Enhanced Networking (SR-IOV)
  - 대역폭 증가, PPS(초당 패킷) 증가, 지연 시간 감소
  - 옵션 1: Elastic Network Adapter (ENA)를 사용하여 100 Gbps까지 속도를 향상시킨다.
  - 옵션 2(Legacy): Intel 82599 VF를 사용하여 10 Gbps까지 속도를 향상시킨다.
- **Elastic Fabric Adapter(EFA)**
  - 향상된 HPC용 ENA로 리눅스에서만 사용할 수 있다.
  - 노드 간 통신, 긴밀하게 연결된 워크로드에 적합하다.
  - 메시지 전달 인터페이스 표준인 MPI(Message Passing Interface) 표준을 활용한다.
  - 기존 리눅스 OS를 우회하여 지연 시간이 적고 안정적인 전송을 제공한다.

#### Storage

- 인스턴스 연결(Instance-attached) 스토리지
  - EBS: io2 Block Express로 최대 256,000 IOPS까지 확장된다.
  - Instance Store: 수백만 IOPS로 확장, EC2 인스턴스와 연결, 지연 시간 감소
- Network 스토리지
  - Amazon S3: 큰 Blob으로 파일 시스템이 아니다.
  - Amazon EFS: 전체 크기에 따라 IOPS를 확장하거나 프로비저닝된 IOPS를 사용한다.
  - Amazon FSx for Lustre
    - HPC 최적화된 분산 파일 시스템
    - S3로 지원한다.

#### Automation & Orchestration

- AWS Batch
  - AWS Batch는 다중 노드 병렬 작업을 지원하므로 여러 EC2 인스턴스에 걸쳐 단일 작업을 실행할 수 있다.
  - 쉽게 작업을 예약하고 그에 따라 EC2 인스턴스를 시작할 수 있다.
- AWS ParallelCluster
  - HPC를 AWS에 배포하는 오픈 소스 클러스터 관리 도구다.
  - 텍스트 파일로 구성되어 있다.
  - VPC, 서브넷, 클러스터 유형 및 인스턴스 유형을 자동으로 생성한다.

---

### Auto Scaling Group

#### Dynamic Scaling Policy

- Target Tracking Scaling
  - 가장 간단하고 쉽게 설정 가능하다.
  - 예를 들어, 평균 ASG CPU 사용률이 40%를 유지하도록 설정할 수 있다.
- Simple / Step Scaling
  - CloudWatch 알람이 트리거되면 (예: CPU 사용률 > 70%) 유닛 2개 추가
  - CloudWatch 알람이 트리거되면 (예: CPU 사용률 < 30%) 유닛 1개 제거
- Scheduled Actions
  - 알려진 사용 패턴을 기반으로 확장을 예상한다.
  - 예를 들어, 금요일 오후 5시에 최소 용량을 10으로 늘린다.

#### Predictive Scaling

- Predictive scaling: 부하를 지속적으로 예측하고 향후 확장 일정을 계획한다.
  - 부하를 유지하기 위해 필요한 인스턴스 양을 예측하기 위해 자동으로 스케일링 액션을 미리 설정한다.

![7-predictive-scaling.png](images%2F7-predictive-scaling.png)

#### 자주 사용되는 지표

- `CPUUtilization`: 인스턴스 전체의 평균 CPU 사용률
- `RequestCountPerTarget`: EC2 인스턴스당 요청 수가 안정적인지 확인
- `Average Network In / Out`: 애플리케이션이 네트워크 바인딩된 경우 네트워크 입출력
- `Any custom metric`: CloudWatch를 사용하여 푸시하는 모든 사용자 지정 메트릭

![8-good-metric-to-scale-on.png](images%2F8-good-metric-to-scale-on.png)

#### Good to know

- Spot Fleet을 지원한다. (Spot 및 On-Demand 인스턴스 혼합)
- Lifecycle Hooks
  - 인스턴스가 실행되기 전 또는 종료되기 전에 작업을 수행한다.
  - 예를 들어 정리, 로그 추출, 특수 상태 검사를 수행한다. 
- AMI를 사용하려면 시작 구성 또는 시작 템플릿을 업데이트해야 한다.
  - 그런 다음 인스턴스를 수동으로 종료한다. (CloudFormation이 도움이 될 수 있음)
  - 또는 EC2 Instance Refresh를 사용하여 Auto Scaling한다.

#### Instance Refresh

- 목표: 시작 템플릿 업데이트 후 모든 EC2 인스턴스를 다시 생성한다.
- 이를 위해 Instance Refresh의 기본 기능을 사용할 수 있다.
- 정상적으로 유지되어야 하는 최소 인스턴스 비율을 지정한다.
  - 예시에서는 60%로 설정되어 있고, 갱신이 되는 중에도 계속 60%로 유지된다.
- 인스턴스가 사용할 준비가 될 때까지 걸리는 시간인 워밍업 시간을 지정한다.

![9-auto-scaling-instance-refresh.png](images%2F9-auto-scaling-instance-refresh.png)

#### Scaling Process

- `Launch`: 새로운 EC2를 그룹에 추가하여 용량을 증대한다.
- `Terminate`: EC2 인스턴스를 그룹에서 제거하여 용량을 줄인다.
- `HealthCheck`: 인스턴스의 상태를 확인한다.
- `ReplaceUnhealthy`: 비정상적인 인스턴스를 종료하고 다시 만든다.
- `AZRebalance`: AZ 전체의 EC2 인스턴스 수 균형을 조정한다.
- `AlarmNotification`: CloudWatch에서 알림을 수락한다.
- `ScheduledActions`: 사용자가 만든 스케줄링된 작업을 수행한다.
- `AddToLoadBalancer`: 로드 밸런서 또는 대상 그룹에 인스턴스를 추가한다.
- `InstanceRefresh` 인스턴스 새로 고침을 수행한다.
- ASG의 장점은 이러한 모든 프로세스를 중단할 수 있다는 것이다.
  - 만약 문제가 있는 인스턴스의 디버깅을 원한다면 Scaling 프로세스를 중단하고 디버깅을 할 수 있다.

#### Health Checks

- 사용 가능한 상태 점검
  - EC2 Status Checks
  - ELB Health Checks (HTTP)
  - Custom Health Check: AWS CLI 또는 AWS SDK(`set-instance-health`)를 사용하여 인스턴스의 상태를 ASG로 전송한다.
- ASG가 상태가 좋지 않은 인스턴스를 종료한 후 새로운 인스턴스를 시작한다.
- 상태 점검이 단순한지 확인하고 올바른 항목을 확인해야 한다.

![10-auto-scaling-health-check.png](images%2F10-auto-scaling-health-check.png)

- 처음 예시의 경우 `/health-server` 경로를 통해서 간단하게 인스턴스의 상태를 확인하고 있다.
- 두번째 잘못된 상태 확인의 경우 `/number-customers` 경로를 통해서 고객의 수를 확인하고 있으며 EC2 인스턴스는 응답을 위해 DB 조회를 해야 한다.
  - 만약 DB의 부하로 인해 Timeout이 발생하는 경우 ASG는 인스턴스에 문제가 발생하였다고 판단하고 인스턴스를 종료한다.

#### 애플리케이션 업데이트

![11-updating-an-application.png](images%2F11-updating-an-application.png)

- ALB에 연결되어 있는 ASG가 있고, ALB는 트래픽을 ASG로 리디렉션한다.
- 동일한 시작 템플릿에서 모든 EC2 인스턴스를 실행 중이다.
- 새로운 시작 템플릿으로 업데이트하기 위한 방법들에 대해서 알아본다.

- **동일한 ASG 유지**

![12-update-same-target-group.png](images%2F12-update-same-target-group.png)

- ASG는 유지하지만 새로운 시작 템플릿으로 새로운 인스턴스를 실행한다.
- 인스턴스가 생성되기 위해 ASG의 용량을 2배로 늘려야 할 수도 있다.
- 업데이트되는 동안 같은 ASG에 속하기 때문에 ALB는 요청을 두 인스턴스로 동시에 리다이렉트한다.
- 두 가지 버전의 애플리케이션이 있으며 "시작 템플릿 버전 2"가 정상적으로 작동한다고 판단되면 "시작 템플릿 버전 1" 인스턴스를 종료한다.

- **새로운 ASG 생성**

![13-update-split-target-group.png](images%2F13-update-split-target-group.png)

- 새로운 ASG를 생성하고 "시작 템플릿 버전 2" 인스턴스를 해당 ASG에서 실행한다.
- ALB 레벨에서 두 개의 Target Group으로 트래픽을 분산해서 전달한다.
- 이렇게 Target Group이 분산된 경우 "시작 템플릿 버전 2"의 인스턴스가 정상인지 확인하기 위해서 소량의 트래픽만 전달하도록 설정할 수 있다.
- "시작 템플릿 버전 2"가 정상이라고 판단되면 "시작 템플릿 버전 1"을 제거하여 모든 트래픽을 새로운 ASG로 전달할 수 있다.

- **새로운 ASG & ALB 생성**

![14-update-client-based-lb.png](images%2F14-update-client-based-lb.png)

- ALB1은 구버전이 실행 중인 ASG1로 트래픽을 전달한다.
- ALB2는 신버전이 실행 중인 ASG2로 트래픽을 전달한다.
- Route 53에서 CNAME을 정의하고 가중치 기반 레코드(Weighed record)를 사용한다.
  - 사용자가 DNS 쿼리를 하면 가중치(Weighted)에 기반하여 ALB를 둘 다 얻을 수 있다.
- 일부 사용자는 ALB1으로 보내고 일부 사용자는 ALB2로 보낼 수 있다.
- 이렇게 아키텍처를 설계하는 경우 테스트 클라이언트가 수동으로 ALB2로 접속하여 테스트를 진행할 수 있다.
- 테스트가 완료되면 모든 요청이 ALB2로 전달되도록 가중치 기반 레코드를 수정한다.

---

### EC2 Spot Instance

- 온디맨드 대비 최대 90% 저렴한 가격으로 사용할 수 있다.
- 최대 스팟의 가격을 정의하고 "현재 스팟의 가격" < "최대 스팟의 가격"인 경우에 인스턴스를 사용할 수 있다.
  - 시간당 스팟 가격은 수요와 제안이 다르고, 수용 용량이 다르면 스팟 가격이 올라갈 수 있다.
  - 스팟 가격이 최대 가격을 초과하면 스팟 인스턴스가 중단하거나 종료될 수 있고 종료될 때까지 유예 기간 2분이 주어진다.
- 배치 작업 데이터 분석 또는 장애에 강한 워크로드에 사용된다.
- 중요한 작업이나 데이터베이스에는 적합하지 않은 유형이다.

![15-ec2-spot-instance.png](images%2F15-ec2-spot-instance.png)

- `m4.large` 유형을 확인해보면 AZ에 따라 가격이 달라지는 것을 확인할 수 있다.
- 또한 시간대에 따라서도 가격의 변동이 있으며, 사용자가 정한 최대값을 넘어가는 시점에는 인스턴스가 회수된다.
- 만약 사용자가 최대 가격을 높게 설정하였다면 인스턴스가 회수되는 확률이 줄어들 것이다.

#### Spot Fleets

- 스팟 인스턴스 + On-Demand 인스턴스 (선택) 모음이다.
- 스팟 인스턴스에 대해 지불할 수 있는 최대 가격을 설정한다.
- 여러 인스턴스 유형을 조합하여 생성할 수 있다.
- EC2 standalone, Auto Scaling Group (launch template), ECS (underlying ASG), AWS Batch를 지원한다.
- 목표는 수요의 기준을 충족하고 훨씬 낮은 가격으로 더 많은 수요에 맞춰 작업량을 최적화하는 것이다.
- 아래와 같은 제한 사항이 있다.
  - 스팟 플릿 or EC2 플릿은 최대 1만개 용량을 가질 수 있다.
  - 한 리전에 있는 스팟 플릿, EC2 플릿은 10만개를 가질 수 있다.

- 스팟 플릿은 가격 제약 조건으로 목표 용량을 충족하려고 노력할 것이다.
  - 인스턴스 유형(`m5.large`), OS, 가용성 영역(AZ)를 정의한다.
  - 플릿이 선택할 수 있도록 여러 개의 Launch Pool을 사용한다.
  - 용량 또는 최대 비용에 도달하면 스팟 플릿에서 인스턴스 실행을 중지한다.
- 스팟 인스턴스 할당 전략
  - `lowestPrice`: 최저 가격으로 풀에서 제공한다. (비용 최적화, 짧은 워크로드)
  - `diversified`: 모든 풀에 분산 (가용성, 긴 워크로드에 적합)
  - `capacityOptimized`: 인스턴스 수에 맞는 최적의 용량을 갖춘 풀을 선택한다.
  - `priceCapacityOptimized` (권장): 사용 가능한 용량이 가장 높은 풀을 선택한 다음, 가장 저렴한 가격으로 풀을 선택한다. (대부분의 워크로드에 적합)
- **스팟 플릿을 사용하면 가장 낮은 가격으로 스팟 인스턴스를 자동으로 요청**할 수 있다.

---

### ECS

#### Docker

- 도커는 앱을 배포하기 위한 소프트웨어 개발 플랫폼이다.
- 앱은 모든 OS에서 실행할 수 있는 컨테이너에 포장되어 있다.
- 앱은 실행 위치에 관계없이 동일하게 실행된다.
  - 모든 머신에서 작동한다. (호환성 문제 X, 예측 가능한 동작)
  - 작업 공수가 적게 든다.
  - 유지 관리 및 구축이 용이하다.
  - 모든 언어, 모든 OS, 모든 기술과 함께 작동한다.
- 컨테이너에 할당되는 메모리/CPU의 수를 제어할 수 있다.
- 컨테이너를 매우 빠르게 스케일링할 수 있다.
- Virtual Machine보다 효율성이 향상된다.

#### 도커 컨테이너 관리

- 컨테이너를 관리하려면 컨테이너 관리 플랫폼이 필요하다.
- Amazon Elastic Container Service (Amazon ECS)
  - Amazon 자체 컨테이너 플랫폼이다.
- Amazon Elastic Kubernetes Service (Amazon EKS)
  - Amazon이 관리하는 Kubernetes다. (오픈 소스)
- AWS Fargate
  - Amazon 자체 서버리스 컨테이너 플랫폼이다.
  - ECS 및 EKS와 함께 작동한다.

#### ECS - 사용 사례

- 마이크로서비스 실행
  - 동일한 시스템에서 여러 개의 오더 컨테이너를 실행할 수 있다.
  - 간편한 서비스 검색 기능을 통해 커뮤니케이션을 향상할 수 있다.
  - 애플리케이션 로드 밸런서 및 네트워크 로드 밸런서와 직접 통합된다.
  - 자동 확장 기능을 제공한다.
- 배치 처리/예약된 작업 실행
  - On-Demand/Reserved/Spot 인스턴스에서 실행할 ECS 작업을 예약한다.
- 애플리케이션을 클라우드로 마이그레이션
  - 사내에서 실행 중인 기존 애플리케이션을 도커화한다.
  - Amazon ECS에서 실행할 도커 컨테이너를 이동한다.

#### ECS - 개념

- ECS Cluster: EC2 인스턴스의 논리적 그룹화
- ECS Service: 실행해야 할 작업 수와 실행 방법을 정의한다.
- Task Definitions: 도커 컨테이너를 실행하는 방법을 ECS에 알려주는 JSON 양식의 메타데이터 (이미지 이름, CPU, RAM 등..)
- ECS Task: 실행 중인 도커 컨테이너인 태스크 정의 인스턴스
- ECS IAM Roles
  - EC2 Instance Profile: EC2 인스턴스에서 사용한다. (예: ECS에 API 호출, 로그 전송 등..)
  - ECS Task IAM Role: 각 작업(Task)이 특정 역할을 갖도록 한다. (예: S3, DynamoDB로 API 호출 등..)

![16-ecs-concepts.png](images%2F16-ecs-concepts.png)

- ECS 클러스터가 있고 EC2 인스턴스 내에서 "Service A"가 실행된다.
  - "Service A"를 실행하려면 먼저 작업 정의(Task Definition)를 제공해야 한다.
  - 작업 정의가 인스턴스화되면 특정 서비스를 위해 EC2 인스턴스에서 실행되는 컨테이너가 생성된다.
  - 사용자는 EC2 인스턴스의 수를 스케일할 수 있다.
- 여러 개의 EC2 인스턴스가 있기 때문에 EC2 인스턴스를 관리할 "Auto Scaling Group"도 가질 수 있다.
  - 이것이 ECS의 일부인지 확인하기 위해 EC2 인스턴스 프로필을 정의해야 한다.
  - EC2 인스턴스가 프로필을 정의하고 EC2 인스턴스가 ECS 서비스에 등록되도록 하고 CLoudWatch 로그 서비스로 로그를 보낼 수 있다.
- 서비스가 직접 S3 버킷에 접근할 수 있도록 하기 위해서 IAM 역할을 위한 서비스 내 작업을 제공해야 한다.
- 사용자가 원하는 경우 새로운 서비스인 "Service B"를 실행할 수 있다.

#### ALB 통합

- 동적 포트 매핑(Dynamic Port Mapping)을 가져온다.
- 동일한 EC2 인스턴스에서 동일한 애플리케이션의 여러 인스턴스를 실행할 수 있다.
- ALB가 EC2 인스턴스에서 올바른 포트를 찾는다.
- 아래와 같은 사용 사례가 있다.
  - 하나의 EC2 인스턴스에서 실행하더라도 복원력을 향상시킨다.
  - CPU/코어 활용률을 극대화한다.
  - 앱 가동 시간에 영향을 주지 않고 롤링 업그레이드를 수행할 수 있는 기능을 제공한다.

![17-ecs-alb-integration.png](images%2F17-ecs-alb-integration.png)

- 서비스 하나에서 실행 중인 ECS 클러스터와 2개의 EC2 인스턴스에 걸쳐 4개의 작업이 있다.
- 각 EC2 인스턴스엔 같은 태스크가 2개 있고 로드 밸런서를 통해 사용자가 애플리케이션에 액세스한다.
- 각 ECS 태스크는 서로 다른 포트를 갖게 된다.
  - 첫 번째 ECS 태스크가 이 포트를 확보하면 동적 포트 매핑을 사용하고 있기 때문에 로드 밸런서가 이 포트를 찾아낸다.
  - 두 번째 ECS 태스크는 다른 포트를 가져오지만 ALB는 첫 번째와 동일하게 포트를 찾을 수 있다.
- 모든 ECS 작업에서 포트가 안정적이지 않더라도 로드 밸런서는 모든 포트를 찾아낸다.

#### AWS Fargate

- AWS에서 도커 컨테이너를 실행시킬 수 있다.
- 인프라를 프로비저닝하지 않으므로 관리할 인스턴스가 없다.
- 작업 정의(Task Definition)를 생성한다.
- AWS는 필요한 CPU/RAM을 기반으로 컨테이너를 실행한다.
- 확장을 위해서는 EC2 인스턴스가 아니라 단순히 작업의 수만 늘리면 된다.

![18-aws-fargate.png](images%2F18-aws-fargate.png)

#### ECS - Security & Networking

- 실행 중인 도커 컨테이너에 환경 변수로 비밀 및 구성을 주입할 수 있다.
  - "SSM Parameter Store" 및 "Secret Manager"와 통합한다.
- ECS Task 네트워킹
  - none: 네트워크 연결 없음, 포트 매핑 없음
  - bridge: 도커의 가상 컨테이너 기반 네트워크 사용
  - host: Docker의 네트워크 우회, 기본 호스트 네트워크 인터페이스 사용
  - awsvpc
    - 인스턴스에서 시작된 모든 작업은 고유한 ENI와 개인 IP 주소를 가진다.
    - 단순화된 네트워킹, 향상된 보운, 보안 그룹, 모니터링, VPCFlowLogs
    - Fargate 작업의 기본 모드다.

#### Service Auto Scaling

- 원하는 작업의 수를 자동으로 증가하거나 감소시킬 수 있다.
- Amazon ECS가 AWS Application Auto Scaling 서비스를 활용한다.
- ECS 서비스 레벨에서 CloudWatch에서 CPU 및 RAM을 추적한다.
- **Target Tracking**: 특정 CloudWatch 메트릭의 목표 값에 기반한 확장을 지원한다.
- **Step Scaling**: 지정된 CloudWatch 알람에 기반한 확장을 지원한다.
- **Scheduled Scaling**: 지정된 날짜/시간을 기준으로 한 스케일링(예측 가능한 변경 사항)을 지원한다.
- ECS Service Auto Scaling(테스크 레벨) <> EC2 Auto Scaling (EC2 인스턴스 레벨)
- Fargate Auto Scaling 설정이 훨씬 용이하다.

#### Spot Instances

- ECS Classic (EC2 Launch Type)
  - 기본 EC2 인스턴스를 Spot Instance(ASG에서 관리)로 사용할 수 있다.
  - 인스턴스는 실행 중인 작업을 제거하기 위해 드레이닝 모드로 전환될 수 있다.
  - 비용 절감 효과는 좋지만 신뢰성에 영향을 미칠 수 있따.
- AWS Fargate
  - 주문형 기준 워크로드에 대한 최소 작업을 지정한다.
  - 비용 절감을 위해 "FARGATE_SPOT"에서 실행 중인 작업을 추가할 수 있다.
  - On-Demand 또는 스팟에 관계없이 Fargate는 부하에 따라 잘 확장된다.

---

### Elastic Container Registry (ECR)

- AWS에 도커 이미지를 저장하고 관리한다.
- Private 및 Public 리포지토리를 지원한다.
- ECS와 완벽하게 통합된다.
- IAM을 통해 접근이 제어된다. (권한 오류 => 정책 확인)
- 이미지 취약성 검사, 버전 지정, 이미지 태그, 이미지 라이프사이클 등을 지원한다.

![19-elastic-container-registry.png](images%2F19-elastic-container-registry.png)

#### Cross Region Replication

- ECR Private 레지스트리는 리전 간 및 계정 간 복제를 모두 지원한다.

![20-cross-region-replication.png](images%2F20-cross-region-replication.png)

- "CodeBuild"로부터 이미지를 빌드하는 파이프라인이 있다.
- "CodeBuild"는 컨테이너를 실행해 이미지를 빌드하고 ECR에 등록하고 푸시한다.
- 만약 교차 지역 복제를 설정했다면 이 이미지들은 다른 리전으로 직접 복제된다.
  - 다른 리전들은 이 이미지들을 다시 빌드할 필요가 없다는 장점이 있다.
  - 그리고 직접 실행할 수도 있으므로 다른 리전의 ECS 상에서 작업을 실행하여 글로벌 애플리케이션을 제공할 수 있다.

#### Image Scanning

- 이미지를 푸시할 때 수동으로 스캔하거나 자동으로 스캔할 수 있다.

![21-ecr-image-scanning.png](images%2F21-ecr-image-scanning.png)

- 기본 스캔은 일반적인 취약점인 CVE를 찾는다.
  - 만약 취약점이 발견되는 경우 EventBridge로 보낼 수 있다.
- "Amazon Inspector"을 활용하여 OS 및 프로그래밍 언어 취약성을 분석하는 항샹된 검색(Enhanced Scanning)을 사용할 수 있다.
  - 만약 취약점이 발견되는 경우 EventBridge로 보낼 수 있다.
  - 검색 결과는 AWS 콘솔 내에서 검색할 수 있다.

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

![22-eks-diagram.png](images%2F22-eks-diagram.png)

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

---

### AWS App Runner

- 웹 애플리케이션과 API를 손쉽게 확장할 수 있는 완벽환 관리 서비스다.
- 인프라스트럭처를 구축한 경험이 필요하지 않다.
- 소스 코드 또는 컨테이너 이미지로 시작된다.
- 웹 앱 자동 구축 및 배포를 지원한다.
- 자동 확장, 고가용성, 로드 밸런싱, 암호화를 지원한다.
- VPC 액세스를 지원한다.
- 데이터베이스, 캐시 및 메시지 대기열 서비스에 연결할 수 있따.
- 소스 코드에서 빌드 된다. (Python, Nodejs, Java)
- 사용 사례: 웹 앱, API, 마이크로서비스, 신속한 운영 구축

![23-app-runner.png](images%2F23-app-runner.png)

#### Multi-Region Architecture

![24-apprunner-multi-region-architecture.png](images%2F24-apprunner-multi-region-architecture.png)

- `us-east-1`과 `us-west-2` 두 개의 리전이 있다.
- 서로 다른 리전의 ECR에 동일한 도커 이미지가 복제 및 저장되어 있다.
- `us-east-1` 리전에서 App Runner 서비스를 실행시키고 데이터를 DynamoDB에 지속시킬 수 있다.
  - DynamoDB 테이블 또한 전역적으로 복사할 수 있다.
- `us-west-2` 리전에서도 App Runner 서비스를 실행시키고 데이터를 DynamoDB에 지속시킬 수 있다.
  - 이미 복제되어 있는 DynamoDB 테이블의 데이터를 참조할 수 있다.
- Route 53의 설정을 살펴본다.
  - 동일한 레코드 이름에 라우팅 정책은 Latency로 설정되어 있다.
  - 자동적으로 대기 시간 측면에서 가장 좋은 엔드포인트로 리다이렉트된다.

---

### Amazon ECS Anywhere

- 사용자가 관리하는 인프라(온프레미스, VM 등)에서 컨테이너를 쉽게 실행할 수 있다.
- 사용자가 모든 환경에서 기본 Amazon ECS 작업을 구현할 수 있도록 지원한다.
- 완전히 관리되는 Amazon ECS Control Plane이다.
- "ECS Container Agent"와 "SSM Agent"를 설치해야 한다.
- 우리 서비스와 작업을 위한 "외부(EXTERNAL)" 실행 유형을 지정한다.
- AWS 영역에서 안정적으로 연결되어 있어야 한다.
- 대표적인 사용 사례는 아래와 같다.
  - 컴플라이언스, 규제 및 지연 시간 요구사항 충족
  - AWS 리전 밖에서 앱을 실행하고 다른 서비스와 더 가까운 곳에서 앱 실행
  - On-Premise 머신러닝, 비디오 처리, 데이터 처리 등..

![25-ecs-anywhere.png](images%2F25-ecs-anywhere.png)

---

### Amazon EKS Anywhere

![26-eks-anywhere.png](images%2F26-eks-anywhere.png)

- AWS 외부에 생성된 Kubernetes 클러스터를 생성 및 운영한다.
- Amazon EKS Distro를 활용한다. (AWS의 Kubernetes 번들 출시)
- 지원 비용 절감 및 중복 타사 툴 유지 관리를 방지한다.
- EKS Anywhere 설치 관리자를 사용하여 설치한다.
- 선택적으로 EKS 커넥터를 사용하여 EKS Anywhere 클러스터를 AWS에 연결한다.
  - **Fully Connected & Partially Disconnected**: Amazon EKS Anywhere 클러스터를 AWS에 연결하고 EKS 콘솔을 활용할 수 있다.
  - **Fully Disconnected**: EKS Distro를 설치하고 오픈 소스 툴을 활용하여 클러스터를 관리해야 한다.

---

### AWS Lambda

- AWS Lambda는 수많은 서비스들과 통합될 수 있다.

![27-lambda-integration-main-one.png](images%2F27-lambda-integration-main-one.png)

#### Serverless Thumbnail creation

![28-serverless-thumbnain-creation.png](images%2F28-serverless-thumbnain-creation.png)

- S3 버킷에 새로운 이미지가 업로드된다.
- 썸네일을 생성하기 위해 Lambda Function을 트리거한다.
- 새롭게 생성된 썸네일이 S3 버킷에 저장되고 DynamoDB에 썸네일의 메타데이터를 푸시한다.
  - 메타데이터는 이미지 이름, 이미지 크기 데이터 등등을 포함하고 있다.

#### Serverless Cronjob

- EventBridge를 이용해 시간 기반 트리거를 생성하여 일정 시간마다 Lambda Function을 트리거하여 작업을 효과적으로 수행하도록 할 수 있다.
  - 관리할 서버가 없는 Cronjob을 지원한다.

![29-serverless-cronjob.png](images%2F29-serverless-cronjob.png)

#### Language Support

- Lambda는 수많은 언어를 지원한다.
- Node.js, Python, Java, C# (.NET Core), Golang, C# / Powershell, Ruby, Custom Runtime API
- Lambda Container Image
  - 컨테이너 이미지는 Lambda Runtime API를 구현해야 한다.
  - 임의의 도커 이미지를 실행하려면 ECS / Fargate가 선호된다.

#### Limit

- RAM: 128MB ~ 12,240MB(10GB)
- CPU: RAM에 연결됨 (수동으로 설정할 수 없음)
  - 2개의 vCPU가 1,769MB의 RAM으로 할당된다.
  - 6개의 vCPU가 10,240MB의 RAM으로 할당된다.
- Timeout: 최대 15분
- `/tmp` Storage: 10,240MB(10GB)
- Deployment Package: 50MB(압축), 250MB(비압축) 레이어 포함
- Concurrent Execution: 1,000개의 동시 실행, 증가가 가능한 Soft Limit
- Container Image Size: 10GB
- 호출 페이로드(요청/응답): 6MB(sync), 256KB(async)

#### Lambda Concurrency & Throttling

- 동시 실행 횟수 제한: 최대 1,000회 동시 실행

![30-lambda-concurrency-throttling.png](images%2F30-lambda-concurrency-throttling.png)

- Function 수준에서 "reserved concurrency"을 설정할 수 있다.
- 동시성 제한을 초과하는 각 호출은 "Throttle"을 트리거한다.
- AWS 서비스 할당량의 할당량 증가를 요청할 수 있다.

#### Lambda Concurrency Issue

- 동시성을 예약하지 않으면 다음과 같은 현상이 발생할 수 있다.

![31-lambda-concurrency-issue.png](images%2F31-lambda-concurrency-issue.png)

- ALB, API Gateway, SDK/CLI가 각각 Lambda Function을 호출하고 있다.
- 갑자기 ALB에 수많은 사용자의 요청이 몰리면서 1,000개의 함수가 동시에 실행된다.
- API Gateway와 SDK/CLI를 통해서 들어오는 요청들은 많지 않지만 ALB를 통해 들어오는 요청에 의해 쓰로틀링된다.

#### Lambda & CodeDeploy

- CodeDeploy를 사용하면 Lambda 별칭에 대한 트래픽 시프트를 자동화할 수 있다.
- 기능이 SAM 프레임워크 내에 통합된다.

![32-lambda-codedeploy.png](images%2F32-lambda-codedeploy.png)

- 처음에는 V1에 100% V2에 0%의 트래픽이 전달된다.
  - 시간이 지남에 따라 80/20과 같은 형식으로 트래픽 전달이 변경된다.
  - CodeDeploy는 트래픽을 전달하는 비율이 시간에 따라 달라진다.

- Liner: 100%까지 N분마다 트래픽이 증가한다.
  - `Linear10PercentEvery3Minutes`
  - `Linear10PercentEvery10Minutes`
- Canary: X퍼센트씩 100%까지 증가시킨다.
  - `Canary10Percent5Minutes`
  - `Canary10Percent30Minutes`
- AllAtOnce: 즉시 트래픽을 변경시킨다.
- Lambda Function의 상태를 점검하기 위해 사전 & 사후 트래픽 후크를 생성할 수 있다.

#### Lambda Logging & Monitoring & Tracing

- CloudWatch
  - AWS Lambda 실행 로그는 AWS CloudWatch 로그에 저장된다.
  - AWS Lambda 메트릭은 AWS CloudWatch 메트릭(성공적인 호출, 오류율, 지연 시간, 시간 초과 등)에 표시된다.
  - AWS Lambda Function에 CloudWatch Logs에 쓰기 권한을 부여하는 IAM 정책과 함께 실행 역할이 있는지 확인한다.
- X-Ray
  - X-Ray로 Lambda를 추적할 수 있다.
  - Lambda 구성에서 활성화한다. (X-Ray 데몬 실행)
  - 코드에서 AWS SDK를 사용한다.
  - Lambda Function에 올바른 IAM 실행 역할이 있는지 확인한다.

#### Lambda in VPC

![33-lambda-in-vpc.png](images%2F33-lambda-in-vpc.png)

- 기본적으로 Lambda를 배포하면 AWS 클라우드에 배포되기 때문에 공용 API에 접속할 수 있다.
  - DynamoDB의 공용 API에 액세스할 수 있다.
  - 하지만 사설 VPC, 사설 서브넷이 있다면 작동하지 않는다.
- VPC에 Lambda Function을 배포하면 Lambda Function은 보안 그룹을 할당받게 된다.
  - VPC에 배포되고 RDS 데이터베이스에 액세스 권한이 작동된다.
  - 하지만 공용 인터넷에 접속하려는 경우 문제가 발생할 수 있다.
  - 만약 사설 서브넷에 Lambda가 배포되어 있다면 NAT 게이트웨이를 배포하여 공용 인터넷에 접속할 수 있도록 설정할 수 있다.
  - 같은 방식으로 NAT 게이트웨이나 인터넷 게이트웨이를 통해서 DynamoDB로 접근할 수 있다.
  - 만약 모든 트래픽을 내부에 유지하고 싶다면 DynamoDB의 VPC 엔드포인트를 생성하고 Lambda Function을 직접 연결할 수 있다.
- 단, Lambda가 CloudWatch Logs로 로그를 전달할 때는 Endpoint나 NAT 게이트웨이가 필요하지 않다.

![34-filxed-public-ip.png](images%2F34-filxed-public-ip.png)

- AWS 공용 클라우드에 Lambda Function을 배포하면 AWS 공용 IP로부터 임의의 공용 IP를 할당받게 된다.
  - 인터넷에 액세스하면 액세스하려는 API가 임의 공용 IP로부터 트래픽을 보게 된다.
- 사설 서브넷에 Lambda Function을 배포하면 NAT 게이트웨이와 통신하고 되고, 인터넷 게이트웨이와도 통신하게 된다.
  - 인터넷에 있는 API와도 통신이 가능하다.
  - NAT 게이트웨이를 통해 인터넷으로 오는 모든 트래픽은 고정되고 EIP에서 보게 된다.
  - 사실상 모든 Lambda Function 트래픽은 같은 EIP에서 보이게 된다.
- 이러한 아키텍처를 통해 Lambda Function에는 인터넷 API 액세스 시 고정 IP를 줬다.
  - 따라서 API 끝에서 공용 사이드의 네트워크 보안을 사용할 수 있다.

#### Synchronous Invocation

- 동기식: CLI, SDK, API Gateway
  - 결과가 바로 반환된다.
  - 클라이언트 측에서 오류 처리가 발생해야 한다. (시도 횟수, 지수 백오프 등..)

![35-synchronous-invocation.png](images%2F35-synchronous-invocation.png)

- SDK가 있고 SDK가 Lambda Function을 호출한다. 
  - Lambda Function은 작업을 수행하고 응답을 반환한다.
- 클라이언트가 API 게이트웨이를 호출하고 요청은 Lambda로 프록시된다.
  - Lambda는 정의된 작업을 진행하고 결과를 API 게이트웨이로 반환한다.
  - 결과를 전달받은 API 게이트웨이는 클라이언트에게 응답을 전달한다.

#### Asynchronous Invocation

- S3, SNS, Amazon EventBridge를 사용한다.
- Lambda는 오류에 대해서 최대 3번까지 재시도한다.
- 처리가 idempotent(재시도인 경우)인지 확인한다.
- 실패한 처리에 대한 DLQ(Dead-letter queue) - SNS 또는 SQS를 정의할 수 있다.

![36-asynchronous-invocation.png](images%2F36-asynchronous-invocation.png)

#### Architecture Discussion

![37-architecture-discussion.png](images%2F37-architecture-discussion.png)

- 위의 예시는 S3가 Amazon SNS에 알림을 발생시키고 Amazon SNS에 Lambda Function을 연결하고 있다.
- 아래의 예시는 S3를 이용한다.
  - SNS를 실생시키고 SQS로 메시지를 보낸다.
  - SQS는 Lambda Function에서 소비하게 된다.
  - 둘은 비슷해 보이지만 S3에 이미지가 저장될 때마다 Lambda가 호출된다.

- 위의 예시에서 만약 1초에 1천 개의 파일을 업로드한다면 S3와 SNS는 바로 실행된다.
  - S3는 SNS에서 알림을 트리거하고 즉시 Lambda Function을 호출한다.
  - SNS에 많은 알림이 있다면 동시에 수많은 Lambda Function이 실행되게 된다.
  - 병렬적으로 동시에 수많은 일을 처리할 수 있지만, 실패한 작업이 유실될 수 있다.
- 아래의 예시는 많은 양의 요청이 들어오면 SQS의 큐가 가득차게 된다.
  - Lmabda Function도 당연히 확장되지만 요청을 하나씩 처리하기 때문에 큐는 빠르게 처리되지 않는다.
  - 배치 실행이 조금 더 효율적이지만 약간의 지연이 발생할 수 있다.
  - 하지만 Lambda Function에서 실패가 발생하는 경우에도 작업은 유실되지 않고 SQS 큐에 보관된다.

---

### Load Balancer

#### Types

- AWS에는 총 4가지 종류의 관리형 로드 밸런서를 보유하고 있다.
- Classic Load Balancer (v1 - old generation): 2009년도에 출시되었으며 CLB로 부른다.
  - HTTP, HTTPS, TCP, SSL, (secure TCP)를 지원한다.
- Application Load Balancer (v2 - new generation): 2016년도에 출시되었으며 ALB로 부른다.
  - HTTP, HTTPS, WebSocket을 지원한다.
- Network Load Balancer (v2 - new generation): 2017년도에 출시되었으며 NLB로 부른다.
  - TCP, TLS (secure TCP), UDP를 지원한다.
- Gateway Load Balancer: 2020년도에 출시되었으며 GWLB라고 부른다.
  - Layer 3에서 작동한다. (IP 프로토콜)
- 전반적으로, 더 많은 기능을 제공하기 때문에 새로운 세대의 로드 밸런서를 사용하는 것이 좋다.
- 일부 로드 밸런서는 내부(Private) 또는 외부(Public) ELB로 설정할 수 있다.

#### Classic Load Balancer (v1)

- 상태 점검은 SSL을 포함하여 HTTP(Layer 7) 또는 TCP(Layer 4) 기반일 수 있다.
- 하나의 SSL 인증서만 지원한다.
  - SSL 인증서에는 여러 SAN(Subject Alternate Name)이 있을 수 있지만 SAN을 추가/편집/제거할 때마다 SSL 인증서를 변경해야 한다.
  - 가능하면 SNI(Server Name Indicator)와 함께 ALB를 사용하는 것이 좋다.
  - 별개의 SSL 인증서를 원하는 경우 여러 CLB를 사용할 수 있다.
- TCP -> TCP가 모든 트래픽을 EC2 인스턴스로 전달한다.
  - 양방향 SSL 인증을 사용하는 유일한 방법이다.

![38-classic-load-balancer.png](images%2F38-classic-load-balancer.png)

#### Application Load Balancer (v2)

- 애플리케이션 로드 밸런서는 HTTP(Layer 7)이다.
- 여러 시스템(대상 그룹)에 걸쳐 여러 HTTP 애플리케이션에 대한 로드 밸런싱을 지원한다.
- 동일한 시스템(예: 컨테이너)에서 여러 애플리케이션에 대한 로드 밸런싱을 지원한다.
  - ECS와의 적합성이 뛰어나고 동적 포트 매핑이 가능하다.
- HTTP/2 및 WebSocket을 지원한다.
- HTTP 요청을 HTTPS로 전달하는 것과 같은 리디렉션을 지원한다.
- 경로, 헤더, 쿼리 문자열에 대한 라우팅 규칙을 지원한다.

![39-http-based-traffic.png](images%2F39-http-based-traffic.png)

- 외부에 ALB가 있고 두 개의 대상 그룹(Target Group)이 있다.
  - 하나의 대상 그룹은 `/user` 경로의 요청을 위해 대상 그룹이며 다른 하나는 `/search` 경로의 요청을 처리하기 위한 대상 그룹이다.
  - 각각의 대상 그룹은 서로 다른 상태 확인을 받게 된다.

- ALB는 여러 종류의 대상 그룹을 가질 수 있다.
  - EC2 Instance: Auto Scaling Group에 의해 관리되고 있어야 하며 HTTP를 사용한다.
  - ECS Task: ECS에 의해 관리되고 있어야 하며 HTTP를 사용한다.
  - Lambda Function: HTTP 요청이 JSON 이벤트로 변환된다.
  - IP 주소: 반드시 사설(Private) IP여야 한다.
  - ALB는 여러 대상 그룹으로 라우팅할 수 있다.
  - 상태 확인은 대상 그룹 수준이다.

#### Network Load Balancer (v2)

- 네트워크 로드 밸런싱 장치(Layer 4)를 통해 다음을 수행할 수 있다.
  - TCP 및 UDP 트래픽을 인스턴스로 전달한다.
  - 초당 수백만 건의 요청을 처리한다.
  - 대기 시간 ~ 100ms(ALB의 경우 400ms)
- NLB는 AZ당 하나의 정적 IP를 가지며, Elastic IP를 할당할 수 있다.
  - 이렇게 할당된 Elastic IP는 특정 IP 화이트리스트에 유용하게 사용된다.
- NLB는 최고 성능, TCP 또는 UDP 트래픽에 사용된다.
- AWS 프리 티어에는 포함되지 않는다.

- NLB는 아래와 같은 대상 그룹을 가질 수 있다.
  - EC2 인스턴스
  - IP 주소: 반드시 Private IP
  - ALB

![40-nlb-target-group.png](images%2F40-nlb-target-group.png)

- 지역 NLB DNS 이름을 확인하면 모든 NLB 노드의 IP 주소가 반환된다.
  - 예시에서는 `my-nlb-1234567890abcdef.elb.us-east-1.amazon.aws.com`
- 영역 DNS 이름 (Zonal DNS Name)
  - NLB에 각 노드에 대한 DNS 이름이 있다.
  - 각 노드의 IP 주소를 결정하는 데 사용한다.
  - 예시에서 `us-east-1a.my-nlb-1234567890abcdef.elb.us-east-1.amazon.aws.com`은 `172.31.7.90` IP와 매핑된다.
  - 지연 시간 및 데이터 전송 비용 최소화에 사용된다.
  - 앱별로 특정한 로직을 구현해야 한다.

![41-nlb-zonal-dns-name.png](images%2F41-nlb-zonal-dns-name.png)

#### Gateway Load Balancer

- AWS에서 타사 네트워크 가상 어플라이언스를 구축, 확장 및 관리할 수 있다.
- 예를 들어, 방화벽, 침입 탐지 및 방지 시스템, 심층 패킷 검사 시스템, 페이로드 조작 등..
- Layer 3(Network Layer)에서 작동하며 IP 패킷을 사용한다.
- 아래의 기능들과 투명하게 결합한다.
  - Transparent Network Gateway: 모든 트래픽에 대해 단일 진입/종료
  - Load Balancer: 트래픽을 가상 어플라이언스로 분산한다.
- 포트 6081에서 GENEVE 프로토콜을 사용한다.

![42-gateway-load-balancer.png](images%2F42-gateway-load-balancer.png)

- 사용자들은 우리의 애플리케이션에 액세스하기를 원한다.
- 하지만 우리는 타사 보안 가상 어플라이언스를 이용해 모든 트래픽을 스캔하기를 원한다.
- GWLB를 배포하고 라우트 테이블을 수정하여 모든 트래픽이 GWLB를 경유하도록 설정한다.
  - GWLB는 모든 트래픽을 수신한 후 보안 가상 어플라이언스가 설치되어 있는 대상 그룹으로 전달하여 트래픽을 분석한다.
  - 문제가 없다면 트래픽을 다시 GWLB로 전달하고 사용자의 요청은 애플리케이션으로 전달된다.

- GWLB는 아래와 같은 대상 그룹을 가질 수 있다.
  - EC2 인스턴스
  - IP 주소: 반드시 Private IP

![43-gwlb-target-group.png](images%2F43-gwlb-target-group.png)

#### Cross-Zone Load Balancing

![44-cross-zone-load-balancing.png](images%2F44-cross-zone-load-balancing.png)

- 두 개의 가용 지역이 `az1`과 `az2`가 있으며 `az1`에는 두 개의 인스턴스가 실행 중이며 `az2`에는 8개의 인스턴스가 실행되고 있다.
- 만약 교차 구역 부하 분산이 활성화되어 있다면 `az1`과 `az2`의 모든 인스턴스에는 동일하게 10의 트래픽이 전달된다.
- 만약 교차 구역 부하 분산이 활성화되어 있지 않다면 `az1`의 두 개의 인스턴스에는 25의 트래픽이 전달되고 `az2`의 8개의 인스턴스에는 6.25의 트래픽이 전달된다.

- **Classic Load Balancer**
  - 기본적으로 비활성화되어 있다.
  - 활성화된 경우 AZ간 데이터 전송에는 비용이 발생하지 않는다.
- **Application Load Balancer**
  - 항상 실행되어 있으며 비활성화할 수 없다.
  - AZ간 데이터 전송에는 비용이 발생하지 않는다.
- **Network Load Balancer**
  - 기본적으로 비활성화되어 있다.
  - 활성화하는 경우 AZ간 데이터 전송에는 비용이 발생한다.
- **Gateway Load Balancer**
  - 기본적으로 비활성화되어 있다.
  - 활성화하는 경우 AZ간 데이터 전송에는 비용이 발생한다.

#### Sticky Session (Session Affinity)

- 동일한 클라이언트가 로드 밸런서 뒤에서 항상 동일한 인스턴스로 리디렉션되도록 "Sticky Session"을 활성화할 수 있다.
- 이 기능은 CLB와 ALB에 적합하다.
- 끈적임에 사용되는 쿠키에는 사용자가 제어하는 유효기간이 있다.
- 사용자가 세션 데이터를 잃어버리지 않도록 제어하고 싶은 경우에 사용된다.
- 활성화하면 백엔드 EC2 인스턴스에 대한 부하 불균형이 발생할 수 있다.

![45-sticky-session-session-affinity.png](images%2F45-sticky-session-session-affinity.png)

#### Request Routing Algorithms - Least Outstanding Requests

- 요청을 수신하게 되년 다음 인스턴스는 보류 중이거나 완료되지 않은 요청 수가 가장 적은 인스턴스다.
- ALB 및 CLB(HTTP/HTTPS)와 함께 작동한다.

![46-least-outstanding-request.png](images%2F46-least-outstanding-request.png)

#### Request Routing Algorithms - Round Robin

- 대상 그룹에서 대상을 동일하게 선택한다.
  - 각 대상에게 순차적으로 요청을 전달한다. 
- ALB 및 TCP(CLB)와 함께 작동한다.

![47-round-robin.png](images%2F47-round-robin.png)

#### Request Routing Algorithms - Flow Hash

- 프로토콜, 소스/대상 IP 주소, 소스 대상 포트 및 TCP 시퀀스 번호를 기반으로 대상을 선택한다.
- 각 TCP/UDP 연결은 연결 수명 동안 단일 대상으로 라우팅된다.
- Network Load Balancer와 함께 작동한다.

![48-flow-hash.png](images%2F48-flow-hash.png)

---

### API Gateway

![49-api-gateway-overview.png](images%2F49-api-gateway-overview.png)

- Lambda, HTTP 및 AWS 서비스를 API로 노출할 수 있도록 지원한다.
- API 버전 관리, 원한 부여, 트래픽 관리(API 키, 쓰로틀링), 대규모, 서버리스, 요청/응답 변환, OpenAPI 사양, CORS를 지원한다.
- **29초의 타임아웃이 있다.**
  - Lambda Function의 최대 작동 시간은 15분이므로 Lambda Function의 작업이 진행 중일 때, 클라이언트에게 Timeout 오류를 전달할 수 있다.
- **최대 페이로드의 크기는 10MB다.**
  - 큰 용량의 파일을 전송하고자할 때는 API 게이트웨이가 적합하지 않다.

#### Deployment Stage

- API 변경 사항은 "Stage" (원하는 만큼)에 배포된다.
- 단계 (dev, test, prod)에 대해 원하는 이름을 사용할 수 있다.
- 배포 기록이 저장되기 대문에 배포가 잘못된 경우 이전 단계로 돌아갈 수 있다.

![50-api-gateway-deployment-stage.png](images%2F50-api-gateway-deployment-stage.png)

#### Integration

- **HTTP**
  - 백엔드에서 HTTP 엔드포인트를 노출한다.
  - 예를 들어, On-Premise의 HTTP API를 노출하거나, ALB를 노출할 수 있다.
  - 속도 제한, 캐싱, 사용자 인증, API 키 추가 등의 기능을 제공받을 수 있다.
- **Lambda Function**
  - Lambda Function을 호출할 수 있다.
  - AWS Lambda에서 지원하는 REST API를 손쉽게 호출할 수 있는 방법이다.
- **AWS Service**
  - API 게이트웨이를 통해 AWS API를 노출할 수 있다.
  - 예를 들어, AWS Step Function 워크플로를 실행하거나, SQS에 메시지를 게시할 수 있다.
  - 인증을 추가하고, 공개적으로 배포하고, 속도 제어를 추가할 수 있다.

#### Solution Architecture

![51-api-gateway-s3.png](images%2F51-api-gateway-s3.png)

- API 게이트웨이는 10MB 페이로드 크기 제한을 받는다.
- 사용자가 API 게이트웨이를 통해서 사설 S3 버킷에 파일을 업로드할 수 있다. 
  - 하지만 파일의 크기가 10이 넘어가는 경우 파일을 업로드할 수 없다.

![52-api-gateway-s3.png](images%2F52-api-gateway-s3.png)

- 개선된 아키텍처다.
- 사용자가 10MB 페이로드가 넘는 크기의 요청을 API 게이트웨이로 전달한다.
- Lambda Function은 파일을 업로드할 수 있는 미리 서명된 URL을 클라이언트 애플리케이션에 전달한다.
- 클라이언트 애플리케이션은 파일을 업로드할 때는 미리 서명된 URL을 사용하기 때문에 페이로드 크기 제한을 받지 않는다.

#### 엔드포인트 유형

- **Edge-Optimized (기본값)**: 글로벌 클라이언트용이다.
  - 요청은 CloudFront Edge 로케이션을 통해 라우팅되어 지연 시간을 향상시킨다.
  - API 게이트웨이는 여전히 한 리전에서만 작동한다.
- **Regional**
  - 동일한 리전 내의 클라이언트의 경우에 사용된다.
  - CloudFront와 수동으로 결합 가능하다. (캐싱 전략 및 배포 제어 강화)
- **Private**
  - 인터페이스 VPC 엔드포인트(ENI)을 사용하여 VPC에서만 액세스할 수 있다.
  - 리소스 정책을 사용하여 액세스를 정의한다.

#### Response Caching

- 캐싱을 통해 백엔드에 대한 호출 수를 감소시킬 수 있다.
- 기본 TTL(라이브 시간)은 300초(최소: 0s, 최대: 3600s)다.
- 캐시는 단계별로 정의된다.
- 메서드별로 캐시 설정을 재정의할 수 있다.
- 클라이언트는 헤더에 `Cache-Control:max-age=0`을 추가하여 캐시를 무효화할 수 있다.
- 전체 캐시를 즉시 플러시(무효화)할 수 있다.
- 캐시 암호화 옵션을 제공한다.
- 0.5GB에서 237GB 사이의 캐시 용량을 제공한다.

![53-response-caching.png](images%2F53-response-caching.png)

#### Error

- 4xx는 클라이언트 오류를 의미한다.
  - 400: Bad Request
  - 403: Access Denied, WAF 필터링
  - 429: Quota exceeded, 쓰로틀링

- 5xx는 서버측 오류를 의미한다.
  - 502: Bad Gateway Exception, 일반적으로 Lambda 프록시 통합 백엔드에서 반환되는 호환되지 않은 출력의 경우와 때때로 과중한 부하로 인해 주문이 맞지 않는 호출의 경우에 발생한다.
  - 503: Service Unavailable Exception
  - 504: Integration Failure - Endpoint 요청 시간 초과 예외 API 게이트웨이 요청 최대 29초 후 시간 초과가 발생한다.

#### Security

- SSL 인증서를 로드하고 Route 53을 사용하여 CNAME을 정의할 수 있다.
- 리소스 정책을 정의할 수 있다. (S3 버킷 정책과 유사)
  - API에 액세스할 수 있는 사용자를 제어한다.
  - AWS 계정, IP 또는 CIDR 블록, VPC 또는 VPC 엔드포인트
- API 수준에서 API 게이트웨이에 대한 IAM 실행 역할을 정의할 수 있다.
  - Lambda Function, AWS 서비스 등을 호출
- CORS (Cross-origin resource sharing)
  - 브라우저 기반 보안을 제공한다.
  - API를 호출할 수 있는 도메인을 제어한다.

#### Authentication

- **IAM based access (AWS_IAM)**
  - 인프라 내에서 액세스를 제공하는 데 적합하다.
  - SigV4를 통해 헤더에 IAM 자격 증명을 전달한다.
- **Lambda Authorizer (formerly Custom Authorizer)**
  - Lambda를 사용하여 사용자 정의 OAuth / SAML / 타사 인증을 확인한다.
- **Cognito User Pools**
  - 클라이언트가 Cognito로 인증한다.
  - 클라이언트가 토큰을 API 게이트웨이로 전달한다.
  - API 게이트웨이가 토큰을 확인하는 방법을 즉시 알 수 있다.

![54-api-gateway-authentication.png](images%2F54-api-gateway-authentication.png)

#### Logging, Monitoring, Tracing

- **CloudWatch Logs**
  - Stage 레벨에서 CloudWatch 로깅을 사용한다. (로그 레벨 - ERROR, INFO)
  - 전체 요청/응답 데이터를 기록할 수 있다.
  - API 게이트웨이 액세스 로그를 전송한다. (사용자 지정 가능)
  - Kinesis Data Firehose (CW Logs 대안)에 로그를 직접 전송할 수 있다.
- **CloudWatch 메트릭**
  - 메트릭은 단계별, 세부 메트릭을 활성화할 수 있는 가능성이 있다.
  - 통합 지연 시간, 지연 시간, 캐시 히트 횟수, 캐시 미스 횟수를 제공한다.
- **X-Ray**
  - API 게이트웨이에서 요청에 대한 추가 정보를 가져오도록 추적을 사용한다.
  - X-Ray API 게이트웨이 + AWS Lambda를 통해 전체적인 그림을 확인할 수 있다.

#### Usage Plans & API Keys

- API를 고객에게 제공하고 비용을 청구하는 경우에 사용될 수 있다.
- **Usage Plan**
  - 배포된 하나 이상의 API 단계 및 방법에 대해 액세스할 수 있는 사용자를 정의한다.
  - 배포된 API 단계와 메서드에 누가 액세스할 수 있고 얼마나 빨리 액세스할 수 있는지 정의한다.
  - API 키를 사용하여 API 클라이언트를 식별하고 액세스를 측정한다.
- **API Key**
  - 고객에게 배포할 영숫자 문자열 값이다.
  - "Usage Plan"과 함께 사용하여 액세스를 제어할 수 있다.
  - API 키에 조절 제한이 적용된다.
  - 할당량 제한은 전체 최대 요청 수다.
- **429 Too Many Request**
  - 한 지역의 모든 API에 걸쳐 계정 수준에서 조절한다.
  - 클라이언트는 재시도 메커니즘을 구현해야 한다.

#### WebSocket API

- WebSocket은 아래와 같은 특성을 가진다.
  - 사용자의 브라우저와 서버 간의 양방향 대화형 통신을 제공한다.
  - 서버가 클라이언트에게 정보를 푸시할 수 있다.
  - 이를 통해 상태에 맞는 애플리케이션 사용 사례를 지원할 수 있다.
- WebSocket API는 채팅 애플리케이션, 협업 플랫폼, 멀티플레이어 게임, 금융거래 플랫폼 등 실시간 애플리케이션에서 자주 사용된다.
- AWS 서비스 (Lambda, DynamoDB) 또는 HTTP 엔드포인트에서 작동한다.

![55-api-gateway-websocket-api.png](images%2F55-api-gateway-websocket-api.png)

- 클라이언트는 API에 있는 웹사이트를 통한 영구 연결(Persistent connection)을 통해서 우리의 API 게이트웨이에 연결된다.
- API 게이트웨이는 `onConnect`라는 Lambda Function을 호출하고 DynamoDB 테이블과 연결을 지속한다.
- API 게이트웨이는 `sendMessage`라는 Lambda Function을 호출하고 DynamoDB에 미시지를 보내고 지속한다.
- API 게이트웨이는 연결이 끊어질 때마다 `onDisconnect`를 호출한다. 

#### @connections

![56-client-messageing-connections.png](images%2F56-client-messageing-connections.png)

- `@connections`라는 WebSocket URL을 사용한다.
- 이 URL을 통해 connectionId를 획득하고 DynamoDB 테이블에 데이터를 남길 수 있다.
- 각각의 연결에 대해 Connection URL이 콜백으로 나온다.
- 해당 URL(`wss//abcdef/execute-api.us-west-1.amazonaws.com/dev/@connections/connectionId`)를 호출하면 Lambda Function이 API 게이트웨이에 URL 콜백 메시지를 포스팅한다.

- 한 가지 방법으로는 클라이언트에서 API 게이트웨이로 가서 람다 함수로 요청이 이동한다.
- `@connections`을 이용하여 이 특별한 콜백 URL에 데이터를 게시하고 클라이언트에게 데이터를 다시 보낼 수 있다.

#### Private API

- VPC 인터페이스 엔드포인트를 사용하여 VPC에서만 액세스할 수 있도록 구성할 수 있다.
- 각 VPC 인터페이스 엔드포인트를 사용하여 여러 개의 Private API에 액세스할 수 있다.
- **API Gateway Resource Policy**
  - AWS 계정 전체를 포함하여 선택한 VPC 및 VPC 엔드포인트의 API 액세스를 허용하거나 거부할 수 있다.
  - `aws:SourceVpc`와 `aws:SourceVpce`를 사용할 수 있다.

![57-api-gateway-private-api.png](images%2F57-api-gateway-private-api.png)

- Private API 게이트웨이가 있고 Private 서브넷에서 연결을 원한다.
- 서브넷에 인터페이스 엔드포인트를 생성하면 EC2 인스턴스는 API 게이트웨이로 접근할 수 있다.
- VPC 엔드포인트 정책을 이용해 인터페이스 엔드포인트로의 접근을 제어할 수 있는 보안 레벨이 있다.

---

### AWS AppSync

- AppSync는 GraphQL을 사용하는 관리형 서비스다.
- GraphQL을 사용하면 애플리케이션이 필요한 데이터를 정확하고 쉽게 얻을 수 있다.
- 여기에는 하나 이상의 소스로부터 데이터를 결합하는 것이 포함된다.
  - NoSQL 데이터 저장소, 관계형 데이터베이스, HTTP API 등..
  - DynamoDB, Aurora, Elasticsearch 등과 통합된다.
  - AWS Lambda를 사용하는 사용자 지정 소스와 통합될 수 있다.
- WebSocket 또는 WebSocket의 MQTT를 통해 실시간으로 데이터를 검색할 수 있다.
- 모바일 앱의 경우 로컬 데이터 액세스 및 데이터 동기화를 할 수 있다.
- 이러한 모든 기능은 하나의 GraphQL 스키마를 업로드하는 것으로 시작된다.

#### Diagram

![58-appsync-diagram.png](images%2F58-appsync-diagram.png)

- 클라이언트(모바일 앱/웹, 실시간 대시보드, 오프라인 동기화)는 AppSync에 액세스한다.
- AppSync 안에는 GraphQL 스키마와 Resolver가 있고 Resolver는 데이터의 다양한 소스에서 데이터를 얻는 방법을 알고 있다.
  - DynamoDB, Aurora, ElasticSearch, Lambda, HTTP와 연결될 수 있다.
- AppSync 모니터링은 CloudWatch 지표와 로그로 할 수 있다.

#### Cognito Integration

- 소속 그룹에 따라 Cognito 사용자에게 권한 부여를 수행한다.
- GraphQL 스키마에서 Cognito 그룹에 대한 보안을 지정할 수 있다.
- 아래의 예시에서는 블로거와 독자 Cognito 그룹을 정의하고 블로거에게만 글을 작성할 수 있는 권한을 부여하고 있다. 

```
type Query {
    posts: [Post!]!
    @aws_auth(cognito_groups: ["Bloggers", "Readers"])
}

Type Mutation {
    addPost(id: ID!, title: String!): Post!
    @aws_auth(cognito_groups: ["Bloggers"])
}
```

![59-appsync-cognito-integration.png](images%2F59-appsync-cognito-integration.png)

- 아키텍처 관점에서 클라이언트는 Cognito로 인증을 하고 JWT 토큰을 얻고 AppSync에 JWT를 전달한다.
- AppSync는 JWT를 확인하고 Cognito로 승인되었는지 확인한다.
- 승인되었다면 GraphQL 스키마 안에 작성한 resolvers에서 확인되는 것처럼 사용자가 속한 그룹을 보게 된다.

---

### Route 53

#### 레코드 유형

- A: 호스트 이름을 IPv4에 매핑한다.
- AAAA: 호스트 이름을 IPv6에 매핑한다.
- CNAME: 호스트 이름을 다른 호스트 이름에 매핑한다.
  - 대상은 A 또는 AAAA 레코드가 있어야 하는 도메인 이름이다.
  - DNS 네임스페이스(Zone Apex)의 상위 노드에 대한 CNAME 레코드를 생성할 수 없다.
  - 예를 들어, `example.com`에 대해서는 만들 수 없지만 `www.example.com`에 대해서는 만들 수 있다.
- NS: 호스트 영역의 이름 서버
  - 도메인에 대한 트래픽 라우팅 방법을 제어한다.

#### A Record

![60-diagram-a-record.png](images%2F60-diagram-a-record.png)

- 클라이언트는 공용 IP를 가진 EC2 인스턴스에 접근하려고 한다.
- 처음으로 Route 53에 쿼리를 날려 `example.com`에 대한 IP를 확보한다.
- 클라이언트는 확보한 IP를 기반으로 EC2 인스턴스에 접속할 수 있다.

#### CNAME vs Alias

- AWS 리소스 (Load Balancer, CloudFront)는 AWS 호스트 이름을 노출한다.
  - 예를 들어, 로드벨런서는 `lb1-1234.us-east-2.elb.amazonaws.com`와 같은 호스트 이름을 노출한다.
  - 하지만 사용자들은 `myapp.mydomain.com`과 같은 간결한 도메인으로 접속하기를 원한다.
- CNAME
  - 호스트 이름을 다른 호스트 이름으로 지정한다. (`app.mydomain.com` -> `blabla.anything.com`)
  - **루트 도메인이 아닌 경우에만 사용할 수 있다.** (`aka.something.mydomain.com`)
- Alias
  - 호스트 이름을 AWS 리소스로 지정한다. (`app.mydomain.com` -> `blabla.amazonaws.com`)
  - 루트 도메인과 루트 도메인이 아닌 모든 경우에 작동한다. (`aka.mydomain.com`)
  - 무료로 사용할 수 있다.
  - 상태 점검(Health Check)를 지원한다.

#### Alias Record Target

- Alias 레코드의 대상이 될 수 있는 여러 AWS 서비스가 있다.
  - Elastic Load Balancer
  - CloudFront Distribution
  - API Gateway
  - Elastic Beanstalk environment
  - S3 Website
  - VPC Interface Endpoint
  - Global Accelerator accelerator
  - 동일한 호스트 존에 있는 Route 53 Record
- **EC2 DNS 이름에 대한 Alias 레코드를 설정할 수 없다.**

#### Record TTL

- 높은 TTL(예: 24시간)
  - Route 53의 트래픽을 감소시킨다. (낮은 비용)
  - 오래된 레코드일 가능성이 높다.
- 낮은 TTL(예: 60초)
  - Route 53에 더 많은 트래픽을 발생시킨다. (높은 비용)
  - 오래된 레코드일 가능성이 낮다.
  - 레코드를 변경할 때 유용하다.
- **Alias 레코드를 제외하고 각 DNS 레코드에는 TTL이 필수다.**

![61-record-ttl.png](images%2F61-record-ttl.png)

- 클라이언트는 웹 서버와 통신하기를 원한다.
- DNS 요청을 통해서 Route 53으로부터 `myapp.example.com`에 대한 IP 주소와 TTL을 응답받는다.
- TTL 시간에 따라 클라이언트에게 캐싱되고 TTL이 만료되면 레코드가 갱신된다.
- TTL이 유효한 시간동안에는 Route 53에 대한 DNS 쿼리를 하지 않고 클라이언트는 바로 웹 서버에 요청을 보낸다.

#### Routing Policy - Simple

![62-routing-policy-simple.png](images%2F62-routing-policy-simple.png)

- 일반적으로 트래픽을 단일 리소스로 라우팅한다.
- 상태 점검과 연결할 수 없다.
- 동일한 레코드에 여러 값을 지정할 수 있다.
- **여러 값이 반환되는 경우 클라이언트에서 임의의 값을 선택한다.**

#### Routing Policy - Weighted

![63-routing-policy-weighted.png](images%2F63-routing-policy-weighted.png)

- 각 특정 리소스로 전달되는 요청 비율을 제어할 수 있다.
- 상태 점검과 연결할 수 있다.
- 지역간 로드 밸런싱, 새로운 애플리케이션 버전 테스트 등의 작업에 사용된다.

#### Routing Policy - Latency-based

![64-routing-policy-latency-based.png](images%2F64-routing-policy-latency-based.png)

- 사용자와 가까운 지연 시간이 가장 적은 리소스로 리디렉션한다.
- 사용자의 대기 시간이 우선시되는 경우 매우 유용하게 사용된다.
- **지연 시간은 사용자와 AWS 지역 간의 트래픽을 기반**으로 한다.
- 예시 이미지를 기준으로 독일의 사용자는 미국으로 연결될 수 있다. (가장 낮은 지연 시간인 경우)
- 상태 점검과 연결할 수 있으며, Failover를 제공한다.

#### Routing Policy - Failover (Active-Passive)

![65-routing-policy-failover-active-passive.png](images%2F65-routing-policy-failover-active-passive.png)

- Primary EC2 인스턴스가 있고, Secondary 장애 조치용 EC2 인스턴스가 있다.
- 만약 Primary EC2 인스턴스에 문제가 생겨서 상태 점검에 실패하는 경우 Route 53은 Secondary EC2 인스턴스에 대한 레코드를 반환한다.

#### Routing Policy - Geolocation

![66-routing-policy-geolocation.png](images%2F66-routing-policy-geolocation.png)

- **지연 시간 기반(Latency-based)** 과는 다르다.
- 사용자 위치를 기반으로 요청을 라우팅한다.
- 대륙, 국가 또는 미국 주별로 위치를 지정하고, 중복되는 경우 가장 정확한 위치를 선택한다.
- 위치에 일치하지 않는 경우에 사용할 수 있는 "Default" 레코드를 만들어야 한다.
- 웹 사이트 현지화, 콘텐츠 배포 제한, 로드 밸런싱 등에 사용된다.
- 상태 점검과 연결할 수 있다.

![66-routing-policy-geolocation.png](images%2F66-routing-policy-geolocation.png)

#### Routing Policy - Geoproximity

- 사용자 및 리소스의 지리적 위치에 따라 리소스로 트래픽을 라우팅한다.
- 정의된 편향(bias)에 따라 더 많은 트래픽을 리소스로 전환하는 기능이 있다.
- 지리적 영역의 크기를 변경하려면 다음과 같이 편향(bias) 값을 지정한다.
  - 1 ~ 99: 리소스에 대한 트래픽이 증가한다.
  - -1 ~ -99: 리소스에 대한 트래픽이 감소한다.
- 리소스는 다음과 같다.
  - AWS 리소스 (AWS 리전 지정)
  - AWS가 아닌 리소스 (위도 및 경도 지정)
- 이 기능을 사용하려면 Route 53 트래픽 흐름(Traffic Flow)을 사용해야 한다.

![67-routing-policy-geoproximity.png](images%2F67-routing-policy-geoproximity.png)

- `us-west-1`과 `us-east-1` 두 개의 리전이 있고 동일한 편향(bias)값을 가지고 있다.
- 사용자들의 요청은 한쪽으로 편향되지 않고 정확하게 반으로 나뉘어진다.

![68-routing-policy-geoproximity.png](images%2F68-routing-policy-geoproximity.png)

- 이번에는 `us-east-1`의 편향(bias)값이 더 높게 설정되어 있다.
- 더 높게 설정된만큼 더 많은 사용자들의 요청을 처리하게 된다.

#### Traffic Flow

![69-traffic-flow.png](images%2F69-traffic-flow.png)

- 대규모 및 복잡한 구성에서 레코드 생성 및 유지 관리 프로세스를 간소화한다.
- 복잡한 라우팅 의사 결정 트리를 관리하는 시각적 편집기다.
- 구성을 트래픽 흐름 정책으로 저장할 수 있다.
  - 다른 Route 53 Hosted Zone에 적용 가능하다.
  - 버전 관리를 지원한다.

#### Routing Policy - Multi-Value

- 트래픽을 여러 리소스로 라우팅할 때 사용한다.
- Route 53이 여러 값/리소스를 반환한다.
- Health Checks(건강한 리소스에 대한 값만 반환)와 연결이 가능하다.
- Multi-Value 쿼리마다 최대 8개의 건강한 레코드가 반환된다.
- **Multi-Value는 ELB를 대체할 수 없다.**

![70-routing-policy-multi-value.png](images%2F70-routing-policy-multi-value.png)

#### Routing Policy - IP-based Routing

- 라우팅은 클라이언트의 IP 주소를 기반으로 한다.
- 클라이언트 및 해당 엔드포인트/위치(사용자-IP-엔드포인트 매핑)에 대한 CIDR 목록을 제공한다.
- 성능 최적화, 네트워크 비용 절감 등의 이유로 사용된다.
- 예를 들어, 최종 사용자를 특정 ISP에서 특정 엔드포인트로 라우팅할 수 있다.

![71-routing-policy-ip-based-routing.png](images%2F71-routing-policy-ip-based-routing.png)

- Route 53에서 두 개의 다른 CIDR 블록이 있는 두 장소를 정의한다.
  - 하나의 IP는 203으로 시작하고 다른 하나는 200으로 시작한다.
  - IP 범위도 정의되어 있다.
- 정의된 장소들을 특정 레코드에 링크한다.
  - 사용자의 IP가 location-1에 해당되는 경우 `1.2.3.4` EC2 인스턴스로 라우팅한다.
  - 사용자의 IP가 location-2에 해당하는 경우 `5.6.7.8` EC2 인스턴스로 라우팅한다.

#### Hosted Zone

- 도메인 및 해당 하위 도메인으로 트래픽을 라우팅하는 방법을 정의하는 레코드 컨테이너다.
- Public Hosted Zone: 인터넷(공용 도메인 이름) `application1.mypublicdomain.com`에서 트래픽을 라우팅하는 방법을 지정하는 레코드가 들어있다.
- Private Hosted Zone: 하나 이상의 VPC(개인 도메인 이름) `application1.company.internal`내에서 트래픽을 라우팅하는 방법을 지정하는 레코드가 포함된다.

![72-public-private-hosted-zone.png](images%2F72-public-private-hosted-zone.png)

- Public Hosted Zone의 응답값은 EC2 인스턴스의 공용 IP, ALB, CloudFront, S3 버킷의 IPㅣㅇㄹ 수 있다.
- Private Hosted Zone은 VPC 내에서 사용된다.
  - EC2 인스턴스의 Private IP나 DB 인스턴스의 Private IP에 링크하는 데에 도움이된다.
  - DNS가 있는 Private Hosted Zone을 가지고 있다면 VPC에서 활성화해야 한다.

#### Good to Know

- 내부 전용 DNS (Private Hosted Zone)의 경우 VPC를 사용하도록 `enableDnsHostnames`와 `enableDnsSupport` 옵션을 활성화해야 한다.
- **DNS Security Extensions (DNSSEC)**
  - DNS 트래픽을 보호하기 위한 프로토콜, DNS 데이터 무결성 및 원본 확인을 제공한다.
  - MITM(Man in the Middle) 공격으로부터 보호한다.
  - Route 53은 도메인 등록을 위한 DNSSEC와 DNSSEC 서명을 모두 지원한다.
  - "Public Hosted Zone"에서만 작동한다.
- **Route 53 with 3rd Registrar**
  - 도메인을 AWS에서 구입하여 DNS 공급자로 Route 53을 사용할 수 있다.
  - 제3자 등록 기관의 NS 레코드를 업데이트해야 한다.

#### Health Checks

- HTTP 상태 검사는 공용 리소스에 대해서만 수행된다.
- 상태 검사 -> 자동화된 DNS Failover
  - 엔트포인트를 모니터링하는 상태 검사 (애플리케이션, 서버, 기타 AWS 리소스)
  - 기타 상태 점검 (계산된 상태 점검)을 모니터링하는 상태 점검
  - CloudWatch 알람을 모니터링하는 상태 점검(전면 제어)
    - 예를 들어, DynamoDB의 스로틀, RDS 알람, 사용자 지정 메트릭 등 (개인 리소스에 유용)
- 상태 점검이 CW 메트릭과 통합된다.

![73-route53-health-checks.png](images%2F73-route53-health-checks.png)

- 여러 상태 검사 결과를 하나의 상태 검사로 결합할 수 있다.
- OR, AND, NOT을 사용할 수 있다.
- 최대 256개의 자식 상태 검사를 모니터링할 수 있다.
- 부모가 상태 검사를 통과하려면 몇 개의 상태 검사를 통과해야 하는지 지정할 수 있다.
- 사용법: 모든 상태 점검에 실패하지 않고 웹 사이트에 대한 유지 관리를 수행한다.

![74-calculated-health-checks.png](images%2F74-calculated-health-checks.png)

#### Health Checks - Monitor as Endpoint

![75-health-checks-monitor-endpoint.png](images%2F75-health-checks-monitor-endpoint.png)

- 약 15명의 글로벌 상태 점검자가 엔드포인트 상태를 확인한다.
- 엔드포인트가 2xx 및 3xx 상태 코드로 응답하는 경우에만 상태 검사를 통과한다.
- 상태 검사는 응답의 처음 5120 바이트의 텍스트를 기준으로 통과/실패하도록 설정할 수 있다.

#### Health Checks - Private Hosted Zone

![76-health-checks-private-hosted-zone.png](images%2F76-health-checks-private-hosted-zone.png)

- Route 53 경로 건강 검진자가 VPC 외부에 있다.
- 건강 검진자는 프라이빗 엔드포인트(프라이빗 VPC 또는 사내 리소스)에 액세스할 수 없다.
- CloudWatch 메트릭을 생성하고 CloudWatch 알람을 연결한 다음 알람 자체를 확인하는 상태 검사를 생성할 수 있다.

#### RDS Multi-Region Failover

![77-rds-multi-region-failover.png](images%2F77-rds-multi-region-failover.png)

- `us-east-1`에 메인 RDS 인스턴스가 있고 `us-west-2`에 읽기 전용 복제본이 있으며 비동기로 복제되고 있다.
- 상태 검사를 위한 하나의 옵션은 Ec2 인스턴스를 설치하여 데이터베이스 상태를 모니터링하고 `/health-db` 경로로 노출하는 방법이다.
- 다른 하나의 옵션은 CW 알람을 정의하고 CloudWatch 알람을 상태 검사에 링크한다. 
  - 상태 검사가 해제되면 CW 알람에도 링크할 수 있다.
- SNS 토픽이나 CloudWatch Event와 연결될 수 있다.
- 만약 문제가 있는 경우 람다 함수를 호출하여 Route 53 레코드를 변경하여 읽기전용 복제본을 메인 DB로 승격할 수 있다.

#### Route 53 - Hybrid DNS

- 기본적으로 Route 53 Resolver는 다음에 대한 DNS 쿼리에 자동으로 응답한다.
  - EC2 인스턴스의 로컬 도메인 이름
  - 개인 호스트 영역의 레코드
  - 공용 네임 서버의 레코드
- Hybrid DNS: VPC(Route 53 Resolver)와 네트워크(기타 DNS Resolver)간 DNS 쿼리를 해결한다.
- 네트워크는 다음과 같다.
  - VPC 자체 / 피어 VPC
  - 사내 네트워크(Direct Connect 또는 AWS VPN을 통한 연결)

![78-route53-hybrid-dns.png](images%2F78-route53-hybrid-dns.png)

#### Route 53 - Resolver Endpoint

- Inbound Endpoint
  - 네트워크의 DNS Resolver가 DNS 쿼리를 Route 53 Resolver로 전달할 수 있다.
  - DNS Resolver가 Route 53 Private Hosted Zone에서 AWS 리소스(예: EC2 인스턴스) 및 레코드에 대한 도메인 이름을 확인할 수 있다.
- Outbound Endpoint
  - Route 53 Resolver가 DNS 쿼리를 DNS Resolver로 조건부 전달한다.
  - Resolver Rules를 사용하여 DNS 쿼리를 DNS Resolver로 전달한다.
- 동일한 AWS 영역에 있는 하나 이상의 VPC와 연결된다.
- 고가용성을 위해 두 개의 AZ로 생성된다.
- 각 엔드포인트는 IP 주소당 초당 10,000개의 쿼리를 지원한다.

#### Resolver Inbound Endpoint

![79-resolver-inbound-endpoint.png](images%2F79-resolver-inbound-endpoint.png)

- AWS의 `us-east-1` 리전에는 VPC내에 EC2 인스턴스가 있고 Private Hosted Zone이 있다.
  - EC2 인스턴스에는 CNAME값인 `app.aws.private`가 매핑되어 있다.
- On-Premise 데이터 센터에는 서버가 실행되고 있다.
- On-Premise 데이터 센터와 AWS 클라우드는 VPN 연결 또는 Direct Connect를 통해 연결되어 있다.
- On-Premise 데이터 센터에는 DNS Resolver가 있고 AWS 클라우드에는 "Resolver Inbound Endpoint"가 있다.
  - Resolver Inbound Endpoint는 고가용성을 위해서 두 개의 ENI를 가지고 있다.
  - 두 개의 ENI는 각각 개인 IP 주소와 연결되어 있다.
- DNS Resolver는 On-Premise내에서 발생하게 되는 도메인 이름을 ENI 두 개의 IP로 전송하도록 설정한다.
- On-Premise 데이터 센터의 서버가 DNS Resolver로 `app.aws.private`에 대한 IP 주소를 질의한다.
  - DNS Resolver는 Resolver Inbound Endpoint에게 `app.aws.private`에 대한 IP 주소를 질의한다.
  - Resolver Inbound Endpoint는 Route 53 Resolver로 질의를 전달하고 "Private Hosted Zone"을 검색하게 된다.

#### Resolver Outbound Endpoint

![80-resolver-outbound-endpoint.png](images%2F80-resolver-outbound-endpoint.png)

- 이번에는 On-Premise 데이터 센터의 DNS 이름을 확인한다.
- EC2 인스턴스는 Route 53 Resolver에게 `web.onprem.private`에 대한 IP 주소를 질의한다.
- Route 53 Resolver는 Resolver Outbound Endpoint에게 질의한다.
  - Resolver Outbound Endpoint에는 `onprem.private` 도메인에 대한 포워딩 규칙이 정의되어 있다.
- Resolver Outbound Endpoint는 포워딩 규칙에 따라 On-Premise 데이터 센터의 DNS Resolver에게 `web.onprem.private`에 대한 IP 주소를 질의한다.

#### Resolver Rules

- 네트워크의 DNS Resolver로 전달되는 DNS 쿼리를 제어한다.
- **Conditional Forwarding Rules (Forwarding Rules)**
  - 지정된 도메인 및 해당 모든 하위 도메인에 대한 DNS 쿼리를 대상 IP 주소로 전달한다.
- **System Rules**
  - 전달 규칙에 정의된 동작을 선택적으로 재정의한다. (예: 하위 도메인 `acme.example.com`에 대한 DNS 쿼리를 전달하지 않음)
- **Auto-defined System Rules**
  - 선택한 도메인에 대한 DNS 쿼리가 해결되는 방법을 정의한다. (예: AWS 내부 도메인 이름, 전용 호스팅 영역)
- 여러 규칙이 일치하면 Route 53 Resolver는 가장 구체적으로 일치하는 항목을 선택한다.
- **AWS RAM을 사용하여 계정 간에 Resolver Rule 공유 가능**
  - 하나의 계정에서 중앙 집중식으로 관리
  - 규칙에 정의된 대상 IP로 여러 VPC에서 DNS 쿼리 전송

![81-resolver-rules.png](images%2F81-resolver-rules.png)

---

### AWS Global Accelerator

- AWS 내부 네트워크를 활용하여 애플리케이션으로 라우팅한다.
- 사용자의 애플리케이션에 2개의 Anycast IP가 생성된다.
- Anycast IP는 트래픽을 엣지 로게이션으로 직접 전송한다.
- 엣지 로케이션에서 트래픽을 사용자의 애플리케이션으로 전송한다.

![82-global-accelerator.png](images%2F82-global-accelerator.png)

- 미귝, 유럽, 호주의 사용자는 엣지 로케이션까지 공용 인터넷을 사용해서 통신한다.
- 이후 엣지 로케이션에서 인도의 ALB까지의 통신은 AWS 내부 네트워크를 통해서 이루어진다.

- Elastic IP, EC2 인스턴스, ALB, NLB, Public IP, Private IP와 작동한다.
- NLB 및 EIP 엔드포인트를 제외한 클라이언트 IP 주소 보존을 지원한다.
- **일관된 성능**
  - 최저 지연 시간 및 빠른 지역별 Failover을 위한 지능적 라우팅을 제공한다.
  - IP가 변경되지 않기 때문에 클라이언트 캐시 문제가 없다.
  - 내부 AWS 네트워크를 사용한다.
- **상태 확인**
  - Global Accelerator가 애플리케이션의 상태 확인을 수행한다.
  - 애플리케이션을 글로벌하게 만드는 데 도움이 된다. (건강하지 못한 경우 Failover에 소요되는 시간이 1분 미만)
  - 재해 복구에 탁월하다.
- **보안**
  - 2개의 외부 IP만 화이트리스트로 지정할 필요가 있다.
  - AWS Shield 덕분에 DDoS 보호를 제공받을 수 있다.

#### Global Accelerator vs CloudFront

- 두 서비스 모두 AWS 글로벌 네트워크와 전 세계 엣지 로케이션을 사용한다.
- 두 서비스 모두 DDoS 보호를 위해 AWS Shield와 통합된다.
- CloudFront
  - 캐시 가능한 콘텐츠(예: 이미지 및 비디오) 모두에 대한 성능이 향상된다.
  - 동적 콘텐츠(API Acceleration과 동적 사이트 제공 등)을 제공한다.
  - 엣지에서 콘텐츠를 제공한다.
- Global Accelerator
  - TCP 또는 UDP를 통한 광범위한 애플리케이션의 성능을 향상한다.
  - 엣지에 있는 패킷을 하나 이상의 AWS 리전에서 실행 중인 애플리케이션에 프록시하는 것이다.
  - 게임(UDP), IoT(MQTT), Voice over IP 등 HTTP가 아닌 사용 사례에 적합하다.
  - 정적 IP 주소가 필요한 HTTP 사용 사례에 적합하다.
  - 결정적이고 빠른 리전별 Failover가 필요한 HTTP 사용 사례에 적합하다.

---

### Solutions Architecting 비교

#### EC2 with Elastic IP

![83-ec2-with-elasticip.png](images%2F83-ec2-with-elasticip.png)

- 사용자가 Public EC2 인스턴스와 연결된 Elastic IP를 통해 인스턴스와 통신하고 있다.
- 장애 조치를 위해서 대기(Standby) 인스턴스가 생성되어 있다.
- 만약 장애가 발생하는 경우 Elastic IP가 대기 인스턴스를 향하도록 수정한다.
- 신속한 장애 조치를 지원하는 방법이지만 확장은 지원하지 않는다.
- 클라이언트는 변경이 이루어지는 것을 확인해서는 안된다.
- 클라이언트가 정적 공용 IP주소로만 해결해야 하는 경우에 유용하다.
- 저럼현 아키텍처다.

#### Stateless web app - scaling horizontally

![84-stateless-web-app-scaling-horizontally.png](images%2F84-stateless-web-app-scaling-horizontally.png)

- 사용자는 애플리케이션이 수평으로 확장되기를 원한다.
- Public이지만 Elastic IP가 없는 세 개의 EC2 인스턴스가 있다.
- 클라이언트는 Route53에서 DNS 쿼리를 수행하고 A 레코드를 받게 된다.
- 레코드 덕분에 EC2 인스턴스와 통신할 수 있게 된다.

![85-stateless-web-app-scaling-horizontally.png](images%2F85-stateless-web-app-scaling-horizontally.png)

- 하지만 EC2 인스턴스 중 하나가 종료되면 클라이언트는 우리의 애플리케이션에 도달할 수 없는 문제가 발생한다.
- 이러한 경우에는 DNS-based 로드 밸런싱을 사용해야 한다.
- 여러 인스턴스를 사용할 수 있는 기능이다.
- Route53 TTL은 고객이 오래된 정보를 얻을 수 있음을 암시한다.
- 클라이언트는 호스트 이름 확인 실패를 처리할 논리가 있어야 한다.
- DNS TTL로 인해 인스턴스를 추가하면 전체 트래픽이 바로 수신되지 않을 수 있다.

#### ALB & ASG

![86-alb-asg.png](images%2F86-alb-asg.png)

- Route53 DNS 쿼리가 Alias 레코드로 ALB를 가르키고 있다.
- ALB는 상태 확인을 활성화하고 다중 AZ가 활성화되어 있는 세 개의 AZ로 라우팅된다.
- Auto Scaling Group에는 총 5개의 EC2 인스턴스가 실행되고 있다.
- 확장성이 뛰어난 고전적인 아키텍처다.
- 새로운 인스턴스가 바로 서비스 중이다.
- 사용자는 서비스가 종료된 인스턴스로 트래픽을 전송하지 않는다.
- 확장 시간이 느리다. (EC2 인스턴스 시작 + 부트스트랩) <- AMI가 도움이 될 수 있다.
- ALB는 탄력적이지만 갑작스럽고 큰 수요를 처리할 수 없다. (미리 예열해야 함)
- 인스턴스가 과부화되면 몇 가지 요청이 손실될 수 있다.
- CloudWatch는 ASG에서 사용될 수 있다.
- 균등한 트래픽 분산을 위한 교차 지역 로드 밸런싱을 활성화해야 한다.
- 목표 활용률은 40% ~ 70%로 설정하는 것이 좋다.

#### ALB & ECS on EC2 (backed by ASG)

![87-alb-ecs-ec2.png](images%2F87-alb-ecs-ec2.png)

- ALB & ASG와 동일한 구조를 가진다.
- 인스턴스에서 애플리케이션을 시작하는 대신 인스턴스에서 ECS 작업을 실행하고 있다.
- 애플리케이션은 Docker 컨테이너에서 실행되어야 한다.
- ECS를 사용하고 있기 때문에 동적 포트 매핑 기능을 사용하여 EC2 인스턴스의 같은 애플리케이션에서 다중 작업을 실행할 수 있다.
- 잠재적으로 EC2 인스턴스 사용을 극대화하는 데 도움이 된다.
- ECS 서비스 자동 확장과 ASG 자동 확장을 함께 사용하는 것은 매우 복잡하다.

#### ALB & ECS on Fargate

![88-alb-ecs-fargate.png](images%2F88-alb-ecs-fargate.png)

- Fargate를 사용하여 ECS를 사용하면 Auto Scaling Group이 필요없다.
- 관리해야 하는 EC2 인스턴스 대신 Fargate가 있다.
- 작업은 AWS 네트워크에서 자동으로 시작되고 관리할 필요도 없다.
- 애플리케이션 역시 도커에서 실행되며 서비스 자동 확장이 아주 용이하다.
- EC2 인스턴스를 미리 시작할 필요가 없기 때문에 서비스를 준비하는 시간이 빨라진다.
- 갑작스럽게 요청이 급증하는 경우 ALB에 의해 여전히 제한된다.
- 서버리스 애플리케이션 계층이다.
- 관리되는 로드 밸런서를 사용한다.

#### ALB & Lambda

![89-alb-lambda.png](images%2F89-alb-lambda.png)

- ALB의 대상 그룹이 람다 함수가 될 수 있다.
- 람다의 런타임에 제한이 생긴다. 
- 람다를 통해서 원활한 확장이 가능하다.
- API 게이트웨이의 모든 기능 없이 람타 함수를 HTTP/S로 노출하는 간단한 방법이다.
- WAF(Web Application Firewall)와 결합할 수 있다.
- 하이브리드 마이크로서비스에 적합하다.
- 일부 요청에는 ECS를 사용하고 다른 요청에는 람다를 사용하도록 구축할 수 있다.

#### API Gateway & Lambda

![90-api-gateway-lambda.png](images%2F90-api-gateway-lambda.png)

- API 게이트웨이와 람다를 조합하는 경우 요청당 비용을 지불해야 한다.
- 하지만 서버없이 원활한 확장을 지원한다.
- API 게이트웨이는 초당 10,000건의 요청, 람다는 1,000 동시성 소프트 제한이 있다.
- API 게이트웨이는 인증, 속도 제한, 캐싱 등의 기능을 제공한다.
- 람다 Cold Start 시간이 일부 요청의 지연 시간을 증가시킬 수 있다.
- X-Ray와 완벽하게 통합되어 인프라 내에서 요청한 모든 것을 추적할 수 있다.

#### API Gateway & AWS Service (Proxy)

![91-api-gateway-aws-service.png](images%2F91-api-gateway-aws-service.png)

- API 게이트웨이는 AWS와 함께 프록시로 사용될 수 있다.
- 클라이언트 API 게이트웨이와 람다가 있고 요청은 SQS 뷰에 들어간다.
- 더 좋은 아키텍처는 클라이언트가 API 게이트웨이와 통신하고 API 게이트웨이가 SQS와 직접 통신하는 것이다.
- 이러한 방식이 더 낮은 지연 시간과 비용을 제공한다.
- 람다의 동시 용량을 사용하지 않으며 사용자 지정 코드가 필요없다.
- API 게이트웨이를 통해 AWS API를 안전하게 노출할 수 있다.
- SQS, SNS< Step Functions등을 통합할 수 있다.
- API 게이트웨이의 페이로드 제한은 10MB이므로 S3 프록시의 경우 문제가 발생할 수 있다.

#### API Gateway & HTTP backend

![92-api-gateway-http-backend.png](images%2F92-api-gateway-http-backend.png)

- 사용자 지정 HTTP 백엔드 위에 API 게이트웨이 기능을 사용한다.
  - 인증, 속도제어, API 키, 캐싱 등..
- 아래의 항목들과 연결할 수 있다.
  - On-Premise 서비스
  - ALB
  - 3rd party HTTP 서비스

---

### AWS Outposts

![93-aws-outposts.png](images%2F93-aws-outposts.png)

- 하이브리드 클라우드: 클라우드 인프라와 함께 사내 인프라를 유지하는 기업에서 사용한다.
- 따라서 IT 시스텀을 처리하는 두 가지 방법이 있다.
  - AWS 클라우드 (AWS 콘솔, CLI 및 AWS API 사용)
  - On-Premise 인프라
- AWS Outposts는 클라우드에서와 마찬가지로 자체 애플리케이션을 구축할 수 있는 동일한 AWS 인프라, 서비스, API 및 툴을 제공하는 "서버 랙"이다.
- AWS는 사내 인프라에 "Outposts Rack"을 설치하고 관리하며 사내 AWS 서비스를 활용할 수 있다.
- 사용자는 Outpost Rack에 대한 물리적 보안을 책임져야한다.

#### 이점
  
- 사내 시스텀에 대한 낮은 대기 시간 액세스를 제공한다.
- 로컬 데이터 처리를 제공한다.
- 데이터 레지던시르 ㄹ제공한다.
- 사내에서 클라우드로 간편하게 마이그레이션할 수 있다.
- 완벽하게 관리되는 서비스다.
- Outpost에서 작동하는 서비스 목록은 아래와 같다.
  - EC2, EBS, S3, EKS, ECS, RDS, EMR

#### S3 on AWS Outposts

- S3 API를 사용하여 AWS Outposts에 데이터를 로컬로 저장 및 검색한다.
- 사내 애플리케이션에 데이터를 가깝게 유지한다.
- AWS 리전으로의 데이터 전송을 감소시킨다.
- S3 스토리지 클래스 이름은 "S3 Outposts"다.
- SS3-S3를 사용한 기본 암호화를 제공한다.

![94-s3-aws-outposts.png](images%2F94-s3-aws-outposts.png)

- Outposts에 S3가 있고 S3 Access Point를 생성한다.
  - VPC의 EC2 인스턴스는 S3 Access Point를 통해서 Outposts의 S3에 접근할 수 있다.
- DataSync를 통해서 Outposts S3와 Amazon S3를 동기화할 수 있다.

---

### AWS WaveLength

- WaveLength Zone은 5G 네트워크 엣지에 있는 통신 공급자의 데이터 센터에 내장된 인프라 구축이다.
- AWS 서비스를 5G 네트워크 엣지에 제공한다.
- 예를 들어, EC2, EBS, VPC가 있다.
- 5G 네트워크를 통한 초저지연 애플리케이션에 적합하다.
- 트래픽이 CSP(통신 서비스 공급자) 네트워크를 이탈하지 않는다.
- 상위 AWS 영역에 대한 고대역폭 및 보안 연결을 지원한다.
- 추가 요금 또는 서비스 계약이 없다.
- Smart City, ML 지원 진단, Connected Vehicle, Interactive Live Video Streams, AR/VR, Real-time Gaming 등에 사용된다.

![95-aws-wevelength.png](images%2F95-aws-wevelength.png)

- 모바일 장치의 5G 사용자가 사용자의 WeveLength Zone에 접속할 때마다 대기 시간이 아주 낮아진다.
  - 애플리케이션이 WeveLength Zone에 배포되기 때문이다.

---

### AWS Local Zones

- AWS 컴퓨팅, 스토리지, 데이터베이스 및 기타 선택된 AWS 서비스를 최종 사용자에 가깝게 배치하여 지연 시간에 민감한 애플리케이션을 실행할 수 있다.
- VPC를 더 많은 장소로 확장한다. - AWS 리전 확장
- EC2, RDS, ECS, EBS, ElastiCache, Direct Connect와 호환 가능하다.
- 예시
  - AWS Region: N.Virginia (`us-east-1`)
  - AWS Local Zone: Boston, Chicago, Dallas, Houston, Miami 등..

![96-aws-local-zones.png](images%2F96-aws-local-zones.png)

---

### 참고한 강의

- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)