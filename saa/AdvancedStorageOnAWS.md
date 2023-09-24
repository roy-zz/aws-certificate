# Advanced Storage on AWS

이번 장에서는 SAA를 준비하며 **Advanced Storage on AWS(AWS 고급 스토리지)**에 대해서 알아보도록 한다.

---

## AWS Snow Family

- **Snow Family**는 Edge에서 데이터를 수집 및 처리하고 AWS 내부 또는 외부로 데이터를 마이그레이션하는 매우 안전한 휴대용 장치다.

![snow-familiy-list.png](images%2FAdvancedStorageOnAWS%2Fsnow-familiy-list.png)

- 데이터를 Migration하기 위한 많은 과제가 있다.
  - 제한된 연결(Limited connectivity)
  - 제한된 대역폭(Limited bandwidth)
  - 높은 네트워크 비용(High network cost)
  - 공유 대역폭으로 인해 라인을 최적화할 수 없음.
  - 접속 안정성(Connection Stability)

![migration-spent-time.png](images%2FAdvancedStorageOnAWS%2Fmigration-spent-time.png)

- **AWS Snow Family**는 데이터 마이그레이션을 수행할 수 있는 오프라인 장치다.
- 만약 네트워크를 통해 전송하는데 일주일 이상의 시간이 소요된다면 **Snowball device**를 고려해볼 수 있다.

### Diagrams

![snow-familiy-diagrams.png](images%2FAdvancedStorageOnAWS%2Fsnow-familiy-diagrams.png)

### Snowball Edge (for data transfer)

- 물리적 데이터 전송 솔루션으로 TB 또는 PB의 데이터를 AWS 내부 또는 외부로 이동시킬 수 있다.
- 네트워크를 통해 데이터를 이동(네트워크 사용료 지불)하는 것에 대한 대안이다.
- 데이터 전송 작업당 비용을 지불한다.
- 블록 스토리지 및 Amazon S3 호환 객체 스토리지를 제공한다.
- 최적화된 Snowball 엣지 스토리지.
  - 블록 볼륨 및 S3 호환 개체를 위한 HDD 용량 80TB를 제공한다.
- Snowball 엣지 컴퓨팅 최적화.
  - 블록 볼륨 및 S3 호환 객체 스토리지를 위한 42TB의 HDD 또는 28TB의 NVMe 용량을 제공한다.
- 대표적인 활용 사례로는 대규모 데이터 클라우드 마이그레이션, DC 중단, 재해 복구가 있다.

![snowball-edge.png](images%2FAdvancedStorageOnAWS%2Fsnowball-edge.png)

### AWS Snowcone & snowcone SSD

- 어디서나 사용할 수 있는 소형 휴대용 컴퓨팅, 견고하고 안전하며 가혹한 환경에서도 견딜 수 있도록 설계되었다.
- 4.5파운드, 2.1kg으로 가볍다.
- 엣지 컴퓨팅, 스토리지, 데이터 전송에 사용되는 장비다.
- Snowcone - 8TB의 HDD 스토리지를 제공한다.
- Snowcone SSD - 14TB의 SSD 스토리지를 제공한다.
- Snowball 장비를 사용하기에 적합하지 않은 환경에 사용된다.
- 배터리/케이블은 사용자가 직접 제공해야 한다.
- 오프라인으로 AWS로 데이터를 전송하거나 인터넷 및 AWS DataSync를 사용하여 데이터를 전송할 수 있다.

![snowcone-ssd.png](images%2FAdvancedStorageOnAWS%2Fsnowcone-ssd.png)

### AWS Snowmobile

- 엑사바이트 단위의 데이터를 전송하기 위해 사용된다. (1EB = 1,000PB = 1,000,000TB)
- 각 Snowmobile의 용량은 100PB로 여러 대를 병렬로 사용할 수 있다.
- 온도 제어, GPS, 연중무휴 비디오 보안등 높은 보안 수준을 제공한다.
- 10PB 이상의 데이터를 전송하는 경우 Snowball보다 우수하다.

![snow-familiy-comparison.png](images%2FAdvancedStorageOnAWS%2Fsnow-familiy-comparison.png)

#### Snow Family - 이용 순서

1. AWS 콘솔에서 Snowball 디바이스를 전송 요청한다.
2. 서버에 Snowball 클라이언트 / AWS OpsHub를 설치한다.
3. 서버에 Snowball을 연결하고 클라이언트를 사용하여 파일을 복사한다.
4. 작업이 완료되면 기기를 다시 AWS로 전송한다.
5. 데이터가 S3 버킷으로 로드된다.
6. Snowball의 데이터가 완벽하게 제거된다.

---

### Edge Computing

- **Edge 위치**에서 데이터를 생성하는 동안 데이터를 처리할 수 있다.
- **Edge 위치**는 도로 위의 트럭, 바다 위의 배, 지하의 광산 등을 의미한다.
- 이러한 위치들은 아래와 같은 특징을 가지고 있다.
  - 제한되거나 단절된 인터넷 환경
  - 제한되거나 용이하지 않은 컴퓨팅 성능
- Snowball Edge / Snowcone 장치를 설정하여 Edge에서 컴퓨팅을 수행할 수 있다.
- Edge 컴퓨팅은 아래와 같은 사용 사례를 가지고 있다.
  - 데이터 전처리
  - Edge 위치에서의 머신러닝
  - 트랜스코딩 미디어 스트림
- 필요한 경우 데이터를 AWS로 반송할 수 있다.

#### Snow Family - Edge Computing

- **Snowcone & Snowcone SSD**
  - CPU 2개, 메모리 4GB, 유선 또는 무선 액세스를 제공한다.
  - 코드 또는 옵션 배터리를 사용한 USB-C타입의 전원을 제공한다.
- **Snowball Edge - Compute Optimized**
  - 104개의 vCPU, 416 GiB의 RAM을 제공한다.
  - 만약 비디오 처리 또는 머신러닝에 사용한다면 GPU 옵션을 사용할 수 있다.
  - 28TB NVMe 또는 42TB HDD의 스토리지가 사용 가능하다.
- **Snowball Edge - 최적화된 스토리지**
  - 최대 40개의 vCPU, 80Gb RAM, 80TB 스토리지를 제공한다.
  - 개체 스토리지 클러스터링이 사용할 수 있다.
- 모든 종류의 Snow Family는 EC2 Instance & AWS Lambda 기능을 실행할 수 있다.(AWS IoT Greengrass)
- 장기 배포를 하는 경우 1년 및 3년 할인이 가능하다.

#### AWS OpsHub

- AWS OpsHub 소프트웨어를 설치하여 사용하면 CLI를 사용하지 않고 Snow Familiy Device를 관리할 수 있다.
  - 단일 또는 클러스터된 장치 잠금 해제 및 구성을 수행할 수 있다.
  - 파일을 전송할 수 있다.
  - Snow Family Device에서 실행되는 인스턴스를 실행 및 관리할 수 있다.
  - Device의 메트릭(스토리지 용량, 활성 인스턴스)을 모니터링할 수 있다.
  - 기기에서 호환되는 EC2, DataSync, NFS 등의 AWS 서비스를 시작할 수 있다.

#### Snowball into Glacier

- Snowball의 데이터를 바로 Gracier로 이동시킬 수 없다.
- Amazon S3로 데이터를 먼저 이동시키고 S3 Lifecycle 정책과 함께 사용해야 한다.

![snowball-with-glacier.png](images%2FAdvancedStorageOnAWS%2Fsnowball-with-glacier.png)

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03