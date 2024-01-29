# Amazon Route 53

- 이번 장에서는 **DevOps Engineer Professional (DOP)** 을 준비하며 "AWS의 DNS 서비스인 Route53"에 대해서 알아보도록 한다.

---

### Route 53

- 가용성과 확장성이 뛰어나고 완벽하게 관리되며 신뢰할 수 있는 DNS다.
  - 권한 있음(Authoritative): 고객이 DNS 레코드를 업데이트할 수 있다.
- Route 53은 도메인 등록자(Domain Registrar)이기도 하다.
- 리소스의 상태를 확인하는 기능을 제공한다.
- 100% 가용성 SLA를 제공하는 유일한 AWS 서비스다.
- Route 53에서 53은 전통적인 DNS 포트를 의미한다.

![1-route53.png](images%2F1-route53.png)

#### Record

- 도메인의 트래픽을 라우팅하려는 방법이다.
- 각각의 레코드에는 아래의 항목이 포함된다.
  - Domain/Subdomain Name: `example.com`
  - Record Type: A 또는 AAAA
  - Value: `12.34.56.78`
  - Routing Policy: Route 53이 쿼리에 응답하는 방법
  - TTL: DNS 확인자가 레코드에 캐시된 시간
- Route 53은 다음과 같은 DNS 레코드 유형을 지원한다.
  - A / AAAA / CNAME / NS
  - CAA / DS / MX / NAPTR / PTR / SOA / TXT / SPF / SRV

#### Record Type

- A: 호스트 이름을 IPv4에 매핑한다.
- AAAA: 호스트 이름을 IPv6에 매핑한다.
- CNAME: 호스트 이름을 다른 호스트 이름에 매핑한다.
  - 대상은 A 또는 AAAA 레코드가 있어야 하는 도메인 이름이다.
  - DNS 네임스페이스의 최상위 노드에 대한 CNAME 레코드를 생성할 수 없다.
  - 예를 들어, `example.com`에 대해서는 생성할 수 없지만 `www.example.com`에 대해서는 생성할 수 있다.
- NS: 호스팅 영역을 위한 이름 서버다.
- 도메인에 대한 트래픽 라우팅 방법을 제어한다.

#### Hosted Zone

- 트래픽을 도메인 및 하위 도메인으로 라우팅하는 방법을 정의하는 레코드용 컨테이너다.
- **Public Hosted Zone**: 인터넷(공개 도메인 이름)에서 트래픽을 라우팅하는 방법을 지정하는 레코드가 포함되어 있다.
  - 예: `application1.mypublicdomain.com`
- **Private Hosted Zone**: 하나 이상의 VPC(비공개 도메인 이름) 내에서 트래픽을 라우팅하는 방법을 지정하는 레코드가 포함되어 있다.
  - 예: `application1.company.internal`

![2-public-private-hosted-zone.png](images%2F2-public-private-hosted-zone.png)

- "Public Hosted Zone"은 퍼블릭 클라이언트의 쿼리에 응답할 수 있다.
  - 예를 들어, 웹 브라우저에서 `example.com`을 달라고하면 IP를 반환한다.
- "Private Hosted Zone"은 VPC내에 있다.
  - 프라이빗 도메인으로 프라이빗 리소스를 식별할 수 있다.
  - 예를 들어, EC2 인스턴스가 하나 있는데 `webapp.example.internal`는 `api.example.internal`를 식별하고자 한다.
  - 또 다른 EC2 인스턴스는 `database.example.internal`을 식별하려고 한다.
  - "Private Hosted Zone"은 이러한 프라이빗 도메인에 대한 리소스의 IP 주소를 각각의 인스턴스에게 반환해준다.

---

#### Routing Policy - Weighted

- 각 특정 리소스로 이동하는 요청 비율을 제어한다.
- 각 레코드에 상대적 가중치를 할당한다.
  - 트래픽(%) = 특정 레코드의 가중치 / 모든 레코드의 가중치 합
  - 가중치의 합이 100이 될 필요는 없다.
- DNS 레코드는 이름과 유형이 동일해야 한다.
- 상태 확인(Health Check)과 연결될 수 있다.
- 사용 사례: 지역 간 로드 밸런싱, 새로운 애플리케이션 버전 테스트 등..
- **리소스에 대한 트래픽 전송을 중지하려면 레코드에 가중치 0을 할당**하면 된다.
- **모든 레코드의 가중치가 0인 경우 모든 레코드가 동일하게 반환**된다.

![3-routing-policy-weighted.png](images%2F3-routing-policy-weighted.png)

#### Routing Policy - Latency-based

- 사용자와 가까운 지연 시간이 가장 짧은 리소스로 리디렉션한다.
- 사용자의 지연 시간이 중요한 경우 매우 유용하게 사용된다.
- **지연 시간은 사용자와 AWS 리전 간의 트래픽을 기준**으로 한다.
- 상태 점검(Health Check)과 연결될 수 있으며, 장애 조치 기능을 지원한다.

![4-routing-policy-latency.png](images%2F4-routing-policy-latency.png)

#### Failover (Active-Passive)

![5-routing-policy-failover.png](images%2F5-routing-policy-failover.png)

- Primary 인스턴스는 평상시에 사용하는 인스턴스이며 Secondary 인스턴스는 장애 발생 시에 사용하는 인스턴스다.
- 만약 Primary 인스턴스의 상태가 좋지 않다면 Route 53은 자동으로 Secondary 인스턴스로 요청을 보내게 된다.
- 클라이언트가 DNS 요청을 하면 정상으로 판명된 리소스를 자동으로 얻게 된다.

#### Geolocation

- Latency-based 라우팅 정책과는 다르다.
- **사용자의 위치를 기반으로 라우팅**한다.
- 대륙, 국가 또는 미국 주별로 위치를 지정한다. (위치가 중복되는 경우 가장 정확한 위치가 선택)
- 사용자의 위치에 해당하는 항목이 없는 경우를 대비하여 "Default" 레코드를 만들어야 한다.
- 사용 사례: 웹사이트 현지화, 콘텐츠 배포 제한, 로드 밸런싱 등..
- 상태 확인(Health Check)와 연결될 수 있다.

![6-routing-policy-gelocation.png](images%2F6-routing-policy-gelocation.png)

#### Geoproximity

- 사용자 및 리소스의 지리적 위치를 기반으로 리소스로 트래픽을 라우팅한다.
- 정의된 **Bias**를 기반으로 더 많은 트래픽을 리소스로 전환하는 기능을 제공한다.
- 지리적 영역의 크기를 변경하려면 Bias 값을 지정한다.
    - 확장 (1 ~ 99): 리소스에 대한 더 많은 트래픽
    - 축소 (-1 ~ -99): 리소스에 대한 트래픽 감소
- 리소스는 아래와 같다.
    - AWS 리소스 (AWS 리전 지정)
    - 비 AWS 리소스 (위도 및 경도 지정)
- 이 기능을 사용하려면 "Route 53 Traffic Flow"를 사용해야 한다.

![7-routing-policy-geoproximity1.png](images%2F7-routing-policy-geoproximity1.png)

- us-west-1과 us-east-1 모두 Bias가 0으로 맞춰진 경우 미국을 둘로 나눈 라인이 생성된다.
- 해당 라인을 기준으로 좌측 절반은 us-west-1로 요청이 전달되고, 우측 절반은 us-east-1로 요청이 전달된다.

![8-routing-policy-geproximity2.png](images%2F8-routing-policy-geproximity2.png)

- us-east-1의 Biasrk 50으로 맞춰진 경우 좌측으로 편향된 라인이 생성된다.
- 해당 라인을 기준으로 좌측의 사용자는 us-west-1로 요청을 하게되고, 우측의 사용자는 us-east-1로 요청을 하게 된다.
- Bias의 값이 높을수록 더 많은 사용자의 요청을 처리하게 된다.

#### Traffic Flow

- 크고 복잡한 구성에서 기록을 생성하고 유지하는 프로세스를 단순화한다.
- 복잡한 라우팅 결정 트리를 관리하는 시각적 편집기를 지원한다.
- **구성을 트래픽 흐름(Traffic Flow) 정책으로 저장**할 수 있다.
    - 다양한 Route 53 호스팅 영역(다른 도메인 이름)에 적용할 수 있다.
    - 버전 관리를 지원한다.

![9-traffic-flow.png](images%2F9-traffic-flow.png)

#### IP-based Routing

- **라우팅은 클라이언트의 IP 주소를 기반**으로 한다.
- 클라이언트 및 해당 엔드포인트/위치에 대한 CIDR 목록을 제공한다. (사용자 - IP - 엔드포인트 매핑)
- 활용 사례: 성능 최적화, 네트워킹 비용 절감 등..
- 예를 들어, 최종 사용자를 특정 ISP에서 특정 엔드포인트로 라우팅한다.

![10-routing-policy-ipbased-routing.png](images%2F10-routing-policy-ipbased-routing.png)

#### Multi-Value

- 트래픽을 여러 리소스로 라우팅할 때 사용한다.
- Route 53은 여러 값/리소스를 반환한다.
- 상태 확인(Health Check)과 연결 가능한 값을 확인할 수 있다.
  정상으로 확인되는 리소스에 대한 값만 반환한다.
- 각 다중 값 쿼리에 대해 최대 8개 이상의 정상 레코드가 반환된다.
- **다중 값 기반의 라우팅은 ELB를 대체할 수 없다.**

![11-routing-policy-multi-value.png](images%2F11-routing-policy-multi-value.png)

---

### 참고한 강의

- [DevOps Engineer Professional](https://www.udemy.com/course/aws-certified-devops-engineer-professional-korean)
- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)