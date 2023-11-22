# Amazon Route 53

이번 장에서는 **SysOps Administrator**를 준비하며 **AWS의 DNS 서비스인 Route 53**에 대해서 알아보도록 한다.

---

### DNS

- Domain Name System의 약자로 인간 친화적인 호스트 이름을 IP 주소로 변환하는 도메인 이름 시스템이다.
- 구글을 예로 들면, `www.google.com`과 같은 호스트 이름을 `172.17.18.36`으로 변환해 준다.
- DNS는 인터넷의 백본이다.
- DNS는 계층적 이름 지정 구조를 사용한다.

#### 용어 (Terminology)

- Domain Registrar: Amazon Route 53, GoDaddy 등..
- DSN Record: A, AAAA, CNAME, NS 등..
- Zone File: DNS 레코드를 포함한다.
- Name Server: DNS 쿼리 해결 (권한 또는 비권한)
- Top Level Domain (TLD): `.com`, `.us`, `.in`, `.gov`, `.org` 등..
- Second Level Domain (SLD): amazon.com, google.com 등..

![1-dns-terminologies.png](images%2F1-dns-terminologies.png)

#### 작동 방식

![2-dns-works.png](images%2F2-dns-works.png)

- 웹 브라우저에서 Local DNS 서버에 `example.com` 도메인의 IP가 있는지 확인한다.
- 저장된 데이터가 없다면 Root DNS 서버에 `example.com`에 대한 정보를 요청한다.
  - Root DNS 서버는 `.com`에 대한 Name Server 정보인 `1.2.3.4`를 반환한다.
- `1.2.3.4`의 목적지인 TLD DNS 서버에 `example.com`에 대한 정보를 요청한다.
  - TLD DNS 서버는 `example.com`에 대한 Name Server 정보인 `5.6.7.8`을 반환한다.
- `5.6.7.8`의 목적지인 SLD DNS 서버에 `example.com`에 대한 정보를 요청한다.
  - SDL DNS 서버는 `example.com`의 IP 주소인 `9.10.11.12`를 반환한다.
- 웹 브라우저는 최초 목적지였던 `example.com`의 IP 주소인 `9.10.11.12` 서버로 요청하게 된다.

---

### Amazon Route 53

- 가용성과 확장성이 뛰어나고 완벽하게 관리되며 신뢰할 수 있는 DNS다.
  - 권한 있음(Authoritative): 고객이 DNS 레코드를 업데이트할 수 있다.
- Route 53은 도메인 등록자(Domain Registrar)이기도 하다.
- 리소스의 상태를 확인하는 기능을 제공한다.
- 100% 가용성 SLA를 제공하는 유일한 AWS 서비스다.
- Route 53에서 53은 전통적인 DNS 포트를 의미한다.

![3-route53.png](images%2F3-route53.png)

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

![4-public-private-hosted-zone.png](images%2F4-public-private-hosted-zone.png)

#### Records TTL (Time To Live)

- 높은 TTL (예. 24시간)
  - Route 53의 트래픽이 감소한다.
  - 오래된 레코드일 수 있다.
- 낮은 TTL (예. 60초)
  - Route 53의 트래픽이 증가하며 비용이 증가한다.
  - 레코드가 짧은 시간마다 변경된다.
  - 레코드 변경에 용이하다.
- **별칭(Alias) 레코드를 제외하고 각 DNS 레코드에는 TTL이 필수**다.

![5-records-ttl.png](images%2F5-records-ttl.png)

#### CNAME vs Alias

- AWS 리소스(Load Balancer, CloudFront 등..)는 AWS 호스트 이름을 노출한다.
  - `lb1-1234.us-east-2.elb.amazonaws.com`, `myapp.mydomain.com`
- CNAME:
  - 호스트 이름이 다른 호스트 이름을 가리킨다. (`app.mydomain.com` => `blabla.anything.com`)
  - **루트 도메인이 아닌 경우에만 해당된다.** (예. `something.mydomain.com`)
- Alias:
  - 호스트 이름이 AWS 리소스를 가리킨다. (`app.mydomain.com` => `blabla.amazonaws.com`)
  - **루트 도메인 및 루트 도메인이 아닌 경우에도 작동**한다.
  - 무료로 사용할 수 있다.
  - Health Check를 지원한다.

#### Alias Records

- 호스트 이름을 AWS 리소스에 매핑한다.
- DNS 기능을 확장한다.
- 리소스의 IP 주소 변경을 자동으로 인식한다.
- CNAME과 달리 DNS 네임 스페이스(Zone Apex)의 최상위 노드에 사용할 수 있다. (예. `example.com`)
- 별칭 레코드는 AWS 리소스(IPv4/IPv6)에 대해 항상 A/AAAA 유형이다.
- **TTL을 설정할 수 없다.**

![6-alias-record.png](images%2F6-alias-record.png)

- 별칭 레코드는 아래와 같은 AWS 리소스를 목적지로 설정할 수 있다.
  - Elastic Load Balancer
  - CloudFront Distribution
  - API Gateway
  - Elastic Beanstalk environment
  - S3 Website
  - VPC Interface Endpoint
  - Global Accelerator accelerator
  - 동일한 Hosted Zone의 Route 53 레코드
- **EC2의 DNS 이름에는 별칭 레코드를 설정할 수 없다.**

---

### Health Check

- HTTP Health Check는 Public 리소스에만 적용된다.
- Health Check => 자동화된 DNS 장애 조치
  1. 엔드 포인트를 모니터링하는 상태 확인 (애플리케이션, 서버, 기타 AWS 리소스)
  2. 다른 Health Check를 모니터링하는 Health Check (계산된 Health Check)
  3. CloudWatch Alarm을 모니터링하는 Health Check (DynamoDB 쓰로틀, RDS 알람, 커스텀 메트릭 등이 있으며 Private 리소스를 확인할 때 유용)
- Health Check는 CloudWatch Metric과 통합된다.

![7-health-check.png](images%2F7-health-check.png)

#### 엔드포인트 모니터링

- 약 15개의 글로벌 Health Checker가 엔드포인트의 상태를 확인한다.
  - Health/Unhealthy Threshold - 3 (기본값)
  - 간격 - 30초 (10초로 설정할 수 있으며 비용이 증가)
  - HTTP, HTTPS, TCP 프로토콜을 지원한다.
  - Health Checker의 18% 이상이 엔드포인트가 정상이라고 보고하면 Route 53은 이를 정상으로 간주한다.
  - Route 53에서 사용할 위치를 선택하는 기능을 제공한다.
- 엔드포인트가 2xx 및 3xx 상태 코드로 응답하는 경우에만 Health Check가 통과한다.
- Health Check는 응답의 처음 5120바이트에 있는 텍스트를 기반으로 통과하거나 실패하도록 설정할 수 있다.
- Route 53은 Health Checker로부터 들어오는 요청을 허용하도록 라우터/방화벽을 구성한다.

![8-health-check-monitor-endpoint.png](images%2F8-health-check-monitor-endpoint.png)

#### Calculated Health Check

- 여러 상태 확인 결과를 단일 상태 확인으로 결합할 수 있다.
- OR, AND, NOT을 사용할 수 있다.
- 최대 256개의 Child 상태 확인을 모니터링할 수 있다.
- 상위 항목을 통과하기 위해 통과해야 하는 Child의 상태 확인 수를 지정할 수 있다.
- 사용 사례: 모든 상태 확인이 실패하지 않고 웹사이트 유지 관리를 수행한다.

![9-calculated-health-check.png](images%2F9-calculated-health-check.png)

#### Private Hosted Zone

- Route 53 상태 검사기는 VPC 외부에 있다.
- Private 엔드포인트(Private VPC 또는 On-Premise 리소스)에 액세스할 수 없다.
- **CloudWatch Metric을 생성하고 CloudWatch Alarm을 연결한 다음 Alarm 자체를 확인하는 상태 확인을 생성할 수 있다**.

![10-private-hosted-zone.png](images%2F10-private-hosted-zone.png)

---

### Routing Policy

- Route 53이 DNS 쿼리에 응답하는 방법을 정의한다.
- "Routing"이라는 단어를 혼동해서는 안된다.
  - 트래픽을 라우팅하는 로드밸런서의 라우팅과 동일하지 않다.
  - DNS 트래픽을 라우팅하지 않고 DNS 쿼리에만 응답한다.
- Route 53은 아래와 같은 라우팅 정책을 지원한다.
  - Simple
  - Weighted
  - Failover
  - Latency based
  - Geolocation
  - Multi-Value Answer
  - Geoproximity (Route 53의 Traffic Flow 기능 사용)

#### Simple

- 일반적으로 트래픽을 단일 리소스로 라우팅한다.
- 동일한 레코드에 여러 값을 지정할 수 있다.
- **여러 값이 반환되는 경우 클라이언트는 임의의 값을 선택**한다.
- 별칭이 활성화된 경우 AWS 리소스를 하나만 지정할 수 있다.
- 상태 확인(Health Check)과 연결할 수 없다.

![11-routing-policy-simple.png](images%2F11-routing-policy-simple.png)

#### Weighted

- 각 특정 리소스로 이동하는 요청 비율을 제어한다.
- 각 레코드에 상대적 가중치를 할당한다.
  - 트래픽(%) = 특정 레코드의 가중치 / 모든 레코드의 가중치 합
  - 가중치의 합이 100이 될 필요는 없다.
- DNS 레코드는 이름과 유형이 동일해야 한다.
- 상태 확인(Health Check)과 연결될 수 있다.
- 사용 사례: 지역 간 로드 밸런싱, 새로운 애플리케이션 버전 테스트 등..
- **리소스에 대한 트래픽 전송을 중지하려면 레코드에 가중치 0을 할당**하면 된다.
- **모든 레코드의 가중치가 0인 경우 모든 레코드가 동일하게 반환**된다.

![12-routing-policy-weighted.png](images%2F12-routing-policy-weighted.png)

#### Latency-based

- 사용자와 가까운 지연 시간이 가장 짧은 리소스로 리디렉션한다.
- 사용자의 지연 시간이 중요한 경우 매우 유용하게 사용된다.
- **지연 시간은 사용자와 AWS 리전 간의 트래픽을 기준**으로 한다.
- 상태 점검(Health Check)과 연결될 수 있으며, 장애 조치 기능을 지원한다.

![13-routing-policy-latency.png](images%2F13-routing-policy-latency.png)

- 위의 이미지를 기준으로 독일 사용자는 미국으로 연결될 수 있다.

#### Failover (Active-Passive)

![14-routing-policy-failover.png](images%2F14-routing-policy-failover.png)

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

![15-routing-policy-gelocation.png](images%2F15-routing-policy-gelocation.png)

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

![16-routing-policy-geoproximity1.png](images%2F16-routing-policy-geoproximity1.png)

- us-west-1과 us-east-1 모두 Bias가 0으로 맞춰진 경우 미국을 둘로 나눈 라인이 생성된다.
- 해당 라인을 기준으로 좌측 절반은 us-west-1로 요청이 전달되고, 우측 절반은 us-east-1로 요청이 전달된다.

![17-routing-policy-geproximity2.png](images%2F17-routing-policy-geproximity2.png)

- us-east-1의 Biasrk 50으로 맞춰진 경우 좌측으로 편향된 라인이 생성된다.
- 해당 라인을 기준으로 좌측의 사용자는 us-west-1로 요청을 하게되고, 우측의 사용자는 us-east-1로 요청을 하게 된다.
- Bias의 값이 높을수록 더 많은 사용자의 요청을 처리하게 된다.

#### Traffic Flow

- 크고 복잡한 구성에서 기록을 생성하고 유지하는 프로세스를 단순화한다.
- 복잡한 라우팅 결정 트리를 관리하는 시각적 편집기를 지원한다.
- **구성을 트래픽 흐름(Traffic Flow) 정책으로 저장**할 수 있다.
  - 다양한 Route 53 호스팅 영역(다른 도메인 이름)에 적용할 수 있다.
  - 버전 관리를 지원한다.

![18-traffic-flow.png](images%2F18-traffic-flow.png)

#### IP-based Routing

- **라우팅은 클라이언트의 IP 주소를 기반**으로 한다.
- 클라이언트 및 해당 엔드포인트/위치에 대한 CIDR 목록을 제공한다. (사용자 - IP - 엔드포인트 매핑)
- 활용 사례: 성능 최적화, 네트워킹 비용 절감 등..
- 예를 들어, 최종 사용자를 특정 ISP에서 특정 엔드포인트로 라우팅한다.

![19-routing-policy-ipbased-routing.png](images%2F19-routing-policy-ipbased-routing.png)

#### Multi-Value

- 트래픽을 여러 리소스로 라우팅할 때 사용한다.
- Route 53은 여러 값/리소스를 반환한다.
- 상태 확인(Health Check)과 연결 가능한 값을 확인할 수 있다. 
  정상으로 확인되는 리소스에 대한 값만 반환한다.
- 각 다중 값 쿼리에 대해 최대 8개 이상의 정상 레코드가 반환된다.
- **다중 값 기반의 라우팅은 ELB를 대체할 수 없다.**

![20-routing-policy-multi-value.png](images%2F20-routing-policy-multi-value.png)

---

### Domain Registrar vs DNS Service

- 일반적으로 연간 요금을 지불하여 도메인 등록 기관에 도메인 이름을 구입하거나 등록한다. (예. GoDaddy, Amazon Registrar 등..)
- 도메인 등록 기관은 일반적으로 DNS 레코드를 관리할 수 있는 DNS 서비스를 제공한다.
- 하지만 다른 DNS 서비스를 사용하여 DNS 레코드를 관리할 수 있다.
- 예를 들어, GoDaddy에서 도메인을 구입하고 Route 53을 사용하여 DNS 레코드를 관리할 수 있다.

![21-domain-registar-dns-service.png](images%2F21-domain-registar-dns-service.png)

- GoDaddy를 기준으로 레코드의 Nameserver를 Route 53의 Nameserver로 변경하면 된다.

![22-godaddy-registrar-route53-dnsservice.png](images%2F22-godaddy-registrar-route53-dnsservice.png)

- 타사 등록 대행자에서 도메인을 구입하는 경우 Route 53을 DNS 서비스 공급자로 계속 사용할 수 있다.
  - Route 53에 호스팅 영역을 생성한다.
  - Route 53의 Nameserver를 사용하여 타사 웹사이트의 NS 레코드를 업데이트한다.
- **Domain Registrar은 DNS Service와 동일하지 않다.**
- 모든 도메인 등록기관에는 일반적으로 일부 DNS 기능이 함께 제공된다.

---

### S3 Website & Route 53

- `acme.example.com`과 같은 도메인이 있다고 가정한다.
- 대상 레코드(`acme.example.com`)와 동일한 이름으로 S3 버킷을 생성한다.
- 버킷에서 S3 웹사이트를 활성화하고 S3 버킷 공개 설정을 활성화한다.
- S3 웹 사이트 엔드포인트에 대한 Route 53 별칭 레코드를 생성하거나 IPv4 주소(A 유형)를 입력한다.
- HTTP 트래픽에만 사용할 수 있으며, HTTPS가 필요한 경우 CloudFront를 사용해야 한다.

![23-s3-website-route53.png](images%2F23-s3-website-route53.png)

---

### Hybrid DNS

- 기본적으로 Route 53 Resolver는 아래에 대한 DNS 쿼리에 자동으로 응답한다.
  - EC2 인스턴스의 로컬 도메인 이름
  - Private 호스팅 영역의 레코드
  - Public Nameserver의 레코드
- Hybrid DNS: VPC(Route 53 Resolver)와 네트워크(기타 DNS Resolver)간의 DNS 쿼리를 확인한다.
- 위에서 말하는 네트워크는 아래와 같다.
  - VPC / 피어링된 VPC
  - 온프레미스 네트워크(Direct Connect 또는 AWS VPN을 통해 연결됨)

![24-hybrid-dns.png](images%2F24-hybrid-dns.png)

#### Resolver Endpoint

- **Inbound Endpoint**
  - 네트워크의 DNS Resolver는 DNS 쿼리를 Route 53 Resolver로 전달할 수 있다.
  - DNS Resolver가 Route 53의 Private 호스팅 영역의 AWS 리소스 및 레코드에 대한 도메인 이름을 확인할 수 있도록 허용한다.
- **Outbound Endpoint**
  - Route 53 Resolver는 조건부로 DNS 쿼리를 DNS Resolver에게 전달한다.
  - Resolver 규칙을 사용하여 DNS 쿼리를 DNS Resolver에게 전달한다.
- 동일한 AWS 리전에 있는 하나 이상의 VPC와 연결된다.
- 고가용성을 위해 2개의 AZ에 생성된다.
- 각 엔드포인트는 IP 주소당 초당 10,000개의 쿼리를 지원한다.

#### Resolver Inbound Endpoint

![25-resolver-inbound-endpoint.png](images%2F25-resolver-inbound-endpoint.png)

- AWS 클라우드와 온프레미스의 데이터 센터가 있고, VPN 또는 Direct Connect로 연결되어 있다.
- Private 인스턴스의 레코드가 DNS의 CNAME으로 연결되어 있다.
- 온프레미스에는 DNS Resolver는 온프레미스의 서버가 `app.aws.private` 주소를 요청하면 AWS의 Resolver Inbound Endpoint로 요청을 보낸다.
- Resolver Inbound Endpoint는 고가용성을 위해 두 개의 ENI를 사용하고 있다.
- Resolver Inbound Endpoint는 Route 53을 통해 Private Hosted Zone에서 실행 중인 인스턴스의 주소를 알 수 있다.

#### Resolver Outbound Endpoint

![26-resolver-outbound-endpoint.png](images%2F26-resolver-outbound-endpoint.png)

- EC2 인스턴스에서 온프레미스의 서버에 접근하는 과정이다.
- EC2 인스턴스는 Route 53에게 `web.onprem.private`의 주소를 요청한다.
- Route 53은 Resolver Outbound Endpoint를 통해서 온프레미스의 DNS Resolver에게 `web.onprem.private` 주소를 응답받는다.

#### Resolver Rule

- 네트워크의 DNS Resolver로 전달되는 DNS 쿼리를 제어한다.
- **조건부 전달 규칙 (Conditional Forwarding Rules)**
  - 지정된 도메인 및 모든 하위 도메인에 대한 DNS 쿼리를 대상 IP 주소로 전달한다.
- **시스템 규칙 (System Rules)**
  - 전달 규칙에 정의된 동작을 선택적으로 재정의한다.
  - 예를 들어, 하위 도메인 `acme.example.com`에 대한 DNS 쿼리를 전달한다.
- **자동 정의 시스템 규칙 (Auto-defined System Rules)**
  - 선택한 도메인 (예. AWS 내부 도메인, Private Hosted Zone)에 대한 DNS 쿼리를 해결하는 방법을 정의한다.
- 여러 규칙이 일치하는 경우 Route 53 Resolver는 가장 구체적인 일치 항목을 선택한다.
- **확인자 규칙은 AWS RAM(Resource Access Manager)을 사용하여 계정 간에 공유**할 수 있다.
  - 하나의 계정으로 중앙에서 관리한다.
  - 여러 VPC에서 규칙에 정의된 대상 IP로 DNS 쿼리를 보낸다.

![27-resolver-rule.png](images%2F27-resolver-rule.png)

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [IP Range](https://ip-ranges.amazonaws.com/ip-ranges.json)