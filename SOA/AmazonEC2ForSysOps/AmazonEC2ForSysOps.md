# Amazon EC2 for SysOps

이번 장에서는 **SysOps Administrator**를 준비하며 **EC2 인스턴스**에 대해서 알아보도록 한다.

---

## EC2

### EC2 인스턴스 유형 변경

[AmazonEC2ForSysOps.md](AmazonEC2ForSysOps.md)

- EBS를 지원하는 인스턴스에서만 유형을 변경할 수 있다.
  - 인스턴스를 중단(Stop) 한다.
  - 인스턴스 설정에서 `Change Instance Type`을 선택한다.
  - 인스턴스를 재실행한다.

### 강화된(Enhanced) 네트워킹

- EC2 Enhanced Networking (SR-IOV)
  - 더 높은 대역폭, 더 높은 PPS(Packet per Second), 더 짧은 지연시간
  - 선택 사항 1: "Elastic Network Adapter(ENA)"는 최대 100 Gbps까지 지원한다.
  - 선택 사항 2: "Intel 82599VF"는 최대 10Gbps까지 지원한다. (Legacy)
  - 최신 세대의 EC2 인스턴스에서 작동한다.
- Elastic Fabric Adapter (EFA)
  - 개선된 HPC(High Performance Computing)용 NEA는 Linux에서만 작동한다.
  - 노드 간 통신, 긴밀하게 결합된 워크로드에 적합하다.
  - Message Passing Interface(MPI) 표준을 활용한다.
  - 기본 Linux OS를 우회하여 지연 시간이 짧고 안정적인 서비스를 제공한다.

---

### 배치 그룹(Placement Group) <- 중요!

- EC2 인스턴스에 배치 전략을 제어하고 싶은 경우에 사용한다.
- 이러한 전략은 배치 그룹(Placement Group)을 사용하여 정의할 수 있다.
- 배치 그룹을 생성할 때, 그룹에 대해 아래의 전략 중 하나를 선택할 수 있다.
  - 클러스터(Cluster): 인스턴스를 **단일 가용 영역(동일한 랙)**의 지연 시간이 짧은 그룹으로 클러스터링한다.
  - 분산(Spread): 기본 하드웨어 전체에 인스턴스를 분산한다. 가용 영역 당 최대 7개의 인스턴스를 사용할 수 있다.
    가용성이 높으므로 중단되면 안되는 애플리케이션을 배치할 때 사용된다.
  - 파티션(Partition): 인스턴스를 다양한 파티션에 분산시킨다. (가용지역 내의 다양한 랙 세트) 그룹당 EC2 인스턴스를 수백 개까지 확장할 수 있다.

#### 클러스터(Cluster) 배치 그룹

![2-cluster-placement-groups.png](images%2F2-cluster-placement-groups.png)

- 장점: 뛰어난 네트워크(Enhanced Networking이 권장되며 인스턴스간 10Gbps 대역폭)를 지원한다.
- 단점: 랙에 장애가 발생하면 모든 인스턴스가 동시에 장애가 발생한다.
- 빠르게 완료되어야 하는 빅데이터 작업, 극도로 낮은 지연 시간과 높은 네트워크 처리량이 필요한 애플리케이션에 사용된다.

#### 분산(Spread) 배치 그룹

![3-spread-placement-groups.png](images%2F3-spread-placement-groups.png)

- 장점: 
  - 가용 지역 전반에 걸쳐 분산될 수 있다.
  - 가용성이 높아지고 동시에 중단될 가능성이 낮아진다.
  - **인스턴스는 물리적으로 다른 하드웨어에서 실행된다.**
- 단점:
  - 배치 그룹당, AZ당 인스턴스가 최대 7개로 제한된다.
- 고가용성을 극대화해야 하는 애플리케이션, 각 인스턴스가 서로 장애로부터 격리되어야 하는 중요한 애플리케이션에 적합하다.

#### 파티션(Partition) 배치 그룹

![4-partition-placement-groups.png](images%2F4-partition-placement-groups.png)

- 가용 지역당 최대 7개의 파티션을 생성할 수 있다.
- 동일한 지역의 여러 가용 지역에 분산될 수 있다.
- 최대 100개의 인스턴스를 생성할 수 있다.
- **파티션의 인스턴스는 절대로 다른 파티션의 인스턴스와 동일한 랙에서 실행되지 않는다.**
- 파티션 오류는 많은 인스턴스에 영향을 미칠 수 있지만 다른 파티션에는 영향을 미치지 않는다.
- 인스턴스는 메타데이터로 파티션 정보에 액세스한다.
- HDFS, HBase, Cassandra, Kafka와 같이 분산 처리 시스템 또는 파티션이 사용되는 애플리케이션에 적합하다.

---

### 종료 동작(Shutdown Behavior)

- Shutdown Behavior: OS를 사용하여 종료가 완료되면 기본적으로 인스턴스는 **중단(Stop)** 상태가 된다. 옵션을 변경하여 **중지(Terminate)**되도록 할 수 있다.
- 이러한 종료 동작은 AWS 콘솔에서 인스턴스를 종료할 때는 적용되지 않는다.
- EC 인스턴스 내에서 Shutdown할 때만 적용된다. CLI의 특징은 인스턴스 "InstanceInitiatedShutdownBehavior"다.

![5-shutdown-behavior.png](images%2F5-shutdown-behavior.png)

#### 종료 방지(Termination Protection)

- 종료 방지를 활성화하여 AWS 콘솔 또는 CLI에서 우발적으로 인스턴스를 종료하는 것을 방지할 수 있다.
- **하지만, 인스턴스 내에서 `shutdown` 커맨드를 입력하여 OS를 종료하는 경우 인스턴스는 중지(Terminate)된다.**

#### EC2 실행 문제 해결

- **InstanceLimitExceeded** 오류가 발생하면 지역당 최대 vCPU 제한에 도달했음을 의미한다.
- 온디맨드 인스턴스 한도는 지역(Region)별로 설정된다.
- 예를 들어, 온디맨드(A, C, D, H, I, M, R, T, Z) 인스턴스 유형은 기본적으로 64개의 vCPU만 실행시킬 수 있다.
- 다른 지역에서 인스턴스를 실행하거나 AWS에 지역 제한을 늘려달라고 요청하여 해결할 수 있다. 
  AWS 콘솔의 "Service Quotas"에서 지역 제한 증가를 요청할 수 있다.
- vCPU 기반 제한은 실행 중인 온디맨드 인스턴스 및 스팟 인스턴스에만 적용된다.

- **InsufficientInstanceCapacity** 오류가 발생하면 인스턴스가 시작되는 특정 AZ에 AWS의 온디맨드 용량이 충분하지 않다는 의미다.
- 이 문제는 사용자가 아닌 AWS에 발생 원인이 있다.
- 해결 방법은 아래와 같다.
  - 다시 요청하기 전에 몇 분 정도 기다린다.
  - 2개 이상을 요청하는 경우 요청을 분류한다. (5개를 요청하는 경우 5개를 한 번에 요청하는 것이 아니라 1개씩 다섯번을 요청한다.)
  - 긴급한 상황에는 나중에 크기를 조정할 수 있는 다른 인스턴스 유형에 대한 요청을 제출한다.
  - 다른 AZ의 인스턴스를 요청하여 문제를 해결한다.
  
- **InstanceTerminatesImmediately** (Pending 중에 종료)
  - EBS 볼륨이 한도에 도달한 경우에 발생한다.
  - EBS 스냅샷이 손상된 경우에 발생한다.
  - 루트 EBS이 암호화되어 있으며 복호화를 위해 KMS 키에 접근하려 하지만 액세스할 수 있는 권한이 없는 경우 발생한다.
  - 인스턴스를 시작하는 데 사용한 인스턴스 스토어 지원 AMI에 필수 부분(`image.part.xx` 파일)이 누락된 경우 발생한다.
- 정확한 이유를 찾으려면 "AWS의 EC2 콘솔" -> "인스턴스" -> "설명" 탭을 확인하고 상태 전환 이유 레이블 옆에 있는 원인을 확인하면 된다.

#### EC2 SSH 문제 해결

- Linux 시스템의 개인 키(pem 파일)가 400 권한을 가지고 있는지 확인한다. 만약 그렇지 않다면 `Unprotected private key file` 오류가 발생한다.
- SSH를 통해 로깅할 때 OS에 대한 사용자 이름이 올바르게 입력하였는지 확인한다. 만약 그렇지 않으면 `Host key not found`, `Permission denied` 또는 `Connection closed by [instance] port 22` 오류가 발생한다.
- SSH를 통해 EC2 인스턴스에 접속할 때, `Connection Timed Out` 오류가 발생하는 여러가지 원인이 있다.
  - Security Group이 올바르게 구성되지 않았다.
  - NACL이 올바르게 구성되지 않았다.
  - 서브넷의 라우팅 테이블을 확인한다(VPC 외부로 향하는 트래픽을 IGW로 라우팅).
  - 인스턴스에 IPv4가 없는 경우.
  - 인스턴스의 CPU 부하가 높은 경우.

#### SSH vs Instance Connect

- SSH를 직접 사용하여 접속하는 것과 AWS 콘솔에서 "Instance Connect"를 통해서 접속하는 것은 아래와 같은 차이가 있다.

![6-ssh-vs-instance-connect.png](images%2F6-ssh-vs-instance-connect.png)

- SSH를 직접 사용하여 접속하는 경우 접속하려는 사용자의 IP가 인바운드 규칙에 추가되어 있어야한다.
- "Instance Connect"를 사용하는 경우 AWS API를 경유하여 접속하게 된다. 그렇기 때문에 사용자의 IP가 아니라 "Instance Connect"의 IP주소를 인바운드 규칙에 추가해야 한다.
  각 리전의 "Instance Connect" 규칙은 [여기](https://ip-ranges.amazonaws.com/ip-ranges.json)에서 확인이 가능하다. 

---

### EC2 구매 옵션

- On-Demand Instances: 짧은 워크로드, 예측 가능한 가격, 초 단위 지출
- Reserved (1 or 3 years)
  - Reserved Instances: 긴 워크로드
  - Convertible Reserved Instances: 긴 워크로드와 함께 유형 변경 가능한 인스턴스
- Savings Plans: 사용량 약정을 사용하여 긴 워크로드를 처리한다.
- Spot Instances: 짧은 워크로드, 저렴한 가격을 제공하지만 손실의 가능성이 있다.(신뢰도가 낮음)
- Dedicated Hosts: 전체 물리 서버를 예약하고 인스턴스 배치를 제어한다.
- Dedicated Instances: 다른 고객이 요청자의 하드웨어를 공유하지 않는다.
- Capacity Reservations: 특정 기간 동안 특정 AZ의 용량을 예약한다.

#### On-Demand Instances

- 사용한 만큼만 지불하면 된다. 
  - Linux 또는 Windows를 지원한다. **처음 1분 사용 이후 초단위로 지불한다.**
  - 이외의 모든 OS에 대해서는 시간당 비용을 청구한다.
- 비용이 가장 높지만 선불 결제는 없다.
- 장기 약정이 필요없다.
- **단기적이고 방해받지 않아야 하는 워크로드에 적합하다.**
- 응용 프로그램이 어떻게 작동할지 예측할 수 없는 곳에 적합하다.

#### Reserved Instances

- On-Demand에 비해 최대 72%까지 할인을 받을 수 있다.
- 특정 인스턴스의 속성을 지정한다. (예. 인스턴스 유형, 지역, 테넌시, OS)
- 더 많은 할인을 받기 위해 1년이나 3년의 예약 기간을 명시하고 부분적으로 선지급을 할지 말지 결정한다.
- 지급 옵션은 선지급 없음(저렴), 부분 선지급(더 저럼), 전부 선지급(매우 저렴)이 있다.
- 예약 인스턴스의 범위는 지역(Regional) 또는 가용 지역(Zonal) 중 선택할 수 있다.
- 데이터베이스와 같이 안정적인 상태(steady-state)를 제공해야 하는 애플리케이션에 적합하다.
- 구매 이후 필요없어진 예약 인스턴스는 Marketplace에서 판매할 수 있다.

- "Convertible Reserved Instance"는 인스턴스 유형, 인스턴스 패밀리, 운영 체제, 범위, 테넌시를 변경할 수 있다.
  물론 할인률은 조금 감소하여 최대 66%까지 할인받을 수 있다.

#### Savings Plans

- 장기 사용량에 따라 할인을 받을 수 있다. 최대 72%까지 할인을 받을 수 있으며 이는 RI와 동일하다.
- 특정 유형의 사용량을 약정한다. (1년 또는 3년)
- 만약 1, 2, 3년간 시간당 $10를 사용한다고 가정하였을 때, 초과하는 비용에 대해서는 On-Demand 가격으로 청구된다.
- Savings Plans는 특정 인스턴스와 지역(Region)에 묶이게 된다.
  예를 들어, m5를 선택하는 경우 특정 지역에서 m5.large, m5.2xlarge와 같이 사이즈를 유연하게 변경할 수 있다.
- 이외에도 OS(예. Linux, Windows), 테넌시(Host, Dedicated, Default)를 유연하게 변경할 수 있다.

#### Spot Instances

- On-Demand 유형 대비 최대 90%까지 할인을 받을 수 있다.
- 경매 방식이기 때문에 "최대 가격"이 "현재의 스팟 가격"을 초과하는 경우 언제든지 잃을 수 있다.
- **AWS에서 가장 비용 효율적인 유형이다.**
- 장애에 대한 복원력이 있는 워크로드에 유용하다.
  - 배치 작업, 데이터 처리, 이미지 처리, 분산된 워크로드, 시간 및 종료 시간이 유연한 워크로드
- 중요한 작업이나 데이터베이스 용도로 사용하기에는 적합하지 않다.

#### Dedicated Hosts

- EC2 인스턴스 용량을 고객 전용으로 갖춘 물리적 서버다.
- **규정 준수 요구 사항을 충족하고 기존 서버 기반의 소프트웨어 라이선스를 사용할 수 있다. (소켓당, 코어당, pe-VM 소프트웨어 라이선스)**
- 초당 비용을 지불하는 On-Demand 방식과 1년 또는 3년을 예약하는 Reserved 방식이 있다.
- **AWS에서 가장 비싼 유형이다.**
- 복잡한 라이선스 모델의 소프트웨어를 사용하는데 유용하다.(BYOL - Bring Your Own License)
- 강력한 규제 또는 규정 준수 요구 사항이 있는 기업들이 사용하기에 유용하다.

#### Dedicated Instances

- **인스턴스는 사용자 전용 하드웨어에서 실행된다.**
- **동일한 계정의 다른 인스턴스와 하드웨어를 공유할 수 있다.**
- 인스턴스 배치를 제어할 수 없다.(중단(Stop)/시작 후 하드웨어 이동이 가능)
- **복잡한 라이선스 모델을 사용할 수 없다.**
- "Dedicated Hosts"와 "Dedicated Instances"의 차이점은 아래의 이미지와 같다.

![7-dedicated-hosts-vs-instances.png](images%2F7-dedicated-hosts-vs-instances.png)

#### Capacity Reservations

- 특정 기간 동안 AZ에 온디맨드 인스턴스 용량을 예약한다.
- 필요할 때 언제든지 EC2 용량에 액세스할 수 있따.
- **시간 약정이 없으므로 언제든지 생성 및 취소할 수 있지만 결제 할인이 되지 않는다.**
- "지역(Regional) Reserved Instances"와 "Saving Plans"와 결합하여 결제 할인 혜택을 누릴 수 있다.
- **인스턴스의 실행 여부와 관계없이 예약한 용량만큼 비용이 지불된다.**
- 특정 AZ에 있어야 하는 중단 없는 단기 워크로드에 적합하다.

#### 가격 비교

- 위에서 알아본 인스턴스 유형들 간의 가격은 아래의 이미지를 참고하면 된다.

![8-instances-types-price-comparison.png](images%2F8-instances-types-price-comparison.png)

---

### Spot Instances & Spot Fleet

- Spot Instances는 On-Demand 대비하여 최대 90%까지 할인받을 수 있다.
- 최대 스팟 가격을 정의하고 "현재의 스팟 가격"보다 "정의한 최대 스팟 가격"이 높은 경우에 인스턴스를 가져올 수 있다.
  - 시간단 스팟 가격은 제안 및 용량에 따라 다르다.
  - 만약 "현재 스팟 가격"이 "정의한 최대 스팟 가격"보다 높은 경우 2분 이후에 인스턴스는 중단(Stop)되거나 중지(Terminate)될 수 있고 사용자가 선택할 수 있다.
- 다른 전략으로는 "스팟 블록"이 있다.
  - 지정된 기간(1 ~ 6시간) 동안 중단 없이 스팟 인스턴스가 방해받지 않도록 하여 회수되지 않도록 한다.
  - 드문 경우이지만 인스턴스가 회수될 수도 있다.
- 배치 작업, 데이터 분석, 장애에 대한 복원력이 있는 워크로드에 적합하다.
- 중요한 작업이나 데이터베이스에는 적합하지 않다.

#### Pricing

![9-spot-instances-pricing.png](images%2F9-spot-instances-pricing.png)

#### Terminate Spot Instances

![10-terminate-spot-instances.png](images%2F10-terminate-spot-instances.png)

- "스팟 인스턴스에 대한 단일 요청"을 할 수도 있고, "스팟 인스턴스에 대한 지속적인 요청"을 할 수도 있다.
- "단일 요청"의 경우 스팟 요청이 수행되지 마자 인스턴스가 실행되고 스팟 요청은 없어질 것이다.
- "지속적인 요청"의 경우 스팟 요청이 유효한 경우 사용자가 원하는 인스턴스의 수가 유지될 수 있도록 지속적으로 요청한다.
  인스턴스가 스팟 가격에 의해 중단되거나 중지되면 스팟 요청이 다시 작동하게 되고, 유효성 검사가 끝나면 인스턴스를 다시 시작할 것이다.
- "지속적인 요청"에서 인스턴스를 정지했는지 요청이 계속 실행 중이라면 다시 요청을 시작할 것이다.
- 만약 요청을 취소하고 싶다면 `open`, `active`, `diasabled` 상태에 있어야 한다. `failed`, `cancelled`, `closed` 상태에서는 요청을 취소할 수 없다.
- **"스팟 요청"을 취소하더라도 이미 생성된 스팟 인스턴스는 자동으로 종료되지 않는다.** 사용자가 직접 종료시켜야 한다.
  먼저 스팟 요청을 취소한 후 연결된 스팟 인스턴스를 종료시켜야 한다.

#### Spot Fleet

- 돈을 절약할 수 있는 최고의 방법으로 "Spot Instance 세트"와 선택적으로 On-Demand 인스턴스를 조합하여 사용한다.
- "Spot Fleet"은 가격 제약으로 목표 용량을 충족시키기 위해 노력한다.
  - "Launch Pool": 인스턴스 유형, OS, AZ
  - 여러 Spot Fleet에서 선택할 수 있도록 여러 "Launch Pool"을 생성할 수 있다.
  - "Spot Fleet"은 용량 또는 최대 비용에 도달하면 인스턴스 시작을 중지한다.
- 스팟 인스턴스 할당 전략은 여러가지가 있다.
  - `lowerPrice`: 최저 가격의 풀에서 인스턴스를 실행하여 비용을 최적화할 수 있게하며 짧은 워크로드를 처리하기 위한 좋은 방법이다.
  - `diversified`: 이전에 정의한 모든 "Launch Pool"에 배포가 될 것이다. 하나의 Pool이 사라지더라도 다른 Pool은 유효하기 때문에 가용성이 높고 긴 워크로드를 처리하기에 좋은 방법이다.
  - `capacityOptimized`: 인스턴스 수에 대한 최적의 용량을 갖춘 Pool에서 선택한다.
  - `priceCapacityOptimized`: 가장 권장되는 방식으로 사용 가능한 용량이 가장 높은 Pool을 선택한 다음 가격이 가장 낮은 Pool을 선택한다.
    대부분의 워크로드에 가장 적합한 방식이다.
- "Spot Fleet"을 사용하면 최저 가격으로 스팟 인스턴스를 자동으로 요청할 수 있다.

---

### Burstable Instances

- AWS에는 T2/T3와 같이 버스트 가능한 인스턴스 유형이 있으며 대부분 "T-패밀리"다.
- 버스트는 인스턴스의 CPU 성능이 전반적으로 양호하다는 것을 의미한다.
- 부하 급증과 같이 머신이 예상하지 못한 일을 처리해야 할 때, 버스트를 사용하여 CPU성능이 매우 좋을 수 있다.
- 버스트 모드가 활성화되면 "Credit"을 활용하게 된다.
- 모든 "Credit"을 사용하면 CPU의 성능은 매우 나빠지고 높아진 부하를 처리할 수 없다.
- 부하가 줄어들어 CPU 사용량이 복구되어 "Credit"을 사용하지 않는 상태가 되면 점차 "Credit"이 복구된다.
- "버스트 가능한 인스턴스"는 예상치 못한 트래픽을 처리하고 트래픽이 올바르게 처리될 것을 보장받는 데 탁월하다.
- **인스턴스의 크레딧이 지속적으로 부족하면 다른 종류의 버스트 불가능한 인스턴스로 이동해야 한다.**
- AWS 콘솔에서 크레딧 사용량 지표와, 크레딧 잔량 지표를 활용하여 얼만큼 버스트 크레딧이 사용되고 남아 있는지 확인할 수 있다.

![11-credit-usage-balance.png](images%2F11-credit-usage-balance.png)

![12-cpu-credit.png](images%2F12-cpu-credit.png)

#### Credit 고갈(Exhausted) 시나리오

- CPU 스트레스 명령을 사용하여 100% 사용률을 만든다.
- 크레딧이 있는 경우 크레딧을 소진하여 100%의 CPU 사용률을 유지할 수 있지만, 크레딧이 소진된 경우 기본 CPU 사용량인 10%로 내려오는 것을 확인할 수 있다.

![13-credit-exhausted.png](images%2F13-credit-exhausted.png)

#### Unlimited

- "무제한 버스트 크레딧 잔액"이 가능하다.
- "Credit Balance"를 초과하면 추가 비용을 지불하지만 성능은 떨어지지 않는다.
- 24시간 동안 평균 CPU 사용량이 기준을 초과하는 경우 vCPU/시간당 추가 사용량에 대해 인스턴스에 요금이 청구된다.
- 인스턴스의 CPU 상태를 모니터링하지 않으면 비용이 많이 발생할 수 있다.

![14-t2-t3-unlimited.png](images%2F14-t2-t3-unlimited.png)

- 이미지를 확인해보면 CPU 사용량이 100% 다, CPU 크레딧도 지속적으로 사용하고 있다.
- 24시간 동안 충전된 여분(Surplus)의 Credit이 먼저 사용된다. 72가 된 경우 1시간 사용할 수 있는 CPU Credit Balance가 충전된다.
- 이런 방식으로 무제한으로 사용할 수 있으며 사용한 만큼의 비용만 지불하면 된다.

---

### Elastics IPs

- EC2 인스턴스를 중단(Stop)했다가 다시 시작하면 Public IP가 변경된다.
- **고정된 Public IP가 필요한 경우 Elastic IP가 필요하다.**
- "Elastic IP"는 삭제하지 않는 한 사용자가 소유한 Public IPv4가 된다.
- 한 번에 하나의 인스턴스에 연결할 수 있다.
- 인스턴스에 연결되어 있는 "Elastic IP"를 다른 인스턴스에 연결되도록 설정할 수 있다.
- **"Elastic IP"가 서버에 연결된 경우 비용을 지불하지 않는다.**
- **"Elastic IP"가 서버에 연결되지 않는 경우 비용을 지불하게 된다.**

![15-elastic-ips.png](images%2F15-elastic-ips.png)

- "Elastic IP" 주소를 사용하면 주소를 계정의 달느 인스턴스에 신속하게 다시 매핑하여 인스턴스나 소프트웨어의 오류를 마스킹할 수 있다.
- 하나의 AWS 계정에 최대 5개까지 "Elastic IP"를 할당받을 수 있다. 필요한 경우 AWS에 요청하여 늘릴 수 있다.
- "Elastic IP" 사용을 피하는 여러가지 방법이 있다.
  - 고정 IP를 사용하기 이전에 다른 방법이 있는지 생각해본다.
  - 임의의 Public IP를 사용하고 여기에 DNS 이름을 등록할 수 있다.
  - 정적 호스트 이름으로 로드 밸런서를 사용할 수 있다.

---

### CloudWatch 지표 & EC2 연동

- AWS 제공 지표(AWS가 수집)
  - 기본 모니터링(기본값): 내부적으로 5분 간격으로 지표가 수집된다.
  - 상세 모니터링(유료): 1분 간격으로 지표가 수집된다.
  - CPU, 네트워크, 디스크 및 상태 확인 지표를 포함한다.
- 사용자 정의(Custom) 지표(사용자가 수집)
  - 기본 수집주기: 1분마다 수집한다.
  - 높은 수집주기: 최소 1초마다 수집하도록 설정할 수 있다.
  - RAM과 애플리케이션 수준의 지표를 수집한다.
  - EC2 인스턴스 역할에 대한 IAM 권한이 올바른지 확인해야 한다.

- 각 지표는 아래와 같은 정보를 포함하고 있다.
  - CPU: CPU 사용률 + 크레딧 사용량/잔량(버스트(T) 시리즈인 경우)
  - Network: 네트워크 In/Out
  - Status Check
    - Instance 상태: EC2 가상머신의 상태
    - System 상태: 기본 하드웨어 상태
  - Dis: 읽기/쓰기에 대한 Ops/Bytes (Instance Store만 해당)
- **RAM은 AWS EC2 지표에 포함되지 않는다.**

#### Unified CloudWatch Agent

- 가상 서버(EC2 인스턴스, On-Premise 서버)의 경우 사용된다.
- RAM, 프로세스, 사용된 디스크 공간 등과 같은 추가 시스템 수준의 지표를 수집할 수 있다.
- **CloudWatch로 보낼 로그를 수집한다.**
  - 에이전트를 사용하지 않으면 EC2 인스턴스 내부의 로그가 CloudWatch Logs로 전송되지 않는다.
- 설정을 위한 JSON 형식의 구성 파일을 "SSM Parameter Store" 저장하고 재사용하여 중앙 집중식으로 구성할 수 있다.
- IAM 권한을 올바르게 설정해야 한다.
- "Unified CloudWatch Agent"가 수집한 지표의 기본 네임스페이스는 `CWAgent`이며 설정 및 변경이 가능하다.

![16-unified-cloudwatch-agent.png](images%2F16-unified-cloudwatch-agent.png)

#### Unified CloudWatch Agent - procstat 플러그인

- 개별 프로세스의 지표를 수집하고 시스템 활용도를 모니터링한다.
- Linux 및 Windows 서버 모두 지원한다.
- 예를 들어, "프로세스가 CPU를 사용하는 시간", "프로세스가 사용하는 메모리의 양"을 수집할 때 사용할 수 있다.
- 모니터링할 프로세스를 선택한다.
  - pid_file: 생성된 프로세스 식별 번호(PID) 파일의 이름
  - exe: 지정한 문자열과 일치하는 프로세스 이름(RegEx)
  - pattern: 프로세스를 시작하는 데 사용되는 커맨드(RegEx)
- "procstat 플러그인"에 의해 수집된 측정항목은 `procstat` 접두사로 시작된다. (`procstat_cpu_time`, `procstat_cpu_usage` 등)

---

### 상태 확인(Status Checks)

- 하드웨어 및 소프트웨어 문제를 식별하기 위한 자동화된 검사를 지원한다.
- **System Status Checks**
  - AWS 시스템(소프트웨어/하드웨어, 물리적 호스트, 시스템 전원 손실 등)의 문제를 모니터링한다.
  - 인스턴스 호스트에 대한 AWS의 예정된 중요 유지 관리가 있는지 "Personal Health Dashboard"에서 확인할 수 있다.
  - 해결방법은 인스턴스를 중단(Stop)하고 다시 시작한다. 이때 인스턴스가 새로운 호스트로 마이그레이션 된다.
    **다른 호스트로 마이그레이션되기 때문에 인스턴스가 재실행되는 경우에 Public IP가 변경되는 것이다.**
  - 시스템 상태 확인은 인스턴스가 실행되는 AWS 시스템을 모니터링한다.
    - 네트워크 연결 끊김
    - 시스템 전원 손실
    - 물리적 호스트의 소프트웨어 문제
    - 네트워크 연결 가능성에 영향을 미치는 물리적 호스트의 하드웨어 문제
    - AWS가 호스트를 수정할 때까지 기다리거나, 또는 EC2 인스턴스를 새 호스트로 이동(Stop & Start, EBS가 지원되는 경우)

![17-instance-status-check.png](images%2F17-instance-status-check.png)

- **Instance Status Checks**
  - 인스턴스의 소프트웨어/네트워크 구성을 모니터링한다. (잘못된 네트워크 구성, 소모된 메모리 등)
  - 해결방법은 인스턴스를 재부팅하거나 인스턴스의 구성을 변경한다.
  - 인스턴스 상태 확인은 개별 인스턴스의 소프트웨어 및 네트워크 구성을 모니터링한다.
    - 잘못된 네트워킹 또는 시작 구성
    - Memory 고갈
    - 손상된 파일 시스템
    - 호환되지 않는 커널
  - 이러한 문제를 해결하기 위해서는 사용자의 참여가 필요하다.
  - 인스턴스를 재실행하거나 인스턴스 구성을 변경해야 한다.

#### CW Metrics & Recovery

- CloudWatch Metrics(1분 주기)
  - `StatusCheckFailed_System`
  - `StatusCheckFailed_Instance`
  - `StatusCHeckFailed` (둘 다 해당)
- 시스템 복구를 위한 두가지 옵션이 있다.
- 옵션 1: CloudWatch Alarm
  - **동일한 Private/Public IP, EIP, 메타데이터 및 배치 그룹을 사용하여 EC2 인스턴스를 복구한다.**
  - SNS를 통해서 알림을 전송한다.

![18-cw-metrics-recovery-1.png](images%2F18-cw-metrics-recovery-1.png)

- 옵션 2: Auto Scaling Group
  - **최소/최대/원하는 인스턴스의 양을 설정한 상태로 유지하도록 도와주지만 "CloudWatch Alarm"을 통한 복구와는 다르게 동일한 Private/Elastic IP는 유지하지 않는다.**
  - 동일한 EBS 볼륨도 지원하지 않는다.
- EC2 인스턴스 하나를 복구하는 경우라면 옵션 1이 선호된다.

![19-cw-metrics-recovery-2.png](images%2F19-cw-metrics-recovery-2.png)

---

### Hibernate(최대 절전 모드)

- 인스턴스는 중단(Stop), 종료(Terminate)할 수 있다.
  - Stop: 디스크(EBS)의 데이터는 다음 시작 시 그대로 유지된다.
  - Terminate: 폐기하도록 설정된 EBS 볼륨이 손실된다.
- 인스턴스 시작 시 아래의 동작들이 일어난다.
  - OS가 부팅되고 EC2 사용자 데이터 스크립트가 실행된다.
  - OS가 부팅된다.
  - 그런 다음 애플리케이션이 시작되고 캐시가 준비될 때까지 시간이 소요된다.

- "EC2 Hibernate"는 아래와 같은 특성을 가지고 있다.
  - in-memory의 상태가 유지된다.
  - OS가 중지/재시작 되지 않으므로 부팅의 속도가 빨라진다.
  - 내부적으로 RAM의 상태가 루트 EBS 볼륨의 파일에 기록된다.
  - 루트 EBS 볼륨은 암호화되어야 한다.
- 사용 사례: 장기 실행 저장, RAM 상태 저장, 초기화에 시간이 걸리는 서비스 

![20-hibernate.png](images%2F20-hibernate.png)

- 대부분의 EC2 인스턴스 유형을 지원한다.(C3, C4, I3, M3, M4, R3, R4...)
- RAM의 크기는 150GB미만이어야 한다.
- 베어메탈 인스턴스는 지원하지 않는다.
- Amazon Linux 2, Linux AMI, Ubuntu, RHEL, CentOS, Windows 등의 AMI를 지원한다.
- 루트 볼륨은 반드시 암호화된 EBS여야 한다. 인스턴스 스토어는 지원하지 않으며 용량이 커야한다.
- On-Demand, Reserved, Spot 인스턴스 유형을 지원한다.
- **최대 Hibernate 기간은 60일이다.**

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [Instance Connect IP 범위](https://ip-ranges.amazonaws.com/ip-ranges.json)
- [InsufficientInstanceCapacity Error](https://aws.amazon.com/premiumsupport/knowledge-center/ec2-insufficient-capacity-errors/)