# Global Infrastructure

이번 장에서는 SAA를 준비하며 **Global Infrastructure**에 대해서 알아보도록 한다.

---

## Amazon CloudFront

- AWS의 CDN 서비스로 Content Delivery Network의 약자다.
- 읽기 성능을 향상하고 콘텐츠는 엣지(Edge)에 캐시되어 고객 경험(Users Experience)을 개선한다.
- 전 세계적으로 216개의 PoP(Point of Presence)이 있다.
- 글로벌 서비스 이므로 DDoS 공격을 방어하며, Shield 및 WAF(AWS Web Application Firewall)과 통합된다.

### CloudFront - Origins

- S3 Bucket
    - 파일 배포 및 엣지 캐싱 용도로 사용된다.
    - CloudFront OAC(Origin Access Control)을 사용하여 보안을 강화한다.
    - OAC가 OAI(Origin Access Identity)를 대체한다.
    - 파일을 S3에 업로드하기 위해 CloudFront를 입력(ingress)으로 사용할 수 있다.

- Custom Origin (HTTP)
    - Application Load Balancer
    - EC2 Instance
    - S3 Website (정적 S3 웹 사이트로 버킷을 활성화해야 한다.)
    - 사용자가 원하는 모든 종류의 HTTP 백엔드 서비스

### 높은 수준의 CloudFront

![high-level-cloudfront.png](images%2FGlobalInfrastructure%2Fhigh-level-cloudfront.png)

### Origin으로서의 S3

![s3-as-an-origin.png](images%2FGlobalInfrastructure%2Fs3-as-an-origin.png)

### CloudFront vs S3 Cross Region Replication

- CloudFront
  - 글로벌 엣지(Edge) 네트워크
  - 파일은 TTL(Time to Live, 일반적으로 하루)동안 캐시된다.
  - 어디에서나 사용할 수 있어야 하는 **정적** 콘텐츠에 적합하다.

- S3 Cross Region Replication
  - 운영자는 복제를 수행하려는 각 지역에 대해 설정해야 한다.
  - 파일은 거의 실시간으로 업데이트되며 읽기 전용이다.
  - 적은 지연 시간으로 사용할 수 있어야 하는 **동적** 콘텐츠에 적합하다.

### Origin으로서의 ALB or EC2

![origin-as-an-ec2.png](images%2FGlobalInfrastructure%2Forigin-as-an-ec2.png)

### CloudFront Geo Restriction(지역 제한)

- 배포물에 액세스할 수 있는 사용자를 제한할 수 있다.
  - 승인 목록(Allowlist): 사용자가 승인된 국가 목록에 있는 경우에만 콘텐츠에 액세스할 수 있도록 설정할 수 있다.
  - 차단 목록(Blocklist): 사용자가 금지된 국가 목록에 있는 국가 중 하나에 있는 경우 사용자가 콘텐츠에 액세스하지 못하도록 한다.
- 여기서 **국가**는 Geo-IP 데이터베이스를 사용하여 결정되며, 대표적인 예시로 콘텐츠에 대한 액세스를 제어하는 저작권법을 위한 사용이 있다.

### CloudFront - 비용

- CloudFront Edge는 전 세계에 있으며, Edge가 위치에 따라 비용이 다르게 발생한다.

![cloudfront-pricing.png](images%2FGlobalInfrastructure%2Fcloudfront-pricing.png)

#### CloudFront - 비용 등급

- 비용을 절감하기 위해서 Edge의 수를 줄일 수 있다.
- 3가지 비용 등급이 있다.
  - Price Class All: 모든 리전을 사용하며 가장 높은 성능을 가지고 있다.
  - Price Class 200: 대부분의 리전을 사용하지만, 가격이 비싼 일부 리전을 제외한다.
  - Price Class 100: 비용이 저렴한 리전만 사용한다.

![cloudfront-pricing-class.png](images%2FGlobalInfrastructure%2Fcloudfront-pricing-class.png)

![cloudfront-pricing-group.png](images%2FGlobalInfrastructure%2Fcloudfront-pricing-group.png)

### CloudFront - 캐시 무효화(Invalidation)

- 백엔드 오리진을 업데이트하는 경우 CloudFront는 Origin이 업데이트 되었다는 사실을 알지 못하기 때문에, TTL이 만료된 후에만 새로운 콘텐츠를 가져올 수 있다.
- 하지만, 사용자가 강제로 캐시 무효화를 수행하여 전체 또는 부분 캐시 새로 고침을 강제로 수행할 수 있다. (TTL ByPass)
- `*`을 사용하여 모든 캐시를 무효화할 수 있고, `/images/*`와 같이 특정 경로만 무효화할 수 있다.

![cloudfront-cache-invalidation.png](images%2FGlobalInfrastructure%2Fcloudfront-cache-invalidation.png)

### 글로벌 사용자

- 응용 프로그램을 배포했으며 응용 프로그램에 직접 액세스하려는 전역 사용자가 있다.
- 공용 인터넷을 통과하므로, 많은 홉으로 인한 대기 시간이 추가될 수 있다.
- AWS 네트워크를 통해 대기시간을 최소화하여 가능한 빠르게 글로벌 사용자가 애플리케이션에 액세스하는 방법에 대해서 알아본다.

![global-users.png](images%2FGlobalInfrastructure%2Fglobal-users.png)

#### Unicast IP vs Anycast IP

- Unicast IP: 하나의 서버는 하나의 IP 주소를 보유하고 있다.
- Anycast IP: 모든 서버가 동일한 IP 주소를 가지고 있고 클라이언트는 가장 가까운 IP 주소로 라우팅된다.

![unicast-vs-anycast.png](images%2FGlobalInfrastructure%2Funicast-vs-anycast.png)

#### AWS Global Accelerator

- AWS 내부 네트워크를 활용하여 애플리케이션으로 라우팅한다.
- 사용자의 애플리케이션에 대해 두개의 Anycast IP가 생성된다.
- Anycast IP는 엣지 로케이션으로 트래픽을 직접 전송한다.
- Edge Location은 트래픽을 애플리케이션으로 전송한다.

![global-accelerator.png](images%2FGlobalInfrastructure%2Fglobal-accelerator.png)

- Global Accelerator는 Elastic IP, EC2 인스턴스, ALB, NLB, Public 또는 Private과 작동한다.
- 일관된 성능(Consistent Performance)
  - 지연 시간이 가장 짧고 지역 장애 조치가 빠른 지능형 라우팅이다.
  - IP가 변경되지 않기 때문에 클라이언트 캐시가 문제가 되지 않는다.
  - AWS 내부 네트워크를 사용한다.
- 상태 확인(Health Checks)
  - Global Accelerator는 애플리케이션의 상태 확인을 수행한다.
  - 애플리케이션을 글로벌하게 만드는 데 도움이 되며, 문제가 발생하는 경우 1분 이내에 장애 조치가 가능한다.
  - 상태 확인을 통해서 재해 복구에 적합하다.
- 보안(Security)
  - 2개의 외부 IP만 화이트리스트로 등록되어 있으면 된다.
  - AWS Shield를 통해서 DDoS 보호가 가능하다.

#### AWS Global Accelerator vs CloudFront

- 두 서비스 모두 AWS 글로벌 네트워크와 글로벌 엣지 로케이션을 사용한다.
- 두 서비스 모두 DDoS 보호를 위해 AWS Shield와 통합된다.
- **CloudFront**
  - 캐싱 가능한 콘텐츠(이미지, 비디오)의 성능 항샹을 위해 사용된다.
  - 동적 콘텐츠(API 가속화 및 동적 사이트 제공)
  - 콘텐츠가 Edge Location에서 제공된다.
- **Global Accelerator**
  - TCP 또는 UDP를 통해 광범위한 애플리케이션의 성능을 개선한다.
  - 하나 이상의 AWS 리전에서 실행되는 애플리케이션에 대해 Edge에서 패킷을 프록시한다.
  - 게임(UDP), IoT(MQTT) 또는 VoIP와 같은 HTTP가 아닌 프로토콜의 사용 사례에 적합하다.
  - 정적 IP 주소가 필요한 HTTP 사용 사례에 적합하다.
  - 결정적이고 빠른 지역 장애 조치가 필요한 HTTP 사용 사례에 적합하다.

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03