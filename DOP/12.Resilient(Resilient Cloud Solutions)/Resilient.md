# Resilient

- 이번 장에서는 **DevOps Engineer Professional (DOP)** 을 준비하며 "탄력성 있는 아키텍처를 구축하는 방법"에 대해서 알아보도록 한다.

---

### Auto Scaling

#### Dynamic Scaling Policy

- **Target Tracking Scaling**
  - 가장 간단하고 설치가 용이하다.
  - 평균 ASG CPU 사용률을 40%로 유지하도록 설정하면 ASG는 자동으로 확장/축소하여 원하는 CPU 사용률이 되도록 한다.
- **Simple / Step Scaling**
  - CloudWatch에서 직접 알람을 설정한다.
  - CloudWatch 경보가 트리거되면(예: CPU Usage > 70%) 장치 2개를 추가한다.
  - CloudWatch 경보가 트리거되면(예: CPU Usage < 30%) 장치 3개를 제거한다.
  - 사용자가 직접 얼마나 추가하고 제거할지를 결정해야 한다.
- **Scheduled Actions**
  - 알려진 사용 패턴을 기반으로 확장을 예측한다.
  - 예를 들어, 금요일 오후 5시에 큰 행사가 있는 것을 알고 있는 경우 트래픽이 몰리는 시간에 최소 용량을 확장한다.
- **Predictive Scaling**
  - 지속적으로 부하를 예측하고 사전 확장을 예약한다.
  - 부하를 보고 예측에 근거해 미리 스케일링 작업이 예정된다.
  - 머신 러닝 기반으로 가장 손쉬운 접근법이다.

![36-predictive-scaling.png](images%2F36-predictive-scaling.png)

#### 스케일링에 사용되는 좋은 지표

- **CPUUtilization**: 인스턴스 전체의 평균 CPU 사용률을 확인한다.
- **RequestCountPerTarget**: EC2 인스턴스당 요청 수가 안정적인지 확인한다.

![37-good-metric-to-scaling.png](images%2F37-good-metric-to-scaling.png)

- **Average Network In / Out**: 응용 프로그램이 네트워크 바인딩된 경우, 많은 업로드와 다운로드가 있다면 인스턴스의 병목현상이 발견될 수 있다.
  평균 네트워크를 스케일아웃하여 어떤 임계값에 도달하면 그에 기반해 스케일아웃한다.
- CloudWatch를 통해 푸시된 커스텀 매트릭

#### Scaling Cooldown

- 조정 활동이 발생한 후에는 "Cooldown"(기본값: 300초) 기간이 된다.
- "Cooldown"기간 동안 ASG는 추가 인스턴스를 시작하거나 종료하지 않는다. (메트릭의 안정화를 위해)

![38-scaling-cooldowns.png](images%2F38-scaling-cooldowns.png)

- 요청을 더 빠르게 처리하고 "Cooldown" 기간을 줄이기 위해 즉시 사용 가능한 AMI를 사용하여 구성시간을 줄일 수 있다.
- 물론 ASG에 대한 상세 모니터링이 가능하도록 해야한다. 1분마다 메트릭이 충분히 업데이트 될 수 있도록 설정한다.

#### ASG Lifecycle Hook

- 기본적으로 ASG에서 인스턴스가 시작되자마자 서비스가 시작된다.
- 인스턴스가 서비스되기 전에 추가 단계를 수행할 수 있다.(Pending 상태)
  - 인스턴스가 시작될 때 실행할 스크립트를 정의한다.
- 인스턴스가 종료되기 전에 몇 가지 작업을 수행할 수 있다.(Terminating 상태)
  - 문제 해결을 위해 인스턴스가 종료되기 전에 일시 중지한다.
- 인스턴스가 시작되어 작동하기 전에 청소, 로그 추출 혹은 특별한 상태 확인 등의 작업에 사용된다.
- EventBridge, SNS, SQS와 통합되어 사용될 수 있다.

![33-lifecycle-hook.png](images%2F33-lifecycle-hook.png)

- 기본적으로 인스턴스를 시작하면 ASG에서 곧바로 InService 상태가 된다.
- 수명 주기 후크를 사용하는 경우 InService 상태가 되기 전에 일부 작업을 실행할 수 있다.
- 스케일 아웃 이벤트가 발생하고 인스턴스가 스크립트를 실행할 시간을 남겨두어야 할 때, "EC2_Instance_Launching" 수명주기 후크를 실행할 수 있다.
    - `Pending:Wait`로 전환되고 스크립트가 완료되면 `Pending:Proceed` 상태로 전환되도록 할 수 있다.
- 인스턴스가 종료될 때도 작업을 실행할 수 있다.
- 인스턴스를 바로 종료하는 대신 로그를 일시 중지하거나 문제 해결을 위해 인스턴스를 일시 중지해야 할 수 있다.
- 종료 시작 시 인스턴스가 Terminating 상태로 전환되면 "EC2_Instance_Terminating" 수명 주기 후크를 실행할 수 있다.
    - `Terminating:Wait` 상태가 되고 원하는 모든 작업을 한 다음 완료되면 ASG에 종료를 진행할 수 있다고 알린다.
    - 그 후 인스턴스는 예정대로 종료된다.
- 수명 주기 이벤트가 EventBridge에서 이벤트를 트리거하면 거기에서 람다 함수 등 원하는 자동화를 호출하여 이러한 수명 주기 후크를 처리할 수 있도록 구성할 수 있다.

![39-asg-lifecycle-hooks.png](images%2F39-asg-lifecycle-hooks.png)

- 인스턴스가 `Terminating:Wait` 상태로 전환되어 종료 중인데 로그를 수집하고 싶은 상황이다.
- 이벤트는 EventBridge에 있게 되고 여기서 Systems Manager Run Command를 호출하여 로그를 수집하고 이를 S3 등에 분석을 위해 보낼 수 있다.
    - CloudWatch Logs로 로그를 전송할 수도 있다.

#### SNS Notifications

- ASG는 다음 이벤트에 대한 SNS 알림 전송을 지원한다.
    - `autoscaling:EC2_INSTANCE_LAUNCH`
    - `autoscaling:EC2_INSTANCE_LAUNCH_ERROR`
    - `autoscaling:EC2_INSTANCE_TERMINATE`
    - `autoscaling:EC2_INSTANCE_TERMINATE_ERROR`

![40-sns-notifications.png](images%2F40-sns-notifications.png)

- 다음 ASG 이벤트와 일치하는 규칙을 생성할 수 있다.
    - EC2 인스턴스 시작, EC2 인스턴스 시작 성공/실패
    - EC2 인스턴스 종료 중, EC2 인스턴스 종료 성공/실패
    - EC2 Auto Scaling 인스턴스 새로 고침 체크포인트 도달
    - EC2 Auto Scaling 인스턴스 새로 고침 시작됨, 성공, 실패, 취소됨

![41-eventbridge-events.png](images%2F41-eventbridge-events.png)

#### Termination Policy













---

### 참고한 강의

- [DevOps Engineer Professional](https://www.udemy.com/course/aws-certified-devops-engineer-professional-korean)
- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)