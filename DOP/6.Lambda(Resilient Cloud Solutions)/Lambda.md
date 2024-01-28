# Lambda

- 이번 장에서는 **DevOps Engineer Professional (DOP)** 을 준비하며 "AWS 람다 서비스"에 대해서 알아보도록 한다.

---

### AWS Lambda

### Version

- 사용자가 람다 함수에 대해 작업할 때 `$LATEST` 버전을 사용한다.
- 람다 함수를 배포할 준비가 되면 버전을 생성한다.
- 버전이 생성되면 코드나 환경 변수를 변경할 수 없다.
- 새로운 버전이 출시되면 버전의 번호가 증가한다.
- 각 버전은 독립적이며 고유한 ARN(Amazon Resource Name)을 가진다.
- 버전은 코드와 구성으로 이루어진다.
- 각 람다 함수 버전에는 `$LATEST`에처럼 액세스할 수 있다.

![1-lambda-version.png](images%2F1-lambda-version.png)

#### Alias

- 별칭은 람다 함수 버전에 대한 "포인터"다.
- "dev", "test", "prod" 별칭을 정의하고 서로 다른 람다 버전을 가리키도록 할 수 있다.
- 별칭은 변경이 가능하다.
- 별칭을 사용하면 람다 함수에 가중치를 할당하여 Canary 배포가 가능하다.
- 별칭을 사용하면 이벤트 트리거/대상을 안정적으로 구성할 수 있다.
- 별칭에는 자체 ARN이 할당된다.
- **별칭은 별칭을 참조할 수 없다.**

![2-alias.png](images%2F2-alias.png)

#### Environment Variables

- 환경 변수는 String 형식의 키/값 쌍으로 이루어져 있다.
- 코드 업데이트 없이 함수 동작을 조정할 수 있다.
- 코드에서 환경 변수를 사용할 수 있다.
- 람다 서비스는 자체 시스템 환경 변수도 추가한다.
- 비밀을 저장하는데 도움이되며 KMS로 암호화된다.
- 비밀은 람다 서비스 키 또는 자체 CMK로 암호화될 수 있다.

#### Lambda Concurrency & Throttling

- 최대 1,000개까지 동시에 실행이 가능하다.

![3-concurrency-throttling.png](images%2F3-concurrency-throttling.png)

- 동시 실행 횟수를 제한할 수 있으며, 권장되는 방식이다.
- 함수 수준에서 "reserved concurrency"을 설정할 수 있다.
- 동시성 제한을 초과하는 각 호출은 "Throttle"를 트리거한다.
- "Throttle" 동작
    - 동기 호출: ThrottleError(429) 반환
    - 비동기 호출: 자동으로 재시도한 후 DLQ로 이동
- 1,000이 넘는 더 많은 한도가 필요하다면 지원 티켓을 사용할 수 있다.

#### Concurrency Issue

- 만약 동시성을 예약(=제한)하지 않으면 아래와 같은 문제가 발생할 수 있다.

![4-concurrency-issue.png](images%2F4-concurrency-issue.png)

- 많은 사용자가 ALB를 통해서 요청을 하고 있다.
- 적은 사용자가 API Gateway와 SDK/CLI를 통해서 요청을 하고 있다.
- 요청의 수가 적을 때는 문제가 없지만 요청의 수가 많아져서 ALB에서 사용하는 함수의 수가 1,000개가 된다면 API Gateway와 SDK/CLI를 통해서 처리되는 함수는 확장될 수 없다.
- **계정에 모든 함수에 동시성 한계가 적용된다는 점을 기억해야 한다.**

#### Asynchronous Invocations

![5-concurrency-asynchronous-invocations.png](images%2F5-concurrency-asynchronous-invocations.png)

- S3 버킷에 파일이 많이 업로드되면 Lambda 함수가 동시에 많이 실행된다.
- 만약 한계에 도달해서 확장할 수 없다면 추가 요청은 제한된다.
- 함수에 모든 이벤트를 처리하는데 사용할 수 있는 동시성이 충분하지 않은 경우 추가 요청이 제한된다.
- Throttling Error(429) 및 시스템 오류(5xx)의 경우 Lambda는 이벤트를 대기열에 반환하고 최대 6시간 동안 함수를 다시 실행하려고 시도한다.
- 재시도 간격은 첫 번째 시도 후 1초에서 최대 5분까지 기하급수적으로 늘어난다.

#### Cold Start & Provisioned Concurrency

- Cold Start
    - 새로운 인스턴스: 코드가 로드되고 핸들러 외부의 코드가 실행된다.(init)
    - 초기화가 큰 경우(code, dependencies, SDK...) 이 프로세스에 시간이 걸릴 수 있다.
    - 새로운 인스턴스에서 제공되는 첫 번째 요청은 나머지 인스턴스보다 지연 시간이 더 길다.
- Provisioned Concurrency
    - 동시성은 함수가 호출되기 전에 할당된다.(미리)
    - 따라서 Cold Start가 발생하지 않으며 모든 호출의 지연 시간이 짧다.
    - Application Auto Scaling은 동시성(일정 또는 목표 활용도)을 관리할 수 있다.
- 2019년 10월/11월에 VPC의 Cold Start가 크게 감소하였다.

![6-reserved-provisioned-concurrency.png](images%2F6-reserved-provisioned-concurrency.png)

- 예약된 동시성과 프로비저닝된 동시성의 개념을 살펴볼 수 있다.
- 자세한 내용은 [공식 홈페이지](https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html)를 참고하도록 한다.

#### File Systems Mounting

- 람다 함수는 VPC에서 실행 중인 경우 EFS 파일 시스템에 액세스할 수 있다.
- 초기화 중에 EFS 파일 시스템을 로컬 디렉토리에 마운트하도록 람다를 구성한다.
- EFS 액세스 포인트를 활용해야 한다.
- EFS 연결 제한(하나의 함수 인스턴스 = 하나의 연결) 및 연결 버스트 제한에 주의해야 한다. 

![7-file-systems-mounting.png](images%2F7-file-systems-mounting.png)

- 람다는 여러 종류의 스토리지를 사용할 수 있으므로 상황에 맞는 스토리지를 사용해야 한다.

![8-storage-options.png](images%2F8-storage-options.png)

#### Cross-Account EFS Mounting

![9-cross-account-efs-mounting.png](images%2F9-cross-account-efs-mounting.png)

- 교차 계정을 사용하는 경우에도 EFS를 마운트할 수 있다.
- 파일 시스템에 대한 액세스 포인트가 계정 B의 VPC B에 있고 람다 함수는 계정 A의 VPC A에 있다.
- 람다 함수에게 액세스 권한을 부여하려면, 먼저 VPC Peering을 설정해야 한다.
- VPC Peering이 설정되면 람다 함수에 충분한 권한이 있는지 확인해야 한다.
- EFS 파일 시스템을 설명할 권한 및 대상을 마운트하기 위해 액세스 포인트를 찾는 권한 등이 있는지 확인해야 한다.
- 계정 B의 EFS 파일 시스템에는 EFS 파일 시스템 정책이 있다.
- 클라이언트가 다른 계정에서 마운트하고 쓸 수 있도록 해준다.
- "Principal"은 계정 A를 가르키고 있어야 한다.

---

### 참고한 강의

- [DevOps Engineer Professional](https://www.udemy.com/course/aws-certified-devops-engineer-professional-korean)
- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)