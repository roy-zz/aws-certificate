# High Availability & Scalability

이번 장에서는 **SysOps Administrator**를 준비하며 **고가용성과 확장성**에 대해서 알아보도록 한다.

---

### 고가용성과 확장성(High Availability & Scalability)

- Load Balancers
  - 트러블슈팅
  - 고급 옵션 및 로깅
  - "CloudWatch"와 통합
- Auto Scaling
  - 트러블슈팅
  - 고급 옵션 및 로깅
  - "CloudWatch"와 통합

- 확장성이란 애플리케이션/시스템이 적응을 통해 더 큰 부하를 처리할 수 있음을 의미한다.
- 확장성에는 두 가지 종류가 있다.
  - 수직 확장성(Vertical Scalability)
  - 수평 확장성(Horizontal Scalability, 탄력성(Elasticity)과 동일한 의미)
- 확장성과 고가용성은 연결되어 있지만 동일하다고 볼 수는 없다.

#### Vertical Scalability

- 수직 확장성은 인스턴스의 크기를 늘리는 것을 의미한다.
- 예를 들어, `t2.micro`에서 실행되는 애플리케이션을 수직 확장하여 `t2.large`에서 실행되도록 변경한다.
- 수직 확장성은 데이터베이스와 같은 비분산 시스템에서 매우 일반적이다.
- RDS, ElastiCache는 수직 확장이 가능한 서비스다.
- 일반적으로 하드웨어 제한으로 인해 수직으로 확장하는 정도에는 제약이 있다.

#### Horizontal Scalability

- 수평 확장성은 애플리케이션의 인스턴스/시스템 수를 늘리는 것을 의미한다.
- 수평 확장은 분산 시스템을 의미한다.
- 수평 확장은 일반적인 웹 애플리케이션이나 최신 애플리케이션에서 매우 일반적이다.
- EC2 인스턴스와 같은 클라우드 서비스를 통해 손쉽게 수평 확장을 할 수 있다.

#### High Availability

- 고가용성은 일반적으로 수평 확장과 함께 진행된다.
- 고가용성은 최소 2개의 데이터 센터(가용 영역)에서 애플리케이션/시스템을 실행하는 것을 의미한다.
- 고가용성의 목표는 데이터 센터 손실을 극복하는 것이다.
- 고가용성은 수동적일 수 있다.(예를 들어, RDS Multi AZ)
- 수평 확장의 경우 고가용성을 활성화할 수 있다.

#### EC2 인스턴스

- 수직 확장: 인스턴스 크기 증가(= scale up / down)
  - `t2.nano`(0.5GiB, 1vCPU) -> `u-12tb1.metal`(12.3TiB, 448vCPU)
- 수평 확장: 인스턴스 수 증가(= scale out / in)
  - Auto Scaling Group
  - Load Balancer
- 고가용성: 다중 AZ에서 동일한 애플리케이션에 대한 인스턴스 실행
  - Auto Scaling 그룹 multi-AZ
  - Load Balancer multi-AZ

---

### Load Balancing

- 로드 밸런서는 트래픽을 여러 서버(예를 들어, EC2 인스턴스) 다운스트림으로 전달하는 서버다.

![1-load-balancing.png](images%2F1-load-balancing.png)

- "Load Balancer"는 아래와 같은 이유로 사용된다.
  - 여러 다운스트림 인스턴스에 부하를 분산시킬 수 있다.
  - 애플리케이션에 단일 액세스 지점(DNS)을 노출한다.
  - 다운스트림 인스턴스의 오류를 원활하게 처리한다.
  - 인스턴스에 대한 정기적인 상태 점검을 수행한다.
  - 웹사이트에 SSL 종료(HTTPS)를 제공한다.
  - 쿠키에 대한 고정성을 강화한다.(Sticky-Session)
  - 영역 전반에 걸친 고가용성을 제공한다.
  - Public 트래픽과 Private 트래픽을 분리한다.

#### Elastic Load Balancer

- "Elastic Load Balancer"는 관리형 로드 밸런서다.
  - AWS는 작동을 보장한다.
  - AWS는 업그레이드, 유지 관리, 고가용성ㅇ르 담당한다.
  - AWS는 몇 가지 구성 손잡이만 제공한다.
- 자체 로드 밸런서를 설정하는 데 비용은 적게 들지만, 작업에는 훨씬 더 많은 노력이 필요하다.
- 다양한 AWS 제품/서비스와 통합된다.
  - EC2, EC2 Auto Scaling Groups, ECS
  - AWS Certificate Manager(ACM), CloudWatch
  - Route 53, AWS WAF, AWS Global Accelerator

#### Health Checks

- "상태 확인"(Health Checks)은 로드 밸런서에서 매우 중요한 기능이다.
- 로드 밸런서는 트래픽을 전달하는 인스턴스가 요청에 응답할 수 있는지 여부를 알 수 있다.
- 상태 확인은 포트와 경로에 대해 수행된다. (`/health`는 공통)
- 응답이 200(OK)이 아니면 인스턴스는 비정상인 것이다.

![2-health-checks.png](images%2F2-health-checks.png)

#### Load Balancer 유형

- AWS에는 4가지 종류의 관리형 로드 밸런서가 있다.
- **Classic Load Balancer(CLB)**: 2009년 출시
  - HTTP, HTTPS, TCP, SSL을 지원한다.
- **Application Load Balancer(ALB)**: 2016년 출시
  - HTTP, HTTPS, WebSocket을 지원한다.
- **Network Load Balancer(NLB)**: 2017년 출시
  - TCP, TLS, UDP를 지원한다.
- **Gateway Load Balancer(GWLB)**: 2020년 출시
  - Layer 3(Network layer)에서 작동한다. (IP Protocol)
- 전반적으로 더 많은 기능을 제공하는 최신 세대 로드 밸런서를 사용하는 것이 좋다.
- 일부 로드 밸런서는 Private 또는 Public ELB로 설정될 수 있다.

#### Security Group

- 애플리케이션의 보안 그룹을 보면 로드 밸런서의 보안 그룹이 Source로 지정된 것을 확인할 수 있다.
  이렇게 애플리케이션 보안 그룹이 설정되는 경우 Source로 지정된 로드 밸런서의 트래픽을 허용한다.

![3-load-balancer-security-group.png](images%2F3-load-balancer-security-group.png)

---

### Classic Load Balancers (v1)

- TCP (Layer 4), HTTP & HTTPS (Layer 7)을 지원한다.
- TCP 또는 HTTP 기반의 상태 확인을 사용한다.
- `XXX.region.elb.amazonaws.com`과 같이 고정된 호스트 이름을 가진다.

![4-classic-load-balancer.png](images%2F4-classic-load-balancer.png)

---

### Application Load Balancers (v2)

- 애플리케이션 로드 밸런서는 HTTP(Layer 7)에서 작동한다.
- 시스템 전체(target group)에 걸쳐 여러 HTTP 애플리케이션에 대한 로드 밸런싱을 지원한다.
- 동일한 시스템의 여러 애플리케이션(예: 컨테이너)에 대한 로드 밸런싱을 지원한다.
- HTTP/2와 WebSocket을 지원한다.
- HTTP 요청을 HTTPS로 전달하는 것과 같은 리디렉션을 지원한다.
- 다양한 대상 그룹으로의 라우팅 테이블이 있다.
  - URL 경로 기반의 라우팅(`example.com/users` & `example.com/posts`)
  - URL의 호스트 이름을 기반으로 하는 라우팅(`one.example.com` 및 `other.example.com`)
  - 쿼리 스트링, 헤더 기반의 라우팅(`example.com/users?id=123&order=false`)
- ALB는 마이크로서비스 및 컨테이너 기반의 애플리케이션에 매우 적합하다.(예를 들어, Docker 및 Amazon ECS)
  - ECS의 동적 포트로 리디렉션하는 포트 매핑을 지원한다.
  - 만약 "Classic Load Balancer"를 사용한다면 애플리케이션마다 고유한 로드 밸런서를 사용해야 한다.

#### HTTP 기반 트래픽

![5-alb-http-based-traffic.png](images%2F5-alb-http-based-traffic.png)

- `/user` 경로의 요청을 처리하기 위한 대상 그룹이 있고 `/search` 경로의 요청을 처리하기 위한 대상 그룹이 있다.
- 하나의 ALB에서 여러개의 대상 그룹으로 현명하게 라우팅을 설정할 수 있다.

#### Target Group

- 여러 종류의 리소스가 "Target Group"이 될 수 있다.
  - EC2 인스턴스 ("Auto Scaling Group"에서 관리될 수 있음) - HTTP
  - ECS 태스크 (ECS에 의해 관리됨) - HTTP
  - Lambda Function - HTTP 요청이 JSON 이벤트로 변환된다.
  - IP Address - Private IP로만 요청을 전달할 수 있다.
- ALB는 여러개의 대상 그룹으로 라우팅하도록 설정할 수 있다.
- 상태 점검은 대상 그룹 수준에서 수행된다.

#### Query String & Parameter 라우팅

![6-alb-query-string-parameter-routing.png](images%2F6-alb-query-string-parameter-routing.png)

- 하나의 애플리케이션에서 Query String의 값에 따라 트래픽을 라우팅할 수 있다.
- 이미지의 경우 `Platform=Mobile`인 경우 EC2 기반의 대상 그룹으로 라우팅한다.
- `Platform=Desktop`인 경우 Private IP를 사용하여 On-Premise 기반의 대상 그룹으로 트래픽을 라우팅한다.

#### ALB 체크 사항

![7-good-to-know.png](images%2F7-good-to-know.png)

- 고정된 호스트 이름을 사용한다.(`XXX.region.elb.amazonaws.com`)
- 애플리케이션 서버는 클라이언트의 IP를 직접 볼 수 없다.
  - 클라이언트의 실제 IP가 `X-Forwarded-For` 헤더에 삽입된다.
  - 포트(`X-Forwarded-Port`)와 프로토(`X-Forwarded-Proto`)를 얻을 수 있다.

---

### Network Load Balancer (v2)

- 네트워크 로드 밸런서(Layer 4)를 통해 아래의 기능들이 가능해진다.
  - **TCP 및 UDP 트래픽을 인스턴스로 전달한다.**
  - 초당 수백만 건의 요청을 처리한다.
  - 짧은 지연 시간 ~ 100ms을 지원한다. ALB의 경우 ~ 400ms를 지원한다.
- **NLB는 AZ당 하나의 고정 IP를 가지며 탄력적 IP 할당을 지원한다.** 
  - 특정 IP를 화이트리스트에 추가하는 데 유용하게 사용된다.
- NLB는 최고의 성능이 필요하거나, TCP 또는 UDP 트래픽을 라우팅해야 할 때 사용된다.
- AWS 프리 티어에는 포함되지 않는다.

#### TCP 기반 트래픽

![8-nlb-tcp-based-traffic.png](images%2F8-nlb-tcp-based-traffic.png)

- 대상 그룹을 생성하면 NLB가 리디렉션한다.
- 예를 들어, TCP 트래픽을 사용할 수 있고 HTTP를 사용하는 백엔드 애플리케이션에서도 사용할 수 있다.

#### Target Group

![9-nlb-target-group.png](images%2F9-nlb-target-group.png)

- 대상 그룹은 EC2 인스턴스가 될 수 있다. NLB가 EC2 인스턴스로 리디렉션해서 TCP 또는 UDP 트래픽을 보낼 수 있다.
- EC2 인스턴스의 Private IP나 On-Premise 서버의 Private IP로 트래픽을 라우팅할 수 있다.
- ALB를 대상 그룹으로 설정할 수 있다. ALB에 고정 IP를 사용하고 싶은 경우에 사용되는 조합이다.
- **TCP, HTTP, HTTPS의 상태 확인을 제공한다.**
  - 백엔드 애플리케이션에서 HTTP나 HTTPS 프로토콜을 사용하는 경우에도 상태 확인이 가능하다.

---

### Gateway Load Balancer

- AWS에서 타사 네트워크 가상 어플라이언스를 배포, 확장 및 관리한다.
- 예를 들어, 방화벽, 침입 탐지 및 예방 시스템, 심층 패킷 검사 시스템, 페이로드 조작 등
- 네트워크 레이어(Layer 3)에서 작동한다 - IP 패킷
- 아래의 기능을 결합한다.
  - 투명한 네트워크 게이트웨이: 모든 트래픽에 대한 단일 진입/출구
  - 로드 밸런서: 가상 어플라이언스에 트래픽을 분산한다.
  - 포트 6081에서 GENEVE 프로토콜을 사용한다.

![10-gateway-load-balancer.png](images%2F10-gateway-load-balancer.png)

#### Target Groups

- Gateway Load Balancer의 대상 그룹은 EC2 인스턴스나, Private IP 주소가 될 수 있다.

![11-gateway-load-balancer-target-group.png](images%2F11-gateway-load-balancer-target-group.png)

---

### Sticky Sessions (세션 선호도(Affinity))

- 동일한 클라이언트가 항상 로드 밸런서 뒤의 동일한 인스턴스로 리디렉션되도록 고정성을 구현할 수 있다.
- 이는 CLB, ALB, NLB에서 작동한다.
- CLB 및 ALB 모두 "Stickiness"에 사용되는 "쿠키"의 만료일을 사용자가 제어할 수 있다.
- 사용자가 세션 데이터를 잃지 않도록 하는데 사용된다.
- "Stickiness"를 활성화하면 백엔드 EC2 인스턴스에 대한 로드 불균형이 발생할 수 있다.

![12-sticky-sessions.png](images%2F12-sticky-sessions.png)

#### Cookie Names

- "Sticky Session"에 사용되는 쿠키는 두 가지가 있다.
- **Application-based Cookies**
  - Custom Cookie
    - 대상(Target)에서 생성된다.
    - 애플리케이션에 필요한 모든 사용자 정의 속성을 포함할 수 있다.
    - "AWSALB", "AWSALBAPP", "AWSALBTG"는 EBL에서 사용해야 하므로 사용자가 직접 상요해서는 안된다.
  - Application Cookie
    - 로드 밸런서에 의해 생성된다.
    - 쿠키 이름은 "AWSALBAPP"이다.
- **Duration-based Cookies**
  - 로드 밸런서가 생성한 쿠키
  - 쿠키 이름은 ALB의 경우 "AWSALB", CLB의 경우 "AWSELB"다.

---

### Cross-Zone Load Balancing

- 교차 영역 로드 밸런싱 사용
  - 각 로드 밸런서 인스턴스는 모든 AZ에 등록된 모든 인스턴스에 고르게 트래픽을 분산한다.

![13-with-cross-zone-load-balancing.png](images%2F13-with-cross-zone-load-balancing.png)

- 교차 영역 로드 밸런싱 미사용
  - 요청은 ELB노드의 인스턴스에 분산된다.

![14-without-cross-zone-load-balancing.png](images%2F14-without-cross-zone-load-balancing.png)

- Application Load Balancer
  - 기본적으로 활성화된다. (대상 그룹 수준에서 비활성화할 수 있음)
  - AZ간 데이터 전송에 대해서는 요금이 부과되지 않는다.
- Network Load Balancer & Gateway Load Balancer
  - 기본적으로 비활성화된다.
  - 활성화된 경우 AZ 간 데이터에 대한 요금을 지불한다.
- Classic Load Balancer
  - 기본적으로 비활성화되어 있다.
  - 활성화된 경우 AZ 간 데이터 전송에 대한 요금이 부과되지 않는다.

---

### SSL/TLS

- SSL 인증서를 사용하면 클라이언트와 로드 밸런서 간의 트래픽을 전송 중에 암호화할 수 있다.
- SSL은 연결을 암호화하는 데 사용되는 SSL(Secure Sockets Layer)을 의미한다.
- TLS는 최신 버전인 "Transport Layer Security"를 의미한다.
- 요즘에는 TLS 인증서를 주로 사용하지만 여전히 SSL이라고 부르는 사람들이 많이 있다.
- 공개 SSL 인증서는 인증 기관(CA)에서 발급된다.
- Comodo, Symantec, GoDaddy, GlobalSign, Digicert, Letsencrypt 등이 있다.
- SSL 인증서에는 만료일이 있으며 갱신해야 한다.

#### Load Balancer - SSL Certificates

![15-load-balancer-ssl-certificates.png](images%2F15-load-balancer-ssl-certificates.png)

- 로드 밸런서는 X.509 인증서(SSL/TLS 서버 인증서)를 사용한다.
- ACM(AWS Certificate Manager)를 사용하여 인증서를 관리할 수 있다.
- 대안으로 자신의 인증서를 업로드할 수 있다.
- HTTPS 리스너:
  - 기본 인증서를 지정해야 한다.
  - 여러 도메인을 지원하기 위해 선택적 인증서 목록을 추가할 수 있다.
  - **클라이언트는 SNI(Server Name Indication)를 사용하여 도달하는 호스트 이름을 지정할 수 있다.**
  - 이전 버전의 SSL/TLS(레거시 클라이언트)를 지원하는 보안 정책을 지원하는 기능을 제공한다.

#### Server Name Indication (SNI)

- SNI는 여러 SSL 인증서를 하나의 웹 서버에 로드하는 문제를 해결한다.(여러 웹사이트를 제공하기 위해)
- 이는 "최신" 프로토콜이며 클라이언트가 초기 "SSL Handshake"에서 대상 서버의 호스트 이름을 나타내도록 요구한다.
- 그러면 서버는 올바른 인증서를 찾거나 기본 인증서를 반환한다.
- ALB 및 NLB, CloudFront에서만 작동한다.
- CLB에서는 작동하지 않는다.

![16-server-name-indication.png](images%2F16-server-name-indication.png)

- Classic Load Balancer (v1)
  - 하나의 SSL 인증서만 지원한다.
  - 여러 SSL 인증서가 있는 여러 호스트 이름에 대해 여러 CLB를 사용해야 한다.
- Application Load Balancer (v2)
  - 여러 SSL 인증서로 여러 리스너를 지원한다.
  - SNI(Server Name Indication)를 사용하여 작동한다.
- Network Load Balancer (v2)
  - 여러 SSL 인증서로 여러 리스너를 지원한다.
  - SNI를 사용하여 작동시킨다.

---

### Connection Draining

- Feature naming
  - Connection Draining: CLB에서 사용되는 이름이다.
  - Deregistration Delay: ALB와 NLB에서 사용되는 이름이다.
- 인스턴스가 등록 취소되거나 비정상인 동안 "진행 중인 요청"을 완료하는 데 걸리는 시간이다.
- 등록 취소 중인 EC2 인스턴스에 대한 새 요청 전송을 중지한다.
- 1 ~ 3,600초 사이의 시간을 선택할 수 있으며 기본값은 300이다.
- 값을 0으로 설정하여 비활성화할 수 있다.
- 요청이 짧은 경우에는 낮은 값으로 설정하는 것이 좋다.

![17-connection-draining.png](images%2F17-connection-draining.png)

--- 

### Health Check

- 아래는 대상(Target)의 건강 상태 목록이다.
  - Initial: 대상 등록
  - Healthy
  - Unhealthy
  - Unused: 대상이 등록되지 않은 경우
  - Draining: 대상 등록이 취소 중인 경우
  - Unavailable: 상태 확인이 비활성화된 경우
- **대상 그룹에 비정상 대상만 포함된 경우 ELB는 비정상 대상 전체로 요청을 라우팅한다.**

![18-health-check.png](images%2F18-health-check.png)

#### Error Code

- 요청 성공: 200
- 클라이언트 측에서 실패: 4XX
  - 400: Bad Request
  - 401: Unauthorized
  - 403: Forbidden
  - 460: Client closed connection
  - 463: IP가 30보다 큰 헤더의 경우 `X-Forwarded`
- 내부 서버 오류: 5XX
  - 500: 내부 서버 오류는 ELB 자체에 오류가 있음을 의미한다.
  - 502: Bad Gateway
  - 503: Service Unavailable
  - 504: Gateway timeout (서버 내부 문제일 수 있음)
  - 561: Unauthorized

---

### Load Balancer Monitoring

- 모든 로드 밸런서 지표는 CloudWatch 지표로 직접 푸시된다.
  - BackendConnectionErrors
  - HealthyHostCount / UnHealthyHostCount
  - HTTPCode_Backend_2XX: 성공적인 요청
  - HTTPCode_Backend_3XX: 리디렉션 요청
  - HTTPCode_ELB_4XX: 클라이언트 오류
  - HTTPCode_ELB_5XX: 로드 밸런서에 의해 생성된 서버 오류 코드
  - Latency
  - RequestCount
  - **RequestCountPerTarget**
  - **SurgeQueueLength**: 정상 인스턴스로의 라우팅을 보류 중인 총 요청(HTTP 리스너) 또는 연결(TCP 리스너)수다. 
    ASG 확장에 도움이 되며 최대값은 1,024다.
  - **SpilloverCount**: 급증 대기열이 가득 차서 거부된 총 요청의 수다.

![19-load-balancer-monitoring.png](images%2F19-load-balancer-monitoring.png)

#### 지표를 활용한 트러블슈팅

- **HTTP 400(Bad Request)**: 클라이언트가 HTTP 사양을 충족하지 않는 잘못된 요청을 보냈다.
- **HTTP 503(Service Unavailable)**: 로드 밸런서가 응답하도록 구성된 모든 가용 영역에 정상적인 인스턴스가 있는지 확인해야 한다.
  CloudWatch에서 HealthyHostCount를 찾는다.
- **HTTP 504(Gateway Timeout)**: EC2 인스턴스의 연결 유지 설정이 활성화되어 있는지 확인하고 연결 유지 시간 초과가 로드 밸런서의 유휴 시간 초과 설정보다 큰지 확인해야 한다.

#### Access Logs

- 로드 밸런서의 액세스 로그는 S3에 저장될 수 있으며 아래의 항목을 포함할 수 있다.
  - Time
  - 클라이언트 IP 주소
  - 지연 시간
  - 요청 경로
  - 서버 응답
  - Trace ID
- S3 스토리지에 대한 비용만 지불한다.
- 규정 준수(Compliance)에 도움이 된다.
- ELB 또는 EC2 인스턴스가 종료된 후에도 액세스 데이터를 유지하는 데 유용하다.
- 액세스 로그는 이미 암호화되어 있다.

#### Request Tracing

- 요청 추적: 각 HTTP 요청에는 추가된 사용자 정의 헤더가 있다. `X-Amzn-Trace-Id`
- 단일 요청을 추적하기 위한 로그/분산 추적 플랫폼에 유용하다.
- Application Load Balancer는 아직 "X-Ray"와 통합되지 않았다.

---

### Target Group 설정

- `deregistration_delay.timeout_seconds`: 로드 밸런서가 대상 등록을 취소하기 전에 기다리는 시간
- `slow_start.duration_seconds`: 새로운 대상에게 요청을 보내기 전까지 제공하는 워밍업 시간
- `load_balancing.algorithm.type`: 요청을 라우팅할 때 로드 밸런서가 대상을 선택하는 방법(Round Robin, Least Outstanding Request)
- `stickiness.enabled`
- `stickiness.type`: application-based, duration-based 쿠키 중에서 설정
- `stickiness.app_cookie.cookie_name`: 애플리케이션 쿠키의 이름
- `stickiness.app_cookie.duration_seconds`: application-based 쿠기의 만료 기간
- `stickiness.lb_cookie.duration_seconds`: duration-based 쿠기의 만료 기간

#### Slow Start Mode

- 기본적으로 대상은 대상 그룹에 등록되면 요청의 전체 공유를 받는다.
- "Slow Start Mode"는 로드 밸런서가 전체 요청 공유를 보내기 전에 정상적인 대상에게 워밍업 시간을 제공한다.
- 로드 밸런서는 대상으로 보내는 요청 수를 선형적으로 늘린다.
- 대상은 아래와 같은 경우 "Slow Start Mode"를 종료한다.
  - 기간이 경과된 경우
  - 대상이 건강하지 않은 경우
- 비활성화하려면 "Slow Start Duration"값을 0으로 설정하면 된다.

![20-slow-start-mode.png](images%2F20-slow-start-mode.png)

#### Least Outstanding Request

- 요청을 수신하는 다음 인스턴스는 보류/완료되지 않은 요청 수가 가장 적은 인스턴스다.
- "ALB", "CLB"와 함께 작동된다. (HTTP/HTTPS)

![21-least-outstanding-request.png](images%2F21-least-outstanding-request.png)

#### Round Robin

- 대상 그룹에서 대상을 동일하게 선택한다.
- "ALB", "CLB"와 함께 작동된다. (TCP)

![22-round-robin.png](images%2F22-round-robin.png)

#### Flow Hash

- 프로토콜, 소스/대상 IP 주소, 소스/대상 포트 및 TCP 시퀀스 번호를 기반으로 대상을 선택한다.
- 각 TCP/UDP 연결은 연결 수명 동안 단일 대상으로 라우팅된다.
- "NLB"와 함께 작동된다.

![23-flow-hash.png](images%2F23-flow-hash.png)

---

### ALB - Listener Rule

- 순서대로 처리된다. (기본 규칙 사용)
- 지원되는 작업(forward, redirect, fixed-response)
- 규칙 조건
  - host-header
  - http-request-method
  - path-pattern
  - source-ip
  - http-header
  - query-string

![24-alb-listener-rule.png](images%2F24-alb-listener-rule.png)

#### Target Group 가중치

- 단일 규칙에 대해 각 대상 그룹에 대한 가중치를 지정한다.
- 예를 들어, 여러 버전의 앱, 블루/그린 배포에 사용한다.
- 애플리케이션에 대한 트래픽 배포를 제어할 수 있다.

![25-target-group-weighting.png](images%2F25-target-group-weighting.png)

---

### Auto Scaling Group

- 실제 웹사이트와 애플리케이션의 부하는 변경될 수 있다.
- 클라우드에서는 매우 빠르게 서버를 생성하고 제거할 수 있다.
- ASG(Auto Scaling Group)의 목표는 아래와 같다.
  - 증가된 부하에 맞게 확장(EC2 인스턴스 추가)
  - 감소된 부하에 맞게 축소(EC2 인스턴스 제거)
  - 실행 중인 EC2 인스턴스의 최소 및 최대 개수를 확인해야 한다.
  - 로드 밸런서에 새 인스턴스를 자동으로 등록한다.
  - 이전 인스턴스가 종료된 경우(비정상인 경우) EC2 인스턴스를 다시 생성한다.
- ASG는 무료로 사용할 수 있으며, 사용한 EC2 인스턴스에 대한 비용만 지불하면 된다.

![26-auto-scaling-group.png](images%2F26-auto-scaling-group.png)

- Minimum Capacity: 최소한의 인스턴스 수를 설정한다. 예시에서는 2로 설정되어 있다.
- Desired Capacity: 원하는 인스턴스 수를 설정한다. 예시에서는 4로 설정되어 있다.
- Maximum Capacity: 최대 인스턴스 수를 설정한다. 예시에서는 7로 설정되어 있다.

![27-auto-scaling-group-load-balancer.png](images%2F27-auto-scaling-group-load-balancer.png)

- ASG는 ELB와 통합되어 사용될 수 있다.

#### Auto Scaling Group Attributes

- 시작 템플릿("Launch Configuration"은 더 이상 사용되지 않음)은 아래의 속성을 가지고 있다.
  - AMI + Instance Type
  - EC2 User Data
  - EBS Volume
  - Security Group
  - SSH Key Pair
  - IAM Role
  - Network, Subnet 정보
  - 로드 밸런서 정보
- 최소, 최대, 초기 용량을 설정할 수 있다.
- 확장 정책을 설정할 수 있다.

![28-auto-scaling-group-attributes.png](images%2F28-auto-scaling-group-attributes.png)

#### CloudWatch Alarms & Scaling

- CloudWatch 경보를 기반으로 ASG를 확장할 수 있다.
- 경보는 지표(예: 평균 CPU 사용량)를 모니터링한다.
- 평균 CPU와 같은 지표는 전체 ASG 인스턴스에 대해 계산된다.
- 알람 기준
  - 확장 정책을 생성할 수 있다.(인스턴스 수 증가)
  - 축소 정책을 생성할 수 있다.(인스턴스 수 감소)

![29-cloudwatch-alarms-scaling.png](images%2F29-cloudwatch-alarms-scaling.png)

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

![30-predictive-scaling.png](images%2F30-predictive-scaling.png)

#### 스케일링에 사용되는 좋은 지표

- **CPUUtilization**: 인스턴스 전체의 평균 CPU 사용률을 확인한다.
- **RequestCountPerTarget**: EC2 인스턴스당 요청 수가 안정적인지 확인한다.

![31-good-metric-to-scaling.png](images%2F31-good-metric-to-scaling.png)

- **Average Network In / Out**: 응용 프로그램이 네트워크 바인딩된 경우, 많은 업로드와 다운로드가 있다면 인스턴스의 병목현상이 발견될 수 있다.
  평균 네트워크를 스케일아웃하여 어떤 임계값에 도달하면 그에 기반해 스케일아웃한다.
- CloudWatch를 통해 푸시된 커스텀 매트릭

#### Scaling Cooldown

- 조정 활동이 발생한 후에는 "Cooldown"(기본값: 300초) 기간이 된다.
- "Cooldown"기간 동안 ASG는 추가 인스턴스를 시작하거나 종료하지 않는다. (메트릭의 안정화를 위해)

![32-scaling-cooldowns.png](images%2F32-scaling-cooldowns.png)

- 요청을 더 빠르게 처리하고 "Cooldown" 기간을 줄이기 위해 즉시 사용 가능한 AMI를 사용하여 구성시간을 줄일 수 있다.
- 물론 ASG에 대한 상세 모니터링이 가능하도록 해야한다. 1분마다 메트릭이 충분히 업데이트 될 수 있도록 설정한다.

---

### ASG Lifecycle Hook

![33-lifecycle-hook.png](images%2F33-lifecycle-hook.png)

- 기본적으로 ASG에서 인스턴스가 시작되자마자 서비스가 시작된다.
- 인스턴스가 서비스되기 전에 추가 단계를 수행할 수 있다.(Pending 상태)
  - 인스턴스가 시작될 때 실행할 스크립트를 정의한다.
- 인스턴스가 종료되기 전에 몇 가지 작업을 수행할 수 있다.(Terminating 상태)
  - 문제 해결을 위해 인스턴스가 종료되기 전에 일시 중지한다.
- 인스턴스가 시작되어 작동하기 전에 청소, 로그 추출 혹은 특별한 상태 확인 등의 작업에 사용된다.
- EventBridge, SNS, SQS와 통합되어 사용될 수 있다.

#### Launch Configuration vs Launch Template

- 시작 구성(Launch Configuration)과 시작 템플릿 아래와 같은 공통된 속성을 갖는다.
  - AMI의 ID, 인스턴스 유형, Key Pair, 보안 그룹 및 기타 EC2 인스턴스를 시작하는 데 사용되는 매개변수(태그, EC2 User Data)
  - 시작 구성과 시작 템플릿은 수정할 수 없으므로 변경이 필요한 경우 새롭게 생성해야 한다.
- **시작 구성**은 레거시다.
  - 변경 사항이 있는 경우 매번 다시 생성해야 한다.
- **시작 템플릿**은 최신 버전이다.
  - 여러 버전이 있을 수 있다.
  - 매개변수 하위 집합을 생성할 수 있다.(재사용 및 상속을 위한 부분 구성)
  - 온디맨드 및 스팟 인스턴스(혼합 가능)를 모두 사용하여 프로비저닝할 수 있다.
  - 배치 그룹, 용량 예약, 전용 지원 호스트, 여러 인스턴스 유형을 지원한다.
  - T2 무제한 버스트 기능을 사용할 수 있다.
  - AWS에서 사용을 권장하고 있다.

#### SQS & ASG

![34-sqs-with-asg.png](images%2F34-sqs-with-asg.png)

- SQS와 다수의 EC2 인스턴스가 메시지를 처리하고 있다.
- SQS의 큐에 메시지가 얼마나 있는지 모니터랑하여 CloudWatch 메트릭을 만들 수 있다.
- 큐의 길이가 길면 처리해야 할 메시지가 많다는 의미이므로 알람을 생성하여 ASG 스케일링 정책이 실행되도록 할 수 있다.

#### ASG Health Check

- 고가용성을 확보하려면 ASG의 2개의 AZ에서 실행되는 인스턴스가 2개 이상 있어야 한다.(다중 AZ ASG를 구성해야 함)
- 아래의 항목에 대해서 상태 확인이 가능하다.
  - EC2 인스턴스 상태 확인
  - ELB 상태 확인
  - 사용자 정의 상태 확인: AWS CLI 또는 AWS SDK를 사용하여 인스턴스 상태를 ASG로 보낸다.
- ASG는 비정상 인스턴스를 종료한 후 새 인스턴스를 시작한다.
- ASG는 비정상 호스트를 재부팅하지 않는다.
- 아래의 CLI는 자주 사용된다.
  - `set-instance-health` (사용자 정의 상태 확인과 함께 사용)
  - `terminate-instance-in-auto-scaling-group`

#### Troubleshooting

- `<number of instances>` 개의 인스턴스가 이미 실행 중이다. EC2 인스턴스 실행에 실패한다.
  - ASG가 `MaximumCapacity`로 설정된 한도에 도달한다.
  - 매개 변수의 최대 용량 값을 재설정하여 ASG를 업데이트 해야한다.
- EC2 인스턴스 실행에 실패한다.
  - 보안 그룹이 존재하지 않는 경우가 있을 수 있다. 보안 그룹이 삭제된 경우에 발생할 수 있다.
  - Key Pair가 존재하지 않는 경우가 있을 수 있다. Key Pair가 삭제된 경우에 발생할 수 있다.
- ASG가 24시간 이상 인스턴스를 시작하지 못하면 자동으로 프로세스를 정지한다. (관리 정지)

---

### CloudWatch Metric

- ASG에 대한 중요한 CloudWatch 지표가 있다.
- 지표는 1분마다 수집되며 아래는 ASG 수준의 지표 목록이다. (선택 사항)
  - GroupMinSize, GroupMaxSize, GroupDesiredCapacity
  - GroupInServiceInstances, GroupPendingInstances, GroupStandbyInstances
  - GroupTerminatingInstances, GroupTotalInstances
  - 이러한 지표를 보려면 지표 수집을 활성화해야 한다.
- EC2 수준의 메트릭이 있으며 기본적으로 활성화되어 있다.
  - CPU 사용률, 네트워크 In/Out 등이 있다.
  - 기본 모니터링은 5분 단위로 수집하며 세부 모니터링을 활성화하여 1분 단위로 지표를 수집할 수 있다.

---

### AWS Auto Scaling

- AWS의 확장 가능한 리소스를 위한 자동 확장 백본 서비스다.
- "Amazon EC2 Auto Scaling Group": EC2 인스턴스를 실행하거나 종료한다.
- "Amazon EC2 Spot Fleet Request": 스팟 집합 요청에서 인스턴스를 시작 또는 종료하거나 가격 또는 용량 문제로 인해 중단된 인스턴스를 자동으로 교체한다.
- "Amazon ECS": 원하는 ECS 서비스 수를 늘리거나 줄인다.
- "Amazon DynamoDB(Table 또는 Global secondary index)": WCU & RCU
- "Amazon Aurora": 동적 읽기 전용 복제본을 확장하거나 축소한다.

#### Scaling Plan

- 두 개의 Scaling Plan이 있다.

![35-dynamic-scaling-plan.png](images%2F35-dynamic-scaling-plan.png)

- **Dynamic Scaling**: 대상 추적 조정 저책을 생성한다.
  - 가용성 최적화: 리소스 활용도 40%
  - 가용성과 비용의 균형: 리소스 활용도 50%
  - 비용 최적화: 리소스 활용도 70%
  - AWS가 제공하는 권고 사항을 따르고 싶지 않다면 자신만의 목표 값을 설정할 수 있다.
  - "Scale Out"만 가능하며 "Scale In"은 지원하지 않는다.
  - Cooldown Period, Warmup Time을 설정할 수 있다.

![36-predictive-scaling-plan.png](images%2F36-predictive-scaling-plan.png)

- **Predictive Scaling**: 지속적으로 부하를 예측하고 사전 확장을 예약한다.
  - AWS가 작업한 머신러닝 알고리즘을 이용해서 이전의 부하를 분석하고 예측이 생성되고 예측에 기반해 추가 스케줄이 자동으로 처리된다.

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [에러 코드를 활용한 트러블슈팅](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/ts-elb-error-message.html)